.PHONY: local mount create browser delete purge
REPO=nginx-local-mount
TIMESTAMP=tmp-$(shell date +%s )

local:
	@eval $$(minikube docker-env) ;\
	docker image build -t $(REPO):$(TIMESTAMP) -f Dockerfile .
	kubectl set image -f kubernetes/deploy/deployment.yaml $(REPO)=$(REPO):$(TIMESTAMP)

mount:
	minikube mount $(PWD)/website:/website

create:
	@eval $$(minikube docker-env) ;\
	docker image build -t $(REPO):create -f Dockerfile .
	kubectl create -f kubernetes/deploy/

browser:
	minikube service $(REPO)

delete:
	kubectl delete -f kubernetes/deploy/

purge:
	@eval $$(minikube docker-env) ;\
	docker rmi $$(docker images $(REPO) -q) -f
