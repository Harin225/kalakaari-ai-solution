# 🎨 KALAKAARI PROJECT - SIMPLIFIED SUMMARY

## What is Kalakaari?

**An AI-powered marketplace that protects artisan intellectual property by:**
1. Using AI (Google Gemini) to verify artwork authenticity
2. Recording proof on blockchain (Polygon) that cannot be changed
3. Detecting AI-generated copies using similarity search
4. Creating digital certificates artisans can share/verify

---

## How It Works in 3 Steps

### 🎨 **Step 1: Artisan Creates Artwork (CraftID)**
```
Artisan visits:  http://localhost:8080 (AI Agent)
                      ↓
Chats with AI:   "Tell me about your artwork"
                      ↓
AI Agent:        Asks follow-up questions
                      ↓
Creates:         CraftID = "CID-00123" 
                      ↓
Stores:          MongoDB (metadata + embeddings)
                      ↓
Queues for:      Blockchain anchoring
```

### ⛓️ **Step 2: Platform Anchors to Blockchain**
```
Chain Batcher:   Wakes up every 10 seconds
                      ↓
Collects:        All pending CraftIDs
                      ↓
Signs:           With private key (proof of authenticity)
                      ↓
Records:         On Polygon blockchain (permanent)
                      ↓
Updates:         Status = "anchored" in MongoDB
```

### 🛍️ **Step 3: Customer Verifies & Buys**
```
Customer visits: http://localhost:4200 (Shop)
                      ↓
Browses:         Products (artisan handcrafted items)
                      ↓
Clicks:          "Verify" on product
                      ↓
Platform checks: Blockchain proof + hash verification
                      ↓
Shows:           ✅ "Authentic" or ❌ "Tampered"
                      ↓
Customer buys:   With confidence
```

---

## 🔌 HOW FRONTEND CONNECTS TO EVERYTHING

### Frontend Code Path

```typescript
// shop-frontend/src/app/services/api-client.service.ts

// Step 1: Frontend sends HTTP request
getProducts(): Observable<Product[]> {
    return this.http.get<Product[]>(
        'http://localhost:8001/get-products'  // Shop Backend
    );
}

// Step 2: Shop Backend receives request
// shop-backend/app/routes/products.py
@router.get("/get-products")
async def get_products():
    db = get_database()  // Connects to MongoDB
    products = db.products.find({})
    return products

// Step 3: MongoDB returns data
// MongoDB: kalaakari_shop_db.products
{
    "artisan_info": {"name": "Harin Savaliya"},
    "art_info": {"name": "Textile", "photo": "..."},
    "verification": {"public_id": "CID-00123"}
}

// Step 4: Frontend displays products
// Angular shows products on web page
```

### Verification Path

```typescript
// Click "Verify" button on product
verifyCraftID(publicId: 'CID-00123')
    .subscribe(result => {
        // result = {
        //   public_id: 'CID-00123',
        //   blockchain_verified: true,
        //   is_tampered: false,
        //   tx_hash: '0x1234...'
        // }
    });

// ↓ Calls Master-IP Backend

// master-ip/server/app/main.py
@app.get("/verify/{public_id}")
async def verify(public_id: str):
    # Query MongoDB for CraftID
    craftid = db.craftids.findOne({public_id})
    
    # Query Polygon blockchain
    tx_data = web3.get_transaction(craftid.tx_hash)
    
    # Compare hashes
    is_tampered = (craftid.stored_hash != computed_hash)
    
    return {
        "blockchain_verified": True,
        "is_tampered": is_tampered,
        "tx_hash": tx_data.hash
    }

// ↓ Returns to Frontend

// Frontend displays: ✅ VERIFIED
```

---

## 📊 DATABASE CONNECTIONS

### Where Data Lives

| Data Type | Storage | Purpose |
|-----------|---------|---------|
| **Products** | MongoDB: `kalaakari_shop_db.products` | Shop items, artisan info |
| **CraftIDs** | MongoDB: `masterip_db.craftids` | Artwork metadata, hashes, embeddings |
| **Pending Anchoring** | MongoDB: `masterip_db.queue` | Items waiting for blockchain |
| **Image Similarity** | Pinecone: `craft-images` index | Find similar images (CLIP vectors) |
| **Text Similarity** | Pinecone: `craft-text` index | Find similar descriptions |
| **AI Chat History** | Neon PostgreSQL | Agent conversation sessions |
| **Blockchain Proof** | Polygon Amoy | Immutable CraftID records |

### Connection Flow

```
Frontend (Angular)
    ↓
Shop Backend (.env: MONGO_URI)
    ↓ Connects
MongoDB Atlas (kalaakari_shop_db)
    ↓ Returns products

Master-IP Backend (.env: MONGO_URI, PINECONE_API_KEY, WEB3_RPC_URL)
    ↓ Connects to
MongoDB (masterip_db) + Pinecone (vectors) + Polygon (blockchain)
    ↓
All return verified data to Frontend
```

---

## 🔐 API KEYS & CREDENTIALS

### Where They Go

```
.env Files (Never committed to git):
│
├─ master-ip/server/.env
│   ├─ MONGO_URI = MongoDB connection
│   ├─ PINECONE_API_KEY = Vector search
│   ├─ WEB3_RPC_URL = Polygon blockchain
│   └─ ANCHORER_PRIVATE_KEY = Sign transactions
│
├─ shop/shop-backend/.env
│   ├─ MONGO_URI = MongoDB connection
│   └─ FIREBASE_CREDENTIALS = User auth
│
└─ agentic/.env
    ├─ GOOGLE_API_KEY = Gemini AI
    └─ SESSION_SERVICE_URI = Neon PostgreSQL
```

### How Frontend Never Sees Secrets

```
❌ WRONG - Frontend has API keys (INSECURE):
Frontend → Google API Key → Gemini

✅ CORRECT - Backend handles secrets:
Frontend → "Please generate CraftID"
           → Backend receives (has API keys)
           → Backend calls Gemini + MongoDB
           → Backend returns result to Frontend
           → Frontend displays result
```

---

## 🚀 Request Lifecycle

### Example: User Clicks "Verify Product"

```
┌─────────────────────────────────────────────────────────────┐
│ USER BROWSER (Angular Frontend)                             │
│ Location: http://localhost:4200                             │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  User clicks "Verify" button on CID-00123                  │
│                                                              │
│  ↓ Angular calls:                                           │
│  this.apiClient.verifyCraftID('CID-00123')                  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
           ↓ HTTP GET Request
           ↓ http://localhost:8000/verify/CID-00123
           ↓

┌─────────────────────────────────────────────────────────────┐
│ MASTER-IP BACKEND (FastAPI)                                 │
│ Location: http://localhost:8000                             │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Receives: GET /verify/CID-00123                            │
│                                                              │
│  ├─ Connects to MongoDB                                     │
│  │  └─ Queries: db.craftids.find({public_id: 'CID-00123'}) │
│  │  └─ Gets: stored_hash, tx_hash, metadata                │
│  │                                                          │
│  ├─ Connects to Polygon Blockchain                          │
│  │  └─ Gets transaction: 0x1234...                          │
│  │  └─ Verifies signature                                   │
│  │  └─ Gets blockchain timestamp                            │
│  │                                                          │
│  ├─ Calculates: computed_hash from metadata                 │
│  │                                                          │
│  ├─ Compares: stored_hash == computed_hash?                 │
│  │  └─ If different: is_tampered = true                     │
│  │                                                          │
│  └─ Returns JSON response:                                  │
│     {                                                       │
│       "public_id": "CID-00123",                              │
│       "status": "anchored",                                  │
│       "blockchain_verified": true,                           │
│       "is_tampered": false,                                  │
│       "tx_hash": "0x1234567890abcdef...",                    │
│       "blockchain_timestamp": "2025-08-01T10:35Z"            │
│     }                                                       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
           ↓ HTTP Response
           ↓ JSON data
           ↓

┌─────────────────────────────────────────────────────────────┐
│ USER BROWSER (Angular Frontend)                             │
│                                                              │
│  Frontend receives JSON response                            │
│                                                              │
│  ├─ Checks: blockchain_verified == true? ✅                │
│  ├─ Checks: is_tampered == false? ✅                        │
│  │                                                          │
│  └─ Displays: "✅ VERIFIED ON BLOCKCHAIN"                   │
│     └─ Shows: tx_hash link to Polygon explorer              │
│     └─ Shows: Blockchain timestamp                          │
│     └─ Shows: Artisan name & info                           │
│                                                              │
│  User now confident artwork is authentic!                   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔄 Complete Data Journey

### When Artisan Registers Artwork

```
1. Artisan chats with AI (Agentic Service)
   "I made a textile artwork"
   ↓

2. AI collects details conversationally
   - Artwork name, description, materials
   - Images uploaded
   ↓

3. AI calls Master-IP: POST /create
   Body: {
       metadata: {title, description, materials, artisan_name},
       images: [base64_encoded_image_1, ...]
   }
   ↓

4. Master-IP creates CraftID
   ├─ Generates: public_id = "CID-00123"
   ├─ Creates embeddings:
   │  ├─ Image embedding using OpenAI CLIP
   │  └─ Text embedding using Sentence-Transformers
   ├─ Stores in MongoDB:
   │  └─ db.masterip_db.craftids
   │     {public_id, metadata, embeddings, stored_hash}
   ├─ Indexes in Pinecone:
   │  ├─ craft-images index (for image similarity)
   │  └─ craft-text index (for text similarity)
   ├─ Creates queue entry:
   │  └─ db.masterip_db.queue
   │     {public_id, status: "pending"}
   └─ Returns: {public_id: "CID-00123", status: "pending"}
   ↓

5. AI confirms to artisan
   "✅ Your artwork registered as CID-00123"
   "Anchoring to blockchain in progress..."
   ↓

6. Chain Batcher (background process)
   ├─ Wakes up every 10 seconds
   ├─ Queries: db.queue where status="pending"
   ├─ Collects batch: [CID-00123, CID-00124, CID-00125]
   ├─ Signs batch with private key (cryptographic proof)
   ├─ Calls: CraftAnchor.anchor() on Polygon smart contract
   ├─ Polygon records on blockchain:
   │  └─ Transaction hash: 0x1234567890abcdef...
   │  └─ Block timestamp: immutable
   └─ Updates MongoDB:
      └─ db.queue where public_id="CID-00123"
         set status="anchored", tx_hash="0x1234..."
   ↓

7. Product added to Shop
   ├─ Artisan (or admin) creates product in Shop
   ├─ Links verification: public_id = "CID-00123"
   ├─ Stored: db.kalaakari_shop_db.products
   │  {
   │    artisan_info: {name, location},
   │    art_info: {name, description, photo},
   │    verification: {public_id: "CID-00123"},
   │    price: 5000
   │  }
   └─ Product appears on Shop Frontend
   ↓

8. Customer verifies
   ├─ Visits: http://localhost:4200
   ├─ Sees product with "Verify" button
   ├─ Clicks: "Verify" → Calls Master-IP verify endpoint
   ├─ Master-IP:
   │  ├─ Gets CraftID from MongoDB
   │  ├─ Checks blockchain for tx_hash
   │  ├─ Compares hashes (tamper detection)
   │  └─ Returns verification status
   └─ Frontend shows: ✅ "VERIFIED - Authentic"
   ↓

9. Customer purchases
   ├─ Adds to cart
   ├─ Checks out via Shop Backend
   ├─ Order stored: db.kalaakari_shop_db.orders
   └─ Artisan receives payment
```

---

## 💾 What Each .env Variable Does

### Master-IP Backend
```bash
# MONGO_URI
# What: Connection string to MongoDB
# Used for: Storing CraftIDs, queue entries, metadata
# Example: mongodb+srv://user:pass@cluster.mongodb.net/

# PINECONE_API_KEY
# What: Authentication for Pinecone vector database
# Used for: Storing/querying image and text embeddings
# Example: pcsk_abc123xyz789...

# WEB3_RPC_URL
# What: Connection to Polygon blockchain
# Used for: Reading/writing smart contract data
# Example: https://rpc-amoy.polygon.technology

# ANCHORER_PRIVATE_KEY
# What: Private key for signing blockchain transactions
# Used for: Proving platform authority when anchoring
# Example: ffd81bcb0d2f2e81d2772476ab7d5d5941db654370f11cf69d32d2b6f2dc1167
```

### Shop Backend
```bash
# MONGO_URI
# What: Connection string to MongoDB
# Used for: Storing products, orders, user data
# Example: mongodb+srv://user:pass@cluster.mongodb.net/

# FIREBASE_CREDENTIALS
# What: Service account for Firebase authentication
# Used for: Verifying user tokens, managing auth
# Example: JSON with private_key, project_id, etc.
```

### Agentic Service
```bash
# GOOGLE_API_KEY
# What: API key for Google Gemini AI
# Used for: Conversational AI responses
# Example: AIzaSyDtZ...

# SESSION_SERVICE_URI
# What: Connection to Neon PostgreSQL
# Used for: Storing chat conversation history
# Example: postgresql://user:pass@ep-cloud-123.neon.tech/neondb
```

---

## 🎯 Complete System Overview (One Diagram)

```
┌────────────────────────────────────────────────────────────────────┐
│                        BROWSER (User)                              │
│                    http://localhost:4200                           │
│                    (Angular Shop Frontend)                         │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  Products Page  │  Verify Button  │  Chat with AI            │ │
│  │  (from MongoDB) │  (from Blockchain) │  (Gemini responses)  │ │
│  └──────────────────────────────────────────────────────────────┘ │
└────────┬──────────────────────────┬──────────────────────────┬─────┘
         │                          │                          │
         │ GET /get-products        │ GET /verify/CID-123      │ Chat messages
         │ (Port 8001)              │ (Port 8000)              │ (Port 8080)
         │                          │                          │
    ┌────▼─────┐          ┌────────▼─────┐        ┌──────────▼──────┐
    │ SHOP      │          │ MASTER-IP    │        │ AGENTIC        │
    │ BACKEND   │          │ BACKEND      │        │ SERVICE        │
    │ FastAPI   │          │ FastAPI      │        │ FastAPI + ADK  │
    │ :8001     │          │ :8000        │        │ :8080          │
    └────┬──────┘          └────┬─────────┘        └────────┬───────┘
         │                      │                          │
    ┌────▼──────────────────────▼──────────────────────────▼────┐
    │ ↓ MongoDB (3 connections, 1 service)                       │
    │ ├─ kalaakari_shop_db.products (Shop Backend)              │
    │ ├─ masterip_db.craftids (Master-IP)                       │
    │ └─ masterip_db.queue (Master-IP + Chain Batcher)          │
    └────────────────────────────────────────────────────────────┘
         │
    ┌────▼──────────────────────────────────────────────────────┐
    │ ↓ Pinecone (Vector Similarity Search)                     │
    │ ├─ craft-images (CLIP embeddings - image similarity)      │
    │ └─ craft-text (text embeddings - description similarity)  │
    └────────────────────────────────────────────────────────────┘
         │
    ┌────▼──────────────────────────────────────────────────────┐
    │ ↓ Polygon Amoy Blockchain (Immutable Records)             │
    │    CraftAnchor Smart Contract                             │
    │    - Stores CraftID proofs                                │
    │    - Records blockchain timestamp                         │
    │    - Enables verification                                 │
    └────────────────────────────────────────────────────────────┘
         │
    ┌────▼──────────────────────────────────────────────────────┐
    │ ↓ Background Process: Chain Batcher (Every 10s)          │
    │    - Polls MongoDB queue                                  │
    │    - Batches pending CraftIDs                             │
    │    - Signs with private key                               │
    │    - Calls smart contract                                 │
    │    - Updates status to "anchored"                         │
    └────────────────────────────────────────────────────────────┘
         │
    ┌────▼──────────────────────────────────────────────────────┐
    │ ↓ External Services                                        │
    │ ├─ Neon PostgreSQL (Agentic sessions)                     │
    │ ├─ Google Gemini 2.0 Flash (AI model)                     │
    │ └─ Firebase (User authentication)                         │
    └────────────────────────────────────────────────────────────┘
```

---

## ✨ Key Insight

**The frontend never directly connects to databases or blockchain.** Instead:

1. **Frontend** makes HTTP requests to backend APIs
2. **Backend APIs** receive requests, load credentials from `.env` files
3. **Backend APIs** connect to MongoDB, Pinecone, Blockchain using those credentials
4. **Backend APIs** return data to frontend
5. **Frontend** displays the data

**This keeps API keys secure and data protected!** 🔐

---

## 🚀 Next Steps

- **Read** [PROJECT_ARCHITECTURE.md](./PROJECT_ARCHITECTURE.md) for technical deep dive
- **Check** individual service READMEs for deployment instructions
- **Test** APIs via Swagger: http://localhost:8000/docs
- **Browse** frontend: http://localhost:4200
- **Chat** with AI: http://localhost:8080

---

**Kalakaari = AI-Verified Artisan Authentication + Blockchain Provenance + Smart Marketplace** 🎨⛓️🛍️
