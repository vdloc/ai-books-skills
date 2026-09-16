# Chapter 8: Multimodal RAG

## Core Idea
Enterprise knowledge lives heavily in tables, images, audio, and video — not just prose — and text-only RAG silently fails on questions whose answer is "locked" in a chart, a diagram, or spoken/visual context; each modality needs its own extraction and retrieval strategy layered onto the same core ingestion/query architecture from Chapter 2.

## Frameworks Introduced
- **Conversion vs. native multimodal approach**: convert non-text data to text/structured representations that fit the standard RAG stack (industry standard today, more reliable/observable) vs. native multimodal LLMs consuming raw embeddings across modalities directly (emerging, simpler pipeline eventually). The book focuses on conversion as the current production standard.
- **Table processing 3-step pipeline**: table extraction (identify grid topology, handle merged cells/borderless tables) → OCR/semantic interpretation (cell-by-cell, classify header vs. data rows) → normalization (strip currency symbols, standardize numeric formats). Never treat parser output as ground truth without validation — even good tools like Docling silently produce empty/malformed tables on complex layouts.
- **Structured-context prompting for tables**: convert tables to JSON (row-dict-of-columns) or Markdown, summarize with an LLM into a "pointer chunk" at ingestion, and inject the *full* table as structured JSON (not flattened prose) into the generation prompt at query time — preserves header-to-cell relationships that naive text chunking destroys.
- **Multi-page table stitching**: detect consecutive-page table fragments with matching column counts, strip duplicate repeated headers, and concatenate into one master dataframe/JSON *before* embedding — prevents both duplicate-header noise and "headless" second-page fragments.
- **Image summarization vs. shared embedding space** (the two core image strategies): summarization = VLM describes the image once at ingestion into a text chunk (cheap at query time, but "locks in" whatever the summary captured, and can't be "re-seen"); shared embedding (CLIP/SigLIP/ImageBind) = image and text share one latent vector space via contrastive learning (preserves detail, better for visual/aesthetic search, but loses fine-grained numeric/textual detail in charts and breaks standard hybrid search/reranking, which need text tokens).
- **Video visual-semantics extraction strategies**: fixed-interval frame extraction (high granularity, weak on temporal/action understanding), temporal segment extraction (captures cause-and-effect over time via temporal-aware VLMs, expensive), keyframe extraction (scene-change detection triggers VLM captioning only at meaningful transitions — best cost/latency trade-off). Hybrid: cheap frame extraction for broad indexing, expensive segment analysis reserved for high-value video.
- **Blind verification workflow** (multimodal hallucination mitigation): pass the image to the VLM with a *neutral* prompt first ("describe the surface condition") to get an unbiased ground-truth description, then have a secondary text-only LLM compare the user's leading question against that neutral description — catches "visual sycophancy" where a leading question (e.g. "where is the rust?") biases the VLM into confirming a premise that isn't visually true.

## Key Concepts
- **The "headless chunk" problem**: naive text-chunking of a table-as-Markdown separates the header row (chunk 1) from data rows (chunks 2+), so a retrieved chunk like `| Model-X | $899 | ... |` loses the column-name context entirely.
- **Orphan chunk**: a vector in the database that has lost its link to the visual/spatial source (bounding box, page location) needed for citation — a chunk must be treated as a complex object (vector + raw content + spatial/temporal metadata), not just a string.
- **Modality alignment**: maintaining a traceable link between text, image, table, and video components of a document so citations and "jump to source" UX remain possible.
- **Speaker diarization**: separating individual speakers in an audio transcript — critical because *who* said something (CFO vs. intern) changes the contextual weight of a retrieved chunk.
- **The "Red Button" problem**: transcription alone misses silent visual actions (e.g. "now, do this" while pressing a specific button) — solved via visual captioning or cross-modal embedding layered on top of the audio transcript.
- **Visual sycophancy**: a VLM hallucination mode where the model prioritizes confirming the user's leading prompt over what the image actually shows (text tokens act as strong cross-attention anchors, lowering the match threshold for the queried concept).
- **Visual prompt injection**: malicious instructions embedded in an image, invisible to humans but legible to a VLM — a multimodal-specific attack surface requiring pixel sanitization/adversarial defense before the image reaches the model.
- **Blur-on-ingest**: model-based redaction (e.g. LayoutLM, YOLO) to detect and blur faces/signatures/license plates in scanned documents or images before they hit the vector store — unstructured PII can't be caught by regex the way text PII can.

## Mental Models
- Treat multimodal support as "specialized parsers and schemas layered on the same ingestion/query stages" (Ch2) — the architecture doesn't change, only the modality-specific extraction and representation logic within each stage.
- Choose image strategy by asking: is the *reasoning inside* the image (numbers, text, precise data) what matters, or is the *concept/aesthetic* of the image what matters? Reasoning-heavy → summarization; concept/aesthetic-heavy → shared embedding.
- Debugging multimodal RAG means moving from "reading logs" to "viewing state" — pull the exact image crop retrieved, overlay OCR bounding boxes, and see literally what the model saw, since text logs alone can't distinguish a retrieval miss from an OCR garble from a model that ignored good visual context.
- Default to restraint: only add multimodal ingestion components once your evaluation framework (Ch6) confirms retrieval failures are specifically caused by answer-locked non-text data — don't apply heavy OCR/ASR/VLM pipelines to every document by default.

## Anti-patterns
- **Chunking a table as flattened Markdown/text with standard character-based splitters**: produces "headless" chunks that sever column-header-to-cell-value relationships, causing the LLM to receive numbers without knowing what they mean.
- **Trusting 100% accuracy from any single table-extraction tool**: even capable tools like Docling silently return empty (0,0) tables on complex layouts — validate against a representative sample before production use, and consider a conditional multi-tool pipeline.
- **Ignoring multi-page table fragmentation**: treating each page's table fragment as independent creates duplicate-header noise or headless second-page data — always stitch fragments by matching column count and page adjacency before embedding.
- **Applying shared embedding models (CLIP/SigLIP) to information-dense charts/infographics**: these models are trained for image-text matching, not precise numeric extraction — they'll get the gist ("a bar chart about sales") but miss the actual trend/values; use summarization instead for reasoning-heavy visuals.
- **Passing a user's leading question directly to a VLM for high-stakes visual judgments** (insurance claims, industrial inspection): risks visual sycophancy — the model confirms what the question implies rather than what the image shows. Use blind/neutral-prompt verification first.
- **Losing bounding-box/timestamp metadata during ingestion**: creates orphan chunks that can't support citations or "jump to source" UX, even if the text/vector content itself is otherwise correct.
- **Defaulting to expensive multimodal pipelines (cloud OCR, VLM summarization, ASR) for every document regardless of value**: architectural tax is real — target the highest-value, clearly-multimodal use cases first rather than blanket-applying heavy processing.

## Code Examples

```python
# Structured (JSON) table context in the generation prompt — preserves header/cell links
def format_context_item(item, max_rows=100):
    if isinstance(item, pd.DataFrame):
        if len(item) > max_rows:
            item = item.head(max_rows)
        return f"- DATAFRAME (Columns: {list(item.columns)}):\n{item.to_json(orient='records', indent=2)}"
    # ... text and generic-dict fallback cases
```
- **What it demonstrates**: instead of flattening a table into prose (which breaks header/cell linkage), pass it as structured JSON in the prompt — modern LLMs trained on code/JSON parse this reliably.

```python
# Multi-page table stitching: detect duplicate headers and merge consecutive fragments
def stitch_tables(extracted_tables):
    tables = sorted(extracted_tables, key=lambda t: t['page'])
    result, current, current_page = [], tables[0]['df'].copy(), tables[0]['page']
    for entry in tables[1:]:
        next_df, next_page = entry['df'].copy(), entry['page']
        if next_page == current_page + 1 and next_df.shape[1] == current.shape[1]:
            if has_duplicate_header(next_df):
                next_df = next_df.iloc[1:].reset_index(drop=True)
            next_df.columns = current.columns
            current = pd.concat([current, next_df], ignore_index=True)
        else:
            result.append(current); current = next_df
        current_page = next_page
    result.append(current)
    return result
# 6 page-fragments (39,39,38,38,40,11 rows) -> stitched into 1 table of 205 rows
```
- **What it demonstrates**: page-adjacency + matching-column-count heuristic reliably unifies a table that spans six PDF pages into one logical dataset before embedding.

```python
# SigLIP shared-embedding retrieval: text query -> closest image via cosine similarity
text_features = model.get_text_features(**text_inputs)
text_features = text_features / text_features.norm(p=2, dim=-1, keepdim=True)
raw_scores = text_features @ image_features.t()   # cosine similarity (vectors normalized)
logits = (raw_scores * model.logit_scale.exp()) + model.logit_bias
sigmoid_probs = torch.sigmoid(logits)  # SigLIP: independent per-pair probability, not softmax ranking
# query "A round Margharitta Pizza" (note the typo) -> correct pizza image, score 0.83
```
- **What it demonstrates**: cross-modal semantic search matches a misspelled, natural-language description to the correct image with no manual tagging — and shows SigLIP scores are independent match probabilities, not relative rankings.

```python
# Deepgram ASR with speaker diarization -> chunked transcript with speaker + timestamp metadata
response = deepgram.listen.v1.media.transcribe_url(
    url=AUDIO_URL, model="nova-2", smart_format=True, diarize=True, utterances=True
)
# Each utterance -> {speaker, transcript, start, end, confidence}
# Chunks group utterances up to a 512-word limit, carrying start/end/speakers as metadata
```
- **What it demonstrates**: treating speaker ID and timestamp as first-class schema fields (not discarded) enables "jump-to-source" playback and speaker-specific filtering/redaction.

## Reference Tables

| Video extraction method | Pros | Cons |
|---|---|---|
| Fixed-interval frame extraction | High granularity, pinpoints entities/timestamps | Struggles with actions/events over time |
| Temporal segment extraction | Rich cause-and-effect summaries | Computationally expensive |
| Keyframe extraction | Optimized cost/latency, solves "Red Button" ambiguity efficiently | Depends on scene-detection accuracy |

| Image strategy | Best for | Weakness |
|---|---|---|
| Image summarization (VLM at ingestion) | Data-dense charts, diagrams, text-in-image reasoning | Locks in whatever the summary captured; can't "re-see" |
| Shared embedding space (CLIP/SigLIP/ImageBind) | Visual concepts, aesthetics, style search | Misses precise numeric/fine-grained detail; breaks text-based hybrid search/reranking |

| Table/OCR tool | Type | Best for |
|---|---|---|
| Amazon Textract / Azure Doc Intelligence / Google Doc AI | Commercial cloud API | Scanned docs, handwriting, high-value documents |
| LlamaParse | Commercial (LlamaCloud) | PDF/PPTX with tables/images to Markdown |
| Docling (IBM) / Unstructured | Open source, local | Native digital files, air-gapped/high-volume, preserves merged cells |
| Gmft | Open source, local | Table extraction specifically (merged cells, multilevel headers) |

## Worked Example
Querying "What was the growth of GenBank Base Pairs between PubMed and PubMed Central?" against three vector stores built from the same NLM report: (1) text-only (ignoring an embedded chart) returns "the provided information does not specify..." — a clear retrieval-blind failure; (2) text + VLM image summaries returns a qualitative but vague answer ("consistent and exponential increase... near zero in 1989 to significantly higher by 2000") — better, but the summary didn't capture exact figures; (3) full multimodal retrieval, passing the actual chart image bytes to a VLM (GPT-4.1-mini) at generation time, returns "increased from approximately 1 billion base pairs to about 10 billion base pairs" — a precise, numerically grounded answer. This progression concretely demonstrates the summarization-vs-full-image trade-off: summaries help but permanently cap the level of detail available at query time, while re-showing the raw image lets the model extract detail the original summary missed.

## Key Takeaways
1. Text-only RAG silently degrades on questions whose answer lives in a table, chart, image, or spoken/visual context — the failure mode looks like "no relevant data" (Ch6) but is really a modality gap.
2. Tables must be treated as structured objects (JSON/dataframe), never flattened text — naive chunking severs the header-to-cell relationship that gives numbers meaning.
3. Choose image strategy based on whether reasoning-over-detail (summarization) or concept/aesthetic search (shared embedding) is the actual need — they have opposite strength/weakness profiles.
4. Video requires solving "silent semantics" (the Red Button problem) — transcription alone misses actions; keyframe extraction is usually the most cost-effective way to add visual grounding.
5. A "chunk" in multimodal RAG must carry spatial/temporal metadata (bounding boxes, timestamps) as first-class schema fields, not optional extras — losing this creates orphan chunks that can't support citations or debugging.
6. Multimodal RAG introduces new attack/compliance surfaces (visual prompt injection, unstructured PII in images/audio) that text-only filters can't catch — dedicated defenses (pixel sanitization, blur-on-ingest) are required.
7. Apply restraint: only invest in multimodal ingestion for the specific use cases where evaluation (Ch6) confirms retrieval is failing because of non-text data — the architectural and cost tax is real, so target high-value cases first.

## Connects To
- **Ch2**: multimodal ingestion is presented explicitly as "the same ingestion/query stages, different modality-specific components" — this chapter builds directly on that architecture.
- **Ch3**: guardrails and hallucination-detection concepts extend here into visual sycophancy and blur-on-ingest PII redaction.
- **Ch4**: the parallelized, component-based production architecture recommended for ingestion cost control is directly reused for multimodal processing.
- **Ch6**: multimodal evaluation (recall@k on image-caption golden sets, VLM-as-judge) directly extends the RAG evaluation framework to a new data type.
- **Ch9**: the next chapter's knowledge graphs are framed as the final step for information multimodal RAG still can't fully capture (relationships across documents).
