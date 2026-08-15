# Kalakaari - AI-Powered Art Authentication & Marketplace

> Empowering artisans with AI-verified digital identity through conversational AI and blockchain provenance.

---

## � Overview

Kalakaari is a comprehensive platform that combines **Google Gemini AI**, **blockchain technology**, and **multi-modal vector search** to protect artisan intellectual property and create a trusted marketplace for authentic art.

### Key Features

- 🤖 **Conversational AI Onboarding** - Natural language art registration using Google Gemini
- 🔍 **AI Duplicate Detection** - Multi-modal similarity search (image + text)
- ⛓️ **Blockchain Provenance** - Immutable proof-of-creation on Polygon
- 🛍️ **Smart Marketplace** - Dual-index vector search with Pinecone
- ✅ **Digital Certificates** - QR-code enabled verification for physical artworks

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│              Google Cloud Run (Hosting)                 │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Shop Frontend (Angular) ←→ Shop Backend (FastAPI)     │
│                                                         │
│  Agentic Web-UI ←→ Agentic Service (Gemini + ADK)     │
│                                                         │
│  Master-IP Backend (FastAPI)                           │
│    ├─ /create    - Generate CraftID                    │
│    ├─ /verify    - Blockchain verification             │
│    ├─ /search    - Multi-modal search                  │
│    └─ Chain Batcher (Background Worker)                │
│                                                         │
│  Databases:                                             │
│    ├─ MongoDB Atlas (CraftIDs, Products)               │
│    ├─ Pinecone (Vector Search)                         │
│    ├─ Neon PostgreSQL (Agent Sessions)                 │
│    └─ Polygon Blockchain (Immutable Anchoring)         │
└─────────────────────────────────────────────────────────┘
```

---

## 🛠️ Technology Stack

### Google AI & Cloud
- **Google Gemini 2.0 Flash** - Conversational AI
- **Google Agent Development Kit (ADK)** - Agent framework
- **Firebase Authentication** - User management
- **Google Cloud Run** - Serverless hosting
- **Angular** - Frontend framework

### AI/ML
- **OpenAI CLIP (ViT-B/32)** - Image embeddings
- **Sentence-Transformers** - Text embeddings
- **Pinecone** - Vector database

### Backend
- **Python & FastAPI** - Microservices
- **MongoDB Atlas** - NoSQL database
- **Neon PostgreSQL** - Relational database
- **Web3.py** - Blockchain integration

### Blockchain
- **Solidity** - Smart contracts
- **Polygon (Amoy Testnet)** - Blockchain network

---

## � Prerequisites

Before setting up the project, ensure you have:

- **Python 3.11+** installed
- **Node.js 18+** and npm installed
- **Git** for version control
- **MongoDB Atlas** account (free tier)
- **Pinecone** account (free tier)
- **Neon** PostgreSQL account (free tier)
- **Google API Key** (from Google AI Studio)
- **MetaMask wallet** with Polygon Amoy testnet configured
- **Polygon Amoy testnet MATIC** (get from faucet)

---

## 🚀 Local Setup

### 1. Clone the Repository

```bash
https://github.com/Harin225/kalakaari-ai-solution
cd kalakaari
```

---

### 2. Master-IP Service Setup

The core service for CraftID generation, verification, and blockchain anchoring.

```bash
# Navigate to service directory
cd master-ip/server

# Create virtual environment
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install --upgrade pip
pip install -r requirements.txt

# Generate signing keys
cd chain/keys
openssl ecparam -name prime256v1 -genkey -noout -out sign_priv.pem
openssl ec -in sign_priv.pem -pubout -out sign_pub.pem

# Create anchorer key file (KEEP THIS SECRET!)
echo "0xYOUR_PRIVATE_KEY_HERE" > anchorer_key.txt
cd ../..

# Create .env file (see Environment Variables section below)
nano .env

# Run the service
PYTHONPATH=.. uvicorn app.main:app --reload --port 8000
```

**In a separate terminal, run the chain batcher:**
```bash
cd master-ip/server
source venv/bin/activate
python -m chain.batcher
```

---

### 3. Agentic Service Setup

The conversational AI agent powered by Google Gemini.

```bash
# Navigate to service directory
cd agentic

# Create virtual environment
python3 -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install dependencies
pip install --upgrade pip
pip install -r requirements.txt

# Create .env file (see Environment Variables section below)
nano .env

# Run the service
PORT=8080 adk web
# OR
uvicorn main:app --reload --host 0.0.0.0 --port 8080
```

---

### 4. Shop Backend Setup

The marketplace backend service.

```bash
# Navigate to service directory
cd shop/shop-backend

# Create virtual environment
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Create .env file (see Environment Variables section below)
nano .env

# Run the service
uvicorn app.main:app --reload --port 8001
```

---

### 5. Shop Frontend Setup

The Angular-based marketplace UI.

```bash
# Navigate to frontend directory
cd shop/shop-frontend

# Install dependencies
npm install

# Configure environment (edit src/environments/environments.ts)
# Add Firebase config and API URLs

# Run development server
npm start
# OR
ng serve

# Access at http://localhost:4200
```

---

## 🔑 Environment Variables

### Master-IP Service (`.env` in `master-ip/server/`)

```bash
# MongoDB
MONGO_URI=mongodb+srv://username:password@cluster.mongodb.net/?retryWrites=true&w=majority
DB_NAME=masterip_db

# JWT
SECRET_KEY=your_secret_key_here

# Pinecone (Image Index)
PINECONE_API_KEY=your_pinecone_api_key
INDEX_HOST=https://your-index.svc.pinecone.io

# Pinecone (Text Index)
PINECONE_ENV=us-east1-aws
PINECONE_TEXT_INDEX=text

# Blockchain
WEB3_RPC_URL=https://rpc-amoy.polygon.technology
CHAIN_ID=80002
ANCHOR_CONTRACT_ADDRESS=0xYourDeployedContractAddress

# Anchorer Private Key (FILE PATH - not the key itself!)
ANCHORER_PRIVATE_KEY=/absolute/path/to/master-ip/server/chain/keys/anchorer_key.txt

# Signing Keys (FILE PATHS)
SIGNER_KEY_PATH=/absolute/path/to/master-ip/server/chain/keys/sign_priv.pem
PLATFORM_PUBKEY_PATH=/absolute/path/to/master-ip/server/chain/keys/sign_pub.pem

# Queue & Batcher
ANCHOR_QUEUE_COLL=anchor_queue
BATCH_LIMIT=5
POLL_INTERVAL=10
```

### Agentic Service (`.env` in `agentic/`)

```bash
# Neon PostgreSQL
SESSION_SERVICE_URI=postgresql://username:password@host.neon.tech/database?sslmode=require
DATABASE_URL=postgresql://username:password@host.neon.tech/database?sslmode=require

# Google AI
MODEL_NAME=gemini-2.0-flash
GOOGLE_API_KEY=your_google_api_key_here
GOOGLE_GENAI_USE_VERTEXAI=FALSE

# Port
PORT=8080
```

### Shop Backend (`.env` in `shop/shop-backend/`)

```bash
MONGO_URI=mongodb+srv://username:password@cluster.mongodb.net/?retryWrites=true&w=majority
DB_NAME=shop_db
```

---

## 🧪 Testing the Complete Flow

### 1. Create a CraftID

```bash
curl -sS -X POST "http://localhost:8000/create" \
  -H "Content-Type: application/json" \
  -d '{
    "artisan": {
      "name": "Ravi Verma",
      "location": "Mithila, Bihar",
      "contact_number": "9876543210",
      "email": "ravi@example.com",
      "aadhaar_number": "1234-5678-9101"
    },
    "art": {
      "name": "Madhubani Painting",
      "description": "Traditional artwork depicting rural folklore",
      "photo": "https://example.com/image.jpg"
    }
  }' | jq
```

### 2. Verify a CraftID

```bash
curl -s "http://localhost:8000/verify/CID-00001" | jq
```

### 3. Search for similar art

```bash
curl -X POST "http://localhost:8000/search" \
  -H "Content-Type: application/json" \
  -d '{
    "query_text": "traditional Madhubani painting",
    "top_k": 5
  }' | jq
```

### 4. Check blockchain anchoring

Monitor the chain batcher terminal, then verify on Polygonscan:
```
https://amoy.polygonscan.com/tx/0xYOUR_TX_HASH
```

---

## 🔧 Troubleshooting

### Port Conflicts
```bash
# Find and kill process using a port
lsof -i :8000
kill -9 <PID>
```

### Pinecone Issues
- Verify all environment variables are set correctly
- Check API key validity
- Ensure index names match

### Chain Batcher Not Working
- Verify anchorer_key.txt exists and contains valid private key
- Check you have testnet MATIC in your wallet
- Verify contract address is correct

### MongoDB Connection
- Whitelist your IP in MongoDB Atlas
- Verify connection string format
- Check database name

---

## 📂 Project Structure

```
kalakaari/
├── master-ip/          # Core CraftID service
├── agentic/            # AI agent service
├── shop/               # Marketplace frontend & backend
└── README.md
```

---

## 📜 License

This project is open-source and available under the MIT License.

---

## 🙏 Acknowledgments

- **Google AI Exchange Program**
- **Google Cloud Platform**
- **Polygon Network**
- **OpenAI & Pinecone**

---

**Built with ❤️ for artisans everywhere**
