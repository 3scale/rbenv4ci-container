DISTRO=centos
DISTRO_SUFFIX=-$(DISTRO)

default: build

.PHONY: build
build: export IMAGE_LOCATION ?= quay.io/3scale
build: export IMAGE_NAME ?= rbenv4ci-container
build: export VERSION ?= $(shell git describe --dirty)
build: export IMAGE_VERSION := $(VERSION)$(DISTRO_SUFFIX)
build: export CONTAINER_USER ?= ruby
build: export IMAGE_FULL_NAME := $(IMAGE_LOCATION)/$(IMAGE_NAME)
build:
	docker build --rm --build-arg USER_NAME=$(CONTAINER_USER) \
		-t $(IMAGE_FULL_NAME):$(IMAGE_VERSION)-layered .
	-docker rm dummy-export-$(IMAGE_NAME)-$(IMAGE_VERSION)
	IMAGE_PATH=$$(docker run \
		--name dummy-export-$(IMAGE_NAME)-$(IMAGE_VERSION) \
		$(IMAGE_FULL_NAME):$(IMAGE_VERSION)-layered /bin/bash -c "echo \$${PATH}") \
	&& echo "Image path is $${IMAGE_PATH}" \
	&& (docker export dummy-export-$(IMAGE_NAME)-$(IMAGE_VERSION) | \
		docker import -c "USER $(CONTAINER_USER)" \
		-c "ENV PATH=$${IMAGE_PATH}" - \
		$(IMAGE_FULL_NAME):$(IMAGE_VERSION)) || \
		(echo Failed to flatten image && \
		docker rm dummy-export-$(IMAGE_NAME)-$(IMAGE_VERSION) && false)
	docker rm dummy-export-$(IMAGE_NAME)-$(IMAGE_VERSION)

.PHONY: destroy-layered
destroy-layered: export IMAGE_LOCATION ?= quay.io/3scale
destroy-layered: export IMAGE_NAME ?= rbenv4ci-container
destroy-layered: export VERSION ?= $(shell git describe --dirty)
destroy-layered: export IMAGE_VERSION := $(VERSION)$(DISTRO_SUFFIX)
destroy-layered: export IMAGE_FULL_NAME := $(IMAGE_LOCATION)/$(IMAGE_NAME)
destroy-layered:
	-docker rmi $(IMAGE_FULL_NAME):$(IMAGE_VERSION)-layered

.PHONY: destroy
destroy: export IMAGE_LOCATION ?= quay.io/3scale
destroy: export IMAGE_NAME ?= rbenv4ci-container
destroy: export VERSION ?= $(shell git describe --dirty)
destroy: export IMAGE_VERSION := $(VERSION)$(DISTRO_SUFFIX)
destroy: export IMAGE_FULL_NAME := $(IMAGE_LOCATION)/$(IMAGE_NAME)
destroy: destroy-layered
	-docker rmi $(IMAGE_FULL_NAME):$(IMAGE_VERSION)
