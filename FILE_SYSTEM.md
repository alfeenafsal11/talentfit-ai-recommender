# TalentFit AI Recommender: File System Documentation

This document provides a comprehensive breakdown of the directory structure and file system of the **TalentFit AI Recommender System**. It explains the purpose of each file and folder, their role in the multi-agent deterministic orchestrator architecture, and how they connect to enable offline hybrid retrieval, intent classification, and API routing.

---

## Directory Tree

Below is the directory tree of the workspace, showing all tracked source code, prompts, tests, and configuration files.

```text
talentfit-ai-recommender/
│
├── .env.example                    # Template for environment variables (API keys)
├── .gitignore                      # Git exclusion rules
├── APPROACH.md                     # Technical architecture and optimization design doc
├── README.md                       # Startup, run, and local setup guide
├── SHL_AI_Intern_Assignment.pdf    # Original take-home assignment description
├── System Approach.docx            # Alternate format of the APPROACH.md document
├── shl_system_architecture.svg     # SVG diagram mapping system routing and processing
│
├── dataset/                        # Reference catalog and evaluation traces
│   ├── catalog.txt                 # Master JSON catalog of 377 SHL assessments
│   └── sample_conversations/       # Benchmark conversational traces
│       └── GenAI_SampleConversations/
│           ├── C1.md to C10.md     # 10 multi-turn test conversation markdown files
│
├── outputs/                        # Output folder for logs and report generation
│   └── final_eval_report.txt       # Saved performance logs and Recall@10 summaries
│
└── shl_recommender/                # Core FastAPI service directory
    ├── Dockerfile                  # Production container definition (precomputes caches)
    ├── render.yaml                 # Infrastructure-as-code configuration for Render
    ├── requirements.txt            # Python dependencies (fastapi, sentence-transformers, faiss, etc.)
    ├── main.py                     # Entrypoint; defines /health, /chat, and server lifecycle
    ├── precompute.py               # Offline embedding generation — run during Docker build
    ├── catalog.py                  # Normalization logic, aliases, and catalog mapping (Agent 0)
    ├── retrieval.py                # Hybrid sparse (BM25) + dense (FAISS) retrieval engine (Agent 1)
    ├── intent.py                   # Stateless message parsing and intent/slot extraction (Agent 2)
    ├── workflow.py                 # Central orchestrator router and specialized workflows (Agents 3-10)
    ├── llm.py                      # LLM API connection (Groq/OpenAI) and fallback handler
    │
    ├── prompts/                    # Delimited instructions for specialized LLM tasks
    │   ├── clarify.txt             # Instructions for asking multi-slot clarification questions
    │   ├── compare.txt             # Guidance for comparing catalog features between products
    │   ├── recommend.txt           # Setup for synthesizing reasons why candidates fit
    │   ├── refine.txt              # Guidelines for tweaking recommendations based on edits
    │   └── refusal.txt             # Direct instructions for rejecting out-of-scope inputs
    │
    ├── tests/                      # Local testing directory
    │   ├── test_behavior.py        # Behavior checks (refusal logic, schema validation)
    │   └── test_api_deployed.py    # Deployed API testing client verifying endpoint status
    │
    └── utils/                      # Helper placeholders for system extension
        ├── normalization.py        # Extension helper for custom string normalization
        ├── scoring.py              # Extension helper for custom metrics
        └── validators.py           # Extension helper for custom schemas
```

---

## Detailed File Descriptions

### 1. Root Configuration & Documentation Files

*   **[.env.example](file:///d:/PROJECTS/talentfit-ai-recommender/.env.example)**
    *   *Purpose*: Templates for the API keys required to run the LLM portions of the system.
    *   *Usage*: Developers copy this to `shl_recommender/.env` and add a valid `GROQ_API_KEY` (primary provider) or `OPENAI_API_KEY` (fallback provider).
*   **[APPROACH.md](file:///d:/PROJECTS/talentfit-ai-recommender/APPROACH.md)**
    *   *Purpose*: A detailed system approach and design report.
    *   *Contents*: Details the hybrid search methodology (BM25 + FAISS Cosine), startup cold-start optimizations, prompt injection guards, and local evaluation results.
*   **[README.md](file:///d:/PROJECTS/talentfit-ai-recommender/README.md)**
    *   *Purpose*: The user guide for the repository.
    *   *Contents*: Instructions for installing local dependencies, configuring environment variables, running the local FastAPI development server, querying HTTP endpoints, and executing the evaluation harness.
*   **[shl_system_architecture.svg](file:///d:/PROJECTS/talentfit-ai-recommender/shl_system_architecture.svg)**
    *   *Purpose*: A detailed SVG diagram representing the request-response lifecycle of the application.
    *   *Visualized Flow*: Shows how request payloads are evaluated by the scope guards, how conversation state is rebuilt, how intents are matched, and how the hybrid retrieval engine feeds candidate items into the LLM synthesis pipeline.

---

### 2. Dataset and Evaluation Folder (`dataset/`)

*   **[dataset/catalog.txt](file:///d:/PROJECTS/talentfit-ai-recommender/dataset/catalog.txt)**
    *   *Purpose*: The master database of available assessments.
    *   *Format*: A JSON array containing 377 products. Each product entry records details such as `entity_id`, `name`, `link` (official SHL product catalog URL), `description`, target `languages`, typical `duration_raw`, `keys` (categories), and target `job_levels`.
*   **[dataset/sample_conversations/GenAI_SampleConversations/](file:///d:/PROJECTS/talentfit-ai-recommender/dataset/sample_conversations/GenAI_SampleConversations/)**
    *   *Purpose*: The benchmark test suite used to evaluate retrieval and workflow effectiveness.
    *   *Contents*: Files `C1.md` through `C10.md` detailing multi-turn simulated dialogues between hiring managers (asking for java devs, graduate trainees, call center reps, etc.) and the agent, ending with expected assessment URLs.

---

### 3. Deployed and Local Logs (`outputs/`)

*   **[outputs/final_eval_report.txt](file:///d:/PROJECTS/talentfit-ai-recommender/outputs/final_eval_report.txt)**
    *   *Purpose*: Summary report capturing evaluation results.
    *   *Contents*: Logs of behavior probes (all 10 passing) and individual Recall@10 scores per conversation template, calculating a final Mean Recall@10 metric.

---

### 4. FastAPI Recommender App Code (`shl_recommender/`)

This directory houses the core application logic. It is structured into stateless functional boundaries:

#### Configuration & Build
*   **[shl_recommender/Dockerfile](file:///d:/PROJECTS/talentfit-ai-recommender/shl_recommender/Dockerfile)**
    *   *Purpose*: Orchestrates building the production-ready Docker container.
    *   *Key Step*: Runs `precompute.py` during the image building phase. This caches all embedding models and precomputes vectors, reducing startup cold-starts to under 51 seconds on memory-constrained hosting tiers like Render.
*   **[shl_recommender/render.yaml](file:///d:/PROJECTS/talentfit-ai-recommender/shl_recommender/render.yaml)**
    *   *Purpose*: Infrastructure-as-code for Render cloud deployments.
    *   *Configuration*: Launches a containerized web service using the Dockerfile, setting environment paths for the catalog text and precomputed embedding caches.
*   **[shl_recommender/requirements.txt](file:///d:/PROJECTS/talentfit-ai-recommender/shl_recommender/requirements.txt)**
    *   *Purpose*: Lists necessary Python packages.
    *   *Pinning*: Restricts `numpy` to `<2.0.0` to preserve binary compatibility with `faiss-cpu`, preventing compilation crashes.

#### Web Interface
*   **[shl_recommender/main.py](file:///d:/PROJECTS/talentfit-ai-recommender/shl_recommender/main.py)**
    *   *Purpose*: FastAPI application entrypoint.
    *   *Endpoints*:
        *   `GET /health`: Used by hosting load balancers for readiness checks.
        *   `POST /chat`: Receives conversation history, triggers processing, and returns recommendations.
    *   *Server Lifespan*: Triggers catalog loading and instantiates the `RetrievalEngine` at server start.

#### Core Logic Modules
*   **[shl_recommender/precompute.py](file:///d:/PROJECTS/talentfit-ai-recommender/shl_recommender/precompute.py)**
    *   *Purpose*: Offline generation of embeddings.
    *   *Workflow*: Loads `catalog.txt`, downloads the `all-MiniLM-L6-v2` transformer model, encodes the catalog descriptions, and saves them to `cache/embeddings.pkl`.
    *   *Impact*: Eliminates the all-MiniLM-L6-v2 model download from runtime, reducing cold-start to under 51 seconds.
*   **[shl_recommender/catalog.py](file:///d:/PROJECTS/talentfit-ai-recommender/shl_recommender/catalog.py)**
    *   *Purpose*: Normalizes raw data and maps aliases (Agent 0).
    *   *Key Components*:
        *   `SENIORITY_MAP` & `SENIORITY_ALIASES`: Normalizes levels (e.g., "fresh", "junior" → "entry").
        *   `ROLE_ALIASES` & `SKILL_KEYWORD_MAP`: Expands short user phrases (e.g., "devops" → Docker, Kubernetes).
        *   `_fix_broken_strings()`: Robust JSON parser that handles multiline unescaped quotes in descriptions.
*   **[shl_recommender/retrieval.py](file:///d:/PROJECTS/talentfit-ai-recommender/shl_recommender/retrieval.py)**
    *   *Purpose*: Main retrieval backend (Agent 1).
    *   *Hybrid Engine*: Performs BM25 keyword matching and FAISS dense vector search, combining scores linearly ($Score = 0.55 \times Dense + 0.35 \times Sparse + 0.1$).
    *   *Filters*: Implements strict post-retrieval filtering on seniority, categories, language, and maximum duration.
*   **[shl_recommender/intent.py](file:///d:/PROJECTS/talentfit-ai-recommender/shl_recommender/intent.py)**
    *   *Purpose*: Dialogue parsing and state tracking (Agent 2).
    *   *ConversationState*: Rebuilds state dynamically from scratch on every call. Extracts constraints, requirements (cognitive/personality tests), and roles.
    *   *Intent Classification*: Detects whether the user wants to compare, refine, greet, or require recommendations.
*   **[shl_recommender/workflow.py](file:///d:/PROJECTS/talentfit-ai-recommender/shl_recommender/workflow.py)**
    *   *Purpose*: Deterministic Orchestrator and Router (Agents 3-10).
    *   *Routing Logic*: Redirects requests to specialized handlers based on parsed intent:
        *   *Greeting Agent*: Returns a friendly introduction.
        *   *Clarification Agent*: Prompts for missing fields (e.g., role or seniority) using deterministic templates first, falling back to LLM-generated questions.
        *   *Recommendation/Refinement Agent*: Executes hybrid search, handles post-filtering, ensures mandatory types (e.g., OPQ for personality) are injected, and calls synthesis templates.
        *   *Comparison Agent*: Identifies two specific products and compares them strictly using catalog properties.
        *   *Scope/Safety Guard*: Detects off-topic input, system prompt jailbreaks, or legal inquiries, returning structured refusals immediately.
        *   *Output Validator*: Strips invented URLs, caps recommendations at 10, and ensures strict schema adherence.
*   **[shl_recommender/llm.py](file:///d:/PROJECTS/talentfit-ai-recommender/shl_recommender/llm.py)**
    *   *Purpose*: LLM endpoint connectors and fallback engine.
    *   *Resilience*: Connects to Groq (`llama-3.3-70b-versatile`) with a fallback option for OpenAI (`gpt-4o-mini`).
    *   *Graceful Fallback*: If both APIs fail (due to rate limits, offline state, or missing keys), the system seamlessly returns a structured, natural-language string listing the matched catalog URLs.

---

### 5. Prompts Folder (`shl_recommender/prompts/`)

Text files containing prompt instructions designed to keep LLM outputs within boundaries:

*   **[prompts/clarify.txt](file:///d:/PROJECTS/talentfit-ai-recommender/shl_recommender/prompts/clarify.txt)**: Directs the LLM on how to prompt for multiple missing items in a concise manner without referencing specific assessments prematurely.
*   **[prompts/compare.txt](file:///d:/PROJECTS/talentfit-ai-recommender/shl_recommender/prompts/compare.txt)**: Restricts product comparisons to catalog data only, prohibiting hallucinated differences.
*   **[prompts/recommend.txt](file:///d:/PROJECTS/talentfit-ai-recommender/shl_recommender/prompts/recommend.txt)**: Restricts synthesis to explaining why the retrieval-chosen assessments match the user's role.
*   **[prompts/refine.txt](file:///d:/PROJECTS/talentfit-ai-recommender/shl_recommender/prompts/refine.txt)**: Acknowledges adjustments to the recommendation list and explains the modifications.
*   **[prompts/refusal.txt](file:///d:/PROJECTS/talentfit-ai-recommender/shl_recommender/prompts/refusal.txt)**: Explains how to decline legal/general advice requests while keeping context focused on assessment selection.

---

### 6. Tests Folder (`shl_recommender/tests/`)

*   **[tests/test_behavior.py](file:///d:/PROJECTS/talentfit-ai-recommender/shl_recommender/tests/test_behavior.py)**
    *   *Purpose*: Test hooks targeting specific conversational patterns.
*   **[tests/test_api_deployed.py](file:///d:/PROJECTS/talentfit-ai-recommender/shl_recommender/tests/test_api_deployed.py)**
    *   *Purpose*: Deployed verification script.
    *   *Functionality*: Pings `GET /health` and runs `POST /chat` payloads against the live Render server to confirm route parsing, validation schemas, and candidate response formatting.

---

### 7. Core Extension Stubs (`shl_recommender/utils/`)

*   **[utils/normalization.py](file:///d:/PROJECTS/talentfit-ai-recommender/shl_recommender/utils/normalization.py)**: Extension stub for custom seniority mapping and string cleaning logic.
*   **[utils/scoring.py](file:///d:/PROJECTS/talentfit-ai-recommender/shl_recommender/utils/scoring.py)**: Extension stub for custom scoring and evaluation metric calculations.
*   **[utils/validators.py](file:///d:/PROJECTS/talentfit-ai-recommender/shl_recommender/utils/validators.py)**: Extension stub for custom schema and input validation logic.
