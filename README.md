# HR AI Agent

An AI-powered HR assistant built with LangGraph that lets you query employee data using natural language. Ask questions like "Find me senior engineers who work remotely" and the agent searches a MongoDB vector database to find relevant employees.

## What it does

- Generates synthetic employee data using Groq (Llama 3.3 70B)
- Embeds employee profiles as semantic vectors using Google Gemini Embeddings and stores them in MongoDB Atlas
- Exposes a REST API where you can chat with an AI agent that searches the HR database using natural language
- Maintains conversation history across messages using MongoDB as a checkpoint store

## Tech Stack

- **LangGraph** — agent orchestration and state management
- **Groq (GPT OSS 120B)** — LLM for the agent and data generation
- **Google Gemini Embeddings** — semantic vector embeddings
- **MongoDB Atlas** — vector search and conversation checkpointing
- **Express** — REST API server
- **TypeScript** — end to end type safety with Zod schema validation

## Architecture

```
User Message (HTTP POST)
        ↓
   Express Server
        ↓
   LangGraph Agent
        ↓
   Should it search the DB?
     ↙           ↘
   Yes            No
    ↓              ↓
MongoDB Atlas   Final Answer
Vector Search
    ↓
Employee Results
    ↓
LangGraph Agent → Final Answer
```

The agent uses a tool-calling loop — it decides when to search the database, retrieves relevant employee profiles via vector similarity search, then synthesizes a final response.

## Getting Started

### Prerequisites

- Node.js 18+
- MongoDB Atlas account with a cluster
- Groq API key (free at [console.groq.com](https://console.groq.com))
- Google AI Studio API key (free at [aistudio.google.com](https://aistudio.google.com))

### Installation

```bash
npm install
```

### Environment Variables

Create a `.env` file in the root:

```
MONGODB_ATLAS_URI=your_mongodb_connection_string
GROQ_API_KEY=your_groq_api_key
GOOGLE_API_KEY=your_google_api_key
```

### Seed the Database

Run this once to generate 10 fictional employees and store their embeddings in MongoDB:

```bash
npx ts-node seed-database.ts
```

> Make sure you have a MongoDB Atlas Vector Search index named `vector_index` on the `hr_database.employees` collection with the `embedding` field configured.

### Run the Server

```bash
npx ts-node index.ts
```

Server starts on `http://localhost:3000`.

## API Usage

### Start a new conversation

```bash
curl -X POST http://localhost:3000/chat \
  -H "Content-Type: application/json" \
  -d '{"message": "Build a team to make an iOS app and tell me the talent gaps."}'
```

Returns a `threadId` you can use to continue the conversation.

### Continue an existing conversation

```bash
curl -X POST http://localhost:3000/chat/{threadId} \
  -H "Content-Type: application/json" \
  -d '{"message": "What team members did you recommend?"}'
```

## Project Structure

```
├── agent.ts          # LangGraph agent — tools, graph definition, state
├── index.ts          # Express server and API routes
├── seed-database.ts  # Generates synthetic employee data and seeds MongoDB
└── .env              # API keys (never commit this)
```
