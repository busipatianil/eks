============================================================================================
![image](https://github.com/user-attachments/assets/24faafff-8017-412e-a89b-2b5e726fb93c)

Developer: The person writing the code and Helm chart.

Version Control System (VCS): (e.g., GitHub, GitLab, Bitbucket) Stores the application code, Dockerfile, and Helm chart.

Jenkins Server: The CI/CD automation server. It will have:

Necessary plugins (Kubernetes, Docker, Helm, Git, etc.).

Credentials to access VCS, Container Registry, and EKS.

Container Registry: (e.g., Amazon ECR, Docker Hub, Quay.io) Stores the built Docker image of the "Hello World" application.

Amazon EKS Cluster: The managed Kubernetes service where the application will be deployed. It consists of:

Control Plane: Managed by AWS.

Worker Nodes: EC2 instances where your pods will run.

Helm: The package manager for Kubernetes, used to define, install, and upgrade the application.

Hello World Application:

Application Code: (e.g., a simple Python Flask app, Node.js Express app, or even just an Nginx container serving static HTML).

Dockerfile: Instructions to build the application into a container image.

Helm Chart: Defines the Kubernetes resources (Deployment, Service, etc.) needed to run the application.

End User: Accesses the deployed "Hello World" application.

======================= Workflow Explanation ================
                  Code & Chart Preparation (Developer):

The developer writes the "Hello World" application code (e.g., a simple web server).

Creates a Dockerfile to containerize the application.

                  Creates a Helm chart. This chart will typically include:

Chart.yaml: Metadata about the chart.
values.yaml: Default configuration values (like image tag, replica count, service type).
templates/: Directory containing Kubernetes manifest templates (e.g., deployment.yaml, service.yaml).

                  
                  Push to Version Control (Developer & Git):

The developer commits and pushes the application code, Dockerfile, and Helm chart to a Git repository (e.g., GitHub, GitLab).
Jenkins Pipeline Trigger (Git & Jenkins):
A webhook configured in the Git repository triggers the Jenkins pipeline upon a push to a specific branch (e.g., main or master).
                  
                  Jenkins Pipeline Execution (Jenkins):

a. Checkout Code: Jenkins checks out the latest code (including the Helm chart and Dockerfile) from the Git repository.
b. Build Docker Image: Jenkins uses Docker (or a Docker plugin) to build a new container image for the "Hello World" application using the Dockerfile. The image is typically tagged with a unique identifier (e.g., Git commit hash or build number).
Example command: docker build -t <your-registry>/hello-world:$BUILD_NUMBER .
c. Push Docker Image to Registry: The newly built Docker image is pushed to a container registry (e.g., Amazon ECR). Jenkins will need credentials to access this registry.
Example command: docker push <your-registry>/hello-world:$BUILD_NUMBER
d. Update Helm Chart (Optional but Recommended):
The Jenkins pipeline might update the values.yaml file in the Helm chart (or pass values via --set) to use the newly built image tag ($BUILD_NUMBER). This ensures the new version is deployed.
e. Deploy/Upgrade with Helm: Jenkins uses the Helm CLI to deploy or upgrade the application in the EKS cluster.
It will need kubeconfig credentials to access the EKS cluster. IAM roles for service accounts (IRSA) are the recommended way for Jenkins (running in or outside EKS) to securely access EKS.
  
                  Example Helm command:
  
  helm upgrade --install hello-world-release ./path/to/helm-chart \
  --namespace hello-world-ns \
  --create-namespace \
  --set image.repository=<your-registry>/hello-world \
  --set image.tag=$BUILD_NUMBER
Use code with caution.
Bash
(Where hello-world-release is the Helm release name, and hello-world-ns is the Kubernetes namespace).

                  EKS Pulls Image (EKS & Container Registry):

The Kubernetes Deployment (created/updated by Helm) in EKS instructs the worker nodes to pull the specified Docker image version from the container registry (ECR).
Worker nodes need IAM permissions to pull from ECR (usually handled by the EKS node IAM role).
  
                  Application Running in EKS (EKS):

EKS schedules Pods (containing the "Hello World" container) onto worker nodes.
A Kubernetes Service (e.g., of type LoadBalancer or NodePort, defined in the Helm chart) exposes the application to be accessible. If it's a LoadBalancer service, AWS will provision an Elastic Load Balancer.
                  
                  User Access (End User):

The end user accesses the "Hello World" application via the URL provided by the Kubernetes Service (e.g., the DNS name of the ELB).

                  Key Considerations:

Jenkins Setup: Jenkins needs appropriate plugins (Git, Docker, Kubernetes CLI, Helm). It can run on an EC2 instance, within EKS itself, or elsewhere.
            
                  IAM Roles & Permissions & enkins needs permissions to:
Read from the Git repository.
Build Docker images.
Push to the Container Registry (ECR).
Interact with the EKS cluster (deploy using Helm).
EKS worker nodes need permission to pull images from ECR.
kubeconfig for Jenkins: Jenkins needs a valid kubeconfig file or service account token to interact with the EKS cluster API. Using IAM Roles for Service Accounts (IRSA) is best practice if Jenkins is running in EKS or has a way to assume an IAM role.
Helm Chart Structure: Ensure your Helm chart is well-structured and parameterizes things like image tags, replica counts, and service types.
Jenkinsfile: The pipeline itself will be defined in a Jenkinsfile (declarative or scripted pipeline) checked into your Git repository.
This setup provides a robust automated way to build and deploy your application to End user (Client)
