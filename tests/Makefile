TAG = edge
PREFIX = nginx-gateway-fabric
NGINX_PREFIX = $(PREFIX)/nginx
PULL_POLICY=Never
GW_API_PREV_VERSION ?= 1.0.0 ## Supported Gateway API version from previous NGF release
GW_API_VERSION ?= 1.0.0 ## Supported Gateway API version for NGF under test
K8S_VERSION ?= latest ## Expected format: 1.24 (major.minor) or latest
GW_SERVICE_TYPE=NodePort
GW_SVC_GKE_INTERNAL=false
GINKGO_LABEL=
GINKGO_FLAGS=

ifneq ($(GINKGO_LABEL),)
    override GINKGO_FLAGS += -ginkgo.label-filter "$(GINKGO_LABEL)"
endif

.PHONY: help
help: Makefile ## Display this help
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "; printf "Usage:\n\n    make \033[36m<target>\033[0m\n\nTargets:\n\n"}; {printf "    \033[36m%-30s\033[0m %s\n", $$1, $$2}'

.PHONY: create-kind-cluster
create-kind-cluster: ## Create a kind cluster
	cd .. && make create-kind-cluster

.PHONY: build-images
build-images: ## Build NGF and NGINX images
	cd .. && make PREFIX=$(PREFIX) TAG=$(TAG) build-images

.PHONY: load-images
load-images: ## Load NGF and NGINX images on configured kind cluster
	kind load docker-image $(PREFIX):$(TAG) $(NGINX_PREFIX):$(TAG)

test: ## Run the system tests against your default k8s cluster
	go test -v ./suite $(GINKGO_FLAGS) -args --gateway-api-version=$(GW_API_VERSION) \
		--gateway-api-prev-version=$(GW_API_PREV_VERSION) --image-tag=$(TAG) \
		--ngf-image-repo=$(PREFIX) --nginx-image-repo=$(NGINX_PREFIX) --pull-policy=$(PULL_POLICY) \
		--k8s-version=$(K8S_VERSION) --service-type=$(GW_SERVICE_TYPE) --is-gke-internal-lb=$(GW_SVC_GKE_INTERNAL)

.PHONY: delete-kind-cluster
delete-kind-cluster: ## Delete kind cluster
	kind delete cluster

.PHONY: run-tests-on-vm
run-tests-on-vm: ## Run the tests on a GCP VM
	bash scripts/run-tests-gcp-vm.sh

.PHONY: create-and-setup-vm
create-and-setup-vm: ## Create and setup a GCP VM for tests
	bash scripts/create-and-setup-gcp-vm.sh

.PHONY: cleanup-vm
cleanup-vm: ## Delete the test GCP VM and delete the firewall rule
	bash scripts/cleanup-vm.sh

.PHONY: create-gke-router
create-gke-router: ## Create a GKE router to allow egress traffic from private nodes (allows for external image pulls)
	bash scripts/create-gke-router.sh

.PHONY: cleanup-router
cleanup-router: ## Delete the GKE router
	bash scripts/cleanup-router.sh

.PHONY: setup-gcp-and-run-tests
setup-gcp-and-run-tests: create-gke-router create-and-setup-vm run-tests-on-vm ## Create and setup a GKE router and GCP VM for tests and run the tests

.PHONY: cleanup-gcp
cleanup-gcp: cleanup-router cleanup-vm ## Cleanup all GCP resources
