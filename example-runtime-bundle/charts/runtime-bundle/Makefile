CURRENT=$(shell pwd)
NAME := runtime-bundle
ORG := $(or $(ORG),activiti)
APP_NAME := $(or $(APP_NAME),$(NAME))
OS := $(shell uname)
RELEASE_ARTIFACT := $(or $(RELEASE_ARTIFACT),$(APP_NAME))

GITHUB_CHARTS_REPO := $(or $(GITHUB_CHARTS_REPO),$(shell git config --get remote.origin.url))
GITHUB_CHARTS_BRANCH := $(or $(GITHUB_CHARTS_BRANCH),gh-pages)

build: clean
	helm init --client-only
	helm repo add charts_activiti https://activiti.github.io/activiti-cloud-charts/
	helm repo add activiti-cloud-helm-charts https://activiti.github.io/activiti-cloud-helm-charts/
	helm dependency build
	helm lint
	#helm install . --name ${NAME} --debug --dry-run

install:
	helm install . --name ${NAME}

upgrade: clean build
	helm upgrade ${NAME} .

dry-run: 
	helm install . --name ${NAME} --dry-run

preview: version package dry-run release

delete:
	helm delete --purge ${NAME}

clean:
	rm -rf requirements.lock
	rm -rf charts
	rm -rf ${NAME}*.tgz

release:
	helm package .


github:
	$(eval GITHUB_CHARTS_DIR := $(shell basename $(GITHUB_CHARTS_REPO) .git))
	rm -rf $$GITHUB_CHARTS_DIR || echo "try to remove $$GITHUB_CHARTS_DIR"
	git clone -b "$(GITHUB_CHARTS_BRANCH)" https://${GITHUB_TOKEN}@github.com/Activiti/activiti-cloud-helm-charts.git $(GITHUB_CHARTS_DIR)
	cp "$(NAME)-$(VERSION).tgz" $(GITHUB_CHARTS_DIR)
	cd $(GITHUB_CHARTS_DIR) && \
	   helm repo index . && \
	   git add . && \
	   git status && \
	   git commit -m "fix:(version) release $(NAME)-$(VERSION).tgz" && \
	   git pull && \
	   git push origin "$(GITHUB_CHARTS_BRANCH)"
	rm -rf $(GITHUB_CHARTS_DIR)

version:
	sed -i -e "s/version:.*/version: $(VERSION)/" Chart.yaml
	sed -i -e "s/tag: .*/tag: $(VERSION)/" values.yaml


tag:
	git add --all
	git commit -m "release $(VERSION)" --allow-empty # if first release then no verion update is performed
	git tag -fa v$(VERSION) -m "Release version $(VERSION)"
	git push origin v$(VERSION)

updatebot/push-version: 
	@echo Resolving push versions for artifacts........
	$(eval ACTIVITI_CLOUD_RB_VERSION=$(shell mvn help:evaluate -Dexpression=activiti-cloud-runtime-bundle-service.version -q -DforceStdout -f ../../pom.xml))
	$(eval MAVEN_VERSIONS=org.activiti.cloud.examples:example-runtime-bundle $(RELEASE_VERSION) org.activiti.cloud.rb:activiti-cloud-runtime-bundle-dependencies $(ACTIVITI_CLOUD_RB_VERSION))

	updatebot -d ../../ push-version --kind maven $(MAVEN_VERSIONS);

	
