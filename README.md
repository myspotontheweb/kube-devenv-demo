# kube-devenv-demo

## Using the minikube docker engine

The first workflow is inspired by the following article

http://kubecloud.io/minikube-workflows/

First start an instance of minikube

   cd nginx-local
   minikube start

Deploy the application

   make create

Make some changes and run the build again

   make local 


