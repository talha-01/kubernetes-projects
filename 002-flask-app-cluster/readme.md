# How to deploy a simple Flask app with Kubernetes
 
In this tutorial, we will create and push our docker image on our local computer, and deploy it using a Kubernetes cluster initialized on EC2 instances.
You can follow the first tutorial to create a Kubernetes cluster with master and worker nodes.

## Download the application from GitHub
```bash
wget -P templates https://raw.githubusercontent.com/talha-01/aws-projects/master/001-roman-numerals-converter/templates/index.html
wget -P templates https://raw.githubusercontent.com/talha-01/aws-projects/master/001-roman-numerals-converter/templates/result.html
wget https://raw.githubusercontent.com/talha-01/aws-projects/master/001-roman-numerals-converter/app.py
```
## Create the Dockerfile
```bash
cat << EOF > Dockerfile -
FROM python:alpine
COPY . /app
WORKDIR /app
RUN pip install flask
EXPOSE 80
CMD python app.py
EOF
```

## Build your image
When creating the image, you sould use the following command.
```bash
docker build -t <DockerHubRepoName/ImageName>:<Version> <DockerfileDirectory>
```
```bash
docker build -t talhas/roman-converter-app:v1 .
```

## Login your Docker Hub account and push the image to your repository
```bash
docker login
```
```bash 
docker push talhas/roman-converter-app:v1
```

## Create Kubernetes Pod
```bash
cat << EOF > roman-app-pod.yaml -
apiVersion: v1
kind: Pod
metadata:
  name: roman-app
  labels:
    app: roman-app
spec:
  containers:
  - name: roman-app
    image: talhas/roman-converter-app:v1
    ports:
    - containerPort: 80
EOF
```
```bash
kubectl apply -f roman-app-pod.yaml
```

## Create Kubernetes Service
```bash
cat << EOF > roman-app-service.yaml -
apiVersion: v1
kind: Service
metadata:
  name: roman-app-service
spec:
  type: NodePort
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30010
  selector:
    app: roman-app
EOF
```
```bash
kubectl apply -f roman-app-service.yaml
```
Check your pod and service if both are running.
```bash
kubectl get nodes
kubectl get services
```

## Check your website from the master node
```bash
curl localhost:30010
```

## Check your website from browser
http:\<PublicIp of master node\>:30010

Make sure that the master node's security group allows inbound traffic at the port 30010!




