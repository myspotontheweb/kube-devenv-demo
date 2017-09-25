
#
# Immediate vars
#
NAME:=demo

#
# Deferred vars
#
BASE_DOMAIN=$(shell minikube ip).nip.io

#
# Global targets
#
all:

init1: init-k8s

init2: init-tools

clean-all: clean-k8s

#
# Init targets
#
init-k8s:
	minikube start 
	minikube addons enable ingress
	minikube addons enable registry
	helm init

init-tools:
	helm install --namespace kube-system charts/registry-ui --set ingress.hosts[0]=registry.$(BASE_DOMAIN)
	helm install --namespace jenkins stable/jenkins --set Master.AdminPassword=admin,Master.ServiceType=ClusterIP,Master.HostName=jenkins.$(BASE_DOMAIN)

#
# Logging targets
#
log-jenkins:
	stern -n jenkins jenkins

log:
	stern $(NAME)

#
# Clean targets
#
clean-k8s:
	minikube delete || true

