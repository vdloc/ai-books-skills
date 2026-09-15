# Chapter 9: Building End-to-End Applications with Generative AI

## Core Idea
End-to-end generative AI applications integrate Bedrock with the broader AWS ecosystem (IoT Core, DynamoDB, Glue, Athena, OpenSearch, Lambda, API Gateway) into complete pipelines — data ingestion → storage → cataloging → AI-powered querying/insight generation → user-facing interface — demonstrated through an IoT environmental-sensor pipeline and a natural-language-to-SQL system.

## Frameworks Introduced
- **End-to-end project lifecycle**: Requirement Analysis & Design Architecture → Model Development & Customization → Testing & Validation → Deployment & Scaling → Continuous Monitoring & Improvement.
  - When to use: any organizational (not prototype) generative AI initiative — this is the macro version of the Ch1 solution life cycle, scaled to cross-departmental, multi-stakeholder projects.
- **Three-tier unstructured-data architecture**: Ingestion (raw data → S3) → Extraction/Analysis (AI services process it) → Metadata layer (Glue Data Catalog connects raw and processed data for discovery).
  - When to use: any pipeline where source data (text, images, sensor readings) lacks a clean schema and needs both storage and semantic discoverability.
- **Text-to-SQL architecture pattern**: User natural-language query → Bedrock (RAG against a schema-metadata knowledge base) → generated SQL → Athena execution → results returned, with retry-on-error loop feeding the SQL error back into regeneration.
  - When to use: giving nontechnical users query access to structured data (databases with cryptic table/column names) without requiring SQL knowledge.

## Key Concepts
- **AWS IoT Core**: managed service for secure device-to-cloud communication via MQTT (lightweight pub/sub protocol suited to constrained/intermittent connections).
- **DynamoDB**: fully managed NoSQL database for low-latency, scalable storage — used here as the landing zone for IoT sensor data.
- **AWS Glue Data Catalog**: serverless metadata repository — crawls S3 data sources and builds a schema catalog that both Athena (for querying) and Bedrock (for text-to-SQL grounding) rely on.
- **Amazon Athena**: serverless SQL query engine over S3 data — pay-per-TB-scanned pricing ($5/TB, 10MB minimum per query), so compression/partitioning/columnar formats matter for cost.
- **Bedrock Knowledge Base as schema context**: instead of just document retrieval, a knowledge base here stores database *metadata* (table/column names, types, descriptions) as the RAG source — grounding SQL generation instead of prose generation.
- **Retry-with-error-feedback pattern**: when a generated SQL query fails execution, the error message is fed back into a regenerated prompt (up to a max retry count) so the model can self-correct.

## Mental Models
- Treat an **end-to-end pipeline as a chain of managed AWS services, each doing one job well**: IoT Core (ingest) → DynamoDB (store) → Lambda (orchestrate/transform) → Bedrock (interpret/generate) → API Gateway (expose) → S3-hosted frontend (present) — no single service does everything; the value is in the composition.
- **Metadata is the bridge between "raw data" and "AI can query it accurately"** — without a Glue Data Catalog or Bedrock Knowledge Base describing schema/columns/types, the LLM has to guess table structure from cryptic names (`KNA1`, `col0`) and will generate incorrect SQL; investing in metadata quality directly improves SQL generation accuracy.
- **Self-correction via error feedback beats one-shot generation for structured output**: rather than hoping the first SQL query is syntactically perfect, the `run_sql` retry loop treats generation as iterative — feed the failure back, let the model see what went wrong, try again (bounded by `retry_max`).

## Anti-patterns
- **Skipping the metadata/knowledge-base step and expecting accurate SQL generation** — the chapter explicitly frames this as the difference between "ambiguous or incorrect SQL" and "precise, context-aware queries"; schema metadata isn't optional polish, it's load-bearing.
- **Not considering Athena's pay-per-TB-scanned pricing model** — unpartitioned, uncompressed, row-based data multiplies query cost; compress/partition/columnarize before scaling up query volume.
- **Rushing end-to-end organizational projects without stakeholder alignment** — the chapter's Strategic Visioning framework explicitly calls out risk/opportunity analysis and change management as prerequisites, not afterthoughts, for multi-stakeholder AI initiatives.
- **Ignoring pagination for large query result sets** — the text-to-SQL Lambda function explicitly notes that large result payloads can exceed client/server limits; implement pagination rather than returning unbounded results.
- **Forgetting resource cleanup after IoT/text-to-SQL exercises** — both worked examples end with an explicit multi-step teardown (Lambda, API Gateway, S3 buckets, Glue crawler/database/table, Knowledge Base, CloudWatch log groups) to avoid ongoing costs.

## Code Examples
```python
# IoT mock data publisher — MQTT to AWS IoT Core
mqtt_client = AWSIoTMQTTClient(CLIENT_ID)
mqtt_client.configureEndpoint(ENDPOINT, 8883)
mqtt_client.configureCredentials(PATH_TO_ROOT, PATH_TO_KEY, PATH_TO_CERT)
mqtt_client.connect()
while True:
    data = {
        'Timestamp': time.strftime("%Y-%m-%dT%H:%M:%S", time.gmtime()),
        'Temperature': round(random.uniform(20.0, 30.0), 2),
        'Humidity': round(random.uniform(30.0, 70.0), 2),
        'CO2': round(random.uniform(400.0, 800.0), 2)
    }
    mqtt_client.publish(TOPIC, json.dumps(data), 1)
    time.sleep(5)
```

```python
# Lambda: aggregate DynamoDB data and invoke Bedrock for insight generation
bedrock = boto3.client('bedrock-runtime')
input_text = f"""Given the following environmental data averages:
- Average Temperature: {round(avg_temp, 2)}°C
- Average Humidity: {round(avg_humidity, 2)}%
- Average CO2 Level: {round(avg_co2, 2)} ppm
Please provide an interpretation of these values, including potential environmental impacts,
health considerations, and any actions that could be taken to improve these conditions."""
payload = {
    "anthropic_version": "bedrock-2023-05-31", "max_tokens": 512,
    "messages": [{"role": "user", "content": input_text.strip()}]
}
response = bedrock.invoke_model(
    modelId='anthropic.claude-sonnet-4-20250514-v1:0',
    contentType='application/json', accept='application/json', body=json.dumps(payload))
```

```python
# Text-to-SQL: schema-grounded generation with retry-on-error
def generate_sql(query, sql_error_message="", db="", schema=""):
    details = """Ensure the SQL query complies with Athena (Presto) syntax..."""
    if sql_error_message == "":
        prompt = f"""You are an expert SQL assistant. {details}
Using the following database schema: {schema}
Generate an SQL query for the following question: "{query}"
Respond with only the SQL query and nothing else."""
    else:
        prompt = f"""The previous SQL query resulted in an error: {sql_error_message}
{details}
Using the following database schema: {schema}
Generate a corrected SQL query for the following question: "{query}"
Respond with only the SQL query and nothing else."""
    return db or 'sql-db', schema, invoke_claude(prompt)

def run_sql(query):
    database = 'sql-db'
    schemas = fetch_table_schema(database)
    schema_info = "\n".join([f"Table {s['Table']}: {s['Schema']}" for s in schemas])
    database, sql_schema, sql = generate_sql(query, schema=schema_info)
    code, sql_results = run_athena_query(database, sql)
    retry_counter = 0
    while code != 'SUCCEEDED' and retry_counter < 3:
        sql_error_message = f"Error: {sql_results}"
        database, sql_schema, sql = generate_sql(query, sql_error_message, database, sql_schema)
        code, sql_results = run_athena_query(database, sql)
        retry_counter += 1
    return sql, (parse_query_results(sql_results) if code == 'SUCCEEDED' else sql_results)
```

```python
# Retrieving schema-descriptive metadata for SQL grounding via Bedrock Knowledge Base RAG
def fetch_knowledge_base_references(query_text, model_identifier="anthropic.claude-sonnet-4-20250514-v1:0"):
    model_arn = f'arn:aws:bedrock:{region}::foundation-model/{model_identifier}'
    response = bedrock_agent_runtime_client.retrieve_and_generate(
        input={'text': query_text},
        retrieveAndGenerateConfiguration={
            'type': 'KNOWLEDGE_BASE',
            'knowledgeBaseConfiguration': {'knowledgeBaseId': knowledge_base_id, 'modelArn': model_arn}
        })
    return [ref.get("content", {}).get("text", "")
            for citation in response.get("citations", [])
            for ref in citation.get("retrievedReferences", [])]
```

- **What it demonstrates**: the full IoT-to-insight pipeline (MQTT publish → DynamoDB read → Bedrock interpretation), and the complete text-to-SQL orchestration (schema fetch → grounded generation → Athena execution → error-feedback retry loop).

## Reference Tables
| Service | Role in Pipeline |
|---|---|
| AWS IoT Core | secure device ingestion via MQTT |
| Amazon DynamoDB | low-latency sensor data storage |
| AWS Lambda | orchestration — read data, invoke Bedrock, format response |
| Amazon Bedrock | insight generation / natural-language-to-SQL translation |
| Amazon API Gateway | REST API exposure for frontend |
| Amazon S3 | static frontend hosting + raw/processed data lake |
| AWS Glue | metadata cataloging (crawler → Data Catalog) |
| Amazon Athena | serverless SQL query execution over S3 |
| Amazon OpenSearch | vector store for schema-metadata embeddings (text-to-SQL RAG) |

| Metadata Field | Purpose |
|---|---|
| table_name | identifies target table for generated SQL |
| columns[].name/type/description | grounds LLM's understanding of schema for accurate query generation |
| table description | overall table purpose context |

## Worked Example
**IoT Environmental Insights Pipeline** (full walkthrough):
1. Create DynamoDB table `EnvironmentalData` (partition key: `Timestamp`).
2. Create an IoT `Thing` (`EnvironmentSensor`) with certificate + policy for MQTT auth.
3. Run `mock_data_publisher.py` to simulate temperature/humidity/CO2 readings published to topic `environment/data`.
4. Configure an IoT Rule (`StoreEnvironmentData`) routing incoming messages into DynamoDB.
5. Lambda function `GetLatestData` scans the last 10 entries, computes averages, and invokes Bedrock (`anthropic.claude-sonnet-4-20250514-v1:0`) with a prompt asking for interpretation of the averages (environmental impact, health considerations, recommended actions).
6. Expose the Lambda via API Gateway (`/get-insights` GET endpoint).
7. Host a static HTML page on S3 (public read policy) that fetches the API and displays averages + AI-generated insights on button click.

**Text-to-SQL System** (full walkthrough):
1. Upload `environment_data.csv` to S3, crawl it with AWS Glue (`environment_data_crawler`) into a Glue Database `sql_db`, producing table `environment_data` with typed schema (Timestamp: string, Temperature/Humidity/CO2: double).
2. Set up Athena to query the S3 data directly via SQL (`SELECT * FROM environment_data LIMIT 10`).
3. Create a Bedrock Knowledge Base (`TextToSQLKB`) and upload `metadata.json` describing the table schema with column-level descriptions — this becomes the RAG grounding source for SQL generation.
4. Build a Lambda function (`TextToSQLFunction`) with a `tools.py` module providing: `fetch_table_schema` (Glue), `generate_sql` (Claude, schema-grounded prompt), `run_athena_query` (execute + poll), `parse_query_results`, and the `run_sql` orchestrator with retry-on-error.
5. Test: query "What is the average CO2 level over December 2024?" → Lambda interprets, generates `SELECT AVG(CO2) FROM environment_data WHERE Timestamp BETWEEN ...`, executes on Athena, returns `[{'_col0': '475.34'}]`.
6. Deploy via API Gateway; test with `curl -X POST .../run_sql -d '{"parameters": {"query": "..."}}'`.
7. Clean up all resources (Lambda, API Gateway, S3 buckets, Glue crawler/database/table, Knowledge Base, CloudWatch logs).

**Why it works**: both examples follow the identical shape (ingest → catalog/store → AI-process → expose → present) at different data types (streaming sensor data vs. structured tabular data), showing the pattern generalizes rather than being pipeline-specific.

## Key Takeaways
1. End-to-end generative AI projects follow a 5-phase lifecycle (Requirement Analysis → Model Dev → Testing → Deployment → Continuous Monitoring) that scales the single-model lifecycle from Ch1 to organizational, multi-stakeholder initiatives.
2. Metadata (Glue Data Catalog, Bedrock Knowledge Base schema descriptions) is what makes AI-generated SQL/queries accurate — without it, the LLM is guessing at cryptic table/column names.
3. Athena's pay-per-TB-scanned pricing makes data format (compression, partitioning, columnar) a direct cost lever, not just a performance one.
4. The retry-with-error-feedback pattern (feed SQL execution errors back into regeneration) is a general-purpose technique for improving structured-output reliability beyond one-shot generation.
5. Composing managed services (IoT Core, DynamoDB, Lambda, Bedrock, API Gateway, S3, Glue, Athena) — each doing one job — is the practical shape of an "end-to-end" AWS generative AI application.
6. Always tear down IoT/database/Lambda/API Gateway/Knowledge Base resources after prototyping to avoid ongoing cost — this is treated as a standard, non-optional step in every worked example.

## Connects To
- **Ch1**: the 7-phase solution life cycle here scales up to the 5-phase organizational end-to-end project lifecycle.
- **Ch3**: the Lambda + API Gateway + S3-hosted-frontend pattern extends the Flask web app pattern from the Bedrock API chapter.
- **Ch6**: the text-to-SQL Knowledge Base is a direct application of the RAG concepts (embeddings, vector store, retrieve-and-generate) from the RAG chapter, applied to schema metadata instead of documents.
- **Ch7**: Step Functions orchestration (mentioned for coordinating Glue/Bedrock/Athena) extends the orchestration patterns from the performance chapter.
- **Ch8**: IAM roles, least-privilege scoping, and encryption practices from the security chapter apply directly to every service touched in this chapter's pipelines.
