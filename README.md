# How to buidl and run
## 1. Build docker image
`docker build -t [tagname] .`
> docker build -t helloworld:v1.1 .

## 2. Run docker image
`docker run --name [SERVICE_NAME] -p 127.0.0.1:8080:8080 -d [tagname]`
> docker run --name my_hellow_world_server -p 127.0.0.1:8080:8080 -d helloworld:v1.1

## 3. Test connection
> curl http://localhost:8080

# How to push image to Container Repository
## 1. Tag image
`docker tag [tagname] gcr.io/[tagname]`
> docker tag helloworld:v1.1 gcr.io/helloworld:lastest

## 2. Push to container repository (gcr.io)
> docker push gcr.io/luiz-gcp-repository/helloworld:latest

(NOTE) The Container Resitry Settings should be set with Public access if it is
to be used for public. If this is the case, you need to change the settings
, Public access , [Container Registry host] , Visibility to *Public*.

## 3. Confirm images
> docker images
```
kazuyukif@carlos:~/dev/docker/first$ docker images
REPOSITORY                              TAG                 IMAGE ID       CREATED          SIZE
helloworld                              v1.1                92bcea9cc902   37 minutes ago   750MB
gcr.io/luiz-gcp-repository/helloworld   latest              92bcea9cc902   37 minutes ago   750MB
```

# How to deploy container to a Kubernetes cluster
## 0. Create the Kubernetes cluster
For this demonstration, Minikube is used.

## 1. Create the manifest file
`helloworld_deploy.yaml`
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: helloworld
  name: helloworld
spec:
  replicas: 1
  selector:
    matchLabels:
      app: helloworld
  template:
    metadata:
      labels:
        app: helloworld
    spec:
      containers:
      - image: gcr.io/luiz-gcp-repository/helloworld:latest
        name: hellogo
```
`helloworld_service.yaml`
```
apiVersion: v1
kind: Service
metadata:
  name: hellowold
  labels:
    app: hellowold
spec:
  type: NodePort
  selector:
    app: hellowold
  ports:
    - protocol: TCP
      targetPort: 8080
      port: 8080
      nodePort: 30080
```

## 2. Apply the manifest files
Set context so you can target the cluster to deploy the manifest.
> kubectl config get-contexts

This should list all available context and you need to pick the target. If this
is for Minikube, then it should be *minikube*.
`kubectl config set-context [target-context]`
> kubectl config set-context minikube

Once the target cluster is set, you now can deploy the manifest using the
folloiwng commands:

> kubectl apply -f helloworld_deploy.yaml

> kubectl apply -f helloworld_service.yaml

## 3. Confirm connection
> curl http://localhost:8080/


