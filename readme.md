
# Deploying YugabyteDB

## Overview

This project involves deploying YugabyteDB on Kubernetes using StatefulSet to ensure high availability, persistence, and scalability. The configuration will include defining the StatefulSet for YugabyteDB, setting up persistent storage, and exposing the database through a Kubernetes Service.

## Objectives

1.  **Deploy YugabyteDB** using StatefulSet for high availability and data persistence.
2.  **Manage storage** using PersistentVolumeClaims.
3.  **Expose the database** through a Kubernetes Service for internal communication.

## 1. Installation

**Prerequisites:**

-   Kubernetes cluster (e.g., Minikube, cloud provider, etc.)
-   `kubectl` CLI tool installed
-   Docker installed

### 1.1 Install Minikube (Optional, if you don’t have a Kubernetes cluster)

Follow the official Minikube installation guide to install Minikube on your system.

### 1.2 Start Minikube


```
minikube start
``` 

## 2. Kubernetes Configuration

### 2.1 Define the YugabyteDB StatefulSet and Service

**File: `yugabyte-db-statefulset.yaml`**



```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: yugabyte-db
spec:
  serviceName: "yugabyte-db"
  replicas: 3
  selector:
    matchLabels:
      app: yugabyte-db
  template:
    metadata:
      labels:
        app: yugabyte-db
    spec:
      containers:
      - name: yugabyte
        image: yugabytedb/yugabyte:latest
        ports:
        - containerPort: 5433
        volumeMounts:
        - name: data
          mountPath: /mnt/data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
``` 

**File: `yugabyte-db-service.yaml`**



```yaml
apiVersion: v1
kind: Service
metadata:
  name: yugabyte-db
spec:
  ports:
  - port: 5433
    targetPort: 5433
  clusterIP: None
  selector:
    app: yugabyte-db
``` 

### 2.2 Define PersistentVolume and PersistentVolumeClaim (Optional, for local storage setup)

**File: `yugabyte-db-pv.yaml`**



```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: yugabyte-db-pv
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 10Gi
  hostPath:
    path: /mnt/data
  storageClassName: manual
``` 

**File: `yugabyte-db-pvc.yaml`**



```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: yugabyte-db-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: manual
``` 

## 3. Apply the Configuration

1.  **Create PersistentVolume and PersistentVolumeClaim (if using local storage):**

    
    ```bash
    kubectl apply -f yugabyte-db-pv.yaml
    kubectl apply -f yugabyte-db-pvc.yaml
    ``` 
    
2.  **Deploy YugabyteDB StatefulSet:**

    
    ```bash
    kubectl apply -f yugabyte-db-statefulset.yaml
    ``` 
    
3.  **Create YugabyteDB Service:**

    
    ```bash
    kubectl apply -f yugabyte-db-service.yaml
    ``` 
    

## 4. Verify the Deployment

1.  **Check the Pods:**

    
    ```bash
    kubectl get pods
    ``` 
    
    You should see pods with names starting with `yugabyte-db` in a `Running` state.
    
2.  **Check the Service:**

    
    ```bash
    kubectl get services
    ``` 
    
    You should see a service named `yugabyte-db` exposing port 5433.
    

## 5. Accessing YugabyteDB

To access YugabyteDB from within the Kubernetes cluster, use the service name `yugabyte-db` and port `5433`. You can connect to the database using a client or application that supports PostgreSQL connections.

**Example Connection String:**

`postgresql://<username>:<password>@yugabyte-db:5433/<database>` 

Replace `<username>`, `<password>`, and `<database>` with your YugabyteDB credentials and database name.

## 6. Cleanup

To delete the deployment and associated resources:

1.  **Delete the Service:**

    
    ```bash
    kubectl delete -f yugabyte-db-service.yaml
    ``` 
    
2.  **Delete the StatefulSet:**

    
    ```bash
    kubectl delete -f yugabyte-db-statefulset.yaml
    ``` 
    
3.  **Delete PersistentVolume and PersistentVolumeClaim (if using local storage):**

    
    ```bash
    kubectl delete -f yugabyte-db-pvc.yaml
    kubectl delete -f yugabyte-db-pv.yaml
    ```