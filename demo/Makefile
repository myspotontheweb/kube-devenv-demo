
HOST_IP:=192.168.192.63

NAME:=demo
REPOSITORY:=myspotontheweb
TAG:=dev-$(shell date '+%Y%m%d%H%M%S')

REGISTRY:=$(HOST_IP):5000
IMAGE_NAME:=$(REGISTRY)/$(REPOSITORY)/$(NAME)
WORKSPACE_DOCKER=$(shell docker-machine config workspace)

NPMRC:=$(shell cat ~/.npmrc)

HELM_RELEASE:=demo

#
# Global targets
#
build: build-image

init: init-registry init-workspace init-k8s init-secrets

clean-all: clean-registry clean-workspace clean-k8s

#
# Init targets
#
init-registry:
	docker run -d -p 5000:5000 --name registry registry:2.2.0
	docker run -d -p 8080:80 --name registry-ui --link registry:registry -e ENV_DOCKER_REGISTRY_HOST=registry -e ENV_DOCKER_REGISTRY_PORT=5000 konradkleine/docker-registry-frontend:v2

init-workspace: 
	docker-machine create workspace --engine-insecure-registry $(REGISTRY)

init-k8s:
	minikube start --insecure-registry $(REGISTRY)
	minikube addons enable ingress
	minikube addons disable registry
	helm init

init-secrets:
	@docker-machine ssh workspace "echo $(NPMRC) > npmrc"
	docker $(WORKSPACE_DOCKER) network create build || true
	docker $(WORKSPACE_DOCKER) run -d --name secrets --network build -v /home/docker:/usr/share/nginx/html nginx:alpine

#
# Build targets
#
build-image:
	docker $(WORKSPACE_DOCKER) build --network build --rm -t $(IMAGE_NAME):$(TAG) .
	docker $(WORKSPACE_DOCKER) push  $(IMAGE_NAME):$(TAG)

#
# Deploy targets
#
deploy: build-image
	helm upgrade --install $(HELM_RELEASE) chart/$(NAME) --namespace $(NAME) --set image.repository=$(IMAGE_NAME),image.tag=$(TAG),image.pullPolicy=Always

#
# Clean targets
#
clean:
	helm delete $(HELM_RELEASE)

clean-registry:
	docker rm -v -f registry-ui 
	docker rm -v -f registry 

clean-workspace:
	docker-machine rm -f workspace

clean-k8s:
	minikube delete

