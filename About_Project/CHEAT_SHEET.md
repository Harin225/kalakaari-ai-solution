# 🎨 KALAKAARI - ONE PAGE CHEAT SHEET

## PROJECT PURPOSE
**AI-Powered Art Authentication & Marketplace** - Protects artisan intellectual property using AI verification, blockchain provenance, and duplicate detection.

---

## 5 MICROSERVICES

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│  4200: SHOP FRONTEND          4080: AGENTIC SERVICE       │
│  ├─ Angular UI                ├─ AI Agents (Google Gemini)│
│  ├─ Browse products           ├─ Chat with AI             │
│  └─ Verify artworks           └─ Conversational AI        │
│                                                            │
│  4001: SHOP BACKEND           4000: MASTER-IP             │
│  ├─ Products API              ├─ CraftID creation         │
│  ├─ Orders API                ├─ Blockchain verification  │
│  └─ FastAPI → MongoDB         ├─ Similarity search        │
│                               └─ FastAPI → Pinecone + Web3│
│                                                            │
│  CHAIN BATCHER (Background)                               │
│  ├─ Polls queue every 10s                                 │
│  ├─ Anchors to Polygon blockchain                         │
│  └─ Proves creation authenticity                          │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## EXTERNAL SERVICES

| Service | Purpose | Stores |
|---------|---------|--------|
| **MongoDB Atlas** | Main database | Products, CraftIDs, Orders |
| **Pinecone** | Vector search | Image & text embeddings |
| **Neon PostgreSQL** | AI sessions | Agent conversation history |
| **Google Gemini 2.0** | AI model | Powers conversational agent |
| **Polygon Amoy** | Blockchain | Immutable CraftID proof |
| **Firebase** | Authentication | User login/signup |

---

## DATA FLOW: 3 MAIN SCENARIOS

### Scenario 1: Browse Products
```
User visits localhost:4200
    ↓
Frontend: GET /get-products (to port 8001)
    ↓
Shop Backend loads MONGO_URI from .env
    ↓
MongoDB returns products
    ↓
Frontend displays product cards with verification links
```

### Scenario 2: Register Artwork (AI Agent)
```
Artisan chats with AI (localhost:8080)
    ↓
AI asks questions about artwork (Google Gemini)
    ↓
Artisan provides details
    ↓
AI calls Master-IP: POST /create
    ↓
Master-IP Backend:
├─ Creates CraftID (e.g., "CID-00123")
├─ Generates embeddings (CLIP + Sentence-Transformers)
├─ Stores in MongoDB
├─ Indexes in Pinecone
└─ Adds to queue for blockchain anchoring
    ↓
Chain Batcher (background, every 10s):
├─ Finds pending items
├─ Signs with private key
├─ Calls Polygon smart contract
└─ Updates status to "anchored"
    ↓
Artwork proof now on blockchain forever ✅
```

### Scenario 3: Verify Authenticity
```
Customer clicks "Verify" on product (CID-00123)
    ↓
Frontend: GET /verify/CID-00123 (to port 8000)
    ↓
Master-IP Backend:
├─ Queries MongoDB for CraftID
├─ Queries Polygon blockchain for proof
├─ Compares stored_hash vs computed_hash
└─ Detects if metadata was tampered
    ↓
Returns: ✅ "Authentic" or ❌ "Tampered"
    ↓
Frontend displays blockchain proof link
```

---

## ARCHITECTURE DIAGRAM

```
                    USER BROWSER
                        │
        ┌───────────────┼───────────────┐
        │               │               │
    localhost:4200   localhost:8000   localhost:8080
    Shop Frontend    Master-IP API    Agentic Service
    (Angular)        (FastAPI)        (FastAPI + ADK)
        │               │               │
        ├─ Firebase   ├─ MongoDB      ├─ Google Gemini
        │ (auth)      ├─ Pinecone     ├─ Neon PostgreSQL
        └─────────────┴─ Polygon  ────┘
                      └─ Web3

    localhost:8001
    Shop Backend
    (FastAPI)
        │
        └─ MongoDB
```

---

## .ENV FILES - WHAT GOES WHERE

### master-ip/server/.env
```
MONGO_URI=mongodb+srv://...                    # MongoDB connection
DB_NAME=masterip_db
PINECONE_API_KEY=pcsk_...                     # Vector database
PINECONE_ENV=us-east-1
INDEX_HOST=craft-images-...
PINECONE_TEXT_INDEX=craft-text
WEB3_RPC_URL=https://rpc-amoy.polygon.technology  # Blockchain
CHAIN_ID=80002
ANCHORER_PRIVATE_KEY=ffd81bcb...              # Signs blockchain txs
SIGNER_KEY_PATH=/path/to/sign_priv.pem
PLATFORM_PUBKEY_PATH=/path/to/sign_pub.pem
```

### shop/shop-backend/.env
```
MONGO_URI=mongodb+srv://...                    # MongoDB connection
DB_NAME=kalaakari_shop_db
FIREBASE_PROJECT_ID=...
FIREBASE_PRIVATE_KEY=...
```

### agentic/.env
```
GOOGLE_API_KEY=AIzaSyDtZ...                   # Google Gemini API
SESSION_SERVICE_URI=postgresql://...           # Neon PostgreSQL
DATABASE_URL=postgresql://...
MODEL_NAME=gemini-2.0-flash
```

---

## CRITICAL CONCEPTS

### CraftID (Artwork Registration)
```
CraftID = "CID-00123"
├─ Metadata: name, description, artisan, materials
├─ Embeddings: Image (CLIP) + Text (Sentence-Transformers)
├─ Hash: SHA-256(metadata) → stored in MongoDB
├─ Status: "pending" → "anchored"
└─ Blockchain: Immutable proof recorded with tx_hash
```

### Vector Search (Duplicate Detection)
```
When new artwork registered:
├─ Generate embedding vectors
├─ Query Pinecone for similar items
├─ If >80% similarity: ⚠️ "Possible Duplicate"
└─ Compare with existing CraftIDs
```

### Blockchain Anchoring
```
Chain Batcher Flow:
1. Query MongoDB: queue where status="pending"
2. Collect batch of pending CraftIDs
3. Sign with ANCHORER_PRIVATE_KEY
4. Call CraftAnchor.sol smart contract
5. Polygon records tx_hash, timestamp
6. Update MongoDB: status="anchored"
```

### Tamper Detection
```
When verifying:
1. Get stored_hash from MongoDB
2. Get metadata from MongoDB
3. Compute hash from metadata: computed_hash = SHA-256(metadata)
4. Compare: stored_hash == computed_hash?
   ✅ YES → "Authentic"
   ❌ NO → "TAMPERED"
```

---

## SECURITY: FRONTEND → BACKEND → DATABASE

```
❌ WRONG (INSECURE)
Frontend has API credentials
    ↓
Browser inspect element reveals password
    ↓
Attacker steals credentials
    ↓
Database hacked 🚨

✅ CORRECT (SECURE)
Frontend has NO credentials
    ↓
Frontend calls: GET http://localhost:8001/get-products
    ↓
Backend loads credentials from .env (server-side)
    ↓
Backend connects to database
    ↓
Backend returns JSON to frontend
    ↓
Frontend displays data safely ✅
```

---

## API ENDPOINTS

### Shop Backend (Port 8001)
```
GET  /               → Status: "Shop backend is running!"
GET  /get-products   → Returns: [{products}]
POST /authenticate   → Login/register users
POST /orders         → Create order
GET  /orders/{id}    → Get user's orders
```

### Master-IP Backend (Port 8000)
```
GET  /               → Status: "Prototype Master-IP backend is running!"
POST /create         → Create CraftID (body: metadata + images)
GET  /verify/{id}    → Verify CraftID (blockchain proof)
POST /search         → Find similar artworks (duplicate detection)
GET  /batch-status   → Check pending anchoring
```

### Agentic Service (Port 8080)
```
GET  /               → Web UI: "Welcome to ADK! Select an agent to begin."
POST /chat           → Send message to AI agent
```

---

## DATABASES AT A GLANCE

### MongoDB: kalaakari_shop_db
```
Collection: products
{
  _id, artisan_info {name, location},
  art_info {name, description, photo},
  verification {public_id, verification_url},
  price, stock, created_at
}

Collection: orders
{
  _id, user_id, products[], total_price,
  status, created_at, updated_at
}
```

### MongoDB: masterip_db
```
Collection: craftids
{
  public_id, artisan_id, metadata {},
  stored_hash, image_embedding [], text_embedding [],
  status, tx_hash, blockchain_timestamp, created_at
}

Collection: queue
{
  public_id, status: "pending|anchored",
  batch_id, created_at
}
```

### Pinecone Indices
```
craft-images:
  - Dimension: 512 (OpenAI CLIP)
  - Metric: cosine
  - Records: Vector(512d) + metadata

craft-text:
  - Dimension: 384 (Sentence-Transformers)
  - Metric: cosine
  - Records: Vector(384d) + metadata
```

### Neon PostgreSQL
```
Table: agent_sessions
{
  id, user_id, agent_type,
  conversation_history {},
  context {}, created_at, updated_at
}
```

### Polygon Amoy Blockchain
```
Smart Contract: CraftAnchor
function anchor(public_ids[], hashes[])
├─ Stores: public_id → hash + timestamp
├─ Network: Polygon Amoy Testnet
├─ Chain ID: 80002
└─ Makes proof immutable
```

---

## KEY FILES TO KNOW

| File | Purpose |
|------|---------|
| `shop-frontend/src/app/services/api-client.service.ts` | API calls to backend |
| `shop-backend/app/main.py` | Shop API endpoints |
| `master-ip/server/app/main.py` | Master-IP API endpoints |
| `agentic/main.py` | Agentic service entry point |
| `agentic/agents/sub_agents/onboarding_agent/agent.py` | AI agent that creates CraftIDs |
| `master-ip/server/chain/batcher.py` | Background process for blockchain |
| `master-ip/contract/CraftAnchor.sol` | Smart contract |

---

## QUICK TROUBLESHOOTING

| Problem | Solution |
|---------|----------|
| Products not loading | Check Shop Backend logs, verify MongoDB MONGO_URI |
| Verification failing | Check Master-IP logs, verify WEB3_RPC_URL |
| AI not responding | Check Agentic logs, verify GOOGLE_API_KEY |
| Not anchoring to blockchain | Check Chain Batcher logs, verify ANCHORER_PRIVATE_KEY |
| API returns 403/401 | Check CORS settings, verify frontend origin allowed |
| Node modules error | Run `npm install --legacy-peer-deps` |
| Python import error | Check virtual environment activated: `source .venv/bin/activate` |

---

## DEPLOYMENT CHECKLIST

- [ ] All services have .env files with production values
- [ ] MongoDB Atlas IP whitelist includes server IPs
- [ ] Pinecone API key updated
- [ ] Google API key with Gemini enabled
- [ ] Polygon RPC endpoint accessible
- [ ] Neon PostgreSQL connection string updated
- [ ] Firebase credentials for production
- [ ] CORS origins updated to production domain
- [ ] SSL certificates installed
- [ ] Environment variables encrypted in secrets manager
- [ ] Database backups configured
- [ ] Monitoring/logging setup
- [ ] Smart contract deployed to Polygon
- [ ] All 5 services containerized with Docker
- [ ] Cloud Run/App Engine deployment configured

---

## METRICS TO MONITOR

| Metric | Where to Check | Healthy Value |
|--------|----------------|---------------|
| API Response Time | Backend logs | <200ms |
| MongoDB Connection | Backend startup logs | Connected ✅ |
| Pinecone Queries | API latency | <1s |
| Blockchain Confirmations | Polygon explorer | Latest block |
| Queue Processing | Chain Batcher logs | <30s per batch |
| Vector Search Accuracy | Test queries | >80% relevant |
| User Registrations | Firebase console | Growing |
| Error Rate | Logs | <1% |

---

## NEXT STEPS

1. **Understand**: Read [SYSTEM_OVERVIEW.md](./SYSTEM_OVERVIEW.md)
2. **Learn Security**: Read [FRONTEND_TO_DATABASE_GUIDE.md](./FRONTEND_TO_DATABASE_GUIDE.md)
3. **Deep Dive**: Read [PROJECT_ARCHITECTURE.md](./PROJECT_ARCHITECTURE.md)
4. **Get Setup**: Follow [SETUP_GUIDE.md](./SETUP_GUIDE.md)
5. **Start Coding**: Pick a service and explore the code
6. **Reference**: Use [DOCUMENTATION_INDEX.md](./DOCUMENTATION_INDEX.md)

---

## LEGEND

```
✅ Working / Healthy
❌ Not working / Error
⚠️  Warning / Needs attention
→ Process flow
├─ Sub-item
```

---

**Everything is running!** 🚀 Visit:
- 🛍️ Shop: http://localhost:4200
- 🔧 Master-IP: http://localhost:8000/docs
- 🤖 AI: http://localhost:8080
- 📦 Backend: http://localhost:8001

Good luck! 🎨⛓️🛍️
