# ⚖️ AI Legal Assistant — Multi-Agent RAG System

A multi-agent legal assistant built with **CrewAI**. Describe a legal problem in plain English and a pipeline of four specialised agents will classify the issue, find applicable **Indian Penal Code (IPC)** sections using a RAG vector database, search for relevant precedent cases, and draft a formal legal document (FIR / complaint / legal notice).

> ⚠️ **Disclaimer:** This project is for educational and research purposes only. It is not a substitute for advice from a qualified lawyer. Outputs from LLMs may be inaccurate or incomplete — always verify with a legal professional.

---

## ✨ Features

- **Multi-agent workflow** — four sequential agents, each with a focused role and shared context
- **RAG over the IPC** — IPC sections stored in a Chroma vector DB with HuggingFace embeddings
- **Live precedent search** — Tavily search restricted to trusted Indian legal sources (Indian Kanoon)
- **Document drafting** — produces a formal complaint / notice with facts, sections, and requested action
- **Two interfaces** — Streamlit web app and a simple CLI script
- **Fast LLM inference** — Llama 3.3 70B served via Groq

---

## 🏗️ Architecture

```
User query (plain English)
        │
        ▼
┌──────────────────┐   structured JSON (case type, domain, summary, entities, jurisdiction)
│ Case Intake Agent│
└────────┬─────────┘
         ▼
┌──────────────────┐   top-3 IPC sections  ◄── Chroma vector DB (ipc.json + HF embeddings)
│ IPC Section Agent│
└────────┬─────────┘
         ▼
┌──────────────────────┐   precedent summary  ◄── Tavily search (indiankanoon.org)
│ Legal Precedent Agent│
└────────┬─────────────┘
         ▼
┌────────────────────────┐
│ Legal Drafter Agent    │ ──► Formal legal document
└────────────────────────┘
```

| Agent | Role | Tool | Temp |
|---|---|---|---|
| Case Intake Agent | Classifies the issue into structured JSON | — | 0 |
| IPC Section Agent | Retrieves the 3 most relevant IPC sections | `search_ipc_sections` (Chroma) | 0.3 |
| Legal Precedent Agent | Finds and summarises precedent cases | `search_legal_precedents` (Tavily) | 0 |
| Legal Drafter Agent | Drafts the final document from all prior outputs | — | 0.4 |

All agents use `groq/llama-3.3-70b-versatile`.

---

## 📁 Project Structure

```
.
├── app.py                      # Streamlit web UI
├── main.py                     # CLI entry point (runs a sample case)
├── crew.py                     # Crew definition (agents + tasks)
├── ipc_vectordb_builder.py     # Builds the Chroma vector DB from ipc.json
├── query_vectordb.py           # Quick script to test vector DB retrieval
├── ipc.json                    # IPC dataset (chapter, section, title, description)
├── sample_inputs.txt           # Example queries
├── env_template.txt            # Template for environment variables
├── requirements.txt
├── agents/
│   ├── case_intake_agent.py
│   ├── ipc_section_agent.py
│   ├── legal_precedent_agent.py
│   └── legal_drafter_agent.py
├── tasks/
│   ├── case_intake_task.py
│   ├── ipc_section_task.py
│   ├── legal_precedent_task.py
│   └── legal_drafter_task.py
└── tools/
    ├── ipc_sections_search_tool.py
    └── legal_precedent_search_tool.py
```

---

## 🚀 Getting Started

### Prerequisites

- Python 3.10+
- A [Groq API key](https://console.groq.com/)
- A [Tavily API key](https://tavily.com/)

### 1. Clone and install

```bash
git clone <your-repo-url>
cd AI-Legal-Assistant_Multi-Agent-RAG-System

python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

pip install -r requirements.txt
```

### 2. Configure environment variables

Create a `.env` file in the project root (use `env_template.txt` as a guide):

```env
GROQ_API_KEY=your_groq_api_key
TAVILY_API_KEY=your_tavily_api_key
IPC_JSON_PATH=./ipc.json
PERSIST_DIRECTORY_PATH=./chroma_vectordb
IPC_COLLECTION_NAME=ipc_collection
```

| Variable | Description |
|---|---|
| `GROQ_API_KEY` | Key for LLM inference via Groq |
| `TAVILY_API_KEY` | Key for precedent web search |
| `IPC_JSON_PATH` | Path to the IPC JSON dataset |
| `PERSIST_DIRECTORY_PATH` | Where the Chroma DB is stored |
| `IPC_COLLECTION_NAME` | Chroma collection name |

### 3. Build the vector database (one-time)

```bash
python ipc_vectordb_builder.py
```

Optionally verify retrieval:

```bash
python query_vectordb.py
```

> Note: `query_vectordb.py` reads `PERSIST_DIRECTORY_NAME` (a folder name relative to the project) and uses the hard-coded collection `ipc_collection`. Add `PERSIST_DIRECTORY_NAME=chroma_vectordb` to your `.env` to use it, or update the script to use `PERSIST_DIRECTORY_PATH`.

### 4. Run

**Web app (Streamlit):**

```bash
streamlit run app.py
```

**Command line:**

```bash
python main.py
```

Edit the `user_input` string in `main.py` to try your own case.

---

## 💡 Example

**Input**

> A man broke into my house at night while my family was sleeping. He stole jewelry and cash from our bedroom. When I confronted him, he threatened me with a knife and ran away. We reported it to the police, but I'm not sure which legal charges should be filed under IPC.

**Output (summary)**

1. Case classification (criminal law — house-breaking, theft, criminal intimidation)
2. Top 3 relevant IPC sections with chapter and text
3. Summary of related precedents from Indian Kanoon
4. A formal complaint draft with parties, facts, applicable sections, and request for action

More examples are in `sample_inputs.txt`.

---

## 🧰 Tech Stack

- [CrewAI](https://github.com/crewAIInc/crewAI) — multi-agent orchestration
- [Groq](https://groq.com/) — Llama 3.3 70B inference
- [ChromaDB](https://www.trychroma.com/) + [LangChain](https://www.langchain.com/) — vector store and retrieval
- [HuggingFace sentence-transformers](https://www.sbert.net/) — embeddings
- [Tavily](https://tavily.com/) — web search for precedents
- [Streamlit](https://streamlit.io/) — UI

---

## 🛣️ Possible Improvements

- Add BNS / BNSS / CrPC datasets
- Cache the vector store and embeddings
- Show each agent's intermediate output in the Streamlit UI
- Export the drafted document to PDF / DOCX
- Add more trusted legal sources and civil-law coverage
- Add evaluation tests for retrieval accuracy

---

## 🤝 Contributing

Issues and pull requests are welcome. Please open an issue first to discuss major changes.


