🚀 3-Tier DevSecOps Mega Project
Docker · Nginx · Node.js · MySQL · Production-Style Architecture
📌 Project Overview

This project is a real-world 3-Tier DevSecOps application built using Docker Compose, demonstrating how modern web applications are deployed in production environments.

The project follows industry-standard separation of concerns:

Frontend (Presentation Tier) served by Nginx

Backend (Application Tier) built with Node.js

Database (Data Tier) using MySQL

Secure internal networking

Production-ready configuration & debugging

This repository is not just about making things work, but about understanding how things break and how to fix them.

🧱 3-Tier Architecture (Logical View)
Client (Browser)
      |
      v
Nginx (Frontend / Reverse Proxy)
      |
      v
Node.js Backend (API Layer)
      |
      v
MySQL Database (Persistent Storage)

Important Note

Nginx is not a separate tier — it is part of the frontend/presentation tier, acting as:

Static file server

Reverse proxy

Single entry point for the system

🐳 Dockerized Architecture
Docker Network (bridge)
├── frontend (Nginx + React build)
├── backend  (Node.js API)
└── mysql    (MySQL Database)


Only frontend is exposed to the outside world.

📁 Project Structure
3-Tier-DevSecOps-Mega-Project/
│
├── frontend/
│   ├── Dockerfile
│   ├── nginx/
│   │   └── default.conf
│   └── build/                # React production build
│
├── backend/
│   ├── Dockerfile
│   ├── app.js
│   ├── routes/
│   └── waitForDb.js
│
├── database/
│   └── init.sql
│
├── docker-compose.yml
└── README.md

⚙️ Key Concepts Implemented
✅ Docker Compose Networking

Containers communicate using service names

No hard-coded IPs

Example:

proxy_pass http://backend:5000;

✅ Persistent Storage (MySQL Volume)
mysql:
  volumes:
    - mysql_data:/var/lib/mysql


Why?

Prevents data loss on container restart

Ensures registered users remain intact

Avoids DB reset on every deployment

Problem Faced
Initially, database data was lost after restart.

Solution
Added named Docker volume and avoided docker-compose down -v.

🧪 Database Initialization (init.sql)
Problem Faced

Database tables were not created on first run

Backend failed because tables didn’t exist

Solution

Used Docker’s built-in MySQL init mechanism:

/docker-entrypoint-initdb.d/init.sql


This script runs only on first container creation, not on restarts.

🔐 Admin Seeding Logic (Idempotent)
Problem Faced

Only admin user was present even after multiple registrations.

Root Cause

Admin seeding logic ran on every startup.

Final Solution

Implemented idempotent seeding:

SELECT * FROM users WHERE email = ?


Admin is created only if not present

No overwriting

No interference with user registration

🔄 Backend Startup Order (Critical)
Problem Faced

Backend tried connecting to MySQL before DB was ready.

Solution

Implemented waitForDb() logic:

await waitForDb();
await seedAdminUser();
app.listen(PORT);


This ensured:

Database readiness

Clean startup

No race conditions

🧠 Build-time vs Runtime Environment Variables
Problem Faced

Environment variables were not available inside containers.

Understanding the Difference
Type	Used When
ARG	Build-time
ENV	Runtime
Solution

Used ARG inside Dockerfile:

ARG NODE_ENV


And ENV for runtime configuration:

ENV NODE_ENV=production


This ensured clean separation of:

Build configuration

Runtime secrets

🧊 Alpine Image Issues (curl missing)
Problem Faced

curl command failed inside Alpine-based container.

Root Cause

Alpine Linux is minimal by default.

Solution

Explicitly installed curl:

RUN apk add --no-cache curl

🌐 Nginx Configuration Deep Dive
Static Files Serving
root /usr/share/nginx/html;
index index.html;


Serves React production build

No backend involvement

API Proxying
location /api/ {
  proxy_pass http://backend:5000/;
}


/api/* forwarded to backend

Backend remains hidden

No CORS issues

SPA Routing Fix (React Router)
Problem Faced

Refreshing /dashboard returned 404.

Solution

Fallback routing:

location / {
  try_files $uri /index.html;
}


Meaning

Nginx serves one HTML file

React Router handles the URL

🔐 Security & DevSecOps Practices

Backend & DB not exposed

Single entry point (Nginx)

Password hashing using bcrypt

Environment variables for secrets

Internal Docker networking

No hard-coded credentials

🧪 Debugging Techniques Used

docker logs

docker exec

curl for API testing

MySQL direct inspection

Network tab debugging

Step-by-step isolation of issues

🎯 Key Learnings

3-Tier architecture is logical, not container-count based

Infrastructure can be correct while application logic fails

Startup order matters in distributed systems

Seeding must always be idempotent

Build-time and runtime configs must be separated

Nginx is a gatekeeper, not a backend

🧠 Interview-Ready Summary

This project demonstrates real-world DevOps problem-solving — not just deployment, but understanding why things fail and how to fix them systematically.

🚀 How to Run
docker-compose up -d


Access application:

http://<server-ip>

📌 Future Enhancements

HTTPS with Let’s Encrypt

CI/CD pipeline

Kubernetes deployment

Secrets management

Rate limiting & WAF

Monitoring & logging

👤 Author

DevOpsHasnain
Aspiring DevOps Engineer | Cloud | Docker | Kubernetes | Linux
