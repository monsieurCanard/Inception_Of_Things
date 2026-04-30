CLUSTER_NAME = bonus-cluster
ARGO_NAMESPACE = argocd
ARGO_DIR = ./argocd

.PHONY: all cluster argocd gitlab clean

all :cluster gitlab argocd

cluster:
	k3d cluster create --config cluster-config.yaml

gitlab:
	helm upgrade --install my-gitlab ./gitlab --namespace gitlab --create-namespace --wait --timeout 10m
	kubectl wait --for=condition=available --timeout=600s deployment/gitlab -n gitlab

gitlab-password:
	@GITLAB_POD=$$(kubectl get pods -n gitlab -l app=gitlab -o jsonpath='{.items[0].metadata.name}'); \
	kubectl exec -n gitlab $$GITLAB_POD -- cat /etc/gitlab/initial_root_password
	

argocd:
	kubectl create namespace $(ARGO_NAMESPACE) || true
	kubectl apply -n $(ARGO_NAMESPACE) -f $(ARGO_DIR)/install-argocd.yaml --server-side
	kubectl wait --for condition=established --timeout=60s crd/applications.argoproj.io
	kubectl wait --for=condition=ready pod --all -n $(ARGO_NAMESPACE) --timeout=2m
	kubectl apply -f argocd/ingress.yaml
	kubectl apply -f argocd/application.yaml
	kubectl patch deployment argocd-server -n argocd --type='json' -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--insecure"}]'

argocd-password:
	@echo "Your admin password for ArgoCd :"
	kubectl -n $(ARGO_NAMESPACE) get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

clean:
	k3d cluster delete $(CLUSTER_NAME)

re : clean all
