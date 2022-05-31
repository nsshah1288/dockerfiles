# https://www.gnu.org/software/make/manual/html_node/Makefile-Conventions.html

SHELL = /bin/bash

.ONESHELL:
.SUFFIXES:

AWS_REGION = us-east-1
AWS_REGISTRY = 660097632732.dkr.ecr.$(AWS_REGION).amazonaws.com
GIT_REGISTRY = registry.gitlab.com
ORG = jupiterintel
REPO = climatescoreglobal
VERSION ?= coder-data-explore
ARGS ?= jupyter lab --no-browser --allow-root --ip=0.0.0.0 --port=8887
IMAGE = $(ORG)/$(REPO)/supernotebook:$(VERSION)


build:
	@git rev-parse HEAD > version
	@DOCKER_BUILDKIT=1 && export DOCKER_BUILDKIT && \
		docker build -t $(IMAGE) --ssh=default .
	@rm version

clean:
	@rm -rf report.xml
	@find . -type d -name '__pycache__' -exec rm -rf {} +
	@find . -type d -name '*pytest_cache*' -exec rm -rf {} +
	@find . -type f -name "*.py[co]" -exec rm -rf {} +

dockerclean:
	@docker images | grep '$(IMAGE)' | awk '{print $$1 ":" $$2}' | xargs -n1 -p docker rmi
	@docker system prune -f

push: build
	@docker tag $(IMAGE) $(GIT_REGISTRY)/$(IMAGE)
	@docker push $(GIT_REGISTRY)/$(IMAGE)

release: build
	@aws ecr get-login-password --region $(AWS_REGION) | docker login --username AWS --password-stdin 660097632732.dkr.ecr.$(AWS_REGION).amazonaws.com
	@docker tag $(IMAGE) $(AWS_REGISTRY)/$(IMAGE)
	@docker push $(AWS_REGISTRY)/$(IMAGE)

run: build
	@AWS_ACCESS_KEY_ID=$$(aws --profile default configure get aws_access_key_id)
	@AWS_SECRET_ACCESS_KEY=$$(aws --profile default configure get aws_secret_access_key)
	@docker run --rm -it \
		-e AWS_ACCESS_KEY_ID="${AWS_ACCESS_KEY_ID}" \
		-e AWS_SECRET_ACCESS_KEY="${AWS_SECRET_ACCESS_KEY}" \
		$(IMAGE) $(ARGS)

.PHONY: build clean dockerclean push release run
