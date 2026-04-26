Kubernetes High Availability & Deployment MasterclassProject OverviewThis project demonstrates the transition from local container development to a high-availability, production-grade Kubernetes environment. It covers the full lifecycle of a containerized application, focusing on Self-Healing, Declarative Configuration, and the Kubernetes Networking Stack.The Architectural FlowThe deployment follows a tiered approach to ensure zero-downtime and scalability:Containerization: Packaging source code into Docker images.Orchestration: Deploying to Amazon EKS (Elastic Kubernetes Service).Networking: Exposing the application via ClusterIP, NodePort, and LoadBalancers.Resilience: Implementing HPA and Readiness Probes.1. Environment Setup & ConfigurationBefore deploying, the environment was configured to bridge the gap between local development (WSL2) and the cloud.Bash# Update local package index
sudo apt-get update

# Verify WSL2 and Docker connectivity
docker ps

# Configure AWS CLI (Used for EKS provisioning)
aws configure
2. Cluster Provisioning I used eksctl to provision a managed Kubernetes cluster on AWS, ensuring high availability across multiple Availability Zones.Bash# Create an EKS cluster
eksctl create cluster --name k8s-pro-cluster --region us-east-1 --nodegroup-name standard-nodes --node-type t3.medium --nodes 2

# Verify node status
kubectl get nodes
3. Deploying the ApplicationUsing a declarative approach, we defined the "Desired State" using YAML manifests.The DeploymentYAMLapiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: web-container
        image: <your-docker-hub-repo>/app:v1
        ports:
        - containerPort: 80
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
Applying the ManifestsBash# Apply deployment
kubectl apply -f deployment.yaml

# Monitor the rollout
kubectl rollout status deployment/web-app
4. Mastering the "Kubernetes Onion" (Networking)Exposing the application through the various layers of the K8s network stack.LayerCommandPurposePod IPkubectl get pods -o wideInternal ephemeral communication.ClusterIPkubectl expose deployment web-app --type=ClusterIPStable internal load balancing.NodePortkubectl expose deployment web-app --type=NodePortOpening a port on every Node (Port range 30000-32767).LoadBalancerkubectl expose deployment web-app --type=LoadBalancerIntegration with AWS ELB for public access.5. Scalability & Self-HealingTesting the "Control Loop" by simulating spikes and failures.Horizontal Pod Autoscaling (HPA)Bash# Autoscale based on CPU usage
kubectl autoscale deployment web-app --cpu-percent=50 --min=2 --max=10

# View HPA status
kubectl get hpa
Testing ResilienceBash# Simulate a pod crash by deleting a pod manually
kubectl delete pod <pod-name>

# Observe the Control Loop instantly recreating the pod
kubectl get pods -w
6. Troubleshooting & Lessons LearnedWSL2 Networking: Resolved connectivity timeouts by adjusting the resolv.conf and ensuring Docker Desktop was correctly integrated with the WSL2 distro.Readiness Probes: Learned that if the probe path (e.g., /health) isn't explicitly defined in the app code, the LoadBalancer will mark the pod as "Unhealthy" and refuse traffic.Quota Management: Encountered AWS EC2 limits; resolved by cleaning up unused volumes and switching regions.ConclusionKubernetes is more than a tool; it's a paradigm shift in how we manage "Reality vs. Design." By mastering the Control Loop and Networking layers, we ensure our applications are not just running, but are resilient, scalable, and self-fixing.
