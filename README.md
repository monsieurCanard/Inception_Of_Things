# Inception of things:


# Exercice 1 
##### Lancer deux Vm a partir d'un VagrantFile et installer k3s sur les deux machines. Une des machines sera le controleur (server) et l'autre le worker (server worker).

> [!NOTE]
> Pourquoi ?
> - Vagrant est un outil de gestion de machines virtuelles qui permet de créer et de configurer des environnements de développement reproductibles. En utilisant Vagrant, vous pouvez facilement lancer plusieurs machines virtuelles pour simuler un cluster Kubernetes local.
> - k3s est une distribution légère de Kubernetes conçue pour les environnements de développement et les clusters de petite taille. En installant k3s sur les machines virtuelles, vous pouvez créer un cluster Kubernetes local pour apprendre et expérimenter avec Kubernetes.

Voici un exemple de Vagrantfile pour lancer deux machines virtuelles et installer k3s :

```ruby
Vagrant.configure("2") do |config|
	# Configuration de la première machine (contrôleur)
	config.vm.define "server" do |server_config|
		server_config.vm.box = "debian/bookworm64"
		server_config.vm.network "private_network", ip: "192.168.10.10"
		server_config.vm.provision "shell", inline: <<-SHELL
			Ecrire ici les commandes pour installer k3s sur la machine serveur
		SHELL
	end

	config.vm.define "worker" do |worker_config|
		worker_config.vm.box = "debian/bookworm64"
		worker_config.vm.network "private_network", ip: "192.168.10.11"
		worker_config.vm.provision "shell", inline: <<-SHELL
			Ecrire ici les commandes pour installer k3s sur la machine worker
		SHELL
	end
end
```

Maitenant qu'on a ecrire notre premiere partie de la config, il faut maintenant ajouter un provider pour lancer nos machines virtuelles. Vagrant supporte plusieurs providers, mais pour cet exercice, nous allons utiliser VirtualBox. Meme si je sais que c'est pas le plus rapide, il est, pour moi, le plus pedagogique.

```ruby

Vagrant.configure("2") do |config|
	config.vm.provider "virtualbox" do |vb|
		vb.memory = "1024"
		vb.cpus = 2
	end

	# Configuration des machines virtuelles (server et worker) ici
end
```

Maintenant nous allons pouvoir lancer nos machines virtuelles en utilisant la commande suivante dans le terminal (assurez-vous d'avoir le provider VirtualBox installé et configuré sur votre machine) :

```bash
vagrant up
```

>[!WARNING]
> Pensez aussi a recuperer le token d'installation de k3s sur la machine serveur pour pouvoir joindre le worker au cluster. Vous pouvez le voir en utilisant la commande suivante sur la machine serveur :
>
>```bash
>sudo cat /var/lib/rancher/k3s/>server/node-token
>```

# Exercice 3
##### Configurer un cluter Kubernetes local en utilisant k3s et déployer une application simple (par exemple, une application web) sur le cluster et la metre a jour automatiquement a chaque changement de version grace a ArgoCD.

Pour cette exercice nous allons utiliser k3d pour creer un cluster Kubernetes local.
>[!NOTE]
> k3d est un outil qui permet de créer des clusters Kubernetes légers en utilisant Docker. 

Voici les étapes pour configurer un cluster Kubernetes local avec k3s et déployer une application simple en utilisant ArgoCD :

## Hierarchie des fichiers
```
.
├── k3d-cluster.yaml
├── app-config/
│   └── deployment.yaml
|   └── service.yaml
|   └── ingress.yaml
├── argo_cd/
		└── install-argocd.yaml
		└── application.yaml
```

## 1. Créer un cluster Kubernetes local avec k3d
```bash
k3d cluster create mycluster --config k3d-cluster.yaml
```

Exemple de fichier de config de k3d (k3d-cluster.yaml) :
```yaml
apiVersion: k3d.io/v1alpha4
kind: Simple 
# Simple = 1 server, 0 workers
# Complex = 1 server, 2 workers
# Custom = 1 server, 0 workers, mais avec des options de configuration personnalisées
metadata:
	name: mycluster
servers: 1
agents: 0
images:
	- rancher/k3s:v1.24.8-k3s1
ports:
	- 80:80
	- 443:443
```

## Créer un namespace pour ArgoCD
```bash
kubectl create namespace argocd
```

## 2. Installer ArgoCD
```bash
kubectl apply -n argocd -f argo_cd/install-argocd.yaml
```

## Appliquer la configuration de l'application dans ArgoCD
```bash
kubectl apply -n argocd -f argo_cd/application.yaml
```

>[!NOTE]
> Exemple de fichier de configuration pour l'application ArgoCD (application.yaml) :
>```yaml
>apiVersion: argoproj.io/v1alpha1
>kind: Application
>metadata:
>  name: wil-app-controller
>  namespace: argocd
>spec:
>  project: default
>  source:
>    repoURL: 'https://github.com/monsieurCanard/Inception_Of_Things'
>    targetRevision: HEAD
>    path: 'p3/app-config'
>  destination:
>    server: 'https://kubernetes.default.svc'
>    namespace: dev
>  syncPolicy:
>    automated:
>      prune: true
>      selfHeal: true
>    syncOptions:
>      - CreateNamespace=true
>```

## 3. Déployer une application simple
Créez les fichiers de configuration pour votre application (deployment.yaml, service.yaml, ingress.yaml) dans le dossier app-config. Par exemple, pour une application web simple, vous pouvez utiliser les configurations suivantes :
### deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
	name: web-app
	namespace: dev
spec:
	replicas: 2
	selector:
		matchLabels:
			app: web-app
	template:
	metadata:
		labels:
			app: web-app
spec:
	containers:
	- name: web-app
	image: nginx:latest
	ports:
	- containerPort: 80
```

### service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: wil-service
  namespace: dev
spec:
  selector:
    app: wil
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8888
```

### ingress.yaml (gestion du routage)
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: wil-ingress
  namespace: dev
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: wil-service
            port:
              number: 80
```
>[!NOTE]
> Dans cette exemple on veux que toutes les requetes qui arrivent sur le port 80 soient redirigées vers le service wil-service, qui lui même redirige vers les pods de l'application web-app.
