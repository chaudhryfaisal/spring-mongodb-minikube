#https://cloud.google.com/community/tutorials/nginx-ingress-gke

#Setup Tiller RBAC Account
kubectl apply -f infra/rbac-config.yaml
helm init --service-account tiller --upgrade


#Install nginx ngress
helm install --name nginx-ingress stable/nginx-ingress --set rbac.create=true

#Deploy Mongo
kubectl apply -f mongo/

#Deploy App
kubectl apply -f app/

#Check Nginx Status
watch kubectl get service nginx-ingress-controller
#Extract ExternalIp
EXTERNAL_IP=$(kubectl get svc nginx-ingress-controller -o jsonpath="{.status.loadBalancer.ingress[0].ip}");echo $EXTERNAL_IP

#Configure Ingress
kubectl apply -f infra/ingress-resource.yaml

#Check Ingres Resources
kubectl describe ingress ingress-resource

#Get Students
curl $EXTERNAL_IP/fic/students
#Create
curl -H "Content-Type: application/json" -X POST -d '{"id":"1","name":"Name","description":"Programmer","courses":[{"id":"1","name":"Minikube on local","description":"Kubenetes","steps":[1]}]}' $EXTERNAL_IP/fic/student

#Get Students
curl $EXTERNAL_IP/fic/students