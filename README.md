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

## Minikube

```
minikube start
```

Setup a secret for auth
```
kubectl create secret docker-registry regcred --docker-server=docker.pkg.github.com --docker-username=JoshWencl --docker-password=<your pat>
```
In the deployment for kubectl add the image pull secret attribute (kompose won't do this for you)

```

      imagePullSecrets:
      - name: regcred
 ```

```
kubectl apply -f api-service.yaml,web-service.yaml,mongo-deployment.yaml,init-deployment.yaml,api-deployment.yaml,web-deployment.yaml 
```


start the dashboard to see if everything is up
```
minikube dashboard
```

Connect to the web
```
minikube service web
```


Setup chaos mesh
```
kubectl create namespace chaos-mesh

helm repo add chaos-mesh https://charts.chaos-mesh.org

helm install chaos-mesh chaos-mesh/chaos-mesh --version 0.4.0 --namespace chaos-mesh --set dashboard.create=true
 
kubectl get deployments,pods,services --namespace chaos-mesh

kubectl patch service chaos-dashboard -n chaos-mesh --type='json' --patch='[{"op": "replace", "path": "/spec/ports/0/nodePort", "value":31111}]'

```

Apply the network test yaml
```
kubectl apply -f network-delay.yaml
```

Going to apply that network latency to the default namespace.

You can check the chaos dashboard 
![latency example](https://user-images.githubusercontent.com/25798273/129636140-b8710ef9-86f8-4a34-9542-987c7028086d.PNG)

Other examples can be found here
https://github.com/chaos-mesh/chaos-mesh/tree/master/examples
