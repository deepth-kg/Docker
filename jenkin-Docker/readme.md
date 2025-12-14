 Jenkins in Docker with Ephemeral Agents (Dev/Test Only)

## Objective
Deploy Jenkins in Docker and spin up ephemeral Docker agents for builds. **For development/testing only.**

---

## Architecture

Host Machine
├─ Docker Daemon

├─ Jenkins Container
     ├─ Jenkins Home (volume)
     ├─ Docker CLI
     └─ docker.sock mounted
     
│ └─ Ephemeral Build Containers
│ ├─ alpine / node / maven / etc.
│ └─ Auto-created & destroyed per build


---

## Prerequisites
- Docker installed on host
- Local user with Docker permissions
- Jenkins Docker image (`jenkins/jenkins:lts-jdk17`)

---

## Step 1: Run Jenkins in Docker

```bash
docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --restart unless-stopped \
  jenkins/jenkins:lts-jdk17
Step 2: Install Docker CLI Inside Jenkins Container
bash
Copy code
docker exec -it jenkins bash
apt-get update
apt-get install -y ca-certificates curl gnupg
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/debian $(. /etc/os-release && echo $VERSION_CODENAME) stable" \
> /etc/apt/sources.list.d/docker.list
apt-get update
apt-get install -y docker-ce-cli
docker ps
Step 3: Install Jenkins Plugins
Docker Pipeline

Docker Commons

Docker API

Pipeline (usually already present)

Restart Jenkins after plugin installation

Step 4: Configure Jenkins Node
Built-in Node

Labels: docker-agent

Executors: 1 or more

Step 5: Create a Sample Pipeline
groovy
Copy code
pipeline {
    agent {
        docker {
            image 'alpine:latest'
            label 'docker-agent'
        }
    }
    stages {
        stage('Ephemeral Build') {
            steps {
                sh '''
                  echo "Running in ephemeral container"
                  hostname
                  sleep 50
                '''
            }
        }
    }
}
You will see a new Docker container spin up.

Once build is completed, the container is deleted.

Security Considerations ⚠️
Mounting /var/run/docker.sock grants root access to host Docker.

Risks: Run privileged containers, escape to host, delete images, mount /.

Only use this setup in local/dev environments.

For production, consider Docker socket proxy or Kubernetes agents.

