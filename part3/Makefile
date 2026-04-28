CLUSTER_NAME = p3-cluster
ARGO_NAMESPACE = argocd
ARGO_DIR = ./argocd

.PHONY: all cluster argocd app clean password info

all :cluster argocd app

cluster:
	k3d cluster create --config cluster-config.yaml

argocd:
	kubectl create namespace $(ARGO_NAMESPACE)
	kubectl apply -n $(ARGO_NAMESPACE) -f $(ARGO_DIR)/install-argocd.yaml --server-side
	kubectl wait --for condition=established --timeout=60s crd/applications.argoproj.io
	kubectl wait --for=condition=ready pod --all -n $(ARGO_NAMESPACE) --timeout=2m

app:
	kubectl apply -f $(ARGO_DIR)/application.yaml

password:
	@echo "Your admin password for ArgoCd :"
	kubectl -n $(ARGO_NAMESPACE) get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

info:
	kubectl port-forward svc/argocd-server -n $(ARGO_NAMESPACE) 9090:443

clean:
	k3d cluster delete $(CLUSTER_NAME)

re : clean all
