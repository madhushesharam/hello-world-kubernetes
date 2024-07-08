# hello-world-kubernetes


Deploy a Full Stack Web Application over Kubernetes

In this project, we'll deploy a web application over a Kubernetes cluster, with separate backend and frontend components. 
We'll also use ConfigMaps to share common information among the services via environment variables in our containers.




Kubernetes provides several distributions to create a cluster. For learning purposes, though, a complete cluster is not required. Instead, you can use any one of the following:

minikube
kind
Docker Desktop
kubeadm
k3's 

We will be using k3's for this excersize on mac 

```
brew install --cask multipass
multipass launch --name k3s --mem 4G --disk 40G
multipass shell k3s

ubuntu@k3s:~$ curl -sfL https://get.k3s.io | sh -
sudo su - 
root@primary:~# kubectl get nodes
NAME      STATUS   ROLES                  AGE   VERSION
primary   Ready    control-plane,master   45m   v1.29.6+k3s2

# intermediate devsetup
snap install docker
docker run hello-world # validate setup

docker --version 
Docker version 24.0.5, build ced0996
git version 
git version 2.43.0

```

We will be considering one of the e-learning commuinity built app for simplicity.

For this task, you are required to do the following:

Change the directory to /usercode/elearning.
Create a Docker image.
Push this image to Docker Hub.

Note: Please create an account on docker hub and use the credentials to login.
      https://docs.docker.com/security/for-developers/access-tokens/

      Use the docker build command to build the Docker image.
      Name the Docker image by using your Docker Hub username and the repository name.
      Log in to Docker Hub by using the docker login command.
      Push the image by using the docker push command.

```
docker login -u my-username -p my-password
docker build -t my-username/e_learning_repo:v1 .
docker push my-username/e_learning_repo:v1 

```

k3 cluster dev setup 

```
 kubectl create namespace dev 
 kubectl config set-context --current --namespace=dev
```


Deploy Backend , this case postgres db

database.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: database-deployment
spec:
  selector:
    matchLabels:
      project: kubernetes-project
      tier: backend
  template:
    metadata:
      labels:
        project: kubernetes-project
        tier: backend
        app: database
    spec: 
      containers:
      - image: postgres:latest
        name: postgres-image
        ports:
          - containerPort: 5432
        imagePullPolicy: Always
        # env: # intentionally commented
        # - name: POSTGRES_PASSWORD
        #   value: "password"
```

kubectl apply -f database.yaml



To deploy the database as a service, you have to create the Kubernetes service object. For that 
database-svc.yaml file.



```
apiVersion: v1
kind: Service
metadata:
  name: database-svc
spec:
  type: ClusterIP
  ports:
  - name: "database"
    port: 5432
    protocol: TCP
  selector:
    project: kubernetes-project
    tier: backend
    app: database
```


kubectl apply -f database-svc.yaml 
service/database-svc created
kubectl get svc
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
database-svc   ClusterIP   10.43.162.174   <none>        5432/TCP   6s



To deploy the frontend application, you have to create another deployment. For that, open the app.yaml file.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  selector:
    matchLabels:
      project: kubernetes-project
      app: application
      tier: frontend
  template:
    metadata:
      labels:
        project: kubernetes-project
        app: application
        tier: frontend
    spec:
      containers:
      - name: app-pod
        image: madhushesharam/e_learning_repo:v1
        imagePullPolicy: Always
        ports:
        - containerPort: 3000
```
