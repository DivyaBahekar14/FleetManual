# Fleet Manual AI Assistant

Fleet Manual AI Assistant is a web-based AI chatbot for answering fleet-management questions using vehicle manuals and IoT truck data.

The project uses **React + Vite** for the frontend and **FastAPI + LangGraph + RAG** for the backend. It retrieves relevant information from fleet manuals using embeddings and ChromaDB, classifies the user's query, and uses Gemini to generate the final answer.

## Features

- Ask questions about fleet vehicles through a chat interface.
- Search information from fleet manuals using RAG.
- Use IoT truck data for vehicle-related queries.
- Classify queries into:
  - Manual
  - IoT
  - Hybrid
  - Invalid
- Use a Supervisor Agent and Information Agent workflow.
- Store document embeddings in ChromaDB.
- Process PDF documents containing text and tables.
- Support OCR for scanned document content.
- Use semantic chunking for better retrieval.
- Generate answers using Google Gemini.

## Technology Stack

### Frontend

- React
- Vite
- Tailwind CSS
- Axios
- JavaScript

### Backend

- Python
- FastAPI
- LangGraph
- LangChain
- Google Gemini
- Sentence Transformers
- ChromaDB
- scikit-learn
- PyPDF
- pdfplumber
- PyMuPDF
- Tesseract OCR

## Project Structure

```text
FleetManualDivya/
│
├── backend/
│   ├── agents/
│   │   ├── information_agent.py
│   │   ├── supervisor_agent.py
│   │   ├── workflow.py
│   │   └── testingFiles/
│   │
│   ├── graph/
│   │   ├── state.py
│   │   └── workflow.py
│   │
│   ├── services/
│   │   └── intent_classifier.py
│   │
│   ├── testingFiles/
│   ├── chromadb/
│   ├── main.py
│   └── store.py
│
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── ChatWindow.jsx
│   │   │   ├── InputBox.jsx
│   │   │   ├── Message.jsx
│   │   │   ├── Navbar.jsx
│   │   │   └── Sidebar.jsx
│   │   │
│   │   ├── pages/
│   │   │   └── Home.jsx
│   │   │
│   │   ├── services/
│   │   │   └── api.js
│   │   │
│   │   ├── App.jsx
│   │   └── main.jsx
│   │
│   ├── package.json
│   └── vite.config.js
│
├── iot_data/
│   └── trucks.json
│
├── manuals/
│   ├── manual.pdf
│   └── blazo-brochure.pdf
│
├── requirements.txt
├── main.py
└── .env
```

## How It Works

The application follows this basic flow:

```text
User Question
      │
      ▼
React Frontend
      │
      ▼
FastAPI Backend
      │
      ▼
Supervisor Agent
      │
      ▼
Intent Classification
      │
      ├── Manual ──────► ChromaDB / Fleet Manual
      │
      ├── IoT ─────────► Truck IoT Data
      │
      └── Hybrid ──────► Manual + IoT Data
                              │
                              ▼
                       Information Agent
                              │
                              ▼
                       Gemini LLM
                              │
                              ▼
                         Final Answer
                              │
                              ▼
                       React Chat UI
```

## RAG Pipeline

The manual documents are processed before they are used for question answering.

1. Read the PDF manuals.
2. Extract text and tables.
3. Extract text from scanned pages using OCR when required.
4. Clean and normalize the extracted content.
5. Split the content into semantic chunks.
6. Generate embeddings for the chunks.
7. Store the chunks and embeddings in ChromaDB.
8. When a user asks a question, generate an embedding for the query.
9. Retrieve the most relevant chunks from ChromaDB.
10. Send the retrieved information to Gemini.
11. Generate an answer using the provided context.

The main ingestion code is available in:

```text
backend/store.py
```

To run the ingestion pipeline:

```bash
python -m backend.store
```

## AI Agent Workflow

### Supervisor Agent

The Supervisor Agent receives the user's question and determines the query intent.

It can classify a question as:

- `manual`
- `iot`
- `hybrid`
- `invalid`

It then routes the request to the Information Agent.

### Information Agent

The Information Agent collects the required information based on the intent.

For a manual query, it retrieves relevant chunks from ChromaDB.

For an IoT query, it uses the truck data stored in:

```text
iot_data/trucks.json
```

For a hybrid query, it uses both manual information and IoT data.

The collected information is then provided to Gemini to generate the final response.

## Installation

### 1. Clone the repository

```bash
git clone <your-repository-url>
cd FleetManualDivya
```

### 2. Create a Python virtual environment

Windows:

```bash
python -m venv venv
venv\Scripts\activate
```

macOS/Linux:

```bash
python3 -m venv venv
source venv/bin/activate
```

### 3. Install backend dependencies

```bash
pip install -r requirements.txt
```

### 4. Configure the API key

Create a `.env` file in the project root:

```env
GOOGLE_API_KEY=your_google_api_key
```

Do not commit your `.env` file to GitHub.

## Running the Backend

From the project root:

```bash
uvicorn main:app --reload
```

The backend will run at:

```text
http://127.0.0.1:8000
```

You can also open the API documentation at:

```text
http://127.0.0.1:8000/docs
```

## Running the Frontend

Open another terminal and move to the frontend folder:

```bash
cd frontend
```

Install the dependencies:

```bash
npm install
```

Start the development server:

```bash
npm run dev
```

The frontend will normally be available at:

```text
http://localhost:5173
```

## API

### Health Check

```http
GET /
```

Example response:

```json
{
  "message": "Fleet Management AI Backend Running"
}
```

### Ask a Question

```http
POST /ask
```

Request:

```json
{
  "question": "What is the specification of BLAZO X 28 CARGO?"
}
```

Response:

```json
{
  "success": true,
  "question": "What is the specification of BLAZO X 28 CARGO?",
  "answer": "..."
}
```

## Example Questions

You can ask questions such as:

```text
What is the specification of BLAZO X 28 CARGO?

What is the maintenance schedule?

How do I reset the GPS tracker?

What is the current fuel level of the truck?

What is the current temperature of the truck?

Tell me about the truck status and the related maintenance procedure.
```

## Important Notes

- The Google Gemini API key is required for Gemini-based generation and Google embeddings.
- The frontend expects the backend to run on `http://127.0.0.1:8000`.
- The backend allows requests from the Vite development server at `http://localhost:5173`.
- The ChromaDB directory contains the locally persisted vector database.
- The manual PDFs are stored in the `manuals` directory.
- IoT sample data is stored in `iot_data/trucks.json`.
- Do not upload API keys or other secrets to GitHub.

## Future Improvements

- Add authentication and user management.
- Add support for uploading new manuals from the UI.
- Add real-time IoT data integration.
- Add conversation history.
- Improve intent classification with additional training examples.
- Add citations showing the manual page used for each answer.
- Deploy the frontend and backend to the cloud.

## Author

**Divya Bahekar**

Fleet Manual AI Assistant project built using React, FastAPI, RAG, LangGraph, ChromaDB, and Google Gemini.
