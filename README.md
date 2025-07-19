# Multi-Agentic Conversational AI System

An intelligent, multilingual, and multi-modal assistant powered by Retrieval-Augmented Generation (RAG), Gemini LLM, a CRM database, and calendar integration.

---
## 📦 Installation

```bash
pip install -r requirements.txt
```
---

## 🚀 Features Implemented

### 🔹 Gemini Chat Endpoint (`/chat/gemini`)

* Handles dynamic user queries.
* NLP-based intent detection (greeting, unrelated, calendar intent).
* Integrates:

  * **RAG** for contextual answers.
  * **Gemini LLM** to generate responses.
  * **Calendar Scheduling** (with conflict checking).
  * **Multi-language Support** (translates queries and responses).
* Logs conversations and classifies with tags.

**Example Test:**

```json
{
  "user_id": "test123",
  "message": "Schedule a meeting with John tomorrow at 3pm for updates."
}
```

---

### 🔹 Upload Document Endpoint (`POST /upload_docs/`)

* Uploads `.pdf`, `.txt`, `.csv` files.
* Indexes content into FAISS vector store using LangChain.

**Test with Swagger:**
Upload a supported file and check for `"File uploaded and processed successfully"`.

---

### 🔹 Calendar API

* **Create Event (auto from chat)**
* **Conflict Detection:** Same-time event not allowed.
* **Update, Fetch, and Delete User Events**

**Endpoints:**

* `POST /calendar/create_event`
* `GET /calendar/user_events/{user_id}`
* `PUT /calendar/update_event/{event_id}`
* `DELETE /calendar/delete_event/{event_id}`

---

### 🔹 User Management

* Create guest or registered users.
* `POST /user/create_user`
* `GET /user/get_user/{user_id}`
* `PUT /user/update_user/{user_id}`
* `DELETE /user/delete_user/{user_id}`

👥 Guest User Handling

If user_id is not provided in the /chat/gemini request, a guest user will automatically be created with the format:

{
  "user_id": "guest-<uuid>",
  "is_guest": true
}

This is inserted into the users_collection in MongoDB

---

### 🔹 Conversation History + Tag Management

* Stores chat logs in MongoDB.
* Filtered by tags or user ID.

**Endpoints:**

* `GET /conversation/user/{user_id}`
* `GET /conversation/filter?tag=calendar`
* `PUT /conversation/update_tags/{conversation_id}`
* `DELETE /conversation/delete_user_conversations/{user_id}`

---

## 🧠 RAG Service (Retrieval-Augmented Generation)

* Uses `faiss-cpu` + LangChain for indexing.
* Dynamically extracts context and checks scope using Gemini.
* Uses Google Generative AI Embeddings.

### Indexing Supported:

* PDF, TXT, CSV, DOC, DOCX

---

## 🌐 Language Support

* Auto-detects language.
* Translates input to English before processing.
* Translates output back to original language.

---

## 🔧 Development Stack

* **Backend:** FastAPI, Python
* **LLM:** Gemini via `google-generativeai`
* **RAG:** LangChain + FAISS
* **NLP:** Custom `analyze_intent` function
* **Multilingual:** `langdetect`, `deep-translator`
* **Document Parsing:** `pypdf`, `python-docx`, `pandas`
* **Database:** MongoDB

---
## 🚀 How to Run the App

✅ 1. Clone the Repository

    git clone https://github.com/Sreekarr29/Multi-Agentic-Conversational-AI-System
    cd multi_agent_chatbot
    cd multi_agent_chatbot

✅ 2. Set Up Virtual Environment (Recommended)

✅ 3. Install Dependencies

    pip install -r requirements.txt

Make sure you have Python 3.10 or 3.11 installed (avoid Python 3.12 for now due to some package issues).

✅ 4. Create .env File

Create a .env file in the root folder and add your API keys:

    GEMINI_API_KEY=your_google_gemini_api_key
    OPENAI_API_KEY=yor_api_key
    MONGO_URI=mongodb://localhost:27017

🗄️ Database: MongoDB Setup

🧪 What DB Are We Using?

We use MongoDB to manage user profiles, conversations, and calendar events.

⚙️ Instructions to Run MongoDB Locally

Install MongoDB:

MongoDB Community Edition

Start MongoDB Server:

mongod --dbpath /your/local/dbpath

Your app connects using:

MONGO_URI=mongodb://localhost:27017
MONGO_DB_NAME=multi_agent_ai_db

Use MongoDB Compass to view data (optional).

✅ 5. Run the FastAPI Server

    *run in root 
    cd multi_agent_chatbot
    cd multi_agent_chatbot

    uvicorn app.main:app --reload

    Server will start on:📍 http://localhost:8000

✅ 6. Access Swagger UI for API Testing

📘 Open your browser:http://localhost:8000/docsYou can test all endpoints from there, including:

POST /chat/gemini

POST /upload_docs/

GET /calendar/user_events/{user_id}... and more!

✅ 7. Example cURL Test
       
        curl -X 'POST' \
          'http://localhost:8000/chat/gemini' \
          -H 'accept: application/json' \
          -H 'Content-Type: application/json' \
          -d '{
          "user_id": "test123",
          "message": "What is the lowest rent available?"
        }'
---

## 🧪 Testing Endpoints

You can test the API using:

* 🔍 **Swagger UI**: [http://localhost:8000/docs](http://localhost:8000/docs)
* 📬 **Postman**: Use the examples below with `http://localhost:8000`

---

### 🤖 POST `/chat/gemini` – Gemini Chat Endpoint (RAG + Language + Calendar)

**Request:**

```json
{
  "user_id": "test123",
  "message": "What is the lowest rent for Broadway properties?"
}
```

✅ **Tests:**

* 💬 Regular rent/property query (uses RAG)
* 📅 Schedule a meeting:

  ```json
  {
    "user_id": "test123",
    "message": "Schedule a meeting with John tomorrow at 3pm"
  }
  ```
* 🌍 Non-English input (auto-translated):

  ```json
  {
    "user_id": "test123",
    "message": "Hola, ¿cuál es la propiedad más barata?"
  }
  ```
* 👋 Greetings intent:

  ```json
  {
    "user_id": "test123",
    "message": "Hi there!"
  }
  ```

---

### 📁 POST `/upload_docs/` – Upload Documents

**Content-Type:** `multipart/form-data`
**Field:**

```
file: (upload a .pdf, .txt, .csv, or .docx file)
```

✅ **Tests:**

* Upload `sample.csv` or `sample.pdf`
* The file will be parsed and indexed for RAG

---

### 📅 Calendar API

#### GET `/calendar/user_events/<user_id>`

Example:

```
GET /calendar/user_events/test123
```

#### POST `/calendar/create_event`

```json
{
  "user_id": "test123",
  "title": "Team Sync",
  "description": "Weekly sync-up",
  "datetime": "2025-07-20T10:00:00",
  "conversation_id": "abc-123"
}
```

#### PUT `/calendar/update_event/<event_id>`

```json
{
  "title": "Updated Team Sync",
  "datetime": "2025-07-21T11:00:00"
}
```

#### DELETE `/calendar/delete_event/<event_id>`

✅ **Tests:**

* Try to schedule overlapping event — it should detect conflict
* Create, update, and delete events

---

### 👤 User Management API

#### POST `/crm/create_user`

```json
{
  "user_id": "test123",
  "name": "Test User",
  "email": "test@example.com"
}
```

#### PUT `/crm/update_user/<user_id>`

```json
{
  "email": "new_email@example.com"
}
```

#### DELETE `/crm/delete_user/<user_id>`

✅ **Tests:**

* Create a new user
* Update their email
* Delete the user and verify removal

---

### 💬 Conversation API

#### GET `/crm/conversations/<user_id>`

Fetch all messages for the user.

#### GET `/crm/conversations/filter_by_tag?tag=calendar`

Filter messages by tag like `calendar`, `greeting`, `unrelated`, etc.

#### PUT `/crm/update_tag/<conversation_id>`

```json
{
  "tags": ["rent", "summary"]
}
```

✅ **Tests:**

* Query all messages
* Filter by tag (e.g. calendar)
* Update tag on specific conversation

---

### 🧼 POST `/reset` – Reset All Data

**Dangerous – clears all user, document, calendar and chat history!**
Used for development and testing resets.

---


### ⚙️ Internal Design Limits

    📌 Org Boundaries: Agent only responds to rent, CRM, calendar, and uploaded document topics.
    
    🌐 Language Handling: Automatically translates non-English inputs to English using langdetect & deep-translator, but:
    
         ✅ Purpose of the Limit:
         Preserves original message without translating if it’s too short.
         
         Avoids misclassification of language for very short messages.
         
         Ensures greetings like "Hi" don’t become "Ciao" (Italian) by accident.
         
         🚫 Limitation Introduced:
         If a valid short query under 6 characters is sent (e.g., "rent?"), the system might skip translation, which could affect understanding in non-English languages.
         
         This limit is a trade-off between translation accuracy and maintaining user intent for short inputs.
    
    🧠 Out-of-Scope Filtering: Uses Gemini + fallback heuristics to detect unsupported queries.
    
    🗓️ Calendar Conflict Check: New event creation is blocked if time overlaps with existing events.
    
    🧠 RAG Scope: check_scope_with_context() uses Gemini to determine if retrieved chunks can answer the question.

---

## 📂 File Structure

```
app/
├── main.py
├── routers/
│   ├── chat_gemini_router.py
│   ├── calendar_router.py
│   ├── upload.py
│   └── user_router.py
├── services/
│   ├── rag_service.py
│   ├── calendar_service.py
│   ├── gemini_service.py
│   ├── language_service.py
│   └── logger_service.py
├── schemas/
│   └── crm_schema.py
```

---




---



---


