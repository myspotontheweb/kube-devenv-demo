.PHONY: local create browser delete purge
NAME=nginx-local
TIMESTAMP=tmp-$(shell date +%s )

local:
	@eval $$(minikube docker-env) ;\
	docker image build -t $(NAME):$(TIMESTAMP) -f Dockerfile .
	kubectl set image -f kubernetes/deploy/deployment.yaml $(NAME)=$(NAME):$(TIMESTAMP)

create:
	@eval $$(minikube docker-env) ;\
	docker image build -t $(NAME):create -f Dockerfile .
	kubectl create -f kubernetes/deploy/

browser:
	minikube service $(NAME)

delete:
	kubectl delete -f kubernetes/deploy/

purge:
	@eval $$(minikube docker-env) ;\
	docker rmi $$(docker images $(NAME) -q) -f
