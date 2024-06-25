# sample-node-app2
## comprehensive DevOps project that demonstrates your skills in deploying a website on Kubernetes

To create a comprehensive DevOps project that demonstrates your skills in deploying a website on Kubernetes, you will need to follow these key steps:

1. **Project Setup and Sample Application**
2. **Infrastructure Setup**
3. **CI/CD Pipeline Setup with Jenkins**
4. **Kubernetes Deployment**
5. **Monitoring with Prometheus and Grafana**

### Step 1: Project Setup and Sample Application

1. **Create a Sample Application**: We will use a simple Node.js application for this purpose.

   **Sample Node.js Application**:
   - Create a directory for the project and navigate into it.
   - Initialize a new Node.js project.

   ```bash
   mkdir sample-node-app
   cd sample-node-app
   npm init -y
   ```

   - Install Express.js.

   ```bash
   npm install express
   ```

   - Create an `index.js` file with the following content:

   ```javascript
   const express = require('express');
   const app = express();
   const port = 3000;

   app.get('/', (req, res) => {
     res.send('Hello World!');
   });

   app.listen(port, () => {
     console.log(`App listening at http://localhost:${port}`);
   });
   ```

   - Add a `Dockerfile` to containerize the application.

   ```dockerfile
   FROM node:14

   WORKDIR /usr/src/app

   COPY package*.json ./
   RUN npm install

   COPY . .

   EXPOSE 3000
   CMD ["node", "index.js"]
   ```

2. **Push the Application to GitHub**:
   - Initialize a git repository, commit your code, and push it to GitHub.

   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git branch -M main
   git remote add origin <your-github-repo-url>
   git push -u origin main
   ```

### Step 2: Infrastructure Setup

1. **Create a Kubernetes Cluster**: Use Google Kubernetes Engine (GKE) to create a Kubernetes cluster.
   - Install `gcloud` CLI and authenticate.

   ```bash
   gcloud init
   ```

   - Create a Kubernetes cluster.

   ```bash
   gcloud container clusters create sample-cluster --num-nodes=3 --zone=<your-zone>
   gcloud container clusters get-credentials sample-cluster --zone=<your-zone>
   ```

### Step 3: CI/CD Pipeline Setup with Jenkins

1. **Install Jenkins**:
   - SSH into your Ubuntu server and install Jenkins.

   ```bash
   sudo apt update
   sudo apt install openjdk-11-jdk
   wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
   sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
   sudo apt update
   sudo apt install jenkins
   sudo systemctl start jenkins
   sudo systemctl enable jenkins
   ```

   - Access Jenkins at `http://<your-server-ip>:8080` and complete the setup wizard.

2. **Configure Jenkins**:
   - Install necessary plugins: Docker, Kubernetes, GitHub, Pipeline.
   - Create a new pipeline job.

3. **Jenkins Pipeline Configuration**:
   - Create a `Jenkinsfile` in your project repository.

   ```groovy
   pipeline {
       agent any

       stages {
           stage('Build') {
               steps {
                   script {
                       dockerImage = docker.build("sample-node-app")
                   }
               }
           }
           stage('Push to Docker Hub') {
               steps {
                   script {
                       docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
                           dockerImage.push('latest')
                       }
                   }
               }
           }
           stage('Deploy to Kubernetes') {
               steps {
                   script {
                       kubernetesDeploy(
                           configs: 'k8s/deployment.yaml',
                           kubeconfigId: 'kubeconfig'
                       )
                   }
               }
           }
       }
   }
   ```

### Step 4: Kubernetes Deployment

1. **Kubernetes Deployment Configuration**:
   - Create a `k8s` directory in your project and add a `deployment.yaml` file.

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: sample-node-app
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: sample-node-app
     template:
       metadata:
         labels:
           app: sample-node-app
       spec:
         containers:
         - name: sample-node-app
           image: <your-dockerhub-username>/sample-node-app:latest
           ports:
           - containerPort: 3000
   ```

2. **Service Configuration**:
   - Add a `service.yaml` file in the `k8s` directory.

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: sample-node-app
   spec:
     type: LoadBalancer
     selector:
       app: sample-node-app
     ports:
       - protocol: TCP
         port: 80
         targetPort: 3000
   ```

3. **Apply the Kubernetes Configuration**:

   ```bash
   kubectl apply -f k8s/deployment.yaml
   kubectl apply -f k8s/service.yaml
   ```

### Step 5: Monitoring with Prometheus and Grafana

1. **Install Prometheus and Grafana**:
   - Create a `monitoring` namespace.

   ```bash
   kubectl create namespace monitoring
   ```

   - Deploy Prometheus using Helm.

   ```bash
   helm install prometheus stable/prometheus --namespace monitoring
   ```

   - Deploy Grafana using Helm.

   ```bash
   helm install grafana stable/grafana --namespace monitoring
   ```

2. **Configure Prometheus and Grafana**:
   - Access Grafana and add Prometheus as a data source.
   - Create dashboards for monitoring.

### Conclusion

This project involves creating a sample Node.js application, containerizing it with Docker, setting up a CI/CD pipeline with Jenkins, deploying the application to a Kubernetes cluster, and configuring monitoring with Prometheus and Grafana. Each step is designed to demonstrate a critical aspect of DevOps engineering, ensuring you have a comprehensive understanding and practical experience to offer as a service to companies and organizations.
