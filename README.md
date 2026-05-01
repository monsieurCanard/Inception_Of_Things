<div align="center">
  <img width="1460" height="349" alt="Image" src="https://github.com/user-attachments/assets/6ee41dfe-fcde-42ad-8f0c-90561d420388" />
  
  <p align="center">
  <h2>Déploiement, GitOps et infrastructure Kubernetes en local</h2>
  <img src="https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white" alt="Kubernetes">
  <img src="https://img.shields.io/badge/Vagrant-1563FF?style=for-the-badge&logo=vagrant&logoColor=white" alt="Vagrant">
  <img src="https://img.shields.io/badge/ArgoCD-EF7B4D?style=for-the-badge&logo=argo&logoColor=white" alt="ArgoCD">
  <img src="https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white" alt="Docker">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Status-Active-success.svg?style=flat-square" alt="Status">
</p>
</div>

Un projet complet pour découvrir l'orchestration de conteneurs, le déploiement continu et la gestion d'infrastructure as code à travers Kubernetes, Vagrant, k3d, et ArgoCD.

---

## 📚 Table des matières

- [Introduction à Kubernetes (K8s)](#-introduction-à-kubernetes-k8s)
  - [Qu'est-ce que Kubernetes ?](#qu'est-ce-que-kubernetes-)
  - [Comment fonctionne Kubernetes ?](#️-comment-fonctionne-kubernetes-)
  - [Quels sont les avantages ?](#-quels-sont-les-avantages-de-kubernetes-)
- [Exercice 1 : Cluster Vagrant & k3s](#1️⃣-exercice-1--le-cluster-avec-vagrant-et-k3s)
- [Exercice 3 : k3d & GitOps avec ArgoCD](#3️⃣-exercice-3--k3d-et-le-gitops-avec-argocd)
- [Bonus : GitLab Interne](#-exercice-bonus--gitlab-interne-100-on-premise)

---

## 🌐 Introduction à Kubernetes (K8s)

### 🧬 Qu'est-ce que Kubernetes ?
Kubernetes (souvent abrégé en K8s) est une plateforme open-source d'orchestration de conteneurs. Développée à l'origine par Google, elle permet d'automatiser le déploiement, la mise à l'échelle, et la gestion des applications conteneurisées.

### ⚙️ Comment fonctionne Kubernetes ?
L'architecture de Kubernetes repose sur le concept de **Cluster**, qui est composé de plusieurs machines physiques ou virtuelles appelées **Nodes** (nœuds). Un cluster se divise en deux rôles principaux :
- **Le Control Plane (Master Node) :** C'est le cerveau du cluster. Il gère l'état global du cluster, planifie les déploiements (Scheduler), stocke l'état du cluster (etcd) et expose l'API Kubernetes.
- **Les Worker Nodes :** Ce sont les machines qui exécutent réellement les applications. Chaque nœud exécute un agent appelé `kubelet` qui communique avec le Control Plane et gère le cycle de vie des **Pods** (la plus petite unité déployable dans K8s qui contient un ou plusieurs conteneurs).

Les utilisateurs ou les outils de CI/CD interagissent avec le Control Plane via des manifestes (fichiers YAML) qui décrivent "l'état désiré" (ex: "je veux 3 réplicas de mon application web"). Dès lors, la boucle de contrôle de K8s travaille en permanence pour s'assurer que l'état actuel du cluster correspond toujours à cet état désiré.

### 🚀 Quels sont les avantages de Kubernetes ?
- **Haute disponibilité (Auto-healing) :** Si un conteneur ou un nœud entier tombe en panne, Kubernetes redémarre ou déplace automatiquement les Pods affectés vers des nœuds sains.
- **Mise à l'échelle (Scalabilité) :** Face à un pic de trafic, K8s peut automatiquement ajouter de nouvelles instances (Pods) ou même de nouveaux nœuds pour absorber la charge.
- **Déploiements fluides (Zero-downtime) :** K8s met à jour l'application progressivement (Rolling updates), ce qui permet des mises à jour sans interruption de service.
- **Agnostique et portable :** Kubernetes fonctionne de la même manière en local (avec k3d, minikube), sur des serveurs physiques (on-premise) ou dans des clouds publics.
- **Load Balancing et Service Discovery :** Il attribue automatiquement des adresses IP aux Pods, gère les noms DNS et distribue le trafic réseau uniformément.

---

## 1️⃣ Exercice 1 : Le Cluster avec Vagrant et k3s

**Le but :** Lancer deux machines virtuelles à partir d'un Vagrantfile et installer k3s. L'une des machines sera le contrôleur (server) et l'autre le worker (agent/node).

> [!NOTE]
> **Pourquoi ces outils ?**
> - **Vagrant** est un outil de gestion de machines virtuelles qui permet de créer et de configurer des environnements de développement reproductibles. En l'utilisant, vous pouvez facilement lancer plusieurs machines virtuelles pour simuler un cluster Kubernetes local.
> - **k3s** est une distribution Kubernetes légère certifiée, conçue pour les environnements de développement, l'IoT et les clusters de petite taille. Son installation très simple nous permet de monter notre cluster rapidement.

Voici un exemple de Vagrantfile pour lancer nos deux machines et préparer l'installation de k3s :

```ruby
Vagrant.configure("2") do |config|
	# Configuration de la première machine (contrôleur / server)
	config.vm.define "server" do |server_config|
		server_config.vm.box = "debian/bookworm64"
		server_config.vm.network "private_network", ip: "192.168.10.10"
		server_config.vm.provision "shell", inline: <<-SHELL
			# Écrire ici les commandes pour installer k3s sur la machine serveur
		SHELL
	end

	# Configuration de la deuxième machine (worker / agent)
	config.vm.define "worker" do |worker_config|
		worker_config.vm.box = "debian/bookworm64"
		worker_config.vm.network "private_network", ip: "192.168.10.11"
		worker_config.vm.provision "shell", inline: <<-SHELL
			# Écrire ici les commandes pour installer k3s sur la machine worker
		SHELL
	end
end
```

Maintenant que nous avons paramétré nos machines, il faut spécifier un "provider" (fournisseur) pour qu'elles puissent être virtuallisées. Vagrant en supporte plusieurs, mais pour cet exercice, nous allons utiliser **VirtualBox**. Bien qu'il ne soit pas le plus performant, il reste très accessible et pédagogique pour ce genre de projet.

```ruby
Vagrant.configure("2") do |config|
	config.vm.provider "virtualbox" do |vb|
		vb.memory = 2048 # Exemple : 2 Go de RAM
		vb.cpus = 2      # Exemple : 2 cœurs
	end

	# (Insérer ici la configuration des machines virtuelles définie plus haut)
end
```

Si vous évoluez sous Linux et souhaitez optimiser les performances, vous pouvez utiliser KVM avec le provider `libvirt` au lieu de `virtualbox` :

```ruby
Vagrant.configure("2") do |config|
	config.vm.provider "libvirt" do |libvirt|
		libvirt.memory = 2048
		libvirt.cpus = 2
	end
end
```

>[!NOTE]
> Assurez-vous d'avoir installé KVM et le plugin `vagrant-libvirt` avant d'utiliser cette configuration.

Nous pouvons à présent lancer nos machines virtuelles avec la commande suivante à la racine du projet :

```bash
vagrant up --provider=libvirt
```
>[!NOTE]
> Si vous voulez éviter de spécifier le provider à chaque fois, vous pouvez le définir avec config.vm.provider (si do |config|) dans le Vagrantfile, ou même utiliser une variable d'environnement `VAGRANT_DEFAULT_PROVIDER=libvirt` pour que ce soit pris en compte par défaut.

>[!WARNING]
> N'oubliez pas de récupérer le *node-token* de k3s sur la machine serveur. Il est indispensable pour faire rejoindre la machine worker à votre cluster. Vous pouvez l'afficher avec la commande suivante (sous la VM serveur) :
>
>```bash
>sudo cat /var/lib/rancher/k3s/server/node-token
>```

## 3️⃣ Exercice 3 : k3d et le GitOps avec ArgoCD

**Le but :** Configurer un cluster Kubernetes local en utilisant k3d, déployer une application simple, et la maintenir à jour automatiquement grâce à ArgoCD (GitOps).

Pour cet exercice, nous allons repousser un peu plus loin le concept et utiliser **k3d**.
>[!NOTE]
> k3d est un wrapper très léger qui exécute k3s à l'intérieur de conteneurs Docker. Il permet de déployer un ou plusieurs clusters locaux en quelques secondes sans avoir besoin de manipuler des VMs avec Vagrant.

Voici les étapes pour configurer un cluster via k3d et déployer notre application de façon continue avec ArgoCD :

### Architecture des fichiers de travail
Voici comment sont structurés nos fichiers pour de ce déploiement (basé sur le dossier `part3/`):
```text
part3/
 ├── cluster-config.yaml         # Configuration du cluster k3d
 ├── Makefile                    # Commandes utiles automatisées
 ├── Vagrantfile                 # (Optionnel) Au cas où l'on déploie k3d sur une VM
 ├── app-config/                 # Application métier (notre serveur web)
 │   ├── deployment.yaml
 │   ├── ingress.yaml
 │   └── service.yaml
 └── argocd/                     # Installation et configuration d'ArgoCD
     ├── application.yaml
     ├── ingress.yaml
     └── install-argocd.yaml
```

## 1. Créer un cluster Kubernetes local avec k3d
Pour provisionner le cluster, on utilise la commande :
```bash
k3d cluster create mycluster --config cluster-config.yaml
```

Exemple d'un fichier de configuration k3d (`cluster-config.yaml`) :
```yaml
apiVersion: k3d.io/v1alpha4
kind: Simple 
metadata:
  name: mycluster
servers: 1      # Nœud Control Plane
agents: 1       # Nœud(s) Worker
image: rancher/k3s:v1.24.8-k3s1
ports:
  - port: 8080:80
    nodeFilters:
      - loadbalancer
  - port: 8443:443
    nodeFilters:
      - loadbalancer
```

## 2. Créer un namespace et installer ArgoCD

Créons le namespace dédié :
```bash
kubectl create namespace argocd
```

Puis, appliquons le manifeste officiel d'installation :
```bash
kubectl apply -n argocd -f argocd/install-argocd.yaml
```

*Remarque : Par sécurité et pour bien manipuler Kubernetes, nous avons choisi de ne pas exposer ArgoCD en dehors du cluster de base via un Ingress public, mais d'utiliser `kubectl port-forward` pour s'y connecter.*

## 3. Demander à ArgoCD de surveiller et déployer l'application
On applique le fichier `application.yaml` qui va configurer ArgoCD pour pointer sur le code source de notre dépôt (GitOps) :
```bash
kubectl apply -n argocd -f argocd/application.yaml
```

>[!NOTE]
> Exemple de fichier décrivant comment l'application ArgoCD (`application.yaml`) synchronise notre dossier :
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
>    path: 'part3/app-config'
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

## 4. Manifestes de l'application surveillée (dans `app-config/`)
Une fois ArgoCD en place, il lira les configurations ci-dessous pour déployer et exposer les Pods. Nous utilisons ici un cas d'école avec le nom `wil` et une image `nginx`.

### `deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wil-deployment
  namespace: dev
spec:
  replicas: 2
  selector:
    matchLabels:
      app: wil
  template:
    metadata:
      labels:
        app: wil
    spec:
      containers:
      - name: wil-app
        image: nginx:latest
        ports:
        - containerPort: 80
```

### `service.yaml`
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
      targetPort: 80
```

### `ingress.yaml` (gestion du routage HTTPS/HTTP)
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
> Dans cet exemple, toutes les requêtes arrivant sur la racine (`/`) du cluster via la gateway sont redirigées vers le service nommé `wil-service`, qui lui-même distribue le trafic vers les pods du `wil-deployment`.

### 💡 Justification de certains choix stratégiques :
- **Sécurité et réseau :** Comme évoqué, j'ai choisi de ne pas exposer ArgoCD directement via Ingress. Cela force l'apprentissage du `port-forwarding` avec `kubectl` afin d'accéder au dashboard applicatif de manière sécurisée sans l'exposer à internet. Côté secret, les mots de passe et configurations ne sont pas en dur dans le code pour encourager les bonnes pratiques. (finalement pour la correction j'ai préféré lancer argocd en --insecure pour éviter de bloquer mon terminal ainsi que les complications liées à la gestion des certificats auto-signés, mais dans un vrai projet il faudrait gérer ça proprement).
- **Automatisation via Makefile :** J'ai mis à disposition un `Makefile` pour regrouper les commandes bash récurrentes (déploiement de k3d, application de Kustomizations, namespaces, mots de passe). C'est beaucoup plus robuste. De plus, si l'on souhaite automatiser cela dans une VM via Vagrant, il suffit que le "shell provisionner" exécute une seule commande (ex: `make all`).

---

## 🦊 Bonus : GitLab Local

**Le but :** Pousser le concept GitOps jusqu'au bout en hébergeant son propre serveur Git localement.

Dans les exercices précédents, ArgoCD surveillait un dépôt hébergé sur GitHub (un service externe). Pour ce bonus, l'objectif est de remplacer GitHub par notre propre instance **GitLab**, déployée de A à Z sur notre cluster Kubernetes.

### ❓ Pourquoi déployer GitLab en local ?
- **Indépendance totale :** Nous ne dépendons plus d'une connexion internet ou des serveurs de Microsoft/GitHub. Tout fonctionne en circuit fermé.
- **Maîtrise de l'infrastructure et sécurité :** Cela simule un environnement d'entreprise "air-gapped" (isolé) ultra-sécurisé où le code source ne quitte jamais le réseau interne (ou le cluster) de la société.

## ⁉️ Pourquoi utiliser Helm pour ce déploiement ?
- **Simplification de la complexité :** GitLab est une solution volumineuse constituée de nombreux composants (base de données PostgreSQL, Redis, serveurs web, workers Sidekiq, Gitaly...). Créer et maintenir chaque ressource YAML à la main serait extrêmement lourd et source d'erreurs.
- **Le concept de Chart :** Helm permet de "packager" toute cette infrastructure sous forme de **Chart Kubernetes** (fichiers `templates/` couplés à un `Chart.yaml`).
- **Configuration centralisée (`values.yaml`) :** Au lieu de modifier vos informations (mots de passe, ingress, certificats) fichier par fichier, Helm permet d'injecter des variables dynamiques à partir d'un seul fichier `values.yaml`.

## Architecture de la partie Bonus
L'intégralité des fichiers se trouve dans le dossier `bonus/`. On y retrouve les configurations pour *k3d* et *ArgoCD*, mais avec un ajout majeur : le dossier `gitlab/`.
```text
bonus/
 ├── cluster-config.yaml
 ├── Makefile
 ├── gitlab/                     # Fichiers de déploiement de GitLab
 │   ├── Chart.yaml              # Structure de type Helm
 │   ├── values.yaml
 │   └── templates/
 │       ├── deployment.yaml
 │       ├── ingress.yaml
 │       └── service.yaml
 ├── app-config/                 # L'application à déployer
 └── argocd/                     # Configuration ArgoCD modifiée
```

## Comment ça fonctionne ?
1. **Déploiement de GitLab :** Nous utilisons les manifestes fournis dans `bonus/gitlab/` (façonnés sous forme de Chart Helm pour faciliter la configuration avec `values.yaml`) pour lancer le serveur GitLab au sein de notre cluster. Un nom de domaine local (via Ingress) lui est attribué pour y accéder depuis notre navigateur.
2. **Configuration du dépôt local :** Une fois notre instance GitLab fonctionnelle, nous nous y connectons, créons notre projet, et utilisons `git push` pour envoyer le dossier `app-config/` directement dans GitLab.
3. **Modification d'ArgoCD :** Le fichier `argocd/application.yaml` est modifié. Au lieu de pointer vers une adresse web comme `https://github.com/...`, la variable `repoURL` va désormais pointer vers notre propre instance interne.

>[!NOTE]
> **Exemple de modification sur la ressource Application d'ArgoCD :**
>```yaml
>spec:
>  source:
>    # Anciennement : repoURL: 'https://github.com/monsieurCanard/Inception_Of_Things.git'
>    repoURL: 'http://gitlab-service.default.svc.cluster.local/root/mon-projet.git'
>    # L'URL ci-dessus permet à ArgoCD de contacter directement le service GitLab via le DNS interne de Kubernetes.
>    targetRevision: HEAD
>    path: 'bonus/app-config'
>```

Avec cette architecture, la boucle **GitOps** est complétée en totale autonomie : le développeur push sur le GitLab interne, ArgoCD le détecte et met automatiquement à jour les pods de l'application sur le même cluster.
