# Chapter 4: Working with Multimodal Foundational Models

## Core Idea
Multimodal models process text and image (and other modality) inputs jointly rather than in isolation, enabling embeddings-based similarity search (Titan Multimodal Embeddings), visual Q&A/captioning (Claude 4 Sonnet, LLaVA), and image editing (inpainting/outpainting/image-to-image via Stable Diffusion/Nova Canvas) — each requiring prompts engineered specifically to bridge text and visual context, not reused text-only prompt habits.

## Frameworks Introduced
- **Seven Best Practices for Multimodal Prompting**: Well-Crafted Prompts, Balanced Detail, Cultural/Societal Sensitivity, Syntactic Consistency, Iterative Testing and Refinement, Leveraging Model Strengths, Encouraging Creativity and Diversity.
  - When to use: any prompt that must coordinate text and image generation/interpretation together.
  - How each applies: precise scene description (well-crafted); enough detail to guide without over-constraining creativity (balanced); avoid stereotypes/bias for global audiences (sensitivity); grammatically clear, stepwise instructions with diagram references (syntactic consistency); refine based on consistency of output, user feedback, and error analysis (iterative); match prompt structure to the specific model's known strengths (leverage strengths); leave visual style/narrative open-ended when novelty matters more than precision (creativity).
- **Model-strength matching**: pick prompting style to the specific multimodal model's known capability profile (e.g., CLIP → detailed descriptions/contextual cues; DALL-E → clear visual style; BLIP → concise image descriptions; Flamingo → balanced text/visual load; Nova Premier → balanced prompts; LLaVA → visual-context-heavy questions).
  - When to use: before committing to a model, check its documented strengths/weaknesses table rather than assuming any multimodal model handles all prompt styles equally.

## Key Concepts
- **Titan Multimodal Embeddings G1**: generates 1024-dimension vector embeddings from text+image jointly (not generative text/image output) — used for similarity search, content classification, recommendation.
- **LLaVA**: CLIP ViT-L/14 visual encoder + projection matrix + Vicuna LLM; two-stage training (Stage 1: align projection layer only; Stage 2: fine-tune projection + LLM). Not natively on Bedrock — deploy via SageMaker.
- **Image-to-image transformation**: modify an existing image per text guidance and a `strength` parameter (0.0=identical to input, 1.0=highly transformed) using Stable Diffusion.
- **Image inpainting**: reconstruct/replace masked regions of an image per a text prompt and binary mask (black = area to inpaint); used via Amazon Nova Canvas (`taskType: INPAINTING`).
- **Image outpainting**: extend an image beyond its original borders, generating new coherent content via diffusion + autoencoder in latent space.
- **Visual Question Answering (VQA)**: combines image feature extraction (CNN or vision-Transformer) with a text-question embedding via attention/fusion, producing an answer; Claude 4 Sonnet handles this natively on Bedrock via multimodal message content.
- **Image captioning**: encoder-decoder pattern — image encoder extracts visual features, language decoder generates descriptive text; Titan Multimodal Embeddings + Nova Premier can compose this pipeline.
- **KNN (K-nearest neighbors) search**: used by OpenSearch to retrieve documents/images with the most similar embedding vectors — the retrieval mechanism underlying embedding-based search.

## Mental Models
- Treat **Titan Multimodal Embeddings and LLaVA as complementary layers**: Titan preprocesses/structures data into embeddings for retrieval; LLaVA (or Claude 4 Sonnet on Bedrock) then reasons over the retrieved content for detailed answers — don't expect one model to do both jobs.
- **Multimodal prompts need MORE detail than text-only prompts**, not the same amount — "Generate an image of a mountain" is underspecified in a way that would be tolerable for pure text tasks but produces wildly inconsistent images; specify style, color, composition explicitly.
- Think of **mask quality as a silent failure mode in inpainting**: an inverted mask, wrong format (must be black/white or grayscale), or a mask too close to image edges causes unnatural blending or no visible change — debug the mask before debugging the prompt.

## Anti-patterns
- **Vague multimodal prompts** ("Beach picture") — produces high output variance; specify scene, lighting, style, composition for consistent results.
- **Over-constraining prompts when creativity is the goal** — detailed prompts are right for product/brand consistency but choke off novelty in marketing/creative use cases; leave visual style or narrative open-ended there instead.
- **Ignoring cultural/societal sensitivity in prompts for global audiences** — risks reinforcing stereotypes and damaging brand reputation; scope target demographic and tone explicitly.
- **Using image-to-image mode without checking `strength` calibration** — a `strength` too high discards the original image's structure entirely; too low makes no meaningful change. Test both extremes (0 and ~0.8) to calibrate.

## Code Examples
```python
# Image-to-image transformation payload (Stable Diffusion 3.5 Large)
def construct_request_payload(alteration_prompt, negative_prompts_list, initial_image_b64):
    payload = {
        "prompt": alteration_prompt,
        "negative_prompt": negative_prompts_list,
        "mode": 'image-to-image',
        "strength": 0.9,
        "image": initial_image_b64,
        "seed": 321,
    }
    return json.dumps(payload)

request_payload = construct_request_payload(new_change_prompt, negative_prompts, encoded_image_b64)
model_identifier = "stability.sd3-5-large-v1:0"
bedrock_response = invoke_model(bedrock_runtime, request_payload, model_identifier)
```

```python
# Image inpainting via Amazon Nova Canvas
inpaint_request = json.dumps({
    "taskType": "INPAINTING",
    "inPaintingParams": {
        "text": inpaint_text,
        "image": image_to_base64(new_img),
        "maskImage": image_to_base64(binary_mask)
    },
    "imageGenerationConfig": {
        "quality": "standard", "numberOfImages": 1,
        "height": 512, "width": 512, "cfgScale": 10
    }
})
model_identifier = "amazon.nova-canvas-v1:0"
```

```python
# Visual Question Answering with Claude 4 Sonnet
with open("image.jpg", "rb") as image_file:
    encoded_image = base64.b64encode(image_file.read()).decode('utf-8')

body = json.dumps({
    "anthropic_version": "bedrock-2023-05-31",
    "max_tokens": 512,
    "messages": [{
        "role": "user",
        "content": [
            {"type": "image", "source": {"type": "base64", "media_type": "image/png", "data": encoded_image}},
            {"type": "text", "text": "What is in the image?"}
        ]
    }]
}).encode()

response = client.invoke_model(
    body=body, modelId="anthropic.claude-sonnet-4-20250514-v1:0",
    accept="application/json", contentType="application/json")
content = json.loads(response["body"].read().decode('utf-8'))['content'][0]['text']
```

```python
# Titan Multimodal Embeddings for image+text search
def get_embedding(image_path, title=None):
    with open(image_path, "rb") as image_file:
        input_image = base64.b64encode(image_file.read()).decode('utf8')
    body = {"inputImage": input_image}
    if title:
        body["inputText"] = title
    response = bedrock_runtime.invoke_model(
        body=json.dumps(body), modelId="amazon.titan-embed-image-v1",
        accept="application/json", contentType="application/json")
    vector_json = json.loads(response['body'].read().decode('utf8'))
    return vector_json
```

- **What it demonstrates**: the full multimodal toolkit — image-to-image transformation, mask-guided inpainting, VQA via Claude's native multimodal message format, and Titan embeddings generation for downstream KNN search.

## Reference Tables
| Model | Input | Output | Best For |
|---|---|---|---|
| Titan Multimodal G1 | Text + Images | Embeddings (1024D) | similarity search, retrieval, recommendation |
| LLaVA | Text + Images | Text (Q&A, reasoning) | visual Q&A, interactive dialogs |
| Nova Canvas | Text + Image/Mask | Generated/edited image | inpainting, outpainting |
| Claude 4 Sonnet | Text + Image | Text | VQA, chart/document interpretation |

| Model | Strengths | Weaknesses | Prompting Tips |
|---|---|---|---|
| CLIP (OpenAI) | understanding visual+textual data | limited novel content generation | detailed descriptions, contextual cues |
| DALL-E (OpenAI) | creative image generation | weaker complex text generation | clear visual descriptions/styles |
| BLIP (Salesforce) | image interpretation, captioning | limited extensive text context | concise image descriptions |
| Flamingo (DeepMind) | multi-modality integration | heavy computational resources | balance text/visual load |
| Nova Premier (Bedrock) | versatile text+image | struggles with niche domain knowledge | balanced prompts, both modalities |
| LLaVA (Bedrock via SageMaker) | VQA, detailed image analysis | limited text generation | focus on visual-context questions |

| Multimodal Prompt Type | Before | After |
|---|---|---|
| Text-only | "Write a summary about global warming." | "Write a 200-word summary... covering main causes... and solutions..." |
| Multimodal | "Generate an image of a mountain." | "Generate an image of a snow-capped mountain with a clear blue sky, surrounded by pine trees." |

## Worked Example
**Movie Recognizer** (practical exercise, uses Titan Multimodal Embeddings + OpenSearch KNN + Gradio):
1. **Data**: MovieLens dataset — 56 movies with title, IMDb ID, poster path, genres, plot summary, release year.
2. **Embeddings**: generate two embeddings per movie poster via `amazon.titan-embed-image-v1` — one image-only, one image+title combined (1024-dim vectors each).
3. **Search Index**: create an OpenSearch/Elasticsearch KNN index (`"index.knn": True`, `knn_vector` field, dimension 1024) storing embeddings + metadata (title, plot summary, movie ID).
4. **Retrieval**: index each movie's embedding as a document via PUT/POST requests.
5. **Demo**: a Gradio web interface takes a natural-language description (e.g., "A space raccoon beside a talking tree on an extraterrestrial world"), embeds the query, and returns the nearest-neighbor movie poster — correctly surfacing "Guardians of the Galaxy" without ever naming it, and "13 Assassins" from "13 warriors band together to save their people."

**Why it works**: it demonstrates that embedding-space proximity captures semantic/thematic similarity, not keyword matching — a query with zero literal title words still retrieves the right result because the embedding encodes meaning, not text.

## Key Takeaways
1. Multimodal prompts require significantly more detail than text-only prompts — vague prompts produce wildly inconsistent visual output.
2. The seven best practices (well-crafted, balanced detail, cultural sensitivity, syntactic consistency, iterative refinement, model-strength leverage, creativity encouragement) form a checklist for any multimodal prompt design.
3. Titan Multimodal Embeddings and LLaVA/Claude 4 Sonnet are complementary — embeddings handle retrieval, generative multimodal models handle reasoning/answering.
4. Image editing splits into three distinct techniques — image-to-image (`strength` controls transformation degree), inpainting (masked region replacement), outpainting (border extension) — each with its own parameter set and failure modes.
5. Mask quality (orientation, format, edge proximity) is a common silent failure point in inpainting — debug the mask before the prompt.
6. Amazon SageMaker JumpStart provides pretrained multimodal models deployable without training from scratch, useful when Bedrock's native model catalog doesn't cover a specific need (e.g., LLaVA).
7. KNN search over embeddings (via OpenSearch or Elasticsearch) is the standard retrieval mechanism for embedding-based semantic search applications.

## Connects To
- **Ch1**: builds on the foundational-model and embedding concepts introduced there.
- **Ch3**: extends the Stable Diffusion / boto3 image-generation code patterns from the Bedrock API chapter into image-to-image, inpainting, and outpainting.
- **Ch5**: fine-tuning (next chapter) is positioned as the next step for increasing relevance of multimodal search/generation results.
- **Ch6**: RAG (referenced here for the SageMaker JumpStart Q&A workflow) is covered in full in the RAG chapter.
