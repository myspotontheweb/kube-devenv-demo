
#
# Immediate vars
#
HELM_RELEASE:=demo
NAME:=demo
REPOSITORY:=myspotontheweb
TAG:=dev-$(shell date '+%Y%m%d%H%M%S')

#
# Deferred vars
#
REGISTRY=$(shell docker-machine ip registry):5000
IMAGE_NAME=$(REGISTRY)/$(REPOSITORY)/$(NAME)
WORKSPACE_DOCKER=$(shell docker-machine config workspace)
BASE_DOMAIN=$(shell minikube ip).nip.io

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
	docker-machine create registry
	docker-machine ssh registry docker network create registry
	docker-machine ssh registry docker run -d -p 5000:5000 --name registry --network registry registry:2.2.0
	docker-machine ssh registry docker run -d -p 8080:80 --name registry-ui --network registry -e ENV_DOCKER_REGISTRY_HOST=registry -e ENV_DOCKER_REGISTRY_PORT=5000 konradkleine/docker-registry-frontend:v2

init-workspace: 
	docker-machine create workspace --engine-insecure-registry $(REGISTRY)
	docker-machine ssh workspace docker network create build

init-k8s:
	minikube start --insecure-registry $(REGISTRY)
	minikube addons enable ingress
	helm init

init-secrets:
	@docker-machine ssh workspace "echo $(shell cat ~/.npmrc) > npmrc"
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
	helm upgrade --install $(HELM_RELEASE) chart/$(NAME) --namespace $(NAME) --set image.repository=$(IMAGE_NAME),image.tag=$(TAG),ingress.basedomain=$(BASE_DOMAIN)

#
# Clean targets
#
clean:
	helm delete $(HELM_RELEASE)

clean-registry:
	docker-machine rm -f registry || true

clean-workspace:
	docker-machine rm -f workspace || true

clean-k8s:
	minikube delete || true

