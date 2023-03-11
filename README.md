# Prerequisities

    1. Install Virtualbox
    2. Install Minikube
       - Runs a single-node Kubernetes cluster inside a Virtual Machine (VM) (Virtualbox) 
       - Run a single-node Kubernetes cluster in DOcker (Hyper-V)
    3. Install kubectl


minikube config set driver virtualbox
minikube stop
minikube delete --all

minikube start

kubectl exec -it postgres-deployment-5f88678b76-kpl8b -n default -- /bin/bash