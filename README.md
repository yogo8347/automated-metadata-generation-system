<div align="center">

<img width="100%" src="https://capsule-render.vercel.app/api?type=waving&color=0:0f2b5b,50:1a5fa8,100:0ea5c9&height=210&section=header&text=MARS&fontSize=80&fontColor=ffffff&fontAlignY=38&desc=Automated%20Metadata%20Generation%20System&descAlignY=60&descSize=20&animation=fadeIn" />

<br/>

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?style=for-the-badge&logo=jupyter&logoColor=white)](https://jupyter.org)
[![Gemini](https://img.shields.io/badge/Gemini_2.0_Flash-4285F4?style=for-the-badge&logo=google&logoColor=white)](https://ai.google.dev)
[![LangChain](https://img.shields.io/badge/LangChain-1C3C3C?style=for-the-badge&logo=langchain&logoColor=white)](https://langchain.com)
[![Gradio](https://img.shields.io/badge/Gradio-FF7C00?style=for-the-badge&logo=gradio&logoColor=white)](https://gradio.app)
[![License: MIT](https://img.shields.io/badge/License-MIT-10b981?style=for-the-badge)](LICENSE)

<br/>

> **MARS** is an end-to-end automated metadata generation system that extracts semantically rich, structured metadata from unstructured documents (PDF, DOCX, TXT) using a multi-stage LLM pipeline with RAG, OCR fallback, and a Gradio web interface.

<br/>

**Submitted by:** Aagam Bandi (22124001)

</div>

---

## 📌 Table of Contents

- [Overview](#-overview)
- [How It Works](#-how-it-works)
- [System Architecture](#-system-architecture)
- [Features](#-features)
- [Tech Stack](#-tech-stack)
- [Installation](#-installation)
- [Usage](#-usage)
- [Project Structure](#-project-structure)
- [Pipeline Steps](#-pipeline-steps-detailed)
- [Web Interface](#-web-interface)
- [Known Limitations](#-known-limitations)
- [Contributing](#-contributing)
- [License](#-license)

---

## 🔍 Overview

MARS solves the problem of **manual, inconsistent metadata tagging** on large document repositories. Given any document in PDF, DOCX, or TXT format, MARS automatically produces structured, semantically rich metadata — including title, authors, topics, keywords, summary, and more — without human intervention.

The system is designed to be:
- **Scalable** — handles large documents by chunking before LLM processing
- **Consistent** — structured JSON output format across all document types
- **Semantically rich** — uses RAG to ensure the most relevant sections inform metadata
- **Accessible** — Gradio web UI requires no technical knowledge to operate

---

## ⚙️ How It Works

The pipeline follows a **5-stage process**:

```
Document Input
     │
     ▼
[1] File Type Detection
     │  .txt / .docx → Direct Text Extraction
     │  .pdf         → PyMuPDF + OCR Fallback (Tesseract)
     ▼
[2] Text Chunking
     │  RecursiveCharacterTextSplitter
     │  chunk_size=1250, chunk_overlap=250
     ▼
[3] Parallel Chunk Summarization
     │  Concurrent async API calls → Gemini 2.0 Flash
     │  Combined summary preserving key metadata signals
     ▼
[4] RAG — Relevant Chunk Retrieval
     │  LLM generates metadata-focused questions from combined summary
     │  ChromaDB + HuggingFace all-MiniLM-L6-v2 embeddings
     │  Top-k relevant chunks retrieved via similarity search
     ▼
[5] Metadata Generation
     │  Retrieved chunks + combined summary → Gemini 2.0 Flash
     ▼
Structured Metadata Output (JSON)
```

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        MARS Pipeline                        │
│                                                             │
│  ┌──────────┐    ┌──────────────┐    ┌───────────────────┐  │
│  │  Input   │───▶│  Extraction  │───▶│    Chunking       │  │
│  │PDF/DOCX/ │    │ PyMuPDF      │    │ RecursiveChar     │  │
│  │  TXT     │    │ Tesseract OCR│    │ TextSplitter      │  │
│  └──────────┘    └──────────────┘    └────────┬──────────┘  │
│                                               │             │
│  ┌────────────────────────────────────────────▼──────────┐  │
│  │           Parallel Summarization (Async)               │  │
│  │           Gemini 2.0 Flash — up to 200 concurrent     │  │
│  └───────────────────────────────┬───────────────────────┘  │
│                                  │                          │
│  ┌───────────────────────────────▼───────────────────────┐  │
│  │     RAG Layer                                          │  │
│  │  ChromaDB  ◀──  HuggingFace Embeddings (MiniLM-L6-v2) │  │
│  │  Retriever: similarity search, k=3                    │  │
│  └───────────────────────────────┬───────────────────────┘  │
│                                  │                          │
│  ┌───────────────────────────────▼───────────────────────┐  │
│  │     Final Metadata Generation                          │  │
│  │     Gemini 2.0 Flash  ──  Structured JSON Output      │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │               Gradio Web Interface                  │    │
│  │         Upload → Process → View Metadata            │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

---

## ✨ Features

- 📄 **Multi-format support** — PDF, DOCX, and TXT files
- 🔍 **Smart OCR fallback** — detects scanned/image-heavy PDF pages using `ocr_fallback_threshold` and runs Tesseract OCR only where needed, avoiding overhead on clean PDFs
- ✂️ **Token-aware chunking** — prevents LLM context overflow via `RecursiveCharacterTextSplitter` with chunk overlap to preserve sentence meaning across boundaries
- ⚡ **Concurrent LLM calls** — parallel async API calls (up to 200 concurrent) for fast chunk summarization using Python `asyncio`
- 🧠 **RAG-enhanced metadata** — uses the combined summary to auto-generate retrieval queries, ensuring the most semantically relevant document sections inform metadata
- 🗃️ **Vector search** — ChromaDB + `all-MiniLM-L6-v2` HuggingFace embeddings for efficient similarity-based chunk retrieval
- 📋 **Structured output** — metadata returned as clean, parseable JSON
- 🌐 **Gradio web interface** — drag-and-drop file upload, real-time processing status, and formatted metadata display
- 🔗 **Public URL sharing** — Gradio generates a shareable public link valid for 1 week

---

## 🛠️ Tech Stack

| Category | Technology |
|---|---|
| **LLM** | Google Gemini 2.0 Flash |
| **LLM Framework** | LangChain, `langchain-google-genai` |
| **PDF Extraction** | PyMuPDF (`fitz`) |
| **OCR** | Tesseract via `pytesseract` |
| **DOCX Extraction** | LangChain `UnstructuredWordDocumentLoader` |
| **TXT Extraction** | LangChain `TextLoader` |
| **Embeddings** | HuggingFace `all-MiniLM-L6-v2` |
| **Vector Store** | ChromaDB |
| **Chunking** | LangChain `RecursiveCharacterTextSplitter` |
| **Web UI** | Gradio |
| **Async Processing** | Python `asyncio` |
| **Environment** | `python-dotenv` |

---

## 📦 Installation

### Prerequisites

- Python 3.10+
- Tesseract OCR installed on your system:
  - **macOS:** `brew install tesseract`
  - **Ubuntu/Debian:** `sudo apt-get install tesseract-ocr`
  - **Windows:** [Download installer](https://github.com/UB-Mannheim/tesseract/wiki)

### Steps

```bash
# 1. Clone the repository
git clone https://github.com/your-username/mars-metadata-generator.git
cd mars-metadata-generator

# 2. Create and activate a virtual environment
python -m venv venv
source venv/bin/activate        # On Windows: venv\Scripts\activate

# 3. Install Python dependencies
pip install -r requirements.txt

# 4. Set up environment variables
cp .env.example .env
# Open .env and add your Google Gemini API key:
# GOOGLE_API_KEY=your_google_gemini_api_key_here
```

### requirements.txt

```
langchain
langchain-community
langchain-text-splitters
langchain-google-genai
google-generativeai
chromadb
sentence-transformers
pymupdf
pytesseract
Pillow
gradio
python-dotenv
unstructured
```

---

## 🚀 Usage

### Option 1: Run the Gradio Web Interface (Recommended)

Open the notebook and run the **last cell** to launch the web app:

```bash
jupyter notebook metadata_generation.ipynb
```

> ⚠️ **Important:** Always run the last code cell to generate a **fresh Gradio link**. Do not use previously generated links — they expire after 1 week.

The terminal will display:
```
* Running on local URL:  http://127.0.0.1:7860
* Running on public URL: https://xxxxxxxxxxxxxx.gradio.live
```

Open either URL in your browser to access the interface.

### Option 2: Run Step-by-Step in the Notebook

The notebook is organized into incremental cells so you can understand each stage:

| Cell | Description |
|---|---|
| Cell 3 | File type detection |
| Cell 4 | Text extraction with OCR fallback |
| Cell 5 | Chunking + embeddings + parallel summarization + RAG + metadata generation |
| Cell 7 | **Full pipeline integrated into Gradio UI** ← Run this for the web app |

### Option 3: Programmatic Usage

```python
import asyncio
from metadata_generation import generate_metadata_from_file

# Pass the path to your document
result = asyncio.run(generate_metadata_from_file("path/to/your_document.pdf"))
print(result)
```

---

## 📁 Project Structure

```
mars-metadata-generator/
│
├── 📓 metadata_generation.ipynb    # Main notebook — full pipeline + Gradio UI
├── 📄 .env                         # API keys (not committed to git)
├── 📄 .env.example                 # Template for environment variables
├── 📋 README.md                    # Project documentation
├── 📦 requirements.txt             # Python dependencies
└── 📂 sample_docs/                 # Sample test documents
    ├── sample.pdf
    ├── sample.docx
    └── sample.txt
```

---

## 🔬 Pipeline Steps — Detailed

### Step 1: File Type Detection
Identifies whether the input is `.txt`, `.docx`, or `.pdf` to route it to the correct extractor.

```python
def get_file_type_safely(file_path):
    _, extension = os.path.splitext(file_path)
    return extension.lower()
```

### Step 2: Text Extraction with OCR Fallback
- **TXT / DOCX:** Direct extraction via LangChain loaders — no OCR needed.
- **PDF:** PyMuPDF (`fitz`) extracts digital text page by page. If a page has images **and** fewer than `ocr_fallback_threshold = 200` characters of digital text, Tesseract OCR is applied to those image elements. This hybrid approach avoids unnecessary OCR on clean PDFs while correctly handling scanned pages.

### Step 3: Chunking
```python
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1250,
    chunk_overlap=250
)
```
Splits the full document into overlapping chunks. Overlap prevents sentences from being cut mid-meaning at chunk boundaries. Each chunk is embedded and stored in ChromaDB.

### Step 4: Parallel Chunk Summarization
Up to 200 concurrent async calls go to **Gemini 2.0 Flash** — one per chunk — for brief summaries that explicitly preserve metadata signals (author names, titles, dates). All summaries are joined into a single `combined_summary`.

### Step 5: RAG — Query Generation & Retrieval
The `combined_summary` is sent to the LLM, which generates metadata-targeted retrieval questions (e.g., *"Who are the authors?"*, *"What is the primary topic?"*). These questions query ChromaDB via similarity search, returning the `k=3` most relevant chunks.

### Step 6: Final Metadata Generation
The retrieved chunks + `combined_summary` are sent together to Gemini 2.0 Flash with a structured extraction prompt. The output is a **JSON object** with fields including title, authors, abstract, keywords, document type, language, and more.

---

## 🌐 Web Interface

The Gradio interface provides:

- 📁 **File upload** — drag-and-drop or browse for PDF, DOCX, or TXT files
- 📊 **Real-time status** — live console output showing page processing, chunk count, and API call progress
- 📋 **Metadata output** — structured JSON displayed in a readable formatted text box
- 🔗 **Public URL** — automatically generated shareable link (valid 1 week)

---

## ⚠️ Known Limitations

| Limitation | Detail |
|---|---|
| **API Rate Limits** | Free-tier Gemini API is capped at 15 requests/minute. Large documents with many chunks may hit quota limits during parallel summarization. Upgrade to a paid tier for production use. |
| **Gradio Link Expiry** | Public Gradio URLs expire after 1 week. Always regenerate by re-running the last notebook cell. |
| **OCR Accuracy** | Tesseract performs best on clean, high-resolution scans. Handwritten or low-quality images may yield reduced extraction quality. |
| **Very Large Documents** | Chunking + summarization is designed to stay within LLM context limits, but extremely large files may still require further chunking optimization. |

---

## 🤝 Contributing

Contributions are welcome! To contribute:

1. Fork the repository
2. Create a new branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -m "Add: your feature description"`
4. Push to the branch: `git push origin feature/your-feature`
5. Open a Pull Request

Please follow PEP 8 style guidelines and add comments to any new pipeline stages.

---

## 📄 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

---

<div align="center">

**If MARS was useful to you, please consider giving it a ⭐**

<img width="100%" src="https://capsule-render.vercel.app/api?type=waving&color=0:0ea5c9,100:0f2b5b&height=100&section=footer" />

</div>
