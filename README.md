# DevOps Assignment

## 1) Dockerization
Create a Dockerfile with your base image of choice. Bonus points if only include what is needed to run the application.

Build the image locally and verify that the server works.

### a. create Dockerfile

```
FROM golang:1.19

WORKDIR /usr/src/app

# pre-copy/cache go.mod for pre-downloading dependencies and only redownloading them in subsequent builds if they change
COPY go.mod go.sum ./
RUN go mod download && go mod verify

COPY . .
RUN go build -v -o /usr/local/bin/app ./...

CMD ["app"]

```
### b. build and run the Docker image

```
docker build . -t test/node-web-app:1.0
docker run -p 9000:4444 -d test/node-web-app:1.0
```
## 2) CICD

Create a new project on gitlab, add application files then create .gitlab-ci.yaml file
```
stages:          # List of stages for jobs, and their order of execution
  - publish
variables:
  TAG_LATEST: rsaeedbakry/k8s:latest

publish:
  image: docker:latest
  stage: publish
  services:
    - docker:dind
  script:
    - docker build -t $TAG_LATEST .
    - docker login -u $CI_REGISTRY_NAME -p $CI_REGISTRY_PASSWORD
    - docker push $TAG_LATEST
```

### 3) Kubernetes
Create the necessary kubernetes resources in order to deploy the service to your kubernetes cluster.
### a. create eks cluster

```
eksctl create iamserviceaccount \                               
       --cluster=k8s \
       --namespace=kube-system \
       --name=alb-ingress-controller \
       --attach-policy-arn="arn:aws:iam::491661417761:policy/ALBIngressControllerIAMPolicy" \
       --override-existing-serviceaccounts \
       --approve
```
### b. create helm chart

```
helm create app
cd app
vim values.yaml
```

### c. install helm
```
helm install myapp . 
```

### 4) Resilience
Make sure that data is not lost in the case of a pod getting rolled.

create presistent volume on deplyment.yaml fole

```
          volumeMounts:
            - name: shared
              mountPath: /home
      volumes:
        - name: shared
          hostPath:
            path: /home/test1
```


### 5) Expose out of cluster
Expose the service outside of the cluster in a way that would be similar to what you would do in a production cluster.

on the helm chart, we create service of type loadbalancer, so you can access your application on 

```
a20916b4ea7124ec9aa90f804fd58642-933938337.eu-west-1.elb.amazonaws.com
```



```
k get svc
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP                                                              PORT(S)        AGE
kubernetes   ClusterIP      10.100.0.1      <none>                                                                   443/TCP        2d1h
myapp        LoadBalancer   10.100.190.83   a20916b4ea7124ec9aa90f804fd58642-933938337.eu-west-1.elb.amazonaws.com   80:30516/TCP   16s
```
