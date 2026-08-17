
# 🔌 HOW FRONTEND CONNECTS TO DATABASES & APIs

## Quick Answer

The **frontend (Angular) never directly touches databases or secret API keys**. Instead:

```
Frontend (Angular)
    ↓ Makes HTTP request
Backend API (FastAPI)
    ↓ Backend reads .env file (has secrets)
Backend connects to: MongoDB, Pinecone, Blockchain
    ↓ Fetches data
Backend returns JSON to Frontend
    ↓ Frontend displays data
```

---

## Visual Example: Browsing Products

### ❌ Wrong Way (DON'T DO THIS)
```
Frontend has MongoDB connection string in code
Frontend connects directly to MongoDB
  ↓ Frontend code is visible in browser (inspect element)
  ↓ Anyone can see: mongodb://username:password@...
  ↓ SECURITY BREACH! 🚨
```

### ✅ Right Way (What We Do)
```
1. Frontend: "Give me products"
   ├─ Code: this.http.get('http://localhost:8001/get-products')
   └─ API calls Shop Backend on port 8001

2. Shop Backend: Receives request
   ├─ Code: @app.get("/get-products")
   ├─ Loads: MONGO_URI from .env file (secrets hidden on server)
   ├─ Connects: db = connect(MONGO_URI)
   ├─ Queries: products = db.products.find({})
   └─ Returns: products as JSON

3. Frontend: Receives JSON
   ├─ Frontend doesn't know MongoDB exists
   ├─ Frontend doesn't have any secrets
   ├─ Frontend displays products on page
   └─ ✅ SECURE!
```

---

## Code Walkthrough: Frontend to Backend to Database

### Frontend Code (Angular)
```typescript
// File: shop-frontend/src/app/services/api-client.service.ts

import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class ApiClientService {
  // ⚠️ Notice: No database connection, no secrets!
  private readonly shopApiUrl = 'http://localhost:8001';
  
  constructor(private http: HttpClient) { }

  // Method 1: Get all products
  getProducts(): Observable<Product[]> {
    // Just makes HTTP request to backend
    return this.http.get<Product[]>(
      `${this.shopApiUrl}/get-products`
    );
  }

  // Method 2: Verify CraftID on blockchain
  verifyCraftID(publicId: string): Observable<VerificationResponse> {
    // Just makes HTTP request to Master-IP backend
    return this.http.get<VerificationResponse>(
      `http://localhost:8000/verify/${publicId}`
    );
  }
}
```

### Backend Code (FastAPI)
```python
# File: shop-backend/app/main.py

from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from dotenv import load_dotenv
import os

from app.routes.products import router as products_router
from app.mongodb import connect_to_mongo

# ✅ Backend loads secrets from .env file
load_dotenv()
MONGO_URI = os.getenv('MONGO_URI')  # Hidden from frontend!

app = FastAPI()

# Add CORS to allow frontend to make requests
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:4200"],  # Only allow frontend
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.on_event("startup")
async def startup_event():
    # Connect to MongoDB on server startup
    await connect_to_mongo(MONGO_URI)

@app.include_router(products_router)

@app.get("/get-products")
async def root():
    return {"message": "Shop backend is running!"}
```

### Backend Database Connection
```python
# File: shop-backend/app/mongodb.py

from motor.motor_asyncio import AsyncClient, AsyncDatabase
import os

client: AsyncClient = None
db: AsyncDatabase = None

async def connect_to_mongo(mongo_uri: str):
    """Connect to MongoDB"""
    global client, db
    client = AsyncClient(mongo_uri)  # MONGO_URI from .env
    db = client.kalaakari_shop_db
    print("✅ Connected to MongoDB")

async def close_mongo_connection():
    """Close MongoDB connection"""
    global client
    if client:
        client.close()
        print("❌ Disconnected from MongoDB")

def get_database():
    """Get database instance"""
    return db
```

### Backend Route Handler
```python
# File: shop-backend/app/routes/products.py

from fastapi import APIRouter
from app.mongodb import get_database

router = APIRouter()

@router.get("/get-products")
async def get_products():
    """Fetch all products from MongoDB"""
    db = get_database()
    
    # Query MongoDB for products
    products_cursor = db.products.find({})
    products = []
    
    async for product in products_cursor:
        # Convert MongoDB _id to string
        product['_id'] = str(product['_id'])
        products.append(product)
    
    return products
```

### Data Returned to Frontend
```json
{
  "products": [
    {
      "_id": "507f1f77bcf86cd799439011",
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
      "stock": 3
    }
  ]
}
```

### Frontend Displays Data
```typescript
// File: shop-frontend/src/app/components/home/home.component.ts

import { Component, OnInit } from '@angular/core';
import { ApiClientService, Product } from '../../services/api-client.service';

@Component({
  selector: 'app-home',
  templateUrl: './home.component.html',
  styleUrls: ['./home.component.scss']
})
export class HomeComponent implements OnInit {
  products: Product[] = [];

  constructor(private apiClient: ApiClientService) {}

  ngOnInit(): void {
    this.loadProducts();
  }

  loadProducts(): void {
    // Call backend API
    this.apiClient.getProducts().subscribe(
      (data: Product[]) => {
        this.products = data;
        // Frontend now has products array
        // Renders in HTML template
      },
      (error) => {
        console.error('Error loading products:', error);
      }
    );
  }

  verifyProduct(publicId: string): void {
    this.apiClient.verifyCraftID(publicId).subscribe(
      (verification) => {
        console.log('Verification result:', verification);
        // Display: ✅ VERIFIED or ❌ TAMPERED
      }
    );
  }
}
```

### Frontend HTML Template
```html
<!-- File: shop-frontend/src/app/components/home/home.component.html -->

<div class="products-container">
  <div *ngFor="let product of products" class="product-card">
    
    <!-- Artisan Info (from MongoDB) -->
    <h3>{{ product.artisan_info.name }}</h3>
    <p>{{ product.artisan_info.location }}</p>
    
    <!-- Art Info (from MongoDB) -->
    <img [src]="product.art_info.photo" alt="{{ product.art_info.name }}">
    <h4>{{ product.art_info.name }}</h4>
    <p>{{ product.art_info.description }}</p>
    
    <!-- Verification (links to blockchain) -->
    <a href="{{ product.verification.verification_url }}" target="_blank">
      🔗 Verify on Blockchain
    </a>
    
    <!-- Price & Buy Button -->
    <p class="price">₹{{ product.price }}</p>
    <button (click)="addToCart(product)">Add to Cart</button>
    
  </div>
</div>
```

---

## 🔄 Complete Data Flow Diagram

```
┌─────────────────────────────────────────────────────────┐
│ 1. BROWSER (Frontend - Angular)                         │
│                                                          │
│ User clicks: "Load Products"                             │
│                                                          │
│ Code executes:                                           │
│   this.apiClient.getProducts()                          │
│                                                          │
│ Creates HTTP GET request:                                │
│   GET http://localhost:8001/get-products                │
│                                                          │
│ ⚠️ No database connection                               │
│ ⚠️ No API keys in frontend code                         │
│ ⚠️ No secrets visible                                   │
└────────────────────┬────────────────────────────────────┘
                     │
                     │ HTTP Request
                     │ (travels over network)
                     ↓
┌─────────────────────────────────────────────────────────┐
│ 2. SERVER (Backend - FastAPI)                           │
│    Port: 8001                                            │
│                                                          │
│ Receives: GET /get-products                              │
│                                                          │
│ Backend loads from .env file:                            │
│   MONGO_URI=mongodb+srv://...                           │
│   (secrets hidden on server!)                            │
│                                                          │
│ Code executes:                                           │
│   db = connect(MONGO_URI)                               │
│   products = db.products.find({})                       │
│                                                          │
│ Creates JSON response:                                   │
│   [                                                      │
│     {artisan_info, art_info, verification},             │
│     {artisan_info, art_info, verification}              │
│   ]                                                      │
└────────────────────┬────────────────────────────────────┘
                     │
        ┌────────────▼──────────┐
        │ MONGODB ATLAS         │
        │                       │
        │ Database: kalaakari_  │
        │ shop_db               │
        │ Collection: products  │
        │                       │
        │ ✅ Stores real data   │
        │ ✅ Secure in cloud    │
        │ ✅ Backed up          │
        └──────────────────────┘
                     │
                     │ Database returns data
                     │ Backend formats as JSON
                     ↓
┌─────────────────────────────────────────────────────────┐
│ 3. SERVER → BROWSER (HTTP Response)                     │
│                                                          │
│ HTTP 200 OK                                              │
│ Content-Type: application/json                           │
│                                                          │
│ Body:                                                    │
│ [                                                        │
│   {                                                      │
│     "_id": "507f1f77bcf86cd799439011",                  │
│     "artisan_info": {                                    │
│       "name": "Harin Savaliya",                          │
│       "location": "Rajasthan, India"                     │
│     },                                                   │
│     "art_info": {                                        │
│       "name": "Indigo Textile",                          │
│       "description": "Hand-dyed...",                     │
│       "photo": "https://..."                             │
│     },                                                   │
│     "verification": {                                    │
│       "public_id": "CID-00123"                           │
│     }                                                    │
│   }                                                      │
│ ]                                                        │
└────────────────────┬────────────────────────────────────┘
                     │
                     │ HTTP Response
                     │ (travels over network)
                     ↓
┌─────────────────────────────────────────────────────────┐
│ 4. BROWSER (Frontend - Angular)                         │
│                                                          │
│ Receives JSON in subscribe():                            │
│   .subscribe((data: Product[]) => {                      │
│     this.products = data;  // Store in component         │
│   })                                                    │
│                                                          │
│ Angular renders HTML:                                    │
│   *ngFor="let product of products"                      │
│                                                          │
│ Displays:                                                │
│   [Product Card 1]                                      │
│   ├─ Name: Harin Savaliya                               │
│   ├─ Location: Rajasthan, India                         │
│   ├─ Art: Indigo Textile                                │
│   ├─ Photo: [image]                                     │
│   └─ Price: ₹5000 [BUY]                                 │
│                                                          │
│   [Product Card 2]                                      │
│   └─ ...                                                │
│                                                          │
│ ✅ User sees products!                                  │
└─────────────────────────────────────────────────────────┘
```

---

## 🔐 Security: Why This Matters

### With Direct Connection (INSECURE ❌)
```
Frontend has MongoDB credentials
  ↓
User opens browser
  ↓
Right-click → Inspect Element
  ↓
See MongoDB connection string with password
  ↓
Attacker steals: mongodb://user:PASSWORD@cluster...
  ↓
Attacker can:
  - Delete all products
  - Modify prices
  - Steal customer data
  - Destroy database
```

### With Backend API (SECURE ✅)
```
Frontend has NO credentials
  ↓
User opens browser
  ↓
Right-click → Inspect Element
  ↓
See only: GET http://localhost:8001/get-products
  ↓
Attacker cannot find any credentials
  ↓
Credentials safe on server in .env file
  ↓
Server validates every request
  ✓ Only authorized operations allowed
```

---

## 📡 Multiple Backend Connections

### How All 3 Backends Connect to Different Databases

```
FRONTEND (One Connection)
├─ Connects to: localhost:8001 (Shop Backend)
└─ Shop Backend:
   └─ Connects to: MongoDB (products, orders)

├─ Connects to: localhost:8000 (Master-IP Backend)
└─ Master-IP Backend:
   ├─ Connects to: MongoDB (CraftIDs, queue)
   ├─ Connects to: Pinecone (vector search)
   └─ Connects to: Polygon Blockchain

├─ Connects to: localhost:8080 (Agentic Service)
└─ Agentic Service:
   ├─ Connects to: Google Gemini API
   └─ Connects to: Neon PostgreSQL (sessions)
```

### Each Backend Has Own .env File

```
master-ip/server/.env
├─ MONGO_URI (connects to MongoDB masterip_db)
├─ PINECONE_API_KEY (connects to Pinecone)
└─ WEB3_RPC_URL (connects to Polygon)

shop/shop-backend/.env
├─ MONGO_URI (connects to MongoDB kalaakari_shop_db)
└─ FIREBASE_CREDENTIALS (Firebase auth)

agentic/.env
├─ GOOGLE_API_KEY (Google Gemini)
└─ SESSION_SERVICE_URI (Neon PostgreSQL)
```

---

## 🔄 Verification Flow (More Complex Example)

### Frontend Clicks "Verify"

```
1. Frontend (Angular)
   └─ Button click: "Verify CID-00123"
   └─ Code: this.apiClient.verifyCraftID('CID-00123')
   └─ HTTP: GET http://localhost:8000/verify/CID-00123

2. Master-IP Backend (Port 8000)
   ├─ Loads from .env:
   │  ├─ MONGO_URI (MongoDB connection)
   │  ├─ WEB3_RPC_URL (Polygon connection)
   │  └─ ANCHORER_PRIVATE_KEY (signing key)
   │
   ├─ Connects to MongoDB:
   │  └─ Queries: db.craftids.findOne({public_id: 'CID-00123'})
   │  └─ Gets: stored_hash, tx_hash, metadata
   │
   ├─ Connects to Polygon Blockchain:
   │  ├─ Gets transaction: web3.getTransaction(tx_hash)
   │  ├─ Verifies signature
   │  └─ Confirms timestamp
   │
   ├─ Computes new hash from metadata
   │
   ├─ Compares: stored_hash == computed_hash?
   │  ├─ If equal: is_tampered = false ✅
   │  └─ If different: is_tampered = true ❌
   │
   └─ Returns JSON:
      {
        public_id: 'CID-00123',
        blockchain_verified: true,
        is_tampered: false,
        tx_hash: '0x1234...',
        blockchain_timestamp: '2025-08-01T10:35Z'
      }

3. Frontend (Angular)
   ├─ Receives JSON in subscribe()
   ├─ Checks: blockchain_verified == true? ✅
   ├─ Checks: is_tampered == false? ✅
   └─ Displays: "✅ VERIFIED - Authentic Artwork"
```

---

## ✨ Summary

| Component | Has Secrets? | Connected To | Purpose |
|-----------|--------------|--------------|---------|
| **Frontend (Angular)** | ❌ NO | Backend APIs only | User interface |
| **Shop Backend** | ✅ YES (.env) | MongoDB | Products & orders |
| **Master-IP Backend** | ✅ YES (.env) | MongoDB, Pinecone, Blockchain | CraftID & verification |
| **Agentic Service** | ✅ YES (.env) | Google Gemini, Neon PostgreSQL | AI conversations |
| **Chain Batcher** | ✅ YES (.env) | MongoDB, Polygon Blockchain | Blockchain anchoring |

**Frontend never knows about databases or secrets!** 🔐

---

## 🎓 Key Takeaway

**Architecture Pattern: API Gateway**

```
Client (Browser)
    ↓
Backend API (Gateway)  ← All secrets here!
    ├─ Database
    ├─ External APIs
    ├─ Blockchain
    └─ Other services
```

This is industry standard for:
- ✅ Security (secrets hidden)
- ✅ Scalability (cache at backend)
- ✅ Flexibility (change backends without frontend changes)
- ✅ Maintainability (one place to update credentials)

Your Kalakaari platform follows this pattern perfectly! 🚀
