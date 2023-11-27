# k8s-tutorial-01

Created for practicing K8s configuration as explained in https://youtu.be/s_o8dwzRlu4 and https://youtu.be/X48VuDVv0do

Later, namespace and Ingress for mongodb-express added.

The ingress doesn't work for WSL & Minikube though as explained here: https://stackoverflow.com/questions/76470764/why-i-cant-get-access-to-app-from-browser-with-kubernetes-minikube

## How to start
0. Install docker, minikube, kubectl.
1. `minikube start`
2. `minikube addons enable ingress`
3. `kubectl apply -f mongodb-namespace.yaml`
4. `kubectl apply -f mongodb-secret.yaml`
5. `kubectl apply -f mongodb-config.yaml`
6. `kubectl apply -f mongodb.yaml`
7. `kubectl apply -f mongodb-express.yaml`
8. `kubectl apply -f mongodb-express-ingress.yaml`
9. `curl --resolve "mongodb-express.k8s.local:80:$( minikube ip )" -i http://mongodb-express.k8s.local`
