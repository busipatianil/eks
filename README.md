#steps

=============== Application work Flow =========================
+-------------------+      +-----------------------+      +---------------------+
|     Developer     |----->| Version Control (Git) |----->|    Jenkins Server   |
+-------------------+      +-----------------------+      +---------------------+
                             ^         |                        |   | (Webhook)
                             |         | (Code, Dockerfile,     |   |
                             |         |  Helm Chart)           |   V (Credentials)
                             |         |                        |
                             |         | 1. Checkout Code       |   +-------------------+
                             |         |----------------------->|   | Build Docker Image|
                             |                                  |   +-------------------+
                             |                                  |            |
                             |                                  |            V (Push Image)
                             |                                  |   +-------------------+
                             |                                  |   | Container Registry|
                             |                                  |   | (e.g.,Registry,ECR)       |
                             |                                  |   +-------------------+
                             |                                  |            ^
                             |                                  |            | (Pull Image)
                             |         +------------------------+            |
                             |         | 2. Helm Deploy/Upgrade              |
                             |         V                                     |
+--------------------------------------------------------------------------------------+
|                                Amazon EKS Cluster                                    |
|                                                                                      |
|   +-----------------------+      +-----------------------------------------------+   |
|   | EKS Control Plane (AWS)|----->| EKS Worker Nodes                              |   |
|   +-----------------------+      |                                               |   |
|                                  |   +---------------------------------------+   |   |
|                                  |   | Helm Installs/Manages:                |   |   |
|                                  |   |   - Deployment (Hello World Pods)     |<--+   |
|                                  |   |   - Service (e.g., LoadBalancer)      |   |   |
|                                  |   +---------------------------------------+   |   |
|                                  +-----------------------------------------------+   |
|                                                  ^                                   |
+--------------------------------------------------|-----------------------------------+
                                                   | (Access App)
                                                   |
                                         +-------------------+
                                         |     End User      |
                                         +-------------------+
              
 ============================================================================================
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
                                         
