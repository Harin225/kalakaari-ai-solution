# 🚀 Complete Setup Guide for Kalakaari Project

This guide takes you from **zero to running** the entire project. Follow each step carefully.

---

## 📋 Step 1: Create All Required Accounts

### 1️⃣ MongoDB Atlas (Database)
**Purpose:** Store CraftIDs, products, and user data

1. Go to: https://www.mongodb.com/cloud/atlas
2. Click **Sign Up** (free account)
3. Fill in your details and verify email
4. Create a free **M0 Cluster**
5. Go to **Database Access** → Add username/password
6. Go to **Network Access** → Add IP `0.0.0.0/0` (for development)
7. Click **Connect** → Choose **Drivers** → Select **Python 3.6 and later**
8. Copy the connection string (looks like):
   ```
   mongodb+srv://USERNAME:PASSWORD@cluster.mongodb.net/
   ```
9. **Save this** - you'll need it later

---

### 2️⃣ Pinecone (Vector Search Database)
**Purpose:** AI-powered image and text search

1. Go to: https://www.pinecone.io
2. Click **Sign Up** (free account)
3. Verify email
4. Create a **free index** with these settings:
   - **Name:** `craft-images` (for images)
   - **Dimension:** `512` (for CLIP embeddings)
   - **Metric:** `cosine`
5. Create another index:
   - **Name:** `craft-text`
   - **Dimension:** `384` (for text embeddings)
   - **Metric:** `cosine`
6. Go to **API Keys** section
7. **Save these values:**
   - `PINECONE_API_KEY` (the key itself)
   - `PINECONE_ENV` (region, e.g., `us-east-1`)
   - `INDEX_HOST` (shown in index details, e.g., `craft-images-xxxxx.svc.pinecone.io`)

---

### 3️⃣ Neon PostgreSQL (Agent Database)
**Purpose:** Store AI agent session data

1. Go to: https://neon.tech
2. Click **Sign Up** (free account)
3. Verify email
4. Create a **new project**
5. Copy the database connection string (looks like):
   ```
   postgresql://user:password@host/dbname?sslmode=require
   ```
6. **Save this** - you'll need it for the agentic service

---

### 4️⃣ Google API Key (Gemini AI)
**Purpose:** Power the conversational AI agent

1. Go to: https://aistudio.google.com/app/apikeys
2. Click **Create API Key**
3. Select your project (or create a new one)
4. Copy the API key
5. **Save this** - keep it secret!

---

### 5️⃣ Polygon Amoy Testnet Wallet
**Purpose:** Blockchain anchoring for artwork provenance

1. Install **MetaMask** browser extension: https://metamask.io
2. Create a new wallet (or import existing)
3. Add Polygon Amoy network:
   - Open MetaMask → Settings → Networks → Add Network
   - Network name: `Polygon Amoy`
   - RPC URL: `https://rpc-amoy.polygon.technology`
   - Chain ID: `80002`
   - Currency symbol: `Mi haveATIC`
   - Block explorer: `https://amoy.polygonscan.com`
4. Get test MATIC tokens from faucet:
   - Go to: https://faucet.polygon.technology
   - Select `Polygon Amoy`
   - Paste your wallet address
   - Click send (you get 0.5 MATIC)
5. **Save your private key** (MetaMask → Account Details → Show Private Key)
   - ⚠️ **NEVER share this publicly**

---

## 🛠️ Step 2: Project Setup (Your Computer)

### Prerequisites Check
Run these commands in terminal to verify you have everything:

```bash
# Check Python
python3 --version  # Should be 3.10 or higher

# Check Node.js
node --version     # Should be 18 or higher
npm --version      # Should be 9 or higher

# Check Git
git --version      # Should be installed
```

If any are missing, install them:
- **Python:** https://www.python.org/downloads
- **Node.js:** https://nodejs.org
- **Git:** https://git-scm.com

### Clone the Project
You already did this, but if not:
```bash
git clone https://github.com/Harin225/kalakaari-ai-solution.git
cd kalakaari-ai-solution
```

---

## ⚙️ Step 3: Setup Environment Variables

### 3.1 Setup Master-IP Service (Blockchain Backend)

Navigate to the master-ip directory:
```bash
cd master-ip/server
```

**Generate cryptographic keys (one-time only):**
```bash
mkdir -p chain/keys
openssl ecparam -name prime256v1 -genkey -noout -out chain/keys/sign_priv.pem
openssl ec -in chain/keys/sign_priv.pem -pubout -out chain/keys/sign_pub.pem
```

Get the full path to your keys:
```bash
pwd
```
This will print something like: `/your/path/to/kalakaari-ai-solution/master-ip/server`
**Save this path** - you'll use it below.

**Create `.env` file** in `master-ip/server/`:
```bash
touch .env
```

Open `.env` with your editor and paste:
```env
# MongoDB
MONGO_URI=mongodb+srv://USERNAME:PASSWORD@cluster.mongodb.net/
DB_NAME=masterip_db

# JWT Secret (create a random strong string)
SECRET_KEY=your_super_strong_random_secret_key_12345

# Pinecone
PINECONE_API_KEY=your_pinecone_api_key
PINECONE_ENV=us-east-1
INDEX_HOST=craft-images-xxxxx.svc.pinecone.io
PINECONE_TEXT_INDEX=craft-text

# Local Key Paths (IMPORTANT: Use the full path from pwd command above)
SIGNER_KEY_PATH=/your/full/path/kalakaari-ai-solution/master-ip/server/chain/keys/sign_priv.pem
PLATFORM_PUBKEY_PATH=/your/full/path/kalakaari-ai-solution/master-ip/server/chain/keys/sign_pub.pem

# Blockchain Configuration
WEB3_RPC_URL=https://rpc-amoy.polygon.technology
CHAIN_ID=80002
ANCHOR_CONTRACT_ADDRESS=0x0000000000000000000000000000000000000000
ANCHORER_PRIVATE_KEY=your_polygon_wallet_private_key_from_metamask

# Queue Settings
ANCHOR_QUEUE_COLL=anchor_queue
QUEUE_FETCH_MAX=5
BATCH_LIMIT=5
POLL_INTERVAL=10

# Optional
GCS_BUCKET_NAME=your-gcs-bucket-name
```

**⚠️ Important replacements:**
- `USERNAME:PASSWORD` → From MongoDB Atlas connection string
- `your_pinecone_api_key` → From Pinecone API Keys section
- `us-east-1` → Your Pinecone environment region
- `craft-images-xxxxx.svc.pinecone.io` → Your Pinecone index host
- `/your/full/path/...` → Use the full path from `pwd` command
- `your_polygon_wallet_private_key_from_metamask` → From MetaMask Account Details

---

### 3.2 Setup Agentic Service

Navigate back and go to agentic directory:
```bash
cd ../../agentic
```

**Create `.env` file:**
```bash
touch .env
```

Open `.env` with your editor and paste:
```env
# Database
SESSION_SERVICE_URI=postgresql://user:password@host/dbname?sslmode=require
DATABASE_URL=postgresql://user:password@host/dbname?sslmode=require

# Google AI
MODEL_NAME=gemini-2.0-flash
GOOGLE_API_KEY=your_google_api_key
GOOGLE_GENAI_USE_VERTEXAI=FALSE
```

**Replace:**
- `postgresql://user:password@...` → From Neon PostgreSQL connection string
- `your_google_api_key` → From Google AI Studio

---

### 3.3 Setup Shop Service Backend

Navigate to shop backend:
```bash
cd ../shop/shop-backend
```

**Create `.env` file:**
```bash
touch .env
```

Open `.env` with your editor and paste:
```env
MONGO_URI=mongodb+srv://USERNAME:PASSWORD@cluster.mongodb.net/
DB_NAME=kalaakari_shop_db
SECRET_KEY=your_super_strong_random_secret_key_12345
```

**Replace:**
- `USERNAME:PASSWORD@cluster` → From MongoDB Atlas
- `SECRET_KEY` → Any strong random string

---

## 🔧 Step 4: Install Dependencies

### Master-IP Service
```bash
cd ../../master-ip/server

# Create virtual environment
python3 -m venv .venv

# Activate it
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install dependencies
pip install --upgrade pip
pip install -r requirements.txt
```

### Agentic Service
```bash
cd ../../agentic

# Create virtual environment
python3 -m venv .venv

# Activate it
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install dependencies
pip install --upgrade pip
pip install -r requirements.txt
```

### Shop Backend
```bash
cd ../shop/shop-backend

# Create virtual environment
python3 -m venv venv

# Activate it
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install --upgrade pip
pip install -r requirements.txt
```

### Shop Frontend
```bash
cd ../shop-frontend

# Install Node.js dependencies
npm install
```

---

## 🚀 Step 5: Running the Project

You need **4 terminal windows**. Open 4 separate terminals and follow below:

### Terminal 1: Master-IP Web Server
```bash
cd master-ip/server
source .venv/bin/activate
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

✅ You should see:
```
Uvicorn running on http://0.0.0.0:8000
```

Access API docs: http://localhost:8000/docs

---

### Terminal 2: Master-IP Chain Batcher (Blockchain Worker)
```bash
cd master-ip/server
source .venv/bin/activate
python -m chain.batcher
```

This processes blockchain transactions in the background.

---

### Terminal 3: Agentic Service
```bash
cd agentic


uvicorn main:app --reload --host 0.0.0.0 --port 8080
```

✅ You should see:
```
Uvicorn running on http://0.0.0.0:8080
```

---

### Terminal 4: Shop Backend + Frontend

**In the 4th terminal, first start backend:**
```bash
cd shop/shop-backend
source venv/bin/activate
uvicorn app.main:app --reload --port 8001
```

✅ You should see:
```
Uvicorn running on http://0.0.0.0:8001
```

**Then in a 5th terminal, start frontend:**
```bash
cd shop/shop-frontend
npm start
```

✅ Frontend will open at: http://localhost:4200

---

## 📍 Access Your Running Services

| Service | URL | Purpose |
|---------|-----|---------|
| **Master-IP API Docs** | http://localhost:8000/docs | Create/verify CraftIDs, search |
| **Agentic Agent** | http://localhost:8080 | AI conversations |
| **Shop Backend** | http://localhost:8001 | Product API |
| **Shop Frontend** | http://localhost:4200 | Web interface |

---

## ✅ Quick Testing

### Test Master-IP Service
1. Open http://localhost:8000/docs in browser
2. Try the `/create` endpoint to create a CraftID
3. Check MongoDB Atlas - you should see data

### Test Agentic Service
1. Open http://localhost:8080 in browser
2. Start a conversation

### Test Shop Frontend
1. Open http://localhost:4200 in browser
2. You should see the shop interface

---

## 🔍 Troubleshooting

### "Connection refused" errors
- Check if the service is running in its terminal
- Check the port (8000, 8080, 8001, 4200)
- Make sure all .env files are created with correct values

### "API key invalid" errors
- Double-check your Google API Key in `.env`
- Make sure it's not expired

### "MongoDB connection failed"
- Verify MongoDB username/password in `.env`
- Check if your IP is whitelisted in MongoDB Atlas (should be 0.0.0.0/0)
- Test connection string in MongoDB Compass

### "Pinecone connection failed"
- Verify API key and environment
- Make sure index names match in code

### Python module not found errors
- Make sure virtual environment is activated
- Run `pip install -r requirements.txt` again

---

## 📚 Next Steps

Once everything is running:
1. Create a new CraftID via Master-IP API
2. Chat with the AI agent via Agentic service
3. Browse products in the Shop frontend
4. Anchor your artwork to blockchain

Enjoy! 🎨
