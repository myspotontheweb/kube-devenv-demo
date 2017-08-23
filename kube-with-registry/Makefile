.PHONY: local create browser delete purge
NAME=nginx-local
TIMESTAMP=tmp-$(shell date +%s )
REGISTRY=$(shell minikube ip -p registry):$(shell kubectl get service registry --context registry --namespace kube-system -o jsonpath='{.spec.ports[0].nodePort}')

init-registry:
	minikube start -p registry
	minikube addons enable registry -p registry
	sleep 10
	kubectl patch service registry -p '{"spec":{"type":"NodePort"}}' --context registry --namespace kube-system

init-dev:
	minikube start --insecure-registry=$(REGISTRY)

local:
	docker image build -t $(NAME):$(TIMESTAMP) -f Dockerfile .
	docker tag $(NAME):$(TIMESTAMP) $(REGISTRY)/$(NAME):$(TIMESTAMP)
	docker push $(REGISTRY)/$(NAME):$(TIMESTAMP)
	kubectl set image -f kubernetes/deploy/deployment.yaml $(NAME)=$(REGISTRY)/$(NAME):$(TIMESTAMP)

create:
	docker image build -t $(NAME):create -f Dockerfile .
	docker tag $(NAME):create $(REGISTRY)/$(NAME):create
	docker push $(REGISTRY)/$(NAME):create
	kubectl run $(NAME) --image=$(REGISTRY)/$(NAME):create --port=80
	kubectl expose deployment $(NAME) --type=NodePort

browser:
	minikube service $(NAME)

delete:
	kubectl delete service $(NAME)
	kubectl delete deployment $(NAME)

purge:
	docker rmi $$(docker images $(NAME) -q) -f

clean-all:
	minikube delete -p registry
	minikube delete
