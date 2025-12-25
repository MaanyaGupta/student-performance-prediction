# Deploying ML Flask App on AWS EC2 with Docker

## 🎯 Project Overview

**Application:** Student Performance Prediction  
**Tech Stack:** Flask (Python) + Machine Learning (CatBoost/XGBoost)  
**Deployment:** AWS EC2 (Free Tier) + Docker  

---

## 📋 What We Did (Step-by-Step)

### 1. Analyzed the Project Structure

The project is a Flask web application with:
- `app.py` — Main Flask application serving predictions
- `src/` — ML pipeline code for data processing and prediction
- `artifacts/` — Trained ML model (`model.pkl`) and preprocessor (`preprocessor.pkl`)
- `templates/` — HTML templates for the web interface

---

### 2. Created Docker Configuration Files

#### **Dockerfile**
```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 5000
ENV FLASK_APP=app.py
ENV FLASK_ENV=production
CMD ["python", "app.py"]
```

**What it does:**
- Uses Python 3.9 slim image (lightweight)
- Copies and installs dependencies first (Docker layer caching)
- Copies application code
- Exposes port 5000 for Flask
- Runs the Flask application

#### **.dockerignore**
Excludes unnecessary files from Docker build:
- `venv/` (virtual environment)
- `notebook/` (Jupyter notebooks)
- `catboost_info/` (training artifacts)
- `__pycache__/` (Python cache)
- `.git/` (version control)

#### **Updated requirements.txt**
- Removed `-e .` (editable install, breaks in Docker)
- Added `gunicorn` (production WSGI server)

---

### 3. AWS EC2 Setup (Free Tier)

**Instance Configuration:**
| Setting | Value |
|---------|-------|
| AMI | Ubuntu (Free tier eligible) |
| Instance Type | t2.micro (750 hrs/month free) |
| Key Pair | Created new .pem file for SSH |

---

### 4. Configured Security Group (Firewall)

**Inbound Rules Added:**
| Port | Purpose |
|------|---------|
| 22 | SSH access |
| 80 | HTTP (optional) |
| **5000** | Flask application |

> This was the key fix when the site was unreachable — port 5000 wasn't open initially.

---

### 5. Installed Docker on EC2

Connected via SSH and ran:
```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER
```

---

### 6. Deployed the Application

```bash
# Cloned repository
git clone <repo-url>
cd <repo-name>

# Built Docker image
docker build -t student-app .

# Ran container in detached mode
docker run -d -p 5000:5000 --name student-app student-app
```

---

### 7. Accessed the Live Application

**URL:** `http://<EC2-Public-IP>:5000`

---

## 🏗️ Architecture Diagram

```
┌─────────────────┐     HTTP:5000     ┌─────────────────────────────────┐
│                 │ ───────────────▶  │          AWS EC2 (t2.micro)    │
│   User Browser  │                   │  ┌─────────────────────────┐   │
│                 │ ◀───────────────  │  │     Docker Container    │   │
└─────────────────┘    HTML Response  │  │  ┌───────────────────┐  │   │
                                      │  │  │   Flask App       │  │   │
                                      │  │  │   (app.py)        │  │   │
                                      │  │  │        │          │  │   │
                                      │  │  │   ML Pipeline     │  │   │
                                      │  │  │   (model.pkl)     │  │   │
                                      │  │  └───────────────────┘  │   │
                                      │  └─────────────────────────┘   │
                                      │                                │
                                      │  Security Group: Port 5000 ✓   │
                                      └─────────────────────────────────┘
```

---

## 🔑 Key Concepts for Interview

### Why Docker?
- **Consistency:** Same environment across dev/prod
- **Isolation:** App runs in its own container
- **Portability:** Works anywhere Docker is installed
- **Easy Deployment:** Single command to build and run

### Why EC2?
- **Full Control:** Complete access over the server
- **Free Tier:** 750 hours/month for 12 months
- **Flexible:** Can install any software (Docker, Git, etc.)

### Security Group Importance
- Acts as a virtual firewall for EC2
- Must explicitly allow inbound traffic on specific ports
- Without port 5000 open, external users can't access the app

### Docker Commands Used
| Command | Purpose |
|---------|---------|
| `docker build -t name .` | Build image from Dockerfile |
| `docker run -d -p 5000:5000` | Run container, map ports |
| `docker ps` | List running containers |
| `docker logs <name>` | View container logs |

---

## 📁 Files Created/Modified

| File | Action | Purpose |
|------|--------|---------|
| `Dockerfile` | Created | Container configuration |
| `.dockerignore` | Created | Exclude files from build |
| `requirements.txt` | Modified | Production dependencies |

---

## ✅ Final Result

Successfully deployed a Machine Learning Flask application to AWS cloud using Docker containerization, accessible via public IP on port 5000.
