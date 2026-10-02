# TalkToPDF

![Node.js](https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)
![Express.js](https://img.shields.io/badge/Express.js-404D59?style=for-the-badge)
![MongoDB](https://img.shields.io/badge/MongoDB-4EA94B?style=for-the-badge&logo=mongodb&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white)
![Gemini](https://img.shields.io/badge/Google%20Gemini-8E75B2?style=for-the-badge&logo=googlebard&logoColor=white)

TalkToPDF is a full-stack RAG application that lets you upload PDF documents and chat with them in natural language. The backend extracts text, chunks it, generates embeddings, stores them in MongoDB Atlas, and uses Gemini to answer questions grounded in the uploaded document.

## What this project does

- Upload a PDF and queue it for background processing
- Extract and clean text from the document
- Split the content into meaningful chunks
- Generate vector embeddings and perform semantic search
- Stream grounded answers back to the UI with source references
- Keep chat history per session using Redis
- Protect the API with a simple rate limiter

## Architecture at a glance

```text
User uploads PDF
  → Express API accepts the file
  → MongoDB stores document metadata
  → Redis queue receives a background job
  → Worker parses PDF, chunks text, embeds chunks, and stores them
  → User asks a question
  → Embedding search retrieves relevant chunks
  → Gemini generates a grounded answer and streams it to the client
```

## Tech stack

| Layer | Technology |
|---|---|
| Frontend | React + Vite |
| Backend | Node.js + Express |
| Database | MongoDB Atlas |
| Queue / Cache | Redis |
| AI | Google Gemini APIs |
| File handling | Multer + pdf-parse |

## Project structure

```text
backend/
  server.js
  src/
    app.js
    runWorker.js
    config/
    controllers/
    middlewares/
    models/
    routes/
    services/
    utils/
    worker/
frontend/
  src/
    components/
    utils/
```

## API overview

### Documents

| Method | Endpoint | Purpose |
|---|---|---|
| POST | /api/documents/upload | Upload a PDF and start ingestion |
| GET | /api/documents | List documents |
| GET | /api/documents/:id | Get document details |
| DELETE | /api/documents/:id | Delete a document and its chunks |

### Chat

| Method | Endpoint | Purpose |
|---|---|---|
| POST | /api/chat/ask | Ask a question and stream the answer |
| GET | /api/chat/history/:sessionId | Retrieve chat history |
| DELETE | /api/chat/history/:sessionId | Clear chat history |

### Jobs

| Method | Endpoint | Purpose |
|---|---|---|
| GET | /api/job/:jobId | Check background job status |

## Getting started

### 1. Prerequisites

Make sure you have:

- Node.js 20+
- MongoDB Atlas account
- Redis running locally or remotely
- A Google Gemini API key

### 2. Install dependencies

```bash
cd backend && npm install
cd ../frontend && npm install
```

### 3. Configure environment variables

Create a file named `.env` inside the backend folder:

```env
PORT=8000
MONGODB_URI=mongodb+srv://<username>:<password>@cluster.mongodb.net/<database>
REDIS_URL=redis://localhost:6379
GEMINI_API_KEY=your_gemini_api_key_here
```

### 4. Set up MongoDB vector search

In MongoDB Atlas, create a search index named `vector_index` on the `chunks` collection with a vector field for `embedding`.

Example shape:

```json
{
  "fields": [
    {
      "type": "vector",
      "path": "embedding",
      "numDimensions": 768,
      "similarity": "cosine"
    },
    {
      "type": "filter",
      "path": "documentId"
    }
  ]
}
```

### 5. Start the services

Start the backend:

```bash
cd backend
npm run dev
```

In a second terminal, start the ingestion worker:

```bash
cd backend
node --env-file=.env src/runWorker.js
```

In a third terminal, start the frontend:

```bash
cd frontend
npm run dev
```

The UI should be available at http://localhost:5173 and the backend at http://localhost:8000.

## Notes

- The app is designed for grounded question answering, so it tries to answer only from the retrieved document chunks.
- Requests are streamed with Server-Sent Events for a more conversational experience.
- The worker handles parsing, chunking, embedding generation, and chunk storage asynchronously.

## License

This project is currently distributed as a personal/demo project and is intended for learning and experimentation.
