# K8s-mongodb-demo

This repository was created in a process of learning Kubernetes.

Originaly following these video tutorials, creating cluster with Mongo DB and Mongo Express deployments:

https://youtu.be/s_o8dwzRlu4

https://youtu.be/X48VuDVv0do

Later, Namespace and Ingress for Mongo Express added.

The Ingress doesn't work in WSL & Minikube setup though as explained here: https://stackoverflow.com/questions/76470764/why-i-cant-get-access-to-app-from-browser-with-kubernetes-minikube

## How to start

### Prereqs

These need to be installed to start the cluster locally:
- docker (https://www.docker.com/)
- minikube (https://minikube.sigs.k8s.io/docs/start/)
- kubectl (https://kubernetes.io/docs/reference/kubectl/)

### Steps

1. Start minikube by running
   ```
   minikube start
   ```
   Wait for it to complete and run
   ```
   minikube status
   ```
   to check if it works. You should get something like
   ```
   minikube
   type: Control Plane
   host: Running
   kubelet: Running
   apiserver: Running
   kubeconfig: Configured
   ```
2. Enable minikube ingress addon
   ```
   minikube addons enable ingress
   ```
   Check it by running
   ```
   kubectl get namespace
   ```
   You should see `ingress-nginx` in the namespaces list
   ```
   NAME              STATUS   AGE
   default           Active   9m36s
   ingress-nginx     Active   2m
   kube-node-lease   Active   9m36s
   kube-public       Active   9m36s
   kube-system       Active   9m36s
   ```
3. Create namespace for mongodb resources in the cluster
   ```
   kubectl apply -f mongodb-namespace.yaml
   ```
   Now you should also see `mongodb` in the namespaces list after running `kubectl get namespace` again
   ```
   NAME              STATUS   AGE
   default           Active   11m
   ingress-nginx     Active   4m12s
   kube-node-lease   Active   11m
   kube-public       Active   11m
   kube-system       Active   11m
   mongodb           Active   8s
   ```
4. Create mongodb secret and config resources
   ```
   kubectl apply -f mongodb-secret.yaml
   kubectl apply -f mongodb-config.yaml
   ```
   Check them by running (`-n mongodb` means we want to see resources in mongodb namespace)
   ```
   kubectl get secret -n mongodb
   kubectl get configmap -n mongodb
   ```
   You should see `mongodb-secret` and `mongodb-config`
   ```
   NAME             TYPE     DATA   AGE
   mongodb-secret   Opaque   2      22s
   ```
   ```
   NAME               DATA   AGE
   kube-root-ca.crt   1      2m18s
   mongodb-config     1      29s
   ```
5. Create the actual mongo database deployment and it's service
   ```
   kubectl apply -f mongodb.yaml
   ```
   Run
   ```
   kubectl get all -n mongodb
   ```
   You should see a pod, service, deployment, and replicaset for mongodb
   ```
   NAME                                      READY   STATUS              RESTARTS   AGE
   pod/mongodb-deployment-5ff4566dc4-vts9l   0/1     ContainerCreating   0          25s
   
   NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)     AGE
   service/mongodb-service   ClusterIP   10.101.159.68   <none>        27017/TCP   25s
   
   NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
   deployment.apps/mongodb-deployment   0/1     1            0           25s
   
   NAME                                            DESIRED   CURRENT   READY   AGE
   replicaset.apps/mongodb-deployment-5ff4566dc4   1         1         0       25s
   ```
   It might not be ready yet (0/1). Give it some more time to spin up and try again.
6. Similarly, create mongodb express
   ```
   kubectl apply -f mongodb-express.yaml
   ```
   And check it by running get all for the namespace
   ```
   kubectl get all -n mongodb
   ```
   There should be a pod, service, deployment, and replicaset for mongodb-express
   ```
   NAME                                      READY   STATUS         RESTARTS   AGE
   pod/mongodb-deployment-5ff4566dc4-vts9l   1/1     Running        0          3m25s
   pod/mongodb-express-b865f59d5-rzfr8       0/1     Running        0          16s
   
   NAME                              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
   service/mongodb-express-service   ClusterIP   10.103.201.180   <none>        8081/TCP    16s
   service/mongodb-service           ClusterIP   10.101.159.68    <none>        27017/TCP   3m25s
   
   NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
   deployment.apps/mongodb-deployment   1/1     1            1           3m25s
   deployment.apps/mongodb-express      0/1     1            0           16s
   
   NAME                                            DESIRED   CURRENT   READY   AGE
   replicaset.apps/mongodb-deployment-5ff4566dc4   1         1         1       3m25s
   replicaset.apps/mongodb-express-788947c4bc      1         1         0       16s
   ```
   Again, it might need more time to start and be ready.
7. Run describe command for the mondodb-express pod. You need to copy the pod name with it's hash. So in our case it's
   ```
   kubectl describe pod/mongodb-express-b865f59d5-rzfr8 -n mongodb
   ```
   You should get information about the pod. The status should be `Running` and port should be `8081:TCP` (in case you did not change it in the yaml file).
   
   Let's check if we can connect to the express running on the pod. To do it run (change the hash in the pod name accordingly) and keed to command running.
   ```
    kubectl port-forward mongodb-express-b865f59d5-rzfr8 -n mongodb 8081:8081
   ```
   You should see something like
   ```
   Forwarding from 127.0.0.1:8081 -> 8081
   Forwarding from [::1]:8081 -> 8081
   ```
   Which means port 8881 from localhost is forwarded to the pod port 8081 in the cluster. Open a browser and navige to `localhost:8081`. It should ask you for username and password. The container was created with default which are (at the time of writing this) `admin` and `pass`. After entering those you should get Mongo Express UI. Close the port-forward command by `ctrl+c`.
8. Create ingress resource for mongo express
   ```
   kubectl apply -f mongodb-express-ingress.yaml
   ```
   Check it by running
   ```
   kubectl get ingress -n mongodb
   ```
   You should see
   ```
   NAME                      CLASS   HOSTS                       ADDRESS   PORTS   AGE
   mongodb-express-ingress   nginx   mongodb-express.k8s.local             80      19s
   ```
9. Run the following command to see if mongo express is accessible through the ingress (unfortunately, this does not work in WSL + minikube setup)
   ```
   curl --resolve "mongodb-express.k8s.local:80:$( minikube ip )" -i http://mongodb-express.k8s.local
   ```
