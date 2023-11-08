# Kubernetes tutorial

* From: [https://youtu.be/s_o8dwzRlu4?si=WPX_LFJVTE4SCipJ](https://youtu.be/s_o8dwzRlu4?si=WPX_LFJVTE4SCipJ)
* Repo: [https://gitlab.com/nanuchi/k8s-in-1-hour](https://gitlab.com/nanuchi/k8s-in-1-hour)

## 1. miniKube & kubectl

* [https://minikube.sigs.k8s.io/docs/start/](https://minikube.sigs.k8s.io/docs/start/)

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

sudo install minikube-linux-amd64 /usr/local/bin/minikube

# start Minikube and check status
minikube start
minikube status

# get minikube node's ip address
minikube ip

# get basic info about k8s components
kubectl get node
kubectl get pod
kubectl get svc
kubectl get all

# get extended info about components
kubectl get pod -o wide
kubectl get node -o wide

# get detailed info about a specific component
kubectl describe svc $SVC_NAME
kubectl describe pod $POD_NAME

# get application logs
kubectl logs $POD_NAME

# stop your Minikube cluster
minikube stop
```

## 2. Deploying a webApp

### 2.1. Configuration files

#### 2.1.1. ConfigMap

[https://kubernetes.io/docs/concepts/configuration/configmap/](https://kubernetes.io/docs/concepts/configuration/configmap/)

<details>
<summary>mongo-config.yaml</summary>

  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: mongo-config
  data:
    mongo-url: mongo-service
  ```
</details>

#### 2.1.2. Secret

[https://kubernetes.io/docs/concepts/configuration/secret/](https://kubernetes.io/docs/concepts/configuration/secret/)

<details>
<summary>mongo-secret.yaml</summary>

  ```yaml
  apiVersion: v1
  kind: Secret
  metadata:
    name: mongo-secret
  type: Opaque
  data:
    mongo-user: bW9uZ291c2VyCg==          # base64 encoded string for "mongouser"
    mongo-password: bW9uZ29wYXNzd29yZAo=  # base64 encoded string for "mongopassword"
  ```
</details>

#### 2.1.3. Deployment & Service

[https://kubernetes.io/docs/concepts/workloads/controllers/deployment/](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

[https://kubernetes.io/fr/docs/concepts/services-networking/service/](https://kubernetes.io/fr/docs/concepts/services-networking/service/)

<details>
<summary>mongo.yaml</summary>

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: mongo-deployment
    labels:
      app: mongo
  spec:
    replicas: 1 # because it's database
    selector:
      matchLabels:
        app: mongo
    template:
      metadata:
        labels:
          app: mongo
      spec:
        containers:
        - name: mongodb
          image: mongo:5.0
          ports:
          - containerPort: 27017
          env:
          - name: MONGO_INITDB_ROOT_USERNAME
            valueFrom:
              secretKeyRef:
                name: mongo-secret
                key: mongo-user
          - name: MONGO_INITDB_ROOT_PASSWORD
            valueFrom:
              secretKeyRef:
                name: mongo-secret
                key: mongo-password
  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: mongodb-service
  spec:
    selector:
      app: MyApp
    ports:
      - protocol: TCP
        port: 27017
        targetPort: 27017
  ```

</details>

<details>
<summary>webapp.yaml</summary>

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: webapp-deployment
    labels:
      app: webapp
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: webapp
    template:
      metadata:
        labels:
          app: webapp
      spec:
        containers:
        - name: webapp
          image: nanjanashia/k8s-demo-app:v1.0
          ports:
          - containerPort: 3000
          env:
          - name: USER_NAME
            valueFrom:
              secretKeyRef:
                name: mongo-secret
                key: mongo-user
          - name: PASSWORD
            valueFrom:
              secretKeyRef:
                name: mongo-secret
                key: mongo-password
          - name: DB_URL
            valueFrom:
              configMapKeyRef:
                name: mongo-config
                key: mongo-url
  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: webapp-service
  spec:
    type: NodePort
    selector:
      app: MyApp
    ports:
      - protocol: TCP
        port: 3000
        targetPort: 3000
        nodePort: 30100
  ```

</details>

## 2.2. Deploying

```bash
kubectl get pod
kubectl apply -f mongo-config.yaml
kubectl apply -f mongo-secret.yaml
kubectl apply -f mongo.yaml
kubectl apply -f webapp.yaml


kubectl get all
kubectl get all -o wide

# ⚠ Known issue - Minikube IP not accessible
# If you can't access the NodePort service webapp with MinikubeIP:NodePort, execute the following command:
minikube service webapp-service

```

## 2.3. Cleaning

```bash
# kubectl stop
kubectl delete -f mongo-config.yaml
kubectl delete -f mongo-secret.yaml
kubectl delete -f mongo.yaml
kubectl delete -f webapp.yaml

kubectl get all

kubectl delete pods --all
kubectl delete services --all
kubectl delete deployments --all
minikube stop

```

## 2.3. Interacting

```bash
allo
```