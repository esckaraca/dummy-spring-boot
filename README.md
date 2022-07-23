# Project Name: Dummy Spring Boot

Tools & Services:

- CI / Build Server - AWS EC2 Instance  ( Ubuntu 20.04, t2.medium)

  - Installed tools:
    - Docker
    - Jenkins
    - aws-cli
    - kubectl

- Local tools needed
  - kubectl
  - eksctl
  - aws-cli

## Part 1

### Application Test

cloned the repo

```bash
git clone https://gitlab.com/bcfmkubilay/dummy-spring-boot
```

build the application building image with docker/build/Dockerfile

```bash
docker build -t helloworld:0.0.1-build docker/build
```

create artifact

```bash
docker run --rm -v $PWD:/app helloworld:0.0.1-build
```

build the release images with /dummy-spring-boot/Dockerfile

```bash
docker build -t helloworld:0.0.1-release .
```

run the application with the release image

```bash
docker run -d -p 8080:8080 helloworld:0.0.1-release 
```

## Part 2

create dummy-cluster

```bash
eksctl create cluster --region eu-west-1 --node-type t2.medium --nodes 1 --nodes-min 1 --nodes-max 2 --node-volume-size 8 --name dummy-cluster
```

configure nginx-ingress controller aws provider

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.3.0/deploy/static/provider/aws/deploy.yaml
```

configure nginx-ingress

```bash
kubectl apply -f ingress.yml
```

ingress.yml

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-service
  annotations:
    kubernetes.io/ingress.class: 'nginx'
    nginx.ingress.kubernetes.io/use-regex: 'true'
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - http:
        paths:
          - path: /?(.*)
            pathType: Prefix
            backend:
              service:
                name: dummy-spring-svc
                port:
                  number: 8080

```

configure service

```bash
kubectl apply -f service.yml
```

service.yml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: dummy-spring-svc
  labels:
    name: dummy-spring-svc
spec:
  type: LoadBalancer 
  ports:
  - port: 8080  
    targetPort: 8080
  selector:
    app: dummy-spring

```

## Part 3

install Jenkins, AWS CLI, kubectl, Docker on EC2 Instance

create IAM User for accessing AWS ECR and AWS EKS services

add aws config

update kube config

```bash
aws eks --region eu-west-1 update-kubeconfig --name dummy-cluster
```

## Part 4

add Jenkins build job
add Jenkins deploy job

run Jenkins build job
run Jenkins deploy job

## Part 5

list ingress services

```bash
kubectl get ingress
```

```plain
a4361f175af46440c935a9cab46afd39-d42970367fc62a67.elb.eu-west-1.amazonaws.com
```
