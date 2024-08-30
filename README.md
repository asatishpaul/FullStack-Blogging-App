**"CI/CD Pipeline Deployment of a Full-Stack Blogging Application on AWS EKS Using Jenkins, Helm, and Terraform"**

### **Detailed Explanation of the Jenkins Pipeline Deployment Project**

#### **1. Prerequisites**

Before diving into the deployment, several prerequisites were set up to ensure a smooth CI/CD process:

- **Jenkins:** Installed and configured with the necessary plugins for Kubernetes, Docker, SonarQube, and Nexus integration.
- **SonarQube:** Set up for static code analysis, running independently and accessible for Jenkins.
- **Docker:** Installed on the Jenkins server to build and manage Docker images.
- **Nexus:** Configured as a Docker registry to store built images securely.
- **Kubernetes Cluster:** Deployed using AWS EKS, provisioned through Terraform.
- **Helm:** Installed on Jenkins for managing Kubernetes deployments and simplifying application release management.

#### **2. Creating the EKS Cluster Using Terraform**

To ensure a reliable and scalable Kubernetes environment, the EKS cluster was created using Terraform. The Terraform scripts included:

- **VPC Creation:** Configured networking, subnets, and security groups required for EKS.
- **IAM Roles:** Created roles and policies necessary for EKS control plane and worker nodes.
- **EKS Cluster and Node Groups:** Provisioned the EKS cluster with the necessary node groups to host the application.

The execution of the Terraform code created an EKS cluster with the necessary configurations to deploy containerized applications.

#### **3. Jenkins Plugins Used**

Several Jenkins plugins were installed and configured to support various stages of the CI/CD pipeline:

1. **SonarQube Scanner:** For static code analysis and ensuring code quality.
2. **Config File Provider:** To manage configuration files within Jenkins.
3. **Maven Integration:** For building and managing Java projects.
4. **Pipeline Maven Integration:** To run Maven commands within Jenkins pipelines.
5. **Kubernetes:** To manage Kubernetes clusters directly from Jenkins.
6. **Kubernetes Client API:** Provides APIs to interact with Kubernetes clusters.
7. **Kubernetes Credentials:** Handles Kubernetes credentials securely in Jenkins.
8. **Kubernetes CLI:** Allows running Kubernetes CLI commands within Jenkins.
9. **Docker:** To interact with Docker within the Jenkins pipeline.
10. **Docker Pipeline:** Enables the use of Docker within Jenkins pipelines.
11. **Pipeline Stage View:** Visualizes the pipeline execution and stages.
12. **Eclipse Temurin Installer:** For managing JDK installations within Jenkins.

#### **4. Setting Up Helm and Creating Helm Charts**

**Helm Installation:**

Helm was installed on Jenkins to manage Kubernetes deployments efficiently. Helm helps in packaging, configuring, and deploying applications on Kubernetes.

**Creating Helm Chart for the Blogging Application:**

The Helm chart was structured as follows:

- **Chart.yaml:** Contains metadata about the chart.
- **values.yaml:** Defines default values for the chart, such as replica count, image repository, and service type.
- **templates/deployment.yaml:** Manages the Kubernetes deployment specifications.
- **templates/service.yaml:** Configures the Kubernetes service, including the LoadBalancer setup.

**Helm Chart Structure:**
```
bloggingapp/
  ├── Chart.yaml
  ├── values.yaml
  └── templates/
      ├── deployment.yaml
      └── service.yaml
```

**Content of Key Files:**

- **Chart.yaml**
  ```yaml
  apiVersion: v2
  name: bloggingapp
  description: A Helm chart for the Blogging Application
  type: application
  version: 0.1.0
  appVersion: "1.0"
  ```

- **values.yaml**
  ```yaml
  replicaCount: 2
  appLabel: bloggingapp
  image:
    repository: asatishpaul/bloggingapp
    tag: latest
    pullPolicy: Always
  service:
    type: LoadBalancer
    port: 80
    targetPort: 8080
  ```

- **deployment.yaml**
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: bloggingapp-deployment
  spec:
    replicas: 2
    selector:
      matchLabels:
        app: eks-app
    template:
      metadata:
        labels:
          app: eks-app
      spec:
        containers:
          - name: bloggingapp
            image: asatishpaul/bloggingapp:latest
            ports:
              - containerPort: 8080
        imagePullSecrets:
          - name: regcred
  ```

- **service.yaml**
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: bloggingapp-ssvc
  spec:
    type: LoadBalancer
    selector:
      app: eks-app
    ports:
      - protocol: TCP
        port: 80
        targetPort: 8080
  ```

#### **5. Kubernetes RBAC Configuration**

To ensure secure access and management of the Kubernetes cluster, RBAC (Role-Based Access Control) was configured:

- **Role:** Defined specific permissions for interacting with Kubernetes resources.
- **RoleBinding:** Associated the role with a specific service account.
- **Service Account:** Used by Jenkins to interact with the Kubernetes cluster securely.
- **Tokens:** Generated tokens for authenticating Jenkins with the Kubernetes API.

**RBAC Example:**

- **role.yaml**
  ```yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: Role
  metadata:
    namespace: webapps
    name: webapp-role
  rules:
  - apiGroups: [""]
    resources: ["pods", "services"]
    verbs: ["get", "list", "create", "delete"]
  ```

- **rolebinding.yaml**
  ```yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    name: webapp-rolebinding
    namespace: webapps
  subjects:
  - kind: ServiceAccount
    name: jenkins
    namespace: webapps
  roleRef:
    kind: Role
    name: webapp-role
    apiGroup: rbac.authorization.k8s.io
  ```

#### **6. Jenkins Pipeline Execution and Console Output**

During the pipeline execution, Jenkins performed the following key steps:

- **Node Verification:** Confirmed the availability of nodes in the EKS cluster.
- **Helm Deployment:** Successfully deployed the Helm chart, with status messages confirming the release.
- **Deployment Verification:** Checked the status of the deployed application, including pods, services, and replicas.

**Console Output Highlights:**
```plaintext
+ kubectl get nodes
NAME                                        STATUS   ROLES    AGE   VERSION
ip-10-0-0-244.ap-south-1.compute.internal   Ready    <none>   12h   v1.30.2-eks-1552ad0
ip-10-0-0-29.ap-south-1.compute.internal    Ready    <none>   12h   v1.30.2-eks-1552ad0
ip-10-0-1-54.ap-south-1.compute.internal    Ready    <none>   12h   v1.30.2-eks-1552ad0

+ echo Deploying Helm chart...
Deploying Helm chart...
+ helm upgrade --install bloggingapp-release /var/lib/jenkins/workspace/helmchart-ci-cd/bloggingapp --namespace webapps
Release "bloggingapp-release" has been upgraded. Happy Helming!

+ echo Verifying Helm deployment...
Verifying Helm deployment...
+ helm status bloggingapp-release --namespace webapps
NAME: bloggingapp-release
LAST DEPLOYED: Fri Aug 30 06:40:43 2024
NAMESPACE: webapps
STATUS: deployed
REVISION: 6

+ echo Listing Kubernetes resources...
Listing Kubernetes resources...
+ kubectl get all --namespace webapps
NAME                                          READY   STATUS    RESTARTS   AGE
pod/bloggingapp-deployment-5bc95575c5-859lp   1/1     Running   0          41m

service/bloggingapp-ssvc   LoadBalancer   172.20.187.34   ad5fc71a3dd9749a3a8b3cfe90334a9b-1580551870.ap-south-1.elb.amazonaws.com   80:32054/TCP   8h

deployment.apps/bloggingapp-deployment   2/2     2            2           41m
```

### **Conclusion**

The entire pipeline executed successfully, deploying a full-stack blogging application on AWS EKS using Jenkins, Helm, and Terraform. The integration of Jenkins plugins, Kubernetes RBAC, and Terraform-based infrastructure ensured a seamless CI/CD process, showcasing the power of modern DevOps practices.
