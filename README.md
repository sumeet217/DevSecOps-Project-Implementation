# Jerney - Blog Platform

A modern blog platform built with a 3-tier architecture: React frontend, Node.js backend, and PostgreSQL database.

**Tech Stack:** React 18 | Node.js 20 | PostgreSQL 16

---

## Overview

Jerney is a full-featured blog application demonstrating cloud-native development practices. It supports creating, editing, and deleting blog posts with a comment system and modern web interface. The project includes multiple deployment options from simple EC2 setup to Kubernetes orchestration with DevSecOps practices.

## Features

- Create and publish blog posts
- Edit existing posts
- Delete posts
- Comment on posts with threaded discussions
- Modern dark UI with responsive design
- Health check and API monitoring endpoints

## Technology Stack

**Frontend:**
- React 18 with Vite
- React Router for client-side navigation
- Axios for HTTP requests
- React Icons and React Hot Toast for UI components
- Date-fns for date formatting

**Backend:**
- Node.js 20 with Express.js
- PostgreSQL 16 database
- CORS enabled for cross-origin requests
- dotenv for environment configuration

**Infrastructure:**
- Nginx reverse proxy
- Docker containerization
- Kubernetes (EKS Auto Mode) orchestration
- Terraform for infrastructure-as-code
- GitHub Actions CI/CD pipeline

## Architecture

```
Frontend (React + Nginx)  ---->  Backend (Node.js + Express)  ---->  PostgreSQL
Port 80                         Port 5000                           Port 5432
```

The application follows a traditional 3-tier architecture with clear separation between presentation, business logic, and data layers.

## Project Structure

```
Jerney/
├── frontend/                  React (Vite) frontend application
│   ├── src/
│   │   ├── components/        Reusable React components
│   │   ├── pages/             Page components
│   │   ├── api.js             API client configuration
│   │   ├── App.jsx            Main application component
│   │   └── index.css          Global styles
│   ├── Dockerfile             Container build instructions
│   ├── nginx.conf             Nginx configuration
│   ├── vite.config.js         Vite configuration
│   ├── package.json           Node.js dependencies
│   └── index.html             HTML entry point
│
├── backend/                   Node.js Express API server
│   ├── src/
│   │   ├── routes/            API route handlers
│   │   │   ├── posts.js       Post endpoints
│   │   │   └── comments.js    Comment endpoints
│   │   ├── db.js              Database connection and initialization
│   │   └── index.js           Express server setup
│   ├── Dockerfile             Container build instructions
│   ├── package.json           Node.js dependencies
│   └── .eslintrc.json         Linting configuration
│
├── deploy/                    AWS EC2 deployment tools
│   ├── setup.sh               One-click EC2 setup script
│   └── jerney-nginx.conf      Nginx reverse proxy configuration
│
├── terraform/                 Infrastructure-as-Code for EKS
│   ├── main.tf                EKS cluster and VPC configuration
│   ├── variables.tf           Terraform input variables
│   ├── outputs.tf             Terraform outputs
│   ├── provider.tf            AWS provider configuration
│   └── terraform.tfvars       Variable values
│
├── k8s/                       Kubernetes manifests
│   └── jerney.yaml            Complete Kubernetes deployment
│
├── .github/
│   └── workflows/
│       └── ci-cd.yml          GitHub Actions pipeline
│
├── docker-compose.yml         Multi-container orchestration
└── README.md                  Project documentation
```

---

## Deployment Options

### Option 1: AWS EC2 (Bare Metal)

**Best for:** Quick setup, simple workloads, cost-conscious development.

#### Prerequisites

- AWS EC2 instance running Ubuntu 22.04 or later
- Security Group allowing inbound traffic on ports 22 (SSH) and 80 (HTTP)
- SSH access to the instance

#### Deployment Steps

1. **Transfer code to EC2:**
```bash
scp -r -i your-key.pem ./Jerney ubuntu@<EC2_PUBLIC_IP>:~/Jerney
```

2. **SSH into the instance:**
```bash
ssh -i your-key.pem ubuntu@<EC2_PUBLIC_IP>
```

3. **Run the automated setup script:**
```bash
cd ~/Jerney
chmod +x deploy/setup.sh
./deploy/setup.sh
```

The setup script performs:
- System package updates
- Installation of Node.js 20.x, PostgreSQL 16, Nginx, and PM2
- Database and user creation
- Backend dependency installation
- React frontend build optimization
- Nginx reverse proxy configuration
- PM2 process management with auto-restart capabilities

4. **Access the application:**
```
http://<EC2_PUBLIC_IP>
```

#### Management Commands

```bash
pm2 status                          View backend service status
pm2 logs                            Display real-time backend logs
pm2 restart all                     Restart backend service
sudo systemctl restart nginx        Restart Nginx reverse proxy
sudo -u postgres psql -d jerney_db  Connect to database
```

---

### Option 2: Docker Compose (Local Development)

**Best for:** Local development, testing multiple services, reproducible environments.

Prerequisites:
- Docker and Docker Compose installed
- All services run in isolated containers

Start the application:
```bash
docker-compose up
```

This starts:
- PostgreSQL database (port 5432)
- Node.js backend (port 5000)
- React frontend with Nginx (port 80)

Access at: `http://localhost`

---

### Option 3: Kubernetes with EKS (Production)

**Best for:** High availability, auto-scaling, production workloads.

#### Infrastructure Setup with Terraform

1. **Configure variables:**
```bash
cd terraform
# Edit terraform.tfvars with your AWS region and settings
```

2. **Deploy infrastructure:**
```bash
terraform init
terraform plan
terraform apply
```

This creates:
- VPC with public and private subnets across 3 availability zones
- EKS cluster with Auto Mode enabled
- NAT Gateway for private subnet egress
- Proper tagging for Kubernetes service discovery

3. **Deploy application:**
```bash
kubectl apply -f k8s/jerney.yaml
```

The Kubernetes manifest includes:
- Namespace isolation
- Secrets management (base64 encoded)
- EBS-backed persistent volumes for database
- Deployments for frontend, backend, and database
- Services for internal and external communication
- Health checks and resource limits

---

## Local Development Setup

### Prerequisites

- Node.js 20.x or later
- PostgreSQL 16 or later
- npm or yarn package manager

### Backend Setup

```bash
cd backend
npm install

# Configure environment variables
export DB_HOST=localhost
export DB_PORT=5432
export DB_USER=jerney_user
export DB_PASSWORD=jerney_pass_2026
export DB_NAME=jerney_db
export PORT=5000

npm start
```

Alternatively, create a `.env` file in the backend directory:
```
DB_HOST=localhost
DB_PORT=5432
DB_USER=jerney_user
DB_PASSWORD=jerney_pass_2026
DB_NAME=jerney_db
PORT=5000
```

Development mode with auto-reload:
```bash
npm run dev
```

Linting:
```bash
npm run lint
```

### Frontend Setup

```bash
cd frontend
npm install
npm run dev
```

The development server runs on `http://localhost:3000` with hot module reloading. API requests to `/api/*` are automatically proxied to the backend at `http://localhost:5000`.

Build for production:
```bash
npm run build
```

Preview production build:
```bash
npm run preview
```

Linting:
```bash
npm run lint
```

### Database Setup

PostgreSQL automatically initializes tables on first backend startup. To manually create tables:

```bash
psql -U jerney_user -d jerney_db
```

Tables created:
- `posts`: Blog post storage (id, title, content, author, emoji, created_at, updated_at)
- `comments`: Comment storage (id, post_id, content, author, created_at)

---

## API Reference

### Posts Endpoints

| Method | Endpoint | Description | Request Body |
|--------|----------|-------------|---------------|
| GET | `/api/posts` | Retrieve all posts | N/A |
| GET | `/api/posts/:id` | Get single post with all comments | N/A |
| POST | `/api/posts` | Create new blog post | `{title, content, author, emoji}` |
| PUT | `/api/posts/:id` | Update existing post | `{title, content, author, emoji}` |
| DELETE | `/api/posts/:id` | Delete blog post | N/A |

### Comments Endpoints

| Method | Endpoint | Description | Request Body |
|--------|----------|-------------|---------------|
| GET | `/api/comments/post/:postId` | Get all comments for a post | N/A |
| POST | `/api/comments` | Create new comment | `{post_id, content, author}` |
| DELETE | `/api/comments/:id` | Delete comment | N/A |

### System Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/health` | Health check endpoint (returns 200 if API is running) |

---

## CI/CD Pipeline

The project includes a GitHub Actions workflow (`/.github/workflows/ci-cd.yml`) that:

1. **Linting Stage**: Runs ESLint on both frontend and backend code
2. **Security Analysis**: Performs SAST with available security tools
3. **Build Stage**: Builds Docker images for frontend and backend
4. **Image Scanning**: Scans container images for vulnerabilities
5. **Infrastructure Scanning**: Validates Terraform code with checkov
6. **Deployment**: Updates Kubernetes manifests with new image tags

Pipeline triggers on all pushes and pull requests to any branch.

---

## Git Branch Strategy

| Branch | Purpose | Deployment Target |
|--------|---------|-------------------|
| `main` | Production-ready source code | EC2 bare-metal deployment |
| `devops` | Full DevSecOps implementation | Kubernetes/EKS with CI/CD |

Switch to DevSecOps branch:
```bash
git checkout devops
```

The `devops` branch includes:
- Docker containerization for all services
- Kubernetes manifests for orchestration
- Terraform code for AWS infrastructure
- GitHub Actions CI/CD pipeline
- Container security scanning
- Infrastructure-as-Code best practices
- Comprehensive DevSecOps tooling

---

## Environment Variables

### Backend

```
DB_HOST          PostgreSQL host address
DB_PORT          PostgreSQL port (default: 5432)
DB_USER          PostgreSQL username
DB_PASSWORD      PostgreSQL password
DB_NAME          PostgreSQL database name
PORT             Express server port (default: 5000)
```

### Frontend

Frontend configuration is handled through Vite environment variables. The development server proxies API requests to the backend.

---

## Security Considerations

- Database credentials are environment-based and not hardcoded
- Docker images run with non-root users
- Read-only filesystems where applicable in Kubernetes deployments
- CORS is configured for controlled cross-origin requests
- PostgreSQL uses parameterized queries to prevent SQL injection
- Secrets are managed through Kubernetes Secrets (not base64 encoded in production)
- All containers include security options and resource limits

---

## Troubleshooting

### Backend fails to connect to database
- Verify PostgreSQL is running: `sudo systemctl status postgresql`
- Check environment variables are set correctly
- Confirm database credentials match: `sudo -u postgres psql`
- Test connection: `psql -h localhost -U jerney_user -d jerney_db`

### Frontend displays blank page
- Check backend is running: `curl http://localhost:5000/api/health`
- Verify frontend is built and Nginx is serving files
- Check browser console for API errors
- Ensure CORS is enabled in backend

### Port already in use
- Find process using port: `lsof -i :5000` or `lsof -i :3000`
- Kill process: `kill -9 <PID>`
- Change port via environment variable if needed

### Docker container exits immediately
- Check logs: `docker-compose logs backend`
- Verify environment variables are set
- Ensure database container is healthy before starting backend

---

## Contributing

When contributing to this project:
1. Create a feature branch from `main`
2. Make your changes with clear commit messages
3. Run linting: `npm run lint` (both frontend and backend)
4. Test functionality locally before submitting PR
5. Ensure CI/CD pipeline passes
6. Submit pull request with detailed description

---

## License

This project is provided as-is for development and learning purposes.

---

## Support and Documentation

For detailed information about specific deployment options or components, refer to:
- Local development: See Backend/Frontend Setup sections above
- EC2 deployment: See AWS EC2 Deployment Option section
- Kubernetes deployment: Refer to `k8s/jerney.yaml` manifest
- Infrastructure-as-Code: Refer to `terraform/` directory documentation

