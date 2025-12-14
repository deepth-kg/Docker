<img width="972" height="251" alt="image" src="https://github.com/user-attachments/assets/624e7b6c-3cc4-4f6c-aaed-29cf92e53354" />
Here What we are trying to achieve is deploy jenkins in docker and then spin up a new docker agent(ephemeral) for build (Dev/testing only) 


so My docker is in my local check whether is docker is installed or not 

Architecture:
Host Machine
 ├─ Docker Daemon
 │
 ├─ Jenkins Container
 │    ├─ Jenkins Home (volume)
 │    ├─ Docker CLI
 │    └─ docker.sock mounted
 │
 └─ Ephemeral Build Containers
      ├─ alpine / node / maven / etc
      └─ Auto-created & destroyed per build

commands:
Docker --version
docker ps 
<img width="972" height="251" alt="image" src="https://github.com/user-attachments/assets/b67d607e-6d48-489f-8935-b24cdbfac3f1" />

once confirmed
STEP 1: Run Jenkins in Docker
✅docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --restart unless-stopped \
  jenkins/jenkins:lts-jdk17

  <img width="1297" height="155" alt="image" src="https://github.com/user-attachments/assets/c68ac661-5941-4728-ac66-248026472ff4" />

  ----------------------------------------------------------

STEP 2: Install Docker CLI INSIDE Jenkins container
   ✅ docker exec -it jenkins bash
Inside container:

✅ apt-get update
✅apt-get install -y ca-certificates curl gnupg
✅install -m 0755 -d /etc/apt/keyrings
✅curl -fsSL https://download.docker.com/linux/debian/gpg \
  | gpg --dearmor -o /etc/apt/keyrings/docker.gpg

✅echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo $VERSION_CODENAME) stable" \
  > /etc/apt/sources.list.d/docker.list

✅apt-get update
✅apt-get install -y docker-ce-cli

Verify:
docker ps
  
------------------------------------------------------------

STEP 3: Jenkins UI – Plugins (MANDATORY)

Manage Jenkins → Manage Plugins

Install:

✅ Docker Pipeline

✅ Docker Commons

✅ Docker API

Pipeline plugins (usually already there)

🔁 Restart Jenkins
<img width="1273" height="638" alt="image" src="https://github.com/user-attachments/assets/8d81ad9f-a845-494e-bc72-4aa46c6d49f6" />

.............................................................
STEP 4: Jenkins Node Configuration
Manage Jenkins → Nodes → Built-in Node

Field	Value
Labels	docker-agent
Executors	1 or more
<img width="1273" height="638" alt="image" src="https://github.com/user-attachments/assets/eb4bd66f-f40c-4aad-a885-8f790badc5c4" />


-----------------------------------------------------------

STEP 5: Pipeline
 create a pipeline and update theii sbelow lines updat ethe label too 
pipeline {
    agent {
        docker {
            image 'alpine:latest'
            label 'docker-agent'  #label name mentioned 
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

<img width="1273" height="638" alt="image" src="https://github.com/user-attachments/assets/f047468d-21c0-4fce-954c-e278ef1081d9" />

now you can see a new docker is spin up 
<img width="1303" height="273" alt="image" src="https://github.com/user-attachments/assets/4eb4757c-c334-44c7-b0b9-5bb22952c3b3" />



