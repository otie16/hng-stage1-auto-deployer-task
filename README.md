

# 🧭 README.md — Automated Docker Deployment Script

## 🚀 Project Overview

`deploy.sh` is a **fully automated Bash deployment script** designed to handle **end-to-end deployment** of Dockerized applications on **remote Linux servers** (e.g., AWS EC2, Azure VM, GCP VM).

It installs Docker, Docker Compose, and Nginx automatically (if missing), sets up the server, and deploys your application behind a reverse proxy — all in one command.

This script was built for DevOps engineers who want a **production-ready**, **idempotent**, and **transparent deployment pipeline** using pure Bash — no Ansible, Terraform, or external tools.

---

## ⚙️ Key Features

- **Full Automation** — One script handles cloning, building, deployment, and proxy configuration.
- **Supports Docker & Docker Compose** — Works with either `Dockerfile` or `docker-compose.yml`.
- **Automatic Remote Setup** — Installs and configures Docker, Compose, and Nginx on the target server.
- **Reverse Proxy Configuration** — Dynamically creates Nginx config that forwards port 80 → app port.
- **Logging & Error Handling** — Every step is logged, with clear failure codes and messages.
- **Idempotent Design** — Safe to re-run; old containers are replaced cleanly.
- **Cleanup Mode** — Run `--cleanup` to safely tear down the app, containers, and Nginx config.

---

## 🏗️ Architecture Overview

The deployment flow looks like this:

```
Local Machine
│
├── Collects repo, branch, and SSH details
│
├── Clones repo + checks for Docker/Docker Compose
│
├── SSH into Remote Server
│   ├── Installs Docker, Docker Compose, Nginx
│   ├── Transfers code
│   ├── Builds and runs containers
│   ├── Creates Nginx reverse proxy config
│   └── Reloads Nginx to route traffic
│
└── Verifies deployment health and logs
```

---

## 🧩 Prerequisites

Before running the script, ensure you have:

| Requirement                       | Description                                         |
| --------------------------------- | --------------------------------------------------- |
| **Linux / macOS Terminal**        | For running the script locally                      |
| **Git Installed**                 | For cloning and pulling repositories                |
| **SSH Key Access**                | The private key that connects to your remote server |
| **Personal Access Token (PAT)**   | For authenticating with private GitHub repos        |
| **Remote Server (AWS/Azure/GCP)** | Ubuntu/Debian preferred                             |
| **Public IP**                     | Of the remote VM you’re deploying to                |

---

## 💻 Installation

Clone this repository to your local system:

```bash
git clone https://github.com/<yourusername>/auto-docker-deploy.git
cd auto-docker-deploy
chmod +x deploy.sh
```

---

## ⚡ Usage

### 🔹 Deploy Your Application

Run the script without arguments:

```bash
./deploy.sh
```

You’ll be prompted for:

| Prompt                        | Description                             |
| ----------------------------- | --------------------------------------- |
| `Git repository URL`          | HTTPS link to your repository           |
| `Personal Access Token (PAT)` | For GitHub authentication               |
| `Branch name`                 | Optional (defaults to `main`)           |
| `Remote server username`      | e.g. `ubuntu`                           |
| `Remote server IP`            | e.g. `3.90.45.100`                      |
| `SSH key path`                | e.g. `~/.ssh/id_rsa`                    |
| `Application port`            | e.g. `5000` (container’s internal port) |

The script will:

1. Clone the repo locally (or pull if it exists)
2. SSH into the remote server
3. Install Docker, Compose, and Nginx
4. Copy your files
5. Build and start the container(s)
6. Set up Nginx reverse proxy
7. Reload and verify deployment

---

### 🔹 Cleanup Mode

To remove everything (containers, images, Nginx config, etc.):

```bash
./deploy.sh --cleanup
```

You’ll only be asked for:

* Remote username
* Server IP
* SSH key path

Then it will:

* Stop and remove containers
* Prune Docker resources
* Remove Nginx site configuration
* Reload Nginx

---

## 🧱 Nginx Reverse Proxy

The script automatically generates a configuration like this:

```nginx
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://127.0.0.1:<APP_PORT>;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

It then removes `/etc/nginx/sites-enabled/default` to ensure your app serves on port 80.

---

## 🧪 Example Application

You can test the script using a simple Node.js app:

**Dockerfile**

```Dockerfile
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 5000
CMD ["npm", "start"]
```

**server.js**

```js
const express = require("express");
const app = express();
app.get("/", (req, res) => res.send("🚀 Deployed successfully!"));
app.listen(5000, () => console.log("App running on port 5000"));
```

Then push this to your GitHub repo and run the deploy script.

---

## 📋 Logging

Each run creates a timestamped log file:

```
deploy_20251021_183045.log
```

This log contains:

* Step-by-step actions
* Success/failure messages
* Exit codes on error

---

## 🧠 Troubleshooting

| Issue                                | Cause                                 | Fix                                                   |
| ------------------------------------ | ------------------------------------- | ----------------------------------------------------- |
| **Still seeing “Welcome to nginx!”** | Default Nginx config not removed      | Script now removes `/etc/nginx/sites-enabled/default` |
| **Permission denied (publickey)**    | Wrong SSH key path                    | Use correct `.pem` or `.ssh/id_rsa` file              |
| **Invalid port in upstream**         | `$APP_PORT` not substituted correctly | Ensure it’s provided during input                     |
| **Docker build fails**               | Missing Dockerfile or syntax error    | Check Dockerfile locally before deploy                |
| **App unreachable**                  | Container crashed or port mismatch    | Run `docker ps` on remote to verify ports             |

---

## 🔐 Security Tips

* Never hardcode your **PAT** or **SSH key paths** into the script.
* Ensure your **PAT** has the least privileges (read-only repo access).
* Rotate keys regularly.
* Use a **non-root user** on remote servers when possible.

---

## 🧰 Extending the Script

You can enhance this script with:

* SSL setup (Certbot for HTTPS)
* CI/CD integration (GitHub Actions)
* Environment variable injection (.env)
* Automatic rollback on failure
* Notifications (Slack/Webhook on completion)

---

## 🌍 Example AWS Setup

If deploying to AWS EC2:

1. Launch an Ubuntu instance.
2. Allow inbound ports 22 (SSH), 80 (HTTP), and your app’s internal port.
3. SSH into the instance manually once:

   ```bash
   ssh -i ~/.ssh/mykey.pem ubuntu@<your-ec2-ip>
   ```
4. Then run the script from your local machine:

   ```bash
   ./deploy.sh
   ```

Your application will be accessible at:

```
http://<your-ec2-ip>
```

---

## 🧹 Cleanup Confirmation

To tear down everything cleanly:

```bash
./deploy.sh --cleanup
```

This removes:

* All app containers/images
* Old Docker networks
* Nginx proxy config
* Leaves system dependencies intact

---

## 🧑‍💻 Author

**Oty**
DevOps Engineer | AWS | Kubernetes | Terraform | Python | Git | Docker | GCP |
GitHub: [@otie16](https://github.com/otie16)



