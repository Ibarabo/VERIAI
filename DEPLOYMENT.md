# VeriAI Africa — Production Deployment Guide
**Intelligence Powering Africa**

This guide documents deploying the full VeriAI Africa stack (Next.js 16 frontend + Python 3.12 FastAPI backend + Neon Cloud PostgreSQL).

---

## 1. Quick Docker Compose (Self-Hosted / VPS)

To deploy both frontend and backend on any cloud VPS (AWS EC2, DigitalOcean, Hetzner, Linode):

```bash
# 1. Clone repository and verify .env
git clone <repo-url>
cd Veriai
cp .env.example .env # Ensure DATABASE_URL is configured to your Neon pooler

# 2. Build and launch containers
docker compose up -d --build

# 3. Check container logs
docker compose logs -f
```

- Frontend is available at `http://<your-server-ip>:3000`
- FastAPI backend documentation is available at `http://<your-server-ip>:8000/docs`

---

## 2. Serverless Cloud Architecture (Recommended for Enterprise)

### A. FastAPI Backend on Google Cloud Run / Render / Fly.io
1. **Google Cloud Run**:
   ```bash
   gcloud builds submit --tag gcr.io/<PROJECT-ID>/veriai-backend -f backend/Dockerfile
   gcloud run deploy veriai-backend \
     --image gcr.io/<PROJECT-ID>/veriai-backend \
     --platform managed \
     --region europe-west1 \
     --allow-unauthenticated \
     --set-env-vars DATABASE_URL="<your-neon-url>"
   ```
2. **Render / Fly.io**:
   - Set Root Directory to `.` and Dockerfile path to `backend/Dockerfile`.
   - Add environment variables: `DATABASE_URL`, `JWT_SECRET`, `PORT=8000`.

### B. Next.js Frontend on Vercel
1. Import the repository on [Vercel](https://vercel.com).
2. Set Environment Variables:
   - `NEXT_PUBLIC_BACKEND_URL`: URL of your deployed FastAPI backend (e.g., `https://veriai-backend-xxxx.a.run.app`).
   - `DATABASE_URL`: Your Neon PostgreSQL connection string.
3. Deploy! The rewrite rule in `next.config.ts` automatically proxies `/api/py/*` to your backend.

---

## 3. Database Migration & Security Checklist
- Ensure `DATABASE_URL` uses Neon's pooled endpoint (`ep-*-pooler.*.neon.tech`).
- Keep `sslmode=require` in database connection strings.
- Restrict `CORS_ORIGINS` in production to your verified domains.
