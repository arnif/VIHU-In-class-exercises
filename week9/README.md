# **Introduction to Kubernetes with Minikube**  

Minikube allows you to run a Kubernetes cluster locally, making it an excellent environment for learning Kubernetes basics. In this exercise, you will:  

✅ Set up Minikube and `kubectl`  
✅ Deploy the Docker container you built in Week 6  
✅ Expose the application and access it  
✅ Learn essential `kubectl` commands  

Let’s get started!  

---

## **Step 1: Install Prerequisites**  

Ensure the following are installed on your machine:  

### **1.1 Install Docker (Preferred) or VirtualBox**  
Minikube can run using different drivers, but we recommend Docker since you already used it in Week 6.  

- **Docker Desktop**: [Download and install Docker](https://www.docker.com/products/docker-desktop)  
- **VirtualBox (alternative)**: [Download VirtualBox](https://www.virtualbox.org/wiki/Downloads)  

### **1.2 Install kubectl**  
`kubectl` is the command-line tool to interact with Kubernetes.  
[Install kubectl](https://kubernetes.io/docs/tasks/tools/)  

### **1.3 Install Minikube**  
Minikube runs a lightweight Kubernetes cluster on your local machine.  
[Install Minikube](https://minikube.sigs.k8s.io/docs/start/)  

---

## **Step 2: Start Minikube**  

Once installed, open a terminal and start Minikube using the **Docker driver** (recommended):  

```sh
minikube start --driver=docker
```

Verify Minikube is running:

```sh
minikube status
```

You should see `Running`.

---

## **Step 3: Build Your Docker Image Inside Minikube**  

Since Minikube runs its own internal Docker daemon, you must build your Docker image inside Minikube’s environment.

1️⃣ **Configure Docker to use Minikube’s daemon:**  

```sh
eval $(minikube docker-env)
```

2️⃣ **Build your Week 6 image inside Minikube:**  

```sh
docker build -t my-app:latest .
```

3️⃣ **Verify Minikube can see your image:**  

```sh
docker images
```

Ensure `my-app` is listed.

---

## **Step 4: Create a Deployment in Kubernetes**  

Instead of manually running `kubectl create deployment`, we will use a **YAML file** to define our deployment correctly.

### **4.1 Create a `deployment.yaml` file**  

In your project folder, create a new file named **`deployment.yaml`** and add the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: my-app:latest
        imagePullPolicy: Never
        ports:
        - containerPort: 3000
```

📌 **Key Notes:**
- `imagePullPolicy: Never` ensures Kubernetes does not try to pull the image from a registry.
- `containerPort: 3000` matches the port used in Week 6.

### **4.2 Deploy the Application**  

Apply the deployment using:

```sh
kubectl apply -f deployment.yaml
```

Verify that the deployment is running:

```sh
kubectl get deployments
```

---

## **Step 5: Expose the Application**  

To make the application accessible, expose it as a **Service**:

```sh
kubectl expose deployment my-app --type=NodePort --port=3000
```

Verify the service is created:

```sh
kubectl get services
```

---

## **Step 6: Access Your Application**  

To get the external URL of your app:

```sh
minikube service my-app --url
```

Copy the output URL and open it in your browser. 🚀  

---

## **Step 7: Basic Kubernetes Commands**  

### **7.1 Check Running Pods**  

```sh
kubectl get pods
```

### **7.2 View Logs of Your Pod**  

```sh
kubectl logs -l app=my-app
```

### **7.3 Scale the Deployment (Increase Replicas)**  

```sh
kubectl scale deployment my-app --replicas=3
```

Check updated pods:

```sh
kubectl get pods
```

### **7.4 Delete the Deployment**  

```sh
kubectl delete deployment my-app
```

---

## **Step 8: Clean Up**  

When you are done, stop Minikube:

```sh
minikube stop
```

To delete everything:

```sh
minikube delete
```

---

## **Conclusion**  

🎉 You successfully:  
✅ Set up a local Kubernetes cluster with Minikube  
✅ Built and deployed your Docker image from Week 6  
✅ Exposed and accessed your application  
✅ Learned essential `kubectl` commands  

As a next step, try modifying your `deployment.yaml` by adding environment variables, mounting a volume, or scaling up your replicas dynamically. Happy learning! 🚀
