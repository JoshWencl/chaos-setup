# you need
minikube
helm
kubectl
docker


# build images
docker-compose -f docker-compose.yml -f local.docker-compose.yml -f docker-compose.init.yml build

# start images
docker-compose -f docker-compose.yml -f local.docker-compose.yml -f docker-compose.init.yml up


# Without compose
Create a network
```
docker network create appnetwork
```
Create and seed the mongo db
```
docker run -d --name mongo --net appnetwork -p 27017:27017 mongo
docker run -ti --name init --net appnetwork init
```

docker run -d --name api -p 3001:3001 --net appnetwork api
docker run -d --name web -p 3000:80 --net appnetwork web

gotta convert the deploy for helm

```
kompose convert
```

start minikube

```
minikube start
```

```
kubectl apply -f api-service.yaml,web-service.yaml,api-deployment.yaml,web-deployment.yaml
```

Setup chaos mesh
```
kubectl create namespace chaos-mesh

helm repo add chaos-mesh https://charts.chaos-mesh.org

helm install chaos-mesh chaos-mesh/chaos-mesh --version 0.4.0 --namespace chaos-mesh --set dashboard.create=true
 
kubectl get deployments,pods,services --namespace chaos-mesh

kubectl patch service chaos-dashboard -n chaos-mesh --type='json' --patch='[{"op": "replace", "path": "/spec/ports/0/nodePort", "value":31111}]'

kubectl apply -f network-delay.yaml
```
