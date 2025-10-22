CNPG_VERSION ?= '0.23.2'
CNPG_NS ?= cnpg-system
KUBECTL_DEPLOY_CONTEXT ?= 'usdf-maf'

ensure-context:
	if [ '$(shell kubectl config current-context)' != $(KUBECTL_DEPLOY_CONTEXT) ]; then echo "Configured deployment context inccorect; expecting $(KUBECTL_DEPLOY_CONTEXT)."; exit 1; fi

helm:
	helm repo add cnpg https://cloudnative-pg.github.io/charts
	helm repo update
	helm template cnpg --namespace $(CNPG_NS) --version $(CNPG_VERSION) --create-namespace cnpg/cloudnative-pg -f cnpg-values.yaml | kubectl slice -o resources/
	mv resources/customresourcedefinition* crd/
	rm -rf resources/-.yaml
	kustomize edit add resource resources/*

dump: helm
	kubectl kustomize .

apply: ensure-context helm
	# CRDs too long
	kubectl apply --server-side --force-conflicts -f crd/
	kubectl apply -k .
