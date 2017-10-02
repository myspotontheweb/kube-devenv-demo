NAME    := demo
TAG     := $(shell git log -1 --pretty=%H)
IMG     := $(NAME):$(TAG)
RELEASE := $(NAME):v0.4.16
LATEST  := $(NAME):latest

build:
	docker build -t $(IMG) .
	docker tag $(IMG) $(RELEASE)
	docker tag $(IMG) $(LATEST)

push: build
	docker push $(IMG)
	docker push $(RELEASE)
	docker push $(LATEST)

package: 
	helm package -d  ~/.helm/repository/local charts/demo

bump:
	bumpversion patch --list --commit --tag

serve:
	helm serve
