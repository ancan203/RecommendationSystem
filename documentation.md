# 🧠 Customall AI Recommendation System

A microservice-based backend application that generates and serves product recommendations using vector embeddings from Amazon Bedrock. Supports multiple merchants, product management, and fast similarity search via Annoy.

---

## 📦 Project Structure

### `serve/` — Recommendation API

Hosts endpoints for serving vector-based product recommendations.

```
serve/
├── Dockerfile
├── main.py              # Exposes /similarity endpoints (single & batch), index refresher
├── helper_log.py        # Logger setup
├── requirements.txt     # FastAPI, boto3, pymongo, annoy
```

### `updater/` — Admin & Index Management

Handles merchant/product management, embedding generation, and index building.

```
updater/
├── Dockerfile
├── main.py                   # Admin endpoints: merchant/product CRUD, model rebuild
├── helper_aws.py             # Bedrock API + S3 utilities
├── helper_documentdb.py      # MongoDB interactions and token auth
├── helper_log.py             # Shared logger config
├── helper_models.py          # Pydantic models for request validation
├── requirements.txt
```

---

## 🧾 System Overview

- **MongoDB** stores merchant and product data with per-merchant isolation.
- **Amazon Bedrock** generates 384-dimensional embeddings using Titan model.
- **Annoy** provides efficient vector similarity search.
- **S3** stores Annoy index files (`.ann`, `.json`).
- **FastAPI** powers REST APIs for both recommendation and management tasks.

---

## 📡 Key API Endpoints

### From `serve/main.py`

- `GET /similarity/single`: Get similar products to a single product ID.
- `GET /similarity/batch`: Recommend products based on a list of product IDs.
- `POST /update_index`: Refresh in-memory Annoy index from S3.

### From `updater/main.py`

- `POST /merchant`: Create new merchant with token.
- `DELETE /merchant/{merchant_id}`: Remove merchant and data.
- `POST /merchant/{merchant_id}/product`: Add product with embeddings.
- `PUT /merchant/{merchant_id}/model`: Rebuild Annoy index and upload to S3.

More routes available for product updates, retrieval, batch upload, and configuration.

---

## 🔐 Authentication

- Admin and merchant tokens stored in MongoDB.
- Role-based access enforced via HTTP Bearer tokens.
- Auth handled in `helper_documentdb.py`.

---

## 🚀 Deployment (Docker Compose)

Run the full stack:

```bash
docker-compose up --build
```

Services:
- `mongo`: MongoDB server
- `updater`: Admin + index management service (port 8000)
- `serve-tmz`: Client API for merchant `tmz` (port 8001)
- `serve-tmz-image`: Image-based variant (port 8002)

---

## 📌 Future Improvements

- Add `last_updated` field to support partial index rebuilds
- Use metadata (e.g., price, number of reviews) to rerank results
- Optimize Bedrock usage when combining multiple embedding sources