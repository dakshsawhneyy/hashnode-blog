---
title: "Deploying 3-tier chat-app on MiniKube"
datePublished: Thu Apr 17 2025 02:17:45 GMT+0000 (Coordinated Universal Time)
cuid: cm9kqday1000009k4c0y6h2wh
slug: deploying-3-tier-chat-app-on-minikube
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1744849496906/8ce258ac-f1b9-4141-b6d4-96a9458e8873.png
tags: docker, kubernetes, dockerhub, minikube, three-tier-architecture

---

## Installing MiniKube on Local

```bash
minikube start --driver=docker
minikube status # Verify if minikube is present
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744850254380/8f091466-4505-405f-bf4f-fa88d46a6e2e.png align="center")

---

## Now, pushing docker images of frontend and backend on DockerHub

In order to make deployment for frontend and backend, we need their images retrieved from Docker Hub, so after building images, we need to push them to Docker Hub

### Login into CLI with your docker credentials

```bash
docker login
```

### Building frontend image and pushing it to Docker Hub

```bash
docker build -t dakshsawhneyy/chatapp-frontend ./frontend
docker push dakshsawhneyy/chatapp-frontend
```

### Building backend image and pushing it to Docker Hub

```bash
docker build -t dakshsawhneyy/chatapp-backend ./backend
docker push dakshsawhneyy/chatapp-backend
```

---

## Starting with K8S

### Creating Namespace of name chat-app

```bash
kubectl create namespace chat-app
```

### Creating Frontend-Deployment.yml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
  namespace: chat-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend-cont
        image: dakshsawhneyy/chatapp-frontend
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 80
        env:
        - name: NODE_ENV
          value: production
```

### Creating Frontend-Deployment.yml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
  namespace: chat-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend-cont
        image: dakshsawhneyy/chatapp-backend
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 5001
        env:
        - name: NODE_ENV
          value: production
        - name: MONGODB_URI        # they are present in env of Docker File
          value: "mongodb://mongoadmin:secret@mongodb"
        - name: JWT_SECRET
          value: your_jwt_secret    
        - name: PORT
          value: 5001
```

### Creating MongoDB-Deployment.yml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-deployment
  namespace: chat-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
      - name: mongo-cont
        image: mongo:latest
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          value: root
        - name: MONGO_INITDB_ROOT_PASSWORD
          value: root
```

---

## Creating Persistent-Volume and Persistent-Volume-Claim for MongoDB

### Creating PV

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongo-pv
  namespace: chat-app
spec:
  capacity:
    storage: 2Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  hostPath:
    path: /data   # for MiniKube
```

### Creating PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc
  namespace: chat-app
spec:
  resources:
    requests:
      storage: 2Gi
  accessModes:
    - ReadWriteOnce
```

### Now, putting these in deployment of MongoDB

Updating MongoDB Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-deployment
  namespace: chat-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
      - name: mongo-cont
        image: mongo:latest
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          value: root
        - name: MONGO_INITDB_ROOT_PASSWORD
          value: root
      volumes:
      - name: mongo-data
        persistentVolumeClaim:
          claimName: mongodb-pvc
```

Apply Both PV and PVC

```bash
kubectl apply -f mongodb-pv.yml
kubectl get pv -n chat-app
kubectl apply -f mongodb-pvc.yml
kubectl get pv -n chat-app
```

---

### Apply Deployment of MongoDB

```bash
kubectl apply -f mongodb-deployment.yml
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744852644442/a17961d7-ffbd-4803-8a61-331587e93011.png align="center")

Error coming, we need to create service for backend, then we’ll able to run frontend pod

### Creating backend-service

```bash
apiVersion: v1
kind: Service
metadata:
  name: backend
  namespace: chat-app
spec:
  selector:
    app:  backend
  ports:
  - port: 5001
    targetPort: 5001
```

### Creating frontend-service

```bash
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: chat-app
spec:
  selector:
    app:  frontend
  ports:
  - port: 80
    targetPort: 80
```

### Creating MongoDB Service

```bash
apiVersion: v1
kind: Service
metadata:
  name: mongo-svc
  namespace: chat-app
spec:
  selector:
    app:  mongo
  ports:
  - port: 27017
    targetPort: 27017
```

Apply ALL

---

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744854848330/bddd601a-0427-457f-b7af-0fd98acbf4b9.png align="center")

And Chat-App is running fine on PORT 80 🎉🎉🎉🎉