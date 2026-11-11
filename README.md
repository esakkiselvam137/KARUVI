<div align="center">

# 🎬 KARUVI

### Story-to-Screenplay AI

*Turn a few lines of story into a properly formatted screenplay part, without losing ownership of your story.*

![Status](https://img.shields.io/badge/status-in%20development-orange)
![Model](https://img.shields.io/badge/model-Qwen3--4B--Instruct--2507-blue)
![Fine--tuning](https://img.shields.io/badge/fine--tuning-QLoRA-purple)
![Backend](https://img.shields.io/badge/backend-FastAPI-009688)
![Database](https://img.shields.io/badge/database-SQLite-003B57)

</div>

---

## ✨ What is KARUVI?

KARUVI helps writers convert a small bunch of story lines into a **screenplay part**: one scene or a short sequence, not an entire movie.

You write:

```text
Maya returns to the old place at night.
She hears a sound upstairs.
Her brother calls her, but she does not answer.
```

KARUVI turns it into structured, validated screenplay content with scene headings, action lines, characters, and dialogue. If something important is missing, it **asks you**. It never makes it up.

---

## 🧭 Core Principles

| Principle | What it means |
|---|---|
| 🛡️ **User ownership** | Your original story belongs to you. It is never silently altered. |
| ✅ **Approval first** | The AI can suggest changes, but nothing becomes permanent until you approve it. |
| 🚫 **No invention** | Missing details are asked about through guided Q&A, never guessed. |
| 🧩 **Partial generation** | Generates only the relevant screenplay part, with no forced plot, theme, or ending. |
| 📐 **Schema-validated** | Every generated part must pass JSON Schema validation before it is shown as final. |
| 🕰️ **Version history** | Every approved version is saved, and you can roll back to any of them. |

---

## 🔄 How It Works

```text
Create project
      ↓
Enter story lines (or audio → transcript)
      ↓
Backend saves the raw story input
      ↓
Model call 1: "Analyze this story"  →  Story Analysis JSON
      ↓
Backend validates analysis + applies completeness rules
      ↓
Missing or unclear details?
   ├── Yes → Guided Q&A (small, specific questions)
   └── No  → continue
      ↓
Model call 2: "Generate screenplay using story + answers"
      ↓
Backend validates Screenplay JSON
      ↓
Readable screenplay preview
      ↓
Request edits → Proposal → Approve / Edit / Reject
      ↓
Approved version saved to version history
```

### Two model calls

| Call | Input | Output |
|---|---|---|
| **1. Analysis** | Original story input | Story Analysis JSON |
| **2. Generation** | Story input + user answers | Screenplay JSON |

> ⚠️ The backend sends the model **exactly** the prompt format used during QLoRA training (labels, order, spacing, field names, and so on). Prompt templates live in `model/prompts/`.

---

## 🧠 The Model

| Item | Detail |
|---|---|
| Base model | `Qwen/Qwen3-4B-Instruct-2507` |
| Fine-tuning | QLoRA |
| Trained to | Identify characters, locations, and events; convert narrative to screenplay format; output schema-conforming JSON; avoid unsupported facts; flag missing details for Q&A |
| Artifacts | Stored privately on Hugging Face as `KARUVI-Qwen3-4B-QLoRA` |
| RAG | Not part of V1. Planned for a later version. |

---

## 🏗️ Architecture

```text
Frontend ──► Router ──► Service ──► Repository ──► SQLite
                           │
                           ├──► Model Adapter ──► LLM (mock / Hugging Face / local)
                           ├──► Validator (JSON Schema)
                           └──► Renderer (JSON → readable screenplay)
```

| Layer | Responsibility |
|---|---|
| **Router** | Handles HTTP requests and responses. No deep business logic. |
| **Service** | Applies KARUVI rules: when to ask questions, when to call the model, when a proposal can be approved. |
| **Repository** | All database reads and writes. |
| **Adapter** | Talks to the model and hides whether it is a mock, Hugging Face, or local backend. |
| **Validator** | Checks model output against the screenplay JSON schema. |
| **Renderer** | Converts validated JSON into readable screenplay text. |

The frontend **never** calls the model directly. It only talks to the backend API.

---

## 📦 Repositories & Storage

| Where | Purpose |
|---|---|
| **GitHub: `story2scriptmodel`** | Dataset only (story-screenplay data, preparation notes, versions) |
| **GitHub: `KARUVI`** *(this repo)* | Training config, prompts, schemas, evaluation, backend, frontend, docs |
| **Hugging Face: `KARUVI-Qwen3-4B-QLoRA`** | Private model artifacts (adapter, tokenizer, model card, results) |
| **Kaggle** | Temporary GPU training environment only, not long-term storage |
| **Hugging Face Space (ZeroGPU)** | Planned public demo |

---

## 🗂️ Project Structure

```text
KARUVI/
├── app/
│   ├── adapters/        # model adapters (mock, Hugging Face) + prompt template
│   ├── core/            # config, database, exceptions, responses
│   ├── repositories/    # database access layer
│   ├── routers/         # API endpoints
│   ├── schemas/         # request/response models
│   ├── services/        # KARUVI business rules
│   ├── utils/           # prompt builder, validator, renderer
│   ├── tests/
│   ├── frontend/
│   └── main.py
├── model/
│   ├── configs/         # training configuration
│   ├── prompts/         # exact training-style prompt templates
│   ├── schemas/         # screenplay + story analysis JSON schemas
│   ├── training/
│   └── evaluation/
├── docs/                # API contract, architecture, DB design, interaction flow
├── deployment/          # Hugging Face Space + local run guides
├── examples/            # demo projects and sample outputs
├── requirements.txt
└── README.md
```

> The structure above is the planned layout. See `docs/architecture.md` for details as it evolves.

---

## 🔌 Planned API

| Method | Endpoint | Purpose |
|---|---|---|
| `POST` | `/projects` | Create a story project |
| `GET` | `/projects` | List projects |
| `GET` | `/projects/{id}` | Load project, current screenplay, and status |
| `PUT` | `/projects/{id}/story` | Save or update a story input |
| `GET` | `/projects/{id}/questions` | Get pending guided questions |
| `POST` | `/projects/{id}/answers` | Save answers |
| `POST` | `/projects/{id}/generate` | Generate the next screenplay part |
| `POST` | `/projects/{id}/edits` | Request an edit |
| `POST` | `/projects/{id}/proposals/{proposal_id}/approve` | Approve a proposal |
| `POST` | `/projects/{id}/proposals/{proposal_id}/reject` | Reject a proposal |
| `GET` | `/projects/{id}/versions` | List approved versions |
| `GET` | `/projects/{id}/export` | Export readable screenplay |
| `GET` | `/system/status` | Live / limited / archive mode and quota |
| `GET` | `/examples` | Saved demo examples |

Full details: [`docs/api-contract.md`](docs/api-contract.md)

---

## 🧾 Screenplay JSON (example)

```json
{
  "title": "",
  "genre": "",
  "characters": [
    { "name": "MAYA", "age": "", "description": "", "status": "active" }
  ],
  "locations": [
    { "name": "OLD HOUSE", "description": "", "status": "active" }
  ],
  "scenes": [
    {
      "scene_number": 1,
      "heading": "OLD HOUSE — NIGHT",
      "location": "OLD HOUSE",
      "time": "NIGHT",
      "characters": ["MAYA"],
      "description": "Maya enters the old house and hears a sound upstairs.",
      "dialogue": [
        { "character": "MAYA", "parenthetical": "", "line": "Hello? Is anyone here?" }
      ],
      "actions": [
        "Maya enters the old house.",
        "A faint sound comes from upstairs."
      ]
    }
  ]
}
```

The exact schema may evolve during implementation. See `model/schemas/screenplay_schema.json`.

---

## 🚦 Application Modes

| Mode | Description |
|---|---|
| 🟢 **Live generation** | GPU quota available; full generate → validate → approve flow |
| 🟡 **Quota limited** | Daily quota reached; project state preserved, archive offered |
| 🔵 **Archive demo** | No live backend; browse saved example projects and Q&A flows |
| ⚙️ **Local run** | Optional advanced mode for users with suitable hardware |

---

## 🗺️ Roadmap

| # | Step | Status |
|---|---|---|
| 1 | Finalize repository structure | 🔄 In progress |
| 2 | Finalize application contract | ⏳ Planned |
| 3 | Design SQLite database | ⏳ Planned |
| 4 | Minimal FastAPI backend | ⏳ Planned |
| 5 | Mock model adapter | ⏳ Planned |
| 6 | JSON Schema validation | ⏳ Planned |
| 7 | Screenplay renderer | ⏳ Planned |
| 8 | Guided Q&A | ⏳ Planned |
| 9 | Approval and version history | ⏳ Planned |
| 10 | Frontend | ⏳ Planned |
| 11 | Replace mock adapter with real model adapter | ⏳ Planned |
| 12 | Saved demo examples | ⏳ Planned |
| 13 | Hugging Face Space demo (ZeroGPU) | ⏳ Planned |
| 14 | Quota, archive, and error-state UI | ⏳ Planned |
| 15 | Research paper artifacts | ⏳ Planned |

**Model training:** ⏸️ paused while the Kaggle GPU setup is being fixed. Next: complete QLoRA training, save the adapter, then evaluate.

---

## 🚧 Current Status

KARUVI is **in early development**. The repository is currently a project skeleton: most files are placeholders, and the backend, frontend, database, and model integration are **not yet implemented**.

---

## 🔒 Security & Privacy

- Never commit `.env` files, tokens, or private keys. Use `.env.example` for non-secret placeholders.
- User story content is treated as private.
- The Hugging Face model repository is private.

---

## 👥 Team

Built by a team of five as a learning and research project.

<!-- Add names / roles / links here -->

---

## 📄 License

License: **Other (karuvi)**. See the license details in the model repository.

<!-- Add a LICENSE file when finalized -->

---

<div align="center">

*KARUVI — your story, your screenplay, your approval.* 🎬

</div>
