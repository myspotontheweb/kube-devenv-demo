
HOST_IP=192.168.192.63

REPOSITORY=myspotonthewev
NAME=demo
TAG=local

REGISTRY=$(HOST_IP):5000
IMAGE_NAME=$(REGISTRY)/$(REPOSITORY)/$(NAME)
DEV_DOCKER=$(shell docker-machine config dev)

NPMRC=$(shell cat ~/.npmrc)

HELM_RELEASE=demo-dev

REPLICAS=3

#
# Global targets
#
build: build-image

init: init-registry init-dev init-k8s init-secrets

clean-all: clean-registry clean-dev clean-k8s

#
# Init targets
#
init-registry:
	docker run -d -p 5000:5000 --name registry registry:2

init-dev: 
	docker-machine create dev --engine-insecure-registry $(REGISTRY)

init-k8s:
	minikube start --insecure-registry $(REGISTRY)
	minikube addons enable ingress
	helm init

init-secrets:
	@docker-machine ssh dev "echo $(NPMRC) > npmrc"
	docker $(DEV_DOCKER) network create build || echo "Network exists"
	docker $(DEV_DOCKER) run -d --name secrets --network build -v /home/docker:/usr/share/nginx/html nginx:alpine

#
# Build targets
#
build-image:
	docker $(DEV_DOCKER) build --network build --rm -t $(IMAGE_NAME):$(TAG) .
	docker $(DEV_DOCKER) push  $(IMAGE_NAME):$(TAG)

#
# Deploy targets
#
deploy:
	helm install chart/$(NAME) --name $(HELM_RELEASE) --namespace $(NAME) --set image.repository=$(IMAGE_NAME),image.tag=$(TAG),replicaCount=$(REPLICAS)

#
# Clean targets
#
clean:
	helm delete --purge $(HELM_RELEASE)

clean-registry:
	docker rm -v -f registry 

clean-dev:
	docker-machine rm -f dev

clean-k8s:
	minikube delete

