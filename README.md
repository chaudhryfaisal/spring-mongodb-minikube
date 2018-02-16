# Spring, MongoDB and Kubernetes
The following document describes the deployment of a basic Spring API Service and MongoDB web stack on Kubernetes. Currently this example does not use replica sets for MongoDB.

Using Replication Controllers for building MongoDB

# Requirements
    Java: JDK 1.8
    Maven Build
    Docker and Minikube on local

### Kubernetes
Using Minikube: build on local

#### Minikube
##### Setup & Start
```
minikube start --vm-driver hyperkit
minikube addons enable ingress
```

##### Stop & Delete
```
minikube stop
minikube delete
```

### APIs Service
##### Build API Service
```
cd api-service && mvn package -DskipTests=True
```

##### Create docker images for APIs Service and push Docker Hub
```
docker build -t chaudhryfaisal/api-service api-service/
docker push chaudhryfaisal/api-service
```

##### Create Persitant Volume
```
kubectl create -f manifests/mongo-pv.yml
kubectl get pv
```

##### Create Persitant Volume Claim
```
kubectl create -f manifests/mongo-pvc.yml
kubectl get pvc
```

##### Create MongoDB Controller
```
kubectl create -f manifests/mongo-controller.yml
```

##### Create MongoDB Service
```
kubectl create -f manifests/mongo-service.yml
```

##### Create API Service Deployment
```
kubectl create -f manifests/api-deploy.yml
```

##### Create API Service
```
kubectl create -f manifests/api-service.yml
```

##### Check Services
```
kubectl describe services
```

```
kubectl describe replicationcontrollers/mongo-controller
```

```
kubectl exec -it mongo-controller-vbqf4 -c mongo bash
```

##### Create Ingress
```
kubectl create -f manifests/ingress.yml
```

##### Add app.k8.fic.com into /etc/hosts
```
sudo sed -i '' '/app.k8.fic.com/d' /etc/hosts
echo "$(minikube ip) app.k8.fic.com " | sudo tee -a /etc/hosts
```

### Notes
##### Config MongoDB
Check config MongoDB in api-service/src/main/resources/application.properties

##### Create API Service Replication Controller
https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux_atomic_host/7/html/getting_started_with_kubernetes/get_started_orchestrating_containers_with_kubernetes#exploring_kubernetes_pods

We can create API Service Replication Controller instead of a API Service Deployment.
Please checkout Git branch: replication-api-service

```
kubectl create -f manifests/api-controller.yml
```

### Usage
##### Create a student

POST http://app.k8.fic.com/student
```
{
	"id": "1",
	"name": "Name",
	"description": "Programmer",
	"courses": [{
		"id": "1",
		"name": "Minikube on local",
		"description": "Kubenetes",
		"steps": [1]
	}]
}
```
```bash
curl -H "Content-Type: application/json" -X POST -d '{"id":"1","name":"Name","description":"Programmer","courses":[{"id":"1","name":"Minikube on local","description":"Kubenetes","steps":[1]}]}' http://app.k8.fic.com/student

```

##### Get all student
```bash
curl -v app.k8.fic.com/students
```

### Reference
[Running a MEAN stack on Google Cloud Platform with Kubernetes](https://medium.com/google-cloud/running-a-mean-stack-on-google-cloud-platform-with-kubernetes-149ca81c2b5d)

[Node.js and MongoDB on Kubernetes](https://github.com/kubernetes/examples/tree/master/staging/nodesjs-mongodb)

[GET STARTED PROVISIONING STORAGE IN KUBERNETES](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux_atomic_host/7/html/getting_started_with_kubernetes/get_started_provisioning_storage_in_kubernetes)
