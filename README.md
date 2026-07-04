# 🚀 Multimodal AI WhatsApp Chatbot (RAG + Conversational Memory)

An enterprise-grade, production-ready **n8n automation workflow** that transforms WhatsApp into an advanced, context-aware AI Assistant. The system handles **multimodal inputs**—including Text, Voice Notes, Images, and structured Documents/PDFs—and leverages **Retrieval-Augmented Generation (RAG)** backed by MongoDB Atlas Vector Search to deliver precise, document-sourced responses with persistent memory.

---

## 🏗️ System Architecture & Dual-Pipeline Logic

The workflow is architected into two separate, optimized pipelines designed for high-performance indexing and low-latency runtime execution.

<img width="1270" height="804" alt="WhatsApp Image 2026-07-04 at 5 36 53 PM" src="https://github.com/user-attachments/assets/8417d524-92bb-4de1-aba4-2cb47ed74ca7" />


### 🏁 1. The Ingestion Pipeline (Knowledge Base Indexing)
* **Trigger:** Initiated via a `Manual Trigger` whenever product documentation or core assets need to be updated.
* **Document Extraction:** Automatically fetches unstructured data files using the `Google Docs Importer`.
* **Chunking & Tokenization:** The `Document Chunker` processes raw text into optimized segments using a `Recursive Character TextSplitter` with a chunk size of `3000` tokens and an overlap of `200` tokens.
* **Vector Embeddings:** Computes high-dimensional vector representations via the `OpenAI Embeddings Generator`.
* **Database Synching:** Streams the generated vectors directly into a centralized **MongoDB Atlas Vector Store** (`n8n-template` collection) mapped under a custom vector index named `data_index`.

### ⚡ 2. The Interactive Chatbot Pipeline (Real-Time Runtime)
When a user interfaces with the integrated WhatsApp business channel, the `WhatsApp Trigger` catches the incoming webhook and passes the message object to a strict structural router (`Route Types`) that processes media payloads contextually:

* **📝 Text Messages:** Plucks the raw string payload body and transfers it instantly to the processing engine.
* **🎙️ Voice Notes:** Fetches the internal binary WhatsApp media URL path, streams the raw audio file via an HTTP download block, and sends it to the `OpenAI Whisper Engine` to handle translation and speech-to-text transcription.
* **🖼️ Images:** Downloads the raw graphical asset and applies computer vision utilizing `OpenAI Vision (GPT-4o-mini)` to perform semantic scene analysis, context extraction, and structural image mapping.
* **📄 Office Documents & PDFs:** Passes structural data down to a dedicated file extraction tree (`Route Document Types`). The framework runs native byte-stream extractions for **PDF, CSV, XLSX, XLS, TXT, HTML, JSON, and XML** formats. Any unsupported file extensions are intercepted and routed to trigger an auto-generated rejection alert to the user.

### 🧠 Core Agent Engine & Omnichannel Response
Once the multimodal input string is fully normalized, mapped, and parsed, it is processed by the **Knowledge Base Agent**:
* **Conversational Buffer Memory:** Links a dynamic `Memory Buffer Window` node pinned explicitly to the incoming sender's WhatsApp tracking ID (`wa_id`) to retain multi-turn dialogue context.
* **Context Retrieval (RAG):** Evaluates the normalized user intent and triggers a semantic similarity lookup over the **MongoDB Atlas Vector Search Index** to pull down the most relevant reference chunks.
* **Inference & Delivery:** Feeds the dynamic conversational history, real-time user input, and fetched document vector context into `GPT-4o-mini` to formulate a highly targeted response, which is then cleanly pushed back to the user via the `Send Response (WhatsApp Node)`.

---

## 📊 MongoDB Atlas Vector Search Index Schema

To enable advanced vector lookup queries, set up your MongoDB Atlas Search Index using the following structural configuration profile:

```json
{
  "mappings": {
    "dynamic": false,
    "fields": {
      "_id": {
        "type": "string"
      },
      "text": {
        "type": "string"
      },
      "embedding": {
        "type": "knnVector",
        "dimensions": 1536,
        "similarity": "cosine"
      },
      "source": {
        "type": "string"
      },
      "doc_id": {
        "type": "string"
      }
    }
  }
}
```
⚙️ Setup, Prerequisites & ConfigurationBefore deploying this blueprint file inside your local or cloud n8n environment, ensure you map the following ecosystem credentials:🔑 Credentials Mapping MatrixNode Service GroupIntegration ProviderRequired Scopes / ConfigurationLLM Inference & VisionOpenAI APIValid API Key with access to gpt-4o-mini and embeddings modelsMessaging WebhooksMeta for DevelopersPermanent WhatsApp System User Token + Verification TokenVector DatabaseMongoDB AtlasCluster Connection String URI with full read/write network accessKnowledge BaseGoogle Docs APIOAuth2 Credentials with read permissions for drive documentsMedia StreamersHTTP Header AuthGeneric HTTP Header authentication matching Meta's asset server requirements🚀 Step-by-Step Deployment GuideImport the Workflow Blueprint: Download the file AI-powered WhatsApp chatbot for text, voice, images, and PDF with RAG.json. Open your n8n workspace dashboard, create a new canvas, and simply press Ctrl+V (or click on the top-right menu and choose Import from File).Resolve Node Credentials: Click into each highlighted error node (OpenAI, MongoDB, Google Docs, and WhatsApp) and assign your respective pre-configured environment credentials from the dropdown selector.Seed your Knowledge Base: Verify that your target Google Doc contains the knowledge base information you want to index. Click Execute Workflow on the Manual Trigger node to run the parsing pipeline and build out your vector listings inside MongoDB.Expose and Activate Webhook: Ensure your n8n instance is configured with a public-facing URL (using a reverse proxy like Ngrok or an active cloud deployment) so Meta can reach your webhook endpoint. Toggle the workflow status switch in the top right corner to Active.
