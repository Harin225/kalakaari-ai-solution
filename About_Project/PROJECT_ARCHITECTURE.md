# 🎨 Kalakaari Project - Complete Architecture Guide

## 📌 PROJECT PURPOSE

**Kalakaari** is an **AI-Powered Art Authentication & Marketplace** platform that helps artisans protect their intellectual property and create a trusted marketplace for authentic handcrafted art.

### 🎯 Core Objectives
1. **AI-Verified Digital Identity** - Use Google Gemini AI to verify artwork authenticity
2. **Blockchain Provenance** - Immutably anchor artwork creation on Polygon blockchain
3. **Duplicate Detection** - Detect AI-generated copies using multi-modal similarity search
4. **Trusted Marketplace** - Enable artisans to sell verified authentic art
5. **Digital Certificates** - QR-code enabled verification for physical artworks

---

## 🏗️ COMPLETE SYSTEM ARCHITECTURE

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         KALAKAARI PLATFORM (5 Services)                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │ 1️⃣  SHOP FRONTEND (Angular)        [PORT 4200]                       │  │
│  │ ┌────────────────────────────────────────────────────────────────┐  │  │
│  │ │ • User Interface (Browse/Search Products)                    │  │  │
│  │ │ • Login/Signup (Firebase Authentication)                     │  │  │
│  │ │ • Product Display with Verification QR codes                 │  │  │
│  │ │ • Cart & Checkout Flow                                       │  │  │
│  │ │ • Technologies: Angular 20, TypeScript, Tailwind CSS         │  │  │
│  │ └────────────────────────────────────────────────────────────────┘  │  │
│  │                              ↓ API Calls                            │  │
│  │                    ┌─────────────────────────┐                      │  │
│  │                    │  axios/HttpClient       │                      │  │
│  │                    │  (api-client.service)   │                      │  │
│  │                    └─────────────────────────┘                      │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │ 2️⃣  SHOP BACKEND (FastAPI)         [PORT 8001]                       │  │
│  │ ┌────────────────────────────────────────────────────────────────┐  │  │
│  │ │ ENDPOINTS:                                                    │  │  │
│  │ │ • GET  /get-products        → Fetch all products from DB     │  │  │
│  │ │ • POST /authenticate        → Login/Register users           │  │  │
│  │ │ • POST /orders              → Create order                   │  │  │
│  │ │ • GET  /orders/{user_id}    → Fetch user orders              │  │  │
│  │ │                                                               │  │  │
│  │ │ Connection Pool: MongoDB Atlas (kalaakari_shop_db)           │  │  │
│  │ │ Technologies: Python, FastAPI, SQLAlchemy, PyMongo           │  │  │
│  │ └────────────────────────────────────────────────────────────────┘  │  │
│  │                              ↓                                       │  │
│  │                    MongoDB Atlas (Shop Collections)                  │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │ 3️⃣  MASTER-IP BACKEND (FastAPI)    [PORT 8000]                       │  │
│  │ ┌────────────────────────────────────────────────────────────────┐  │  │
│  │ │ CORE FUNCTIONALITY:                                            │  │  │
│  │ │                                                                │  │  │
│  │ │ CREATE CraftID:                                               │  │  │
│  │ │ • POST /create → {artwork_metadata, images} → CraftID        │  │  │
│  │ │   - Generates vector embeddings (CLIP + text-embedding)      │  │  │
│  │ │   - Stores in MongoDB (masterip_db.craftids)                 │  │  │
│  │ │   - Indexes in Pinecone (craft-images & craft-text)          │  │  │
│  │ │   - Returns: {"public_id": "CID-00123", ...}                 │  │  │
│  │ │                                                                │  │  │
│  │ │ VERIFY CraftID:                                               │  │  │
│  │ │ • GET /verify/{public_id} → Check blockchain status          │  │  │
│  │ │   - Compares stored hash with blockchain                     │  │  │
│  │ │   - Returns tamper detection result                          │  │  │
│  │ │                                                                │  │  │
│  │ │ SEARCH (Duplicate Detection):                                │  │  │
│  │ │ • POST /search → Similar artworks (>80% match)               │  │  │
│  │ │   - Queries Pinecone vectors                                 │  │  │
│  │ │   - Returns similar CraftIDs                                 │  │  │
│  │ │                                                                │  │  │
│  │ │ BATCH OPERATIONS:                                            │  │  │
│  │ │ • GET /batch-status → Check pending anchoring                │  │  │
│  │ │                                                               │  │  │
│  │ │ Technologies: Python, FastAPI, OpenAI CLIP, Sentence Trans.  │  │  │
│  │ └────────────────────────────────────────────────────────────────┘  │  │
│  │                                    ↓                                  │  │
│  │                    ┌────────────────────────────┐                    │  │
│  │                    │  MongoDB Atlas             │                    │  │
│  │                    │  (masterip_db.craftids)    │                    │  │
│  │                    └────────────────────────────┘                    │  │
│  │                              ↓                                       │  │
│  │                    ┌────────────────────────────┐                    │  │
│  │                    │  Pinecone Vector DB        │                    │  │
│  │                    │  • craft-images            │                    │  │
│  │                    │  • craft-text              │                    │  │
│  │                    └────────────────────────────┘                    │  │
│  │                              ↓                                       │  │
│  │                    Queue (MongoDB):                                  │  │
│  │                    Pending blockchain anchoring                      │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │ 4️⃣  CHAIN BATCHER (Background Worker)    [No Port - Background]     │  │
│  │ ┌────────────────────────────────────────────────────────────────┐  │  │
│  │ │ • Polls MongoDB queue every 10 seconds                       │  │  │
│  │ │ • Collects pending CraftIDs needing blockchain anchoring     │  │  │
│  │ │ • Batches multiple transactions to save gas                  │  │  │
│  │ │ • Calls smart contract on Polygon Amoy                       │  │  │
│  │ │ • Updates status in MongoDB (status: "anchored")             │  │  │
│  │ │ • Stores tx_hash for verification                            │  │  │
│  │ │                                                               │  │  │
│  │ │ Smart Contract: CraftAnchor.sol                              │  │  │
│  │ │ • Receives batch of CraftIDs with hashes                     │  │  │
│  │ │ • Records timestamp and tx_hash on blockchain                │  │  │
│  │ │                                                               │  │  │
│  │ │ Technologies: Web3.py, Solidity, Polygon, OpenSSL (signing)  │  │  │
│  │ └────────────────────────────────────────────────────────────────┘  │  │
│  │                              ↓                                       │  │
│  │                    Polygon Amoy Blockchain                           │  │
│  │                    CraftAnchor Smart Contract                        │  │
│  │                    (Immutable Ledger)                               │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │ 5️⃣  AGENTIC SERVICE (Google ADK)   [PORT 8080]                       │  │
│  │ ┌────────────────────────────────────────────────────────────────┐  │  │
│  │ │ AI AGENT ORCHESTRATION:                                        │  │  │
│  │ │                                                                │  │  │
│  │ │ Sub-Agents:                                                   │  │  │
│  │ │ 1. Onboarding Agent                                           │  │  │
│  │ │    - Natural language artwork registration                   │  │  │
│  │ │    - Guides users through CraftID creation                   │  │  │
│  │ │    - Calls Master-IP /create endpoint                        │  │  │
│  │ │                                                               │  │  │
│  │ │ 2. IP Agent                                                  │  │  │
│  │ │    - Handles IP verification queries                         │  │  │
│  │ │    - Checks blockchain status                                │  │  │
│  │ │    - Performs similarity search                              │  │  │
│  │ │                                                               │  │  │
│  │ │ 3. Shop Agent                                                │  │  │
│  │ │    - Marketplace product queries                             │  │  │
│  │ │    - Order management                                        │  │  │
│  │ │    - Product recommendations                                 │  │  │
│  │ │                                                               │  │  │
│  │ │ Model: Google Gemini 2.0 Flash (Multimodal)                  │  │  │
│  │ │ Session Storage: Neon PostgreSQL (Agent context/history)    │  │  │
│  │ │ Technologies: Google ADK, LangChain, FastAPI, Gemini API     │  │  │
│  │ └────────────────────────────────────────────────────────────────┘  │  │
│  │                              ↓                                       │  │
│  │                    ┌────────────────────────────┐                    │  │
│  │                    │  Neon PostgreSQL           │                    │  │
│  │                    │  (Agent Sessions)          │                    │  │
│  │                    └────────────────────────────┘                    │  │
│  │                              ↓                                       │  │
│  │                    ┌────────────────────────────┐                    │  │
│  │                    │  Google Gemini 2.0 Flash   │                    │  │
│  │                    │  (AI Language Model)       │                    │  │
│  │                    └────────────────────────────┘                    │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 🔄 DATA FLOW: HOW EVERYTHING CONNECTS

### **FLOW 1: USER BROWSING MARKETPLACE**

```
User Opens Browser (localhost:4200)
         ↓
[Shop Frontend - Angular]
    - Renders login page with Firebase
    - User logs in with email/password
         ↓
Authentication Service (Firebase)
    - Creates session token
    - Stores in localStorage
         ↓
API Call: GET /get-products
    (api-client.service.ts → axios)
         ↓
[Shop Backend - Port 8001]
    - Connects to MongoDB
    - Queries: db.products collection
    - Returns: [{artisan_info, art_info, verification: {public_id, ...}}]
         ↓
Frontend Receives Products
    - Renders product cards
    - Shows artisan name, art image, description
    - Displays verification QR code (links to Master-IP verify endpoint)
         ↓
User Scans QR Code or Clicks "Verify"
    - Frontend calls: GET /verify/{public_id} on Master-IP (Port 8000)
         ↓
[Master-IP Backend - Port 8000]
    - Queries MongoDB: masterip_db.craftids collection
    - Gets CraftID metadata and stored_hash
    - Queries blockchain: Polygon Amoy for tx_hash
    - Compares: stored_hash vs computed_hash (tamper detection)
    - Returns: {public_id, status, is_tampered, blockchain_verified, ...}
         ↓
Frontend Displays Verification Status
    - ✅ "Authentic - Verified on Blockchain"
    - ❌ "Tampered - Hash Mismatch"
    - ⏳ "Pending - Waiting for blockchain confirmation"
```

### **FLOW 2: ARTISAN CREATING A CRAFTID (AI Agent)**

```
Artisan visits Agentic Service (localhost:8080)
         ↓
[ADK Interface]
    - "Welcome to ADK! Select an agent to begin."
    - Clicks: "Onboarding Agent"
         ↓
[Onboarding Agent - Conversational AI]
    Agent: "Tell me about your artwork"
    Artisan: "It's a traditional indigo-dyed textile..."
         ↓
Agent sends message to Google Gemini 2.0
    - Context: Artwork description
    - Model: Gemini 2.0 Flash (multimodal)
    - Session stored in: Neon PostgreSQL
         ↓
Gemini generates next question:
    "How long did it take to create?"
    Artisan: "About 3 days of hand-dyeing..."
         ↓
Agent collects all details, then calls:
    POST /create (Master-IP Backend)
    Payload: {
        artwork_metadata: {...descriptions...},
        images: [base64_encoded_images],
        artisan_id: "ARN-123"
    }
         ↓
[Master-IP Backend - Port 8000]
    1. Generate CraftID: "CID-00456"
    2. Create embeddings:
       - Image embedding using OpenAI CLIP
       - Text embedding using Sentence-Transformers
    3. Store in MongoDB:
       - db.masterip_db.craftids collection
       - Include: metadata, hashes, embeddings
    4. Index in Pinecone:
       - craft-images index (image vectors)
       - craft-text index (text vectors)
    5. Create queue entry:
       - db.masterip_db.queue collection
       - status: "pending"
    6. Return: {public_id: "CID-00456", status: "pending", ...}
         ↓
Agent confirms to artisan:
    "✅ Your artwork registered as CID-00456"
    "Anchoring to blockchain in progress..."
         ↓
[Chain Batcher - Background Process]
    - Runs every 10 seconds
    - Queries MongoDB queue: {status: "pending"}
    - Collects batch of pending CraftIDs
    - Signs with private key (OpenSSL)
    - Calls Smart Contract: CraftAnchor.anchor()
    - Updates MongoDB: {status: "anchored", tx_hash: "0x..."}
         ↓
[Polygon Amoy Blockchain]
    - Transaction mined
    - CraftID permanently recorded on blockchain
    - Timestamp immutable
         ↓
Artisan can now:
    - Sell on marketplace (appears in Shop Frontend)
    - Share verification QR code
    - Verify blockchain proof anytime
```

### **FLOW 3: DUPLICATE DETECTION**

```
Artisan uploads new artwork to create CraftID
         ↓
[Master-IP Backend]
    - Generates embeddings for new artwork
    - Queries Pinecone similarity search
    - Request: "Find similar items (cosine_similarity > 0.8)"
         ↓
[Pinecone Vector DB]
    - Searches craft-images index
    - Searches craft-text index
    - Returns: [CID-00123 (92% match), CID-00234 (85% match), ...]
         ↓
Master-IP API Response:
    {
        public_id: "CID-00456",
        similar_items: [
            {public_id: "CID-00123", similarity: 0.92, reason: "Image match"},
            {public_id: "CID-00234", similarity: 0.85, reason: "Text match"}
        ],
        is_duplicate: true/false
    }
         ↓
If is_duplicate = true:
    - Agent warns artisan about potential copy
    - Suggests contacting original creator (CID-00123 owner)
    - Both artisans can prove ownership via blockchain
```

---

## 🗄️ DATABASE SCHEMA & FLOW

### **1. MONGODB ATLAS**

#### Collection: `masterip_db.craftids`
```json
{
    "_id": ObjectId(),
    "public_id": "CID-00123",
    "artisan_id": "ARN-001",
    "metadata": {
        "title": "Indigo Textile",
        "description": "Traditional hand-dyed...",
        "materials": ["cotton", "indigo dye"],
        "creation_date": "2025-08-01",
        "artisan_name": "Harin Savaliya",
        "artisan_location": "Rajasthan, India"
    },
    "stored_hash": "abc123def456...",
    "image_embedding": [0.123, 0.456, ...],      // CLIP embeddings
    "text_embedding": [0.789, 0.012, ...],       // Sentence-Transformer embeddings
    "image_url": "s3://...",
    "status": "anchored",                         // pending, anchored, failed
    "tx_hash": "0x1234567890abcdef...",          // Blockchain tx
    "blockchain_timestamp": "2025-08-01T10:30Z",
    "created_at": "2025-08-01T10:25Z",
    "updated_at": "2025-08-01T10:35Z"
}
```

#### Collection: `masterip_db.queue`
```json
{
    "_id": ObjectId(),
    "public_id": "CID-00124",
    "status": "pending",
    "created_at": "2025-08-01T10:26Z",
    "batch_id": null              // Assigned when batched
}
```

#### Collection: `kalaakari_shop_db.products`
```json
{
    "_id": ObjectId(),
    "artisan_info": {
        "name": "Harin Savaliya",
        "location": "Rajasthan, India"
    },
    "art_info": {
        "name": "Traditional Indigo Textile",
        "description": "Hand-dyed cotton fabric...",
        "photo": "https://..."
    },
    "verification": {
        "public_id": "CID-00123",
        "verification_url": "http://localhost:8000/verify/CID-00123"
    },
    "price": 5000,
    "stock": 3,
    "created_at": "2025-08-01T10:35Z"
}
```

### **2. PINECONE VECTOR DB**

#### Index: `craft-images`
```
Dimension: 512 (OpenAI CLIP image embeddings)
Metric: cosine
Records:
  - Namespace: "craft-images"
  - Vector ID: "CID-00123"
  - Vector: [0.123, 0.456, ..., 512 dimensions]
  - Metadata: {public_id, artisan_id, creation_date, ...}
```

#### Index: `craft-text`
```
Dimension: 384 (Sentence-Transformers embeddings)
Metric: cosine
Records:
  - Namespace: "craft-text"
  - Vector ID: "CID-00123"
  - Vector: [0.789, 0.012, ..., 384 dimensions]
  - Metadata: {public_id, title, description, ...}
```

### **3. NEON POSTGRESQL**

#### Table: `agent_sessions`
```sql
CREATE TABLE agent_sessions (
    id UUID PRIMARY KEY,
    user_id VARCHAR(255),
    agent_type VARCHAR(50),  -- 'onboarding', 'ip', 'shop'
    conversation_history JSONB,
    context JSONB,
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);
```

### **4. POLYGON AMOY BLOCKCHAIN**

#### Smart Contract: `CraftAnchor.sol`
```solidity
struct CraftRecord {
    string public_id;        // CID-00123
    bytes32 craft_hash;      // Immutable hash
    uint256 timestamp;       // Block timestamp
    address signer;          // Who signed it
}

mapping(string => CraftRecord) public craftRecords;

function anchor(string[] memory publicIds, bytes32[] memory hashes) {
    // Records batch of CraftIDs on-chain
}
```

---

## 🔐 API KEYS & CREDENTIALS FLOW

### **Frontend Configuration** (Environment Variables)
```javascript
// shop/shop-frontend/environments/environments.ts
export const environment = {
  production: false,
  firebase: {
    apiKey: "AIzaSyDtZ...",           // Google Firebase
    authDomain: "kalakaari.firebaseapp.com",
    projectId: "kalakaari-2025",
    storageBucket: "kalakaari-2025.appspot.com",
    messagingSenderId: "123456789",
    appId: "1:123456789:web:abc123def"
  },
  shopApiUrl: "http://localhost:8001",    // Local Dev
  masterIpApiUrl: "http://localhost:8000"  // Local Dev
};

// Production (uses deployed URLs):
// shopApiUrl: "https://kalakaari-shop-backend-*.run.app"
// masterIpApiUrl: "https://master-ip-service-*.run.app"
```

### **Backend Configuration** (Environment Variables)

#### Shop Backend `.env`
```bash
# MongoDB
MONGO_URI=mongodb+srv://harinsavaliya07_db_user:password@cluster0.mongodb.net/?appName=Cluster0
DB_NAME=kalaakari_shop_db

# API Keys
SECRET_KEY=your_secret_key_here

# Firebase Service Account
FIREBASE_PROJECT_ID=kalakaari-2025
FIREBASE_PRIVATE_KEY=-----BEGIN PRIVATE KEY-----...
FIREBASE_CLIENT_EMAIL=firebase-adminsdk@...iam.gserviceaccount.com
```

#### Master-IP Backend `.env`
```bash
# MongoDB
MONGO_URI=mongodb+srv://harinsavaliya07_db_user:password@cluster0.mongodb.net/?appName=Cluster0
DB_NAME=masterip_db

# Pinecone Vector DB
PINECONE_API_KEY=pcsk_ytxkV_4t9cjCU...
PINECONE_ENV=us-east-1
INDEX_HOST=craft-images-q60kye7.svc.aped-4627-b74a.pinecone.io
PINECONE_TEXT_INDEX=craft-text

# Blockchain
WEB3_RPC_URL=https://rpc-amoy.polygon.technology
CHAIN_ID=80002  # Polygon Amoy
ANCHORER_PRIVATE_KEY=ffd81bcb0d2f2e81d2772476ab7d5d5941db654370f11cf69d32d2b6f2dc1167

# Signing Keys
SIGNER_KEY_PATH=/workspaces/kalakaari-ai-solution/master-ip/server/chain/keys/sign_priv.pem
PLATFORM_PUBKEY_PATH=/workspaces/kalakaari-ai-solution/master-ip/server/chain/keys/sign_pub.pem

SECRET_KEY=your_secret_key
```

#### Agentic Service `.env`
```bash
# Google Cloud
GOOGLE_API_KEY=AIzaSyDtZ...
GOOGLE_GENAI_USE_VERTEXAI=FALSE

# Neon PostgreSQL (Agent Sessions)
SESSION_SERVICE_URI=postgresql://user:password@ep-spring-cloud-12345.us-east-2.neon.tech/neondb
DATABASE_URL=postgresql://user:password@ep-spring-cloud-12345.us-east-2.neon.tech/neondb

# AI Model
MODEL_NAME=gemini-2.0-flash

# Service URLs
MASTER_IP_URL=http://localhost:8000
SHOP_BACKEND_URL=http://localhost:8001

SECRET_KEY=your_secret_key
```

---

## 🔄 REQUEST/RESPONSE CYCLE

### **Example 1: Frontend → Shop Backend → MongoDB**

```
FRONTEND (Angular)
  └─ api-client.service.ts
     └─ this.http.get<Product[]>(`http://localhost:8001/get-products`)

SHOP BACKEND (FastAPI)
  └─ routes/products.py
     └─ @router.get("/get-products")
        └─ db = get_database()  // MongoDB connection
           └─ products = db.products.find({})
              └─ return [Product(...), Product(...)]

MONGODB ATLAS
  └─ kalaakari_shop_db.products collection
     └─ Returns: [{artisan_info, art_info, verification}, ...]

RESPONSE TO FRONTEND
  └─ Angular displays products on page
```

### **Example 2: Frontend → Master-IP → Pinecone → Blockchain**

```
FRONTEND (Angular)
  └─ Click "Verify" on product CID-00123

MASTER-IP BACKEND (FastAPI)
  └─ GET /verify/CID-00123
     ├─ Query MongoDB: craftids.findOne({public_id: "CID-00123"})
     │  └─ Get: stored_hash, tx_hash, metadata
     │
     ├─ Connect to Web3 (Polygon Amoy)
     │  └─ Get transaction details from blockchain
     │  └─ Verify signature
     │
     └─ Return: {
          public_id: "CID-00123",
          status: "anchored",
          is_tampered: false,
          blockchain_verified: true,
          tx_hash: "0x1234...",
          blockchain_timestamp: "2025-08-01T10:35Z"
        }

FRONTEND
  └─ Display: ✅ "Verified on Blockchain"
     └─ Show transaction link
     └─ Display artist info
```

---

## 🚀 DEPLOYMENT FLOW

### **Local Development** (Current)
```
localhost:4200  → Shop Frontend
localhost:8001  → Shop Backend
localhost:8000  → Master-IP Backend
Port 8080       → Agentic Service
Background      → Chain Batcher
```

### **Production** (Google Cloud Run)
```
Cloud DNS → https://kalakaari-shop.app
           → Cloud Run Container: Shop Frontend (Angular)
           → Cloud Run Service: Shop Backend (FastAPI)
           → Cloud Run Service: Master-IP Backend (FastAPI)
           → Cloud Run Service: Agentic Service (FastAPI + ADK)

External Services:
├─ MongoDB Atlas (Global Database)
├─ Pinecone (Vector Search)
├─ Neon PostgreSQL (Agent Sessions)
├─ Google Gemini 2.0 Flash (AI Model)
└─ Polygon Amoy Blockchain (Immutable Ledger)
```

---

## 📊 TECHNOLOGY STACK SUMMARY

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Frontend** | Angular 20, TypeScript, Tailwind CSS | User Interface |
| **Frontend Auth** | Firebase Authentication | User Login/Signup |
| **Backend APIs** | Python, FastAPI, Uvicorn | REST APIs |
| **Databases** | MongoDB Atlas, Neon PostgreSQL | Data Storage |
| **Vector Search** | Pinecone, OpenAI CLIP, Sentence-Transformers | Similarity Detection |
| **AI/ML** | Google Gemini 2.0 Flash, Google ADK | Conversational AI |
| **Blockchain** | Solidity, Web3.py, Polygon Amoy | Immutable Provenance |
| **Cryptography** | OpenSSL, RSA Signing | Digital Signatures |
| **Hosting** | Docker, Google Cloud Run | Deployment |

---

## ✅ HOW EVERYTHING WORKS TOGETHER

### **The Complete User Journey**

```
1. USER REGISTERS
   ├─ Visits localhost:4200
   ├─ Signs up with email (Firebase stores credentials)
   └─ Session token saved locally

2. USER BROWSES
   ├─ Frontend requests /get-products
   ├─ Shop Backend queries MongoDB
   ├─ Returns all verified artisan products
   └─ User sees products with CraftID verification links

3. ARTISAN REGISTERS ARTWORK
   ├─ Visits localhost:8080 (Agentic Service)
   ├─ Chats with "Onboarding Agent" (Google Gemini AI)
   ├─ Agent collects artwork details conversationally
   ├─ Agent calls POST /create (Master-IP)
   └─ Master-IP creates CraftID with embeddings

4. CRAFTID GETS ANCHORED
   ├─ Chain Batcher polls queue every 10s
   ├─ Collects pending CraftIDs
   ├─ Signs with private key
   ├─ Calls smart contract on Polygon Amoy
   └─ Blockchain records proof permanently

5. USER VERIFIES
   ├─ Scans QR code on product
   ├─ Frontend calls GET /verify/CID-00123
   ├─ Master-IP checks blockchain
   ├─ Returns: ✅ "Authentic" or ❌ "Tampered"
   └─ User confident about authenticity

6. MARKETPLACE LISTING
   ├─ Product appears on Shop Frontend
   ├─ User adds to cart
   ├─ Checkout via Shop Backend
   ├─ Order stored in MongoDB
   └─ Artisan receives payment
```

---

## 🎯 KEY SECURITY FEATURES

1. **Blockchain Immutability** - Once anchored, cannot be changed
2. **Tamper Detection** - Hash comparison detects metadata changes
3. **Digital Signatures** - RSA signing proves platform authority
4. **Duplicate Detection** - Pinecone similarity search catches copies
5. **User Authentication** - Firebase prevents unauthorized access
6. **Environment Secrets** - API keys not exposed in frontend
7. **CORS Protection** - Backend restricts requests to authorized origins

---

## 📈 SCALABILITY

| Component | Scaling Strategy |
|-----------|------------------|
| **Frontend** | CDN caching, lazy loading, code splitting |
| **Shop Backend** | Docker containers, load balancing |
| **Master-IP** | Horizontal scaling on Cloud Run |
| **MongoDB** | Sharding by public_id, indexing |
| **Pinecone** | Auto-scaling vector queries |
| **Blockchain** | Batching reduces transaction count |
| **Agent Sessions** | PostgreSQL connection pooling |

---

## 🔍 DEBUGGING: WHERE TO LOOK

```
User sees no products?
├─ Check Shop Backend logs: localhost:8001/docs
├─ Verify MongoDB connection: .env MONGO_URI
└─ Check browser console for API errors

CraftID not anchored?
├─ Check Chain Batcher: Terminal output
├─ Verify blockchain RPC: WEB3_RPC_URL
├─ Check queue in MongoDB: masterip_db.queue

AI Agent not responding?
├─ Check Agentic Service logs: localhost:8080
├─ Verify Google API Key: .env GOOGLE_API_KEY
└─ Check Neon PostgreSQL connection

Verification failing?
├─ Check blockchain status: Polygon Amoy explorer
├─ Verify Pinecone connection: .env PINECONE_API_KEY
└─ Compare stored_hash vs computed_hash in MongoDB
```

---

## 🎓 LEARNING RESOURCES

- **FastAPI**: https://fastapi.tiangolo.com/
- **Angular**: https://angular.io/docs
- **MongoDB**: https://docs.mongodb.com/
- **Pinecone**: https://docs.pinecone.io/
- **Web3.py**: https://web3py.readthedocs.io/
- **Google ADK**: https://ai.google.dev/agentic/
- **Polygon**: https://wiki.polygon.technology/

---

**Your Kalakaari platform is a sophisticated full-stack application that combines modern web technologies with AI and blockchain!** 🚀🎨
