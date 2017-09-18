
#
# Immediate vars
#
NAME:=demo

#
# Deferred vars
#
PROXY=http://$(shell docker-machine ip proxy):5000
BASE_DOMAIN=$(shell minikube ip).nip.io

#
# Global targets
#
all:

init: init-proxy init-k8s init-draft

clean-all: clean-proxy clean-k8s clean-draft

#
# Init targets
#
init-proxy:
	docker-machine create proxy
	docker-machine scp config/proxy.config.yml proxy:config.yml
	docker-machine ssh proxy docker network create proxy
	docker-machine ssh proxy docker run -d -p 5000:5000 --name proxy    --restart=always --network proxy -v /home/docker/config.yml:/etc/docker/registry/config.yml registry:2
	docker-machine ssh proxy docker run -d -p 8080:80   --name proxy-ui --restart=always --network proxy -e ENV_DOCKER_REGISTRY_HOST=proxy -e ENV_DOCKER_REGISTRY_PORT=5000 konradkleine/docker-registry-frontend:v2

init-k8s:
	minikube start --registry-mirror $(PROXY)
	minikube addons enable ingress
	minikube addons enable registry
	helm init
	kubectl apply -n kube-system -f config/proxy-ui.yml

init-draft:
	draft create --app $(NAME) --pack go
	echo "n|\$$(REGISTRY_SERVICE_HOST)|||teamwork|$(BASE_DOMAIN)" | tr '|' '\n' | script -c "draft init"

#
# Logging targets
#
log-draft:
	stern -n kube-system draft

log:
	stern $(NAME)

#
# Clean targets
#
clean-proxy:
	docker-machine rm -f proxy || true

clean-k8s:
	minikube delete || true

clean-draft:
	rm -rf chart
	rm -f Dockerfile
	rm -f draft.toml
	rm -f typescript
	rm -f .draftignore
