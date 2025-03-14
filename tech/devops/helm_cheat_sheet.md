# ⛵ Helm Cheat Sheet: Commands, Templates & Best Practices

This cheat sheet provides **essential Helm CLI commands, Helm chart creation steps, and templating examples** for Kubernetes deployments.

📌 **Helm CLI Reference**: [Helm Commands](https://helm.sh/docs/helm/)  
📌 **Helm Chart Template Guide**: [Helm Templating](https://helm.sh/docs/chart_template_guide/)  
📌 **Helm Repository Management**: [Helm Repos](https://helm.sh/docs/helm/helm_repo/)  

---

## **1. Helm CLI Commands**  

### **1.1 Install & Verify Helm**  
```sh
curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version  # Verify installation
```

### **1.2 Add & Update Helm Repositories**  
```sh
helm repo add stable https://charts.helm.sh/stable
helm repo update  # Refresh repositories
helm search repo nginx  # Search for a chart
```

### **1.3 Install, List & Uninstall Releases**  
```sh
helm install my-nginx stable/nginx --set service.type=LoadBalancer
helm list  # View installed Helm releases
helm uninstall my-nginx  # Remove a Helm release
```

🔗 **More Helm CLI Commands**: [Helm CLI Reference](https://helm.sh/docs/helm/)  

---

## **2. Creating a Helm Chart from Scratch**  

### **2.1 Create a New Chart**
```sh
helm create mychart
```

### **2.2 Helm Chart Structure**
```
mychart/
  ├── charts/        # Dependencies
  ├── templates/     # Kubernetes YAML templates
  ├── values.yaml    # Default configuration values
  ├── Chart.yaml     # Chart metadata
```

### **2.3 Define Chart Metadata (Chart.yaml)**
```yaml
apiVersion: v2
name: mychart
description: A sample Helm chart
version: 0.1.0
appVersion: 1.0.0
```

🔗 **Helm Chart Development**: [Helm Chart Guide](https://helm.sh/docs/topics/charts/)  

---

## **3. Using Helm Templates**  

### **3.1 Dynamic Deployment Template (templates/deployment.yaml)**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp-container
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: 80
```

### **3.2 Configurable Values (values.yaml)**
```yaml
replicaCount: 2
image:
  repository: nginx
  tag: latest
service:
  type: ClusterIP
  port: 80
```

### **3.3 Rendering & Debugging Templates**
```sh
helm template mychart  # Render templates without installing
helm lint mychart  # Validate Helm chart syntax
```

🔗 **Helm Templating Guide**: [Helm Templates](https://helm.sh/docs/chart_template_guide/)  

---

## **4. Managing Helm Releases**  

### **4.1 Upgrade an Existing Release**
```sh
helm upgrade myrelease ./mychart --set replicaCount=3
```

### **4.2 Rollback to a Previous Version**
```sh
helm rollback myrelease 1
```

### **4.3 Dry-Run Before Applying Changes**
```sh
helm upgrade myrelease ./mychart --dry-run --debug
```

🔗 **Helm Release Management**: [Helm Upgrade & Rollback](https://helm.sh/docs/helm/helm_upgrade/)  

---

## **5. Helm Security & Best Practices**  

### **5.1 Secure Helm with Role-Based Access Control (RBAC)**
```sh
kubectl create serviceaccount helm-user
kubectl create clusterrolebinding helm-user-binding --clusterrole=cluster-admin --serviceaccount=default:helm-user
```

### **5.2 Use Secrets for Sensitive Data**
```yaml
env:
  - name: DATABASE_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: password
```

🔗 **Helm Security Guide**: [Helm Security](https://helm.sh/docs/topics/security/)  

---

## **6. Helm in CI/CD Pipelines**  

### **6.1 Package & Push Helm Charts**
```sh
helm package mychart  # Package chart into a tar.gz file
helm push mychart-0.1.0.tgz myrepo  # Push chart to repo
```

### **6.2 Deploy in CI/CD Pipelines (GitHub Actions Example)**
```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Install Helm
        run: curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
      - name: Deploy Helm Chart
        run: helm upgrade --install myrelease ./mychart
```

🔗 **Helm & CI/CD**: [Helm in GitHub Actions](https://github.com/helm/helm/blob/main/docs/helm)  

---

## **7. Helm Troubleshooting & Debugging**  

### **7.1 Check Helm Release Status**
```sh
helm status myrelease
```

### **7.2 Inspect Generated Kubernetes Manifests**
```sh
helm get manifest myrelease
```

### **7.3 Debugging Chart Issues**
```sh
helm template mychart | kubectl apply --dry-run=client -f -
helm lint mychart  # Check for template errors
```

🔗 **Helm Debugging**: [Helm Troubleshooting Guide](https://helm.sh/docs/chart_template_guide/debugging/)  

---

### **Final Thoughts**  
Helm simplifies **Kubernetes deployments** by providing **templated, reusable configurations**. Whether deploying **microservices, managing CI/CD pipelines, or handling complex Kubernetes apps**, Helm **streamlines workflows and improves maintainability**.

### **Happy Deploying with Helm! ⛵🚀**  
