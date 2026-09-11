## Setup Guide: Installing n8n with Docker Compose and PostgreSQL
This guide provides step-by-step instructions to deploy n8n using Docker Compose with a PostgreSQL database backend on Ubuntu Desktop within VirtualBox.
------------------------------
## 📋 Prerequisites
Before you begin, ensure your Ubuntu VM has:

* Docker and Docker Compose installed.
* sudo privileges.
* A user account named serveradmin (or adjust the directory permissions step to match your current username).

------------------------------
## 🛠️ Step-by-Step Installation## Step 1: Verify Docker Installation
Ensure Docker is installed and checking the version:

docker -v

If this returns a version number, you are ready to proceed.
## Step 2: Create the Project Directory
Organize the deployment files inside the /opt/stacks directory and configure the correct ownership permissions:

sudo mkdir -p /opt/stacks/n8n
sudo chown serveradmin:serveradmin -R /opt/stacks
cd /opt/stacks/n8n

## Step 3: Create the Docker Compose File
Create a new configuration file named compose.yaml:

nano compose.yaml

Paste the following configuration into the file, then save (Ctrl+O, Enter) and exit (Ctrl+X):

services:
  n8n:
    image: n8nio/n8n:latest
    restart: always
    ports:
      - "5678:5678"
    env_file:
      - .env
    volumes:
      - ./data:/home/node/.n8n
      - ./files:/files
    depends_on:
      - postgres

  postgres:
    image: postgres:15
    restart: always
    env_file:
      - .env
    volumes:
      - ./postgres-data:/var/lib/postgresql/data

## Step 4: Create the Environment Variable File
Create the .env file to manage configuration settings securely:

nano .env

Paste the following values, modify the configuration properties (such as POSTGRES_PASSWORD) to match your environment, then save and exit:

# n8n Settings
DOMAIN_NAME=example.com
SUBDOMAIN=n8n
GENERIC_TIMEZONE=America/New_York
N8N_HOST=n8n.example.com
N8N_PROTOCOL=https
WEBHOOK_URL=https://n8n.example.com/
N8N_SECURE_COOKIE=false
NODE_ENV=production

# Traefik (optional)
TRAEFIK_ROUTER_NAME=n8n
TRAEFIK_DOMAIN=n8n.example.com
TRAEFIK_CERT_RESOLVER=mytlschallenge

# PostgreSQL
POSTGRES_DB=n8n
POSTGRES_USER=n8n
POSTGRES_PASSWORD=changeme123

## Step 5: Start the Stack
Launch the containers in detached mode (running in the background):

docker compose up -d

Docker will automatically download the necessary images, set up the data volumes, and launch both the n8n and PostgreSQL containers.
------------------------------
## 🔍 Verification & Troubleshooting## Check Container Status
Verify that both containers are running properly:

docker ps

You should see active containers for both n8n and postgres.
## View System Logs
If any container fails to start, inspect the runtime logs to diagnose the issue:

docker compose logs

------------------------------
## 🚀 Accessing the n8n Web Interface

* From Inside the Ubuntu VM: Open a web browser and navigate to http://localhost:5678.
* From the Host Machine: If VirtualBox is configured with a Bridged Adapter or Port Forwarding (Port 5678), open your host browser and navigate to http://<VM_IP_ADDRESS>:5678.

Because n8n runs inside a Docker container using a non-root `node` user (UID `1000`), the container needs permission to read and write to the local folders you mapped in the `compose.yaml` file. 


## 🚀 Error Fixed
If you see permission errors or if n8n fails to save workflows or docker would not able start the n8n (**keep restart**), run the following commands on your Ubuntu terminal to grant correct ownership to the container:

```bash
# Change ownership of the n8n data folder to the container's node user (UID 1000)
sudo chown -R 1000:1000 /opt/stacks/n8n/data

# Change ownership of the shared files folder if you plan to import/export files
sudo chown -R 1000:1000 /opt/stacks/n8n/files

# Ensure correct read/write permissions are applied
sudo chmod -R 775 /opt/stacks/n8n/data /opt/stacks/n8n/files
```

> ⚠️ **Note:** If you haven't started the stack yet, the `data` and `files` folders might not exist. Docker will create them automatically when you run `docker compose up -d`, after which you should run these permission commands



## 🌐 Official Project References

A huge thank you to **Techno Tim** for providing such a clear visual automation roadmap. For deeper implementation details, consult the following verified channels:

* **Official Video Tutorial:** (https://www.youtube.com/watch?v=gyn8bcOLdcA)
* **Step-by-Step Documentation:** (https://technotim.com/posts/n8n-self-hosted/)
* **Source Repository:** https://github.com/n8n-io/n8n




