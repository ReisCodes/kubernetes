# Kubernetes (K8s)

Kubernetes is an open source container orchestration engine for automating deployment, scaling, and management of containerized applications.

#### When not to use Microservices

Microservices architecture is a great way to manage a large project, thanks to it flexibility and scalibility. However, it also requires a high maintanance and a high cost. So if your project is of a small scale and there is no plan of expanding and growing in the future, then it might be much simplier and more cost effective to use a Monolith or N-tier architecture instead.

#### When not to use K8 - disadvantages

* You don't need K8 unless you have a large-scale application that run on multiple services that communicate with each other
* It involves steep learning curve, so it might take time to learn how to use it
* Requires learning of multiple tools
* Requires high maintanance and cost

### Kubernetes Deployment

1. Within docker we first need to enable kubernetes in the settings.

<br>

2. within our directory we need to create a YAML file 

- ```sudo nano nginx-deploy.yml```

3. In this file you need to add the information neccissary for kubernetes 

```
apiVersion: apps/v1 # which api to use for deployment

kind: Deployment # what kind of service/object do you want to create?

# what would you like to call it - name the service/object


metadata:
  name: nginx-deployment
spec:
  selector: 
    matchLabels:
      app: nginx
  replicas: 3

  template:
    metadata:
      labels:
        app: nginx
    
    spec: 
      containers:
      - name: nginx
        image: reiscodes/sparta-app:v2
        ports:
        - containerPort: 80

```

4. We then need to create the cluster 

- ```kubectl create -f nginx-deploy.yml```

5. Can check the clusters 

- ```kubectl get deploy```

6. we can check specific pods with:

- ```kubectl get pods```

7. But now we need to start a service, to expose the image on the broswer, we can check services with 

- ```kubectl get service```

- ``` kubectl delete deploy <name_of_deployement>```

### Creating a service

Now we have deployed our nodes we need to expose them using a service. 

1. firstly create a YAML file `nginx-service.yml`, within this add the following code:

```
# Select the type of API version and type of service/object
apiVersion: v1
kind: Service

# Metadata for name
metadata:
  name: nginx-svc
  namespace: default #sre
# Specification to include ports Selector to connect to the ?
spec:
  ports:
  - nodePort: 30001 # range is 30000-32768
    port: 80


    targetPort: 80

  # lets define the selector and label to connect to nginx deploment
  selector:
    app: nginx 

  # Creating NodePort type of deployement 
  type: NodePort 
```

2. Now within our terminal we can start this service 

- ``` create -f nginx-service.yml ```

3. We can verify this service exposed the node by navigating on our local host with port `30001`

![](nginx.png)


#### deploying a exposing node.js app 

We need to follow the above steps for nginx but ammending files to open ports to 3000 and change respective names 

1. Make a deploy YAML file called: `sparta-app-deploy.yml`

<br>

2. Within this file add:

```
apiVersion: apps/v1 # which api to use for deployment

kind: Deployment # what kind of service/object do you want to create?

# what would you like to call it - name the service/object


metadata:
  name: sparta-app-deployment
spec:
  selector: 
    matchLabels:
      app: node
  replicas: 3

  template:
    metadata:
      labels:
        app: node
    
    spec: 
      containers:
      - name: node
        image: reiscodes/sparta_app:v2
        ports:
        - containerPort: 80
```

3. We can then deploy this with 

- ```kubectl create -f sparta-app-deploy.yml ```

4. Then we need to make the service file, `sparta-app-service.yml`

<br>

5. Add the following code to this file

```
# Select the type of API version and type of service/object
apiVersion: v1
kind: Service

# Metadata for name
metadata:
  name: node-svc
  namespace: default #sre
# Specification to include ports Selector to connect to the ?
spec:
  ports:
  - nodePort: 30002 # range is 30000-32768
    port: 3000


    targetPort: 3000

  # lets define the selector and label to connect to nginx deploment
  selector:
    app: node 

  # Creating NodePort type of deployement 
  type: NodePort 
```

6. Starting this service with 

- ```kubectl create -f sparta-app-deploy.yml```

7. We can again verify this has been deployed and exposed succesfully by navigating to our localhost on port 30002. 

![](sparta_app.png)

#### Deploying node and mongodb pod on K8 cluster

1. first we need to create a pv or PersistnatVolume in a file called `mongo-pv.yml`

<br>

2. within this file we need to add the following script:

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

3. Then use `kubectl create -f mongo-pv.yml` to create the pv 

<br>

4. We then need to make a file to create a pvc (PersistantVolumeClient), Service and Deployment for mongodb called `mongo-deploy.yml`

<br>

5. Within this file add the following script:

```
# Creating the mongodb PVC

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 256Mi
      
# Creating the mongodb service   

--- 
# Select the type of API version and type of service/object
apiVersion: v1
kind: Service

# Metadata for name
metadata:
  name: mongodb-svc
  namespace: default #sre
# Specification to include ports Selector to connect to the ?
spec:
  ports:
   # expose the ports
  - port: 27017


    targetPort: 27017

  # lets define the selector and label to connect to nginx deploment
  selector:
    app: mongodb 


# Creating the mongodb deployment

---
apiVersion: apps/v1 # which api to use for deployment

kind: Deployment # what kind of service/object do you want to create?

# what would you like to call it - name the service/object


metadata:
  name: mongodb
spec:
  selector: 
    matchLabels:
      app: mongodb
  replicas: 3

  template:
    metadata:
      labels:
        app: mongodb
    
    spec: 
      containers:
      - name: mongodb
        image: mongo:latest
        ports:
        - containerPort: 27017
        volumeMounts:
        - name: storage
          mountPath: /data/db

      volumes:
      - name: storage
        persistentVolumeClaim:
          claimName: mongo-pvc
```

6. We can launch these all within our cluster with `kubectl create -f mongo-deploy.yml`

<br>

7. We can use the same deployment and service file for the node app we created previously, but to connect the node app to the database we need to add the env variable into the deployment

```
apiVersion: apps/v1 # which api to use for deployment

kind: Deployment # what kind of service/object do you want to create?

# what would you like to call it - name the service/object


metadata:
  name: sparta-app-deployment
spec:
  selector: 
    matchLabels:
      app: node
  replicas: 3

  template:
    metadata:
      labels:
        app: node
    
    spec: 
      containers:
      - name: node
        image: reiscodes/sparta_app:v2
        ports:
        - containerPort: 3000
        env:
        - name: MONGO_URL
          value: mongodb://mongodb:27017/posts
        imagePullPolicy: Always
```
8. similarly use the same create command to execute these files. 

9. After some time, you'll be able to see all the pods running with 

```kubectl get pods```
