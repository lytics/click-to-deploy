APP_NAME = wordpress

tools_path = ../vendor/marketplace-tools

include $(tools_path)/gcloud.Makefile

include $(tools_path)/crd.Makefile
include $(tools_path)/app.Makefile

REGISTRY_FOR_DEPLOYER ?= $(APP_REGISTRY)
$(info ---- REGISTRY_FOR_DEPLOYER = $(REGISTRY_FOR_DEPLOYER))

app/build:: .build/deployer .build/wordpress .build/mysql

.build/deployer: deployer/* manifest/* $(MARKETPLACE_BASE_BUILD)/deployer-kubectl $(APP_BUILD)/registry_prefix | app/setup
	docker build \
	    --build-arg REGISTRY="$(REGISTRY_FOR_DEPLOYER)" \
	    --build-arg TAG="$(APP_TAG)" \
	    --build-arg MARKETPLACE_REGISTRY="$(MARKETPLACE_REGISTRY)" \
	    --tag "$(APP_DEPLOYER_IMAGE)" \
	    -f deployer/Dockerfile \
	    .
	gcloud docker -- push "$(APP_DEPLOYER_IMAGE)"
	@touch "$@"

# Simulate building of primary app image. Actually just copying public image to
# local registry.
.build/wordpress: $(APP_BUILD)/registry_prefix | app/setup
	gcloud docker -- pull launcher.gcr.io/google/wordpress4-php5-apache
	docker tag launcher.gcr.io/google/wordpress4-php5-apache "$(APP_REGISTRY):$(APP_TAG)"
	gcloud docker -- push "$(APP_REGISTRY):$(APP_TAG)"
	@touch "$@"

# Relocate public mysql image to $APP_REGISTRY.
.build/mysql: $(APP_BUILD)/registry_prefix | app/setup
	gcloud docker -- pull launcher.gcr.io/google/mysql5
	docker tag launcher.gcr.io/google/mysql5 "$(APP_REGISTRY)/mysql:$(APP_TAG)"
	gcloud docker -- push "$(APP_REGISTRY)/mysql:$(APP_TAG)"
	@touch "$@"