# 🚀 Complete Service Startup Guide

## Prerequisites
- Python 3.12+
- Node.js + npm
- All dependencies installed

---

## Step 1: Install Dependencies (First Time Only)

### Master-IP Backend
```bash
cd /workspaces/kalakaari-ai-solution/master-ip/server
pip install -r requirements.txt
```

### Shop Backend
```bash
cd /workspaces/kalakaari-ai-solution/shop/shop-backend
pip install -r requirements.txt
```

### Frontend
```bash
cd /workspaces/kalakaari-ai-solution/shop/shop-frontend
npm install
```

### Agentic Service
```bash
cd /workspaces/kalakaari-ai-solution/agentic
pip install -r requirements.txt
```

---

## Step 2: Start Services (Open 4 Terminals)

### Terminal 1: Master-IP Backend (Port 8000)
```bash
cd /workspaces/kalakaari-ai-solution/master-ip/server
python -m uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```
✅ Wait for: `INFO:     Uvicorn running on http://0.0.0.0:8000`

### Terminal 2: Shop Backend (Port 8001)
```bash
cd /workspaces/kalakaari-ai-solution/shop/shop-backend
python -m uvicorn app.main:app --host 0.0.0.0 --port 8001 --reload
```
✅ Wait for: `INFO:     Uvicorn running on http://0.0.0.0:8001`

### Terminal 3: Frontend (Port 4200)
```bash
cd /workspaces/kalakaari-ai-solution/shop/shop-frontend
ng serve --port 4200
```
✅ Wait for: `✔ Compiled successfully`

### Terminal 4: Agentic Service (Port 8080) [Optional for now]
```bash
cd /workspaces/kalakaari-ai-solution/agentic
python main.py
```
✅ Wait for: `Running on http://0.0.0.0:8080`

---

## Step 3: Verify Services Are Running

### Test Master-IP (Port 8000)
```bash
curl http://localhost:8000/
# Expected: {"message": "Prototype Master-IP backend is running!"}
```

### Test Shop Backend (Port 8001)
```bash
curl http://localhost:8001/
# Expected: {"message": "Shop backend is running!"}
```

### Test Frontend
Open browser: `http://localhost:4200`
✅ Should see Kalaakari login page

### Test Product Listing
```bash
curl http://localhost:8001/get-products
# Expected: [] (empty array, since no products yet)
```

---

## Step 4: Test the Flow

1. **Login as Art Lover**
   - Email: test@example.com
   - Password: Test@123456
   - Expected: See home page with "No artisan products found yet"

2. **Check Console for Errors**
   - Open Browser DevTools (F12)
   - Check Console tab
   - Check Network tab for failed requests

3. **If Products Don't Load**
   - Check Shop Backend logs (Terminal 2)
   - Check for connection errors to Master-IP
   - Verify MASTER_IP_URL is set in shop backend .env

---

## Troubleshooting

### Error: "Port 8000/8001 already in use"
```bash
# Find and kill process
lsof -i :8000
kill -9 <PID>
```

### Error: "Connection refused to localhost:8000"
- Master-IP backend not started in Terminal 1
- Or service crashed (check for error messages)

### Error: "MASTER_IP_URL not set"
- Check `/workspaces/kalakaari-ai-solution/shop/shop-backend/.env`
- Ensure it has: `MASTER_IP_URL=http://localhost:8000`

### Error: "Products failed to fetch"
- Check browser Network tab (F12)
- See which endpoint is failing
- Check backend logs for errors

---

## Quick Health Check

```bash
# All in one check
echo "=== Master-IP ===" && curl -s http://localhost:8000/ | jq . && \
echo "=== Shop Backend ===" && curl -s http://localhost:8001/ | jq . && \
echo "=== Get Products ===" && curl -s http://localhost:8001/get-products | jq .
```

---

## Environment Variables Check

**Shop Backend** (`.env` must have):
```
MONGO_URI=mongodb+srv://...
DB_NAME=kalaakari_shop_db
SECRET_KEY=Harin@2205
MASTER_IP_URL=http://localhost:8000
```

**Master-IP** (`.env` must have):
```
MONGO_URI=mongodb+srv://...
DB_NAME=masterip_db
PINECONE_API_KEY=pcsk_...
WEB3_RPC_URL=https://rpc-amoy.polygon.technology
ANCHOR_CONTRACT_ADDRESS=0x...
```
