# 📄 RAG Workflow for Company Documents (n8n + Google Drive + OpenAI)

## 🚀 Overview

This project implements a **Retrieval-Augmented Generation (RAG)** workflow using **n8n**, allowing employees to chat with internal company documents stored in Google Drive.

The system:

* Automatically ingests documents from Google Drive
* Splits and embeds content
* Stores vectors in a vector store
* Enables AI-powered Q&A over company documents

---

## 🧠 Architecture

### 🔄 Two Main Flows

#### 1. Document Ingestion Flow

Triggered when files are **created or updated** in Google Drive.

```
Google Drive Trigger → Download File → Text Split → Embeddings → Vector Store
```

#### 2. Chat / Query Flow

Triggered when a user sends a message.

```
Chat Trigger → AI Agent → Vector Store Tool → Response
```

---

## ⚙️ Prerequisites

Before setting up, ensure you have:

### 1. Google Cloud

* Create a project
* Enable required APIs
* Setup OAuth credentials for Google Drive

### 2. OpenAI API Key

* Required for:

  * Embeddings
  * Chat model

### 3. n8n Setup

* Self-hosted or cloud instance
* Configure credentials:

  * Google Drive OAuth2
  * OpenAI API

---

## 📂 Workflow Setup (Step-by-Step)

### Step 1: Google Drive Folder

* Create a dedicated folder
* Store all company documents here
* Copy the **Folder ID**

---

### Step 2: Import Workflow

* Import the provided JSON into n8n
* Open the workflow editor

---

### Step 3: Configure Google Drive Triggers

#### Nodes:

* `Google Drive File Created`
* `Google Drive File Updated`

✅ Update:

```
folderToWatch → Your Google Drive Folder ID
```

---

### Step 4: File Download

#### Node:

* `Download File From Google Drive`

✅ Function:

* Downloads file using:

```
fileId = {{$json.id}}
fileName = {{$json.name}}
```

---

### Step 5: Document Processing

#### Nodes:

* `Default Data Loader`
* `Recursive Character Text Splitter`

✅ Config:

* Chunk overlap: `100`
* Splits documents into smaller chunks for embeddings

---

### Step 6: Embeddings

#### Nodes:

* `Embeddings OpenAI`
* `Embeddings OpenAI1`

✅ Purpose:

* Convert text chunks into vector embeddings

---

### Step 7: Vector Storage

#### Nodes:

* `Simple Vector Store1` (Insert Mode)
* `Simple Vector Store` (Query Mode)

✅ Key:

```
memoryKey = vector_store_key
```

---

### Step 8: AI Agent Setup

#### Node:

* `AI Agent`

✅ System Prompt:

```
You are a helpful AI assistant designed to answer employee questions based on company policies.

Retrieve relevant information from the provided internal documents and provide a concise, accurate, and informative answer.

Use the tool "company_documents_tool".

If no answer is found:
"I cannot find the answer in the available resources."
```

---

### Step 9: Vector Store Tool

#### Node:

* `Vector Store Tool`

✅ Purpose:

* Enables AI Agent to retrieve document data

---

### Step 10: Chat Trigger

#### Node:

* `When chat message received`

✅ Function:

* Entry point for user queries

---

### Step 11: Memory Handling

#### Node:

* `Window Buffer Memory`

✅ Purpose:

* Maintains conversational context

---

### Step 12: Output Formatting

#### Node:

* `Edit Fields`

✅ Output:

```
Result = {{$json.output}}
```

---

## 🔌 Connections Overview

### Document Ingestion Flow

* Google Drive Trigger → Download File
* Download File → Vector Store (Insert)
* Text Splitter → Data Loader → Vector Store

### Chat Flow

* Chat Trigger → AI Agent
* AI Agent → Vector Store Tool
* AI Agent → Output

---

## 💬 How It Works

1. Upload or update a document in Google Drive

2. Workflow automatically:

   * Downloads file
   * Splits content
   * Generates embeddings
   * Stores in vector DB

3. User sends a query:

   * AI Agent retrieves relevant chunks
   * Generates contextual answer

---

## 🧪 Example Queries

* "What is the leave policy?"
* "Explain reimbursement rules"
* "What are company working hours?"

---

## ⚠️ Limitations

* In-memory vector store (data lost on restart)
* No document versioning
* Limited scalability for large datasets

---

## 🔧 Improvements (Recommended)

* Replace in-memory store with:

  * Pinecone / Weaviate / Supabase
* Add:

  * File type filtering (PDF, DOCX)
  * Metadata (filename, department)
  * Re-ingestion logic for updates
* Add authentication for chat users

---

## 📦 Tech Stack

* n8n (workflow automation)
* OpenAI (LLM + embeddings)
* Google Drive API
* LangChain nodes (n8n)

---

## 🧑‍💻 Author

**Vignesh Nagarajan**

---

## 📜 License

MIT License