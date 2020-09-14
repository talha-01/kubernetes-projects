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
The command below will create a Dockerfile with the instructions of pulling python:alpine image, copying all the files and folders in this directory into the image, installing flask, opening port 80, and running the application. The exposed port and the port that is specified in your app file should match.
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
Make sure that you have a repository in your Docker Hub account. If not, you need to create a repository on Docker Hub. Alternatively, you can use other registries such as AWS ECR. However, you need to follow thier instructions for the login and push process if you choose so.

```bash 
docker push talhas/roman-converter-app:v1
```
The steps above can be completed using any computer, however, you should use the Kubernetes master node for the rest of the tutorial.

## Create Kubernetes Pod
The commands below will create a pod with roman-converter-app running inside at port 80, which is the same port that was exposed when creating the image above.
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
The commands below will create a Kubernetes service object for the pods labeled with `app: roman-app`, expose the application outside at the port 30010. 
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
You can use the commands below on the master node to check your pod and service if both are running.
```bash
kubectl get nodes
kubectl get services
```

## Check your website from the master node
```bash
curl localhost:30010
```

## Check your website from browser
http://\<PublicIp of master node\>:30010

Make sure that the master node's security group allows inbound traffic at the port 30010!




