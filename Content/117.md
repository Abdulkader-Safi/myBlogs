# Retrieval-Augmented Generation (RAG) with Local PDFs and Ollama: A Developer’s Guide

Large Language Models (LLMs) like GPT-4, Claude, and Llama 2 are powerful, but they have limits: they hallucinate, miss domain knowledge, and can’t stay up-to-date without retraining.

Retrieval-Augmented Generation (RAG) solves these challenges by combining retrieval from an external knowledge base with language generation. And best of all—you don’t need OpenAI or Anthropic APIs. With Ollama, you can run LLMs locally on your machine and power them with your own data (like PDFs in a directory).

## In this guide, we’ll walk through:

- What RAG is and why it matters
- How to build a RAG pipeline over a directory of PDFs
- Using Ollama as your local LLM backend
- Full Python code example

---

## What is Retrieval-Augmented Generation?

At its core, RAG is a pipeline:

1. User Query → “What does this contract say about termination terms?”
2. Retriever → Searches a vector database of your documents (PDFs, knowledge base).
3. Augmentation → Injects the retrieved text into the LLM’s context.
4. Generator → The LLM (Ollama, GPT, Claude, etc.) produces an answer grounded in your data.

This lets you:

- Reduce hallucinations
- Use private, domain-specific data
- Keep knowledge fresh without retraining

---

## Why Use Ollama for RAG?

Ollama makes running LLMs locally as simple as ollama run llama2. Advantages:

- No API costs: No tokens, no rate limits.
- Full privacy: Your PDFs never leave your machine.
- Custom models: Use Llama 2, Mistral, Phi, or even fine-tuned models.

This is perfect for developers working with sensitive corporate documents, legal contracts, or research papers.

---

## Full Example: RAG on a PDF Directory with Ollama

Here’s a working Python script that:

- Loads all PDFs in a folder
- Splits them into chunks
- Stores embeddings in FAISS (vector database)
- Runs queries against them using Ollama as the LLM

### 1. Install dependencies

```bash
pip install langchain langchain-community pytesseract pdf2image pillow pymupdf cryptography faiss-cpu pypdf sentence-transformers
```

And install Ollama if you haven’t already.

### 2. Python code: rag_pdf_ollama.py

```python
import io
import os
import warnings
from pathlib import Path
from typing import List
import hashlib
import pickle
from concurrent.futures import ThreadPoolExecutor, as_completed
from functools import lru_cache

import fitz  # PyMuPDF
import pytesseract
from PIL import Image

# Suppress all warnings
warnings.filterwarnings("ignore")

from langchain.chains import RetrievalQA
from langchain.docstore.document import Document
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_community.vectorstores import FAISS
from langchain_community.chat_models import ChatOllama
from langchain_community.document_loaders import PyPDFLoader
from langchain_community.embeddings import HuggingFaceEmbeddings

# --- Config ---
DATA_DIR = Path("./my_pdfs")  # put PDFs here
INDEX_DIR = Path("./vector_index")  # FAISS index
CACHE_DIR = Path("./.pdf_cache")  # Cache for extracted text
TOP_K = 3  # Reduced from 4 for faster retrieval
OLLAMA_MODEL = "mistral"  # or "mistral", "phi", etc.
MAX_WORKERS = 4  # Parallel processing workers
OCR_IMAGE_MAX_SIZE = 2000  # Max image dimension for OCR

# Singleton for embeddings model
_EMBEDDINGS_MODEL = None


def _get_image_hash(image_bytes: bytes) -> str:
    """Calculate hash for image deduplication"""
    return hashlib.md5(image_bytes).hexdigest()


def _preprocess_image(image: Image.Image) -> Image.Image:
    """Optimize image for faster OCR"""
    # Resize large images
    if max(image.size) > OCR_IMAGE_MAX_SIZE:
        ratio = OCR_IMAGE_MAX_SIZE / max(image.size)
        new_size = (int(image.size[0] * ratio), int(image.size[1] * ratio))
        image = image.resize(new_size, Image.Resampling.LANCZOS)

    # Convert to grayscale for faster OCR
    if image.mode != 'L':
        image = image.convert('L')

    return image


def _process_single_image(args):
    """Process a single image for OCR (for parallel processing)"""
    image_bytes, pdf_path, page_num = args
    try:
        image = Image.open(io.BytesIO(image_bytes))
        image = _preprocess_image(image)
        ocr_text = pytesseract.image_to_string(image)

        if ocr_text.strip():
            return Document(
                page_content=ocr_text,
                metadata={
                    "source": str(pdf_path),
                    "page": page_num,
                    "type": "image_ocr",
                },
            )
    except Exception:
        pass
    return None


def extract_text_from_images(pdf_path: Path) -> List[Document]:
    """Extract text from images in PDF using OCR with parallel processing"""
    docs = []
    pdf_document = fitz.open(str(pdf_path))

    # Collect all images with deduplication
    image_tasks = []
    seen_hashes = set()

    for page_num in range(len(pdf_document)):
        page = pdf_document[page_num]
        images = page.get_images()

        for img in images:
            try:
                xref = img[0]
                base_image = pdf_document.extract_image(xref)
                image_bytes = base_image["image"]

                # Skip duplicate images
                img_hash = _get_image_hash(image_bytes)
                if img_hash in seen_hashes:
                    continue
                seen_hashes.add(img_hash)

                image_tasks.append((image_bytes, pdf_path, page_num))
            except Exception:
                pass

    pdf_document.close()

    # Process images in parallel
    if image_tasks:
        with ThreadPoolExecutor(max_workers=MAX_WORKERS) as executor:
            results = executor.map(_process_single_image, image_tasks)
            docs = [doc for doc in results if doc is not None]

    return docs


def _get_pdf_cache_path(pdf_path: Path) -> Path:
    """Get cache file path for a PDF"""
    CACHE_DIR.mkdir(parents=True, exist_ok=True)
    pdf_hash = hashlib.md5(str(pdf_path).encode()).hexdigest()
    return CACHE_DIR / f"{pdf_hash}.pkl"


def _get_pdf_mtime(pdf_path: Path) -> float:
    """Get modification time of PDF"""
    return pdf_path.stat().st_mtime


def _load_single_pdf(pdf_path: Path) -> List[Document]:
    """Load a single PDF with caching"""
    cache_path = _get_pdf_cache_path(pdf_path)
    current_mtime = _get_pdf_mtime(pdf_path)

    # Check cache
    if cache_path.exists():
        try:
            with open(cache_path, 'rb') as f:
                cached_data = pickle.load(f)
                if cached_data['mtime'] == current_mtime:
                    print(f"[CACHE HIT] Loading from cache: {pdf_path.name}")
                    return cached_data['docs']
        except Exception:
            pass

    # Load fresh
    print(f"[PROCESSING] Extracting text from: {pdf_path.name}")
    docs = []

    # Load text content
    loader = PyPDFLoader(str(pdf_path))
    text_docs = loader.load()
    docs.extend(text_docs)

    # Extract text from images using OCR
    image_docs = extract_text_from_images(pdf_path)
    docs.extend(image_docs)

    # Cache results
    try:
        with open(cache_path, 'wb') as f:
            pickle.dump({'mtime': current_mtime, 'docs': docs}, f)
        print(f"[CACHED] Saved to cache: {pdf_path.name}")
    except Exception:
        pass

    return docs


def load_pdfs(pdf_dir: Path) -> List[Document]:
    """Load PDFs with parallel processing and caching"""
    pdf_files = list(pdf_dir.glob("**/*.pdf"))

    if not pdf_files:
        return []

    # Process PDFs in parallel
    all_docs = []
    with ThreadPoolExecutor(max_workers=MAX_WORKERS) as executor:
        future_to_pdf = {executor.submit(_load_single_pdf, pdf): pdf for pdf in pdf_files}
        for future in as_completed(future_to_pdf):
            try:
                docs = future.result()
                all_docs.extend(docs)
            except Exception:
                pass

    return all_docs


def chunk_docs(docs: List[Document]) -> List[Document]:
    """Chunk documents with parallel processing"""
    splitter = RecursiveCharacterTextSplitter(
        chunk_size=1000, chunk_overlap=200, separators=["\n\n", "\n", " ", ""]
    )

    # Split documents in parallel batches
    if len(docs) <= 10:
        return splitter.split_documents(docs)

    batch_size = max(1, len(docs) // MAX_WORKERS)
    batches = [docs[i:i + batch_size] for i in range(0, len(docs), batch_size)]

    all_chunks = []
    with ThreadPoolExecutor(max_workers=MAX_WORKERS) as executor:
        futures = [executor.submit(splitter.split_documents, batch) for batch in batches]
        for future in as_completed(futures):
            try:
                chunks = future.result()
                all_chunks.extend(chunks)
            except Exception:
                pass

    return all_chunks


def get_embeddings():
    """Get embeddings model with singleton pattern"""
    global _EMBEDDINGS_MODEL
    if _EMBEDDINGS_MODEL is None:
        _EMBEDDINGS_MODEL = HuggingFaceEmbeddings(
            model_name="sentence-transformers/all-MiniLM-L6-v2"
        )
    return _EMBEDDINGS_MODEL


def get_llm():
    return ChatOllama(
        model=OLLAMA_MODEL,
        temperature=0,
        num_ctx=2048,  # Reduced context window for faster inference
        num_predict=256,  # Limit response length for faster generation
    )


def ensure_index():
    INDEX_DIR.mkdir(parents=True, exist_ok=True)

    # Check if index already exists
    if (INDEX_DIR / "index.faiss").exists():
        return FAISS.load_local(
            folder_path=str(INDEX_DIR),
            embeddings=get_embeddings(),
            allow_dangerous_deserialization=True,
        )

    # Index doesn't exist, need to build it
    docs = load_pdfs(DATA_DIR)
    if not docs:
        raise RuntimeError("No PDFs found in directory.")
    chunks = chunk_docs(docs)

    vs = FAISS.from_documents(chunks, get_embeddings())
    vs.save_local(str(INDEX_DIR))
    return vs


def make_qa_chain(vs: FAISS):
    retriever = vs.as_retriever(search_type="similarity", search_kwargs={"k": TOP_K})
    llm = get_llm()
    return RetrievalQA.from_chain_type(
        llm=llm,
        retriever=retriever,
        return_source_documents=True,
        chain_type="stuff",
    )


def ask(qa, query: str):
    result = qa({"query": query})
    print(result["result"])


if __name__ == "__main__":
    import argparse
    import sys

    # Suppress stderr to hide all warnings
    sys.stderr = open(os.devnull, 'w')

    parser = argparse.ArgumentParser()
    parser.add_argument("-q", "--query", required=True, help="Your question")
    parser.add_argument(
        "--rebuild", action="store_true", help="Rebuild the FAISS index"
    )
    args = parser.parse_args()

    if args.rebuild and INDEX_DIR.exists():
        for p in INDEX_DIR.glob("*"):
            p.unlink()

    vs = ensure_index()
    qa = make_qa_chain(vs)
    ask(qa, args.query)
```

### 3. Run it

```bash
# Ask about your PDFs
python rag_pdf_ollama.py -q "Summarize the refund policy."

# If you add new PDFs, rebuild the index:
python rag_pdf_ollama.py --rebuild -q "What are the contract termination clauses?"
```

---

## Best Practices

- Choose the right Ollama model:
  - **llama2**: balanced
  - **mistral**: lightweight, fast
  - **phi**: very small, efficient
- **Chunk wisely**: ~1000 tokens with 200 overlap is a strong default.
- **Keep sources**: Always return source documents for transparency.
- **Optimize embeddings**: For large corpora, test different embedding models.

---

## Conclusion

With Ollama and LangChain, you can build a completely local RAG pipeline:

- Index your PDF directory
- Run queries against it
- Get answers grounded in your private data

No API keys. No data leakage. Just LLMs running locally, powered by your documents.

If you’re serious about AI + knowledge management, RAG with Ollama is a must-have in your toolkit.

---

### 🤝 Need a Custom RSVP System or Dashboard?

I help businesses build tools that _actually work_ , even on tight deadlines.

Whether you're planning an event, need internal tools, or want a custom dashboard for your team , I can help.

### Reach out

📧 Email: [safi.abdulkader@gmail.com](mailto:safi.abdulkader@gmail.com) | 💻 LinkedIn: [@abdulkader-safi](https://www.linkedin.com/in/abdulkader-safi/) | 📱 Instagram: [@abdulkader.safi](https://www.instagram.com/abdulkader.safi/) | 🏢 [DSRPT](https://www.dsrpt.com.au/kw/contact)

_Drop me a line, I’m always happy to collaborate!_ 🚀
