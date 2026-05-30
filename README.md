<div align="center">
  <img width="200" alt="Kubernetes Banner" src="https://upload.wikimedia.org/wikipedia/commons/3/39/Kubernetes_logo_without_workmark.svg" />
  
<h1> ☸️ Inception of things </h1>
  <p align="center">
  <h2>Deployment, GitOps and local Kubernetes infrastructure</h2>
  <img src="https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white" alt="Kubernetes">
  <img src="https://img.shields.io/badge/Vagrant-1563FF?style=for-the-badge&logo=vagrant&logoColor=white" alt="Vagrant">
  <img src="https://img.shields.io/badge/ArgoCD-EF7B4D?style=for-the-badge&logo=argo&logoColor=white" alt="ArgoCD">
  <img src="https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white" alt="Docker">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Status-Active-success.svg?style=flat-square" alt="Status">
</p>
</div>

A complete project to discover container orchestration, continuous deployment, and infrastructure as code management using Kubernetes, Vagrant, k3d, and ArgoCD.

---

## 📚 Table of Contents

- [Introduction to Kubernetes (K8s)](#-introduction-to-kubernetes-k8s)
  - [What is Kubernetes?](#what-is-kubernetes)
  - [How does Kubernetes work?](#️-how-does-kubernetes-work)
  - [What are the benefits?](#-what-are-the-benefits-of-kubernetes)
- [Exercise 1: Vagrant & k3s Cluster](#1️⃣-exercise-1--the-cluster-with-vagrant-and-k3s)
- [Exercise 3: k3d & GitOps with ArgoCD](#3️⃣-exercise-3--k3d-and-gitops-with-argocd)
- [Bonus: Internal GitLab](#-bonus-exercise--internal-gitlab-100-on-premise)

---

## 🌐 Introduction to Kubernetes (K8s)

### 🧬 What is Kubernetes?
Kubernetes (often abbreviated as K8s) is an open-source container orchestration platform. Originally developed by Google, it automates the deployment, scaling, and management of containerized applications.

### ⚙️ How does Kubernetes work?
The architecture of Kubernetes relies on the concept of a **Cluster**, which is made of several physical or virtual machines called **Nodes**. A cluster is divided into two primary roles:
- **The Control Plane (Master Node):** This is the brain of the cluster. It manages the global state of the cluster, schedules deployments (Scheduler), stores the cluster's state (etcd), and exposes the Kubernetes API.
- **The Worker Nodes:** These are the machines that actually run the applications. Each node runs an agent called `kubelet` which communicates with the Control Plane and manages the lifecycle of **Pods** (the smallest deployable unit in K8s that contains one or more containers).

Users or CI/CD tools interact with the Control Plane via manifests (YAML files) that describe the "desired state" (e.g., "I want 3 replicas of my web app"). The K8s control loop works continuously to ensure that the cluster's current state always matches this desired state.

### 🚀 What are the benefits of Kubernetes?
- **High availability (Auto-healing):** If a container or an entire node fails, Kubernetes automatically restarts or moves the affected Pods to healthy nodes.
- **Scaling (Scalability):** During a traffic spike, K8s can automatically add new instances (Pods) or even new nodes to handle the load.
- **Seamless deployments (Zero-downtime):** K8s updates the application progressively (Rolling updates), allowing updates without service interruption.
- **Agnostic and portable:** Kubernetes works the same way locally (with k3d, minikube), on physical servers (on-premise), or in public clouds.
- **Load Balancing and Service Discovery:** It automatically assigns IP addresses to Pods, manages DNS names, and distributes network traffic evenly.

---

## 1️⃣ Exercise 1: The Cluster with Vagrant and k3s

**The Goal:** Launch two virtual machines from a Vagrantfile and install k3s. One machine will be the controller (server) and the other the worker (agent/node).

> [!NOTE]
> **Why these tools?**
> - **Vagrant** is a virtual machine management tool that allows you to create and configure reproducible development environments. By using it, you can easily launch multiple virtual machines to simulate a local Kubernetes cluster.
> - **k3s** is a certified lightweight Kubernetes distribution, designed for development environments, IoT, and small clusters. Its very simple installation allows us to set up our cluster quickly.

Here is an example of a Vagrantfile to launch our two machines and prepare the k3s installation:

```ruby
Vagrant.configure("2") do |config|
	# Configuration of the first machine (controller / server)
	config.vm.define "antgabriS" do |server_config|
		server_config.vm.box = "debian/trixie64"
		server_config.vm.network "private_network", ip: "192.168.56.110"
		server_config.vm.provision "shell", inline: <<-SHELL
			# Write commands here to install k3s on the server machine
		SHELL
	end

	# Configuration of the second machine (worker / agent)
	config.vm.define "antgabriSW" do |server_worker_config|
		server_worker_config.vm.box = "debian/trixie64"
		server_worker_config.vm.network "private_network", ip: "192.168.56.111"
		server_worker_config.vm.provision "shell", inline: <<-SHELL
			# Write commands here to install k3s on the worker machine
		SHELL
	end
end
```

Now that we have configured our machines, we need to specify a "provider" so they can be virtualized. Vagrant supports several, but for this exercise, we will use **VirtualBox**. Although it is not the most performant, it remains very accessible and educational for this kind of project.

```ruby
Vagrant.configure("2") do |config|
	config.vm.provider "virtualbox" do |vb|
		vb.memory = 2048 # Example: 2 GB RAM
		vb.cpus = 2      # Example: 2 cores
	end

	# (Insert the virtual machines configuration defined above here)
end
```

If you are working on Linux and want to optimize performance, you can use KVM with the `libvirt` provider instead of `virtualbox`:

```ruby
Vagrant.configure("2") do |config|
	config.vm.provider "libvirt" do |libvirt|
		libvirt.memory = 2048
		libvirt.cpus = 2
	end
end
```

>[!NOTE]
> Make sure you have installed KVM and the `vagrant-libvirt` plugin before using this configuration.

We can now launch our virtual machines with the following command at the root of the project:

```bash
vagrant up --provider=libvirt --no-parallel
```

>[!NOTE]
> If you want to avoid specifying the provider every time, you can define it with config.vm.provider (under do |config|) in the Vagrantfile, or even use an environment variable `VAGRANT_DEFAULT_PROVIDER=libvirt` so it is taken into account by default.

>[!WARNING]
> Do not forget to retrieve the k3s *node-token* on the server machine. It is essential for joining the worker machine to your cluster. You can display it with the following command (on the server VM):
>
>```bash
>sudo cat /var/lib/rancher/k3s/server/node-token
>```

# Inception of Things - Part 2

## K3s and three simple applications

This repository contains the configuration for **Part 2** of the 42 **Inception of Things** project.

The goal of this part is to run **one virtual machine** with **K3s in server mode**, deploy **three web applications**, and access them through the same IP address by using the HTTP `Host` header.

## Subject requirements

For Part 2, the project must provide:

| Requirement | Implementation in this project |
|---|---|
| One virtual machine | Vagrant VM using Debian Trixie |
| K3s server mode | K3s installed inside the VM |
| VM name | `thitranS` |
| VM IP address | `192.168.56.110` |
| Three web applications | `app-one`, `app-two`, `app-three` |
| App 2 replicas | `app-two` has `3` replicas |
| Host-based routing | Kubernetes Ingress with Traefik |
| Default application | `app-three` is used when the host is not `app1.com` or `app2.com` |

## Architecture

```text
Browser / curl on the Linux host
        |
        | Host: app1.com / app2.com / app3.com / anything else
        v
192.168.56.110
        |
        v
K3s virtual machine: thitranS
        |
        v
Traefik Ingress Controller
        |
        +-- app1.com        -> Service app-one   -> Pod app-one
        +-- app2.com        -> Service app-two   -> 3 Pods app-two
        +-- other hosts     -> Service app-three -> Pod app-three
```

## Project files

```text
p2/
 ├── confs/
 │   └── apps.yaml
 └── Vagrantfile
```

### `Vagrantfile`

The `Vagrantfile` creates one Debian virtual machine:

```ruby
config.vm.box = "debian/trixie64"
config.vm.hostname = "thitranS"
config.vm.network "private_network", ip: "192.168.56.110"
```

The VM is configured with:

- hostname: `thitranS`
- private IP: `192.168.56.110`
- provider: `libvirt`
- memory: `2048 MB`
- CPUs: `2`

### `apps.yaml`

The `apps.yaml` file defines:

- `Deployment` for `app-one` with `1` replica
- `Deployment` for `app-two` with `3` replicas
- `Deployment` for `app-three` with `1` replica
- one `ClusterIP` service for each application
- one `Ingress` named `apps-ingress`

The applications use the image:

```text
paulbouwer/hello-kubernetes:1.10
```

Each application displays a different message:

```text
Hello from app1.
Hello from app2.
Hello from app3.
```
## Install vagrant

```bash
sudo apt update
sudo apt install -y wget gpg coreutils
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update
sudo apt install -y vagrant
```
## Install the required packages of Libvirt

```bash
sudo apt update
sudo apt install -y \
  qemu-kvm \
  libvirt-daemon-system \
  libvirt-clients \
  libvirt-dev \
  build-essential \
  ruby-dev \
  pkg-config \
  dnsmasq-base \
  ebtables
sudo systemctl enable --now libvirtd
sudo usermod -aG libvirt,kvm $USER
newgrp libvirt
```
  
## Install Vagrant libvirt plugin

```bash
vagrant plugin install vagrant-libvirt
```
## Check existing Vagrant machines

```bash
vagrant global-status
```

## Then clean invalid entries

```bash
vagrant global-status --prune
```
## Direct fix with libvirt

```bash
virsh list --all
virsh destroy thitranSdefault
```

## Then undefine it:

```bash
virsh undefine thitranSdefault --remove-all-storage
```

## Start the virtual machine

Run these commands from the project directory on the host machine:

```bash
vagrant up --provider=libvirt
vagrant ssh
```

After `vagrant ssh`, you are inside the VM.

## Install K3s inside the VM

Inside the VM, install the required packages:

Install curl:

```bash
sudo apt update
sudo apt install -y curl ca-certificates
```

Install K3s in server mode:

```bash
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server --write-kubeconfig-mode=644" sh -
```

Check that K3s is running:

```bash
sudo systemctl status k3s --no-pager
sudo k3s kubectl get nodes -o wide
```

Expected result:

```text
NAME       STATUS   ROLES           VERSION
thitrans   Ready    control-plane   vX.XX.X+k3sX
```

The exact K3s version can be different depending on the installation date.

## Deploy the applications

From inside the VM, apply the Kubernetes configuration:

```bash
sudo k3s kubectl apply -f /vagrant/apps.yaml
```
Then check the resources:

```bash
sudo k3s kubectl get all -o wide
sudo k3s kubectl get ingress -o wide
sudo k3s kubectl describe ingress apps-ingress
```

## Expected Kubernetes resources

You should see three deployments:

```text
app-one     1/1
app-two     3/3
app-three   1/1
```

You should also see three services:

```text
app-one
app-two
app-three
```

And one ingress:

```text
apps-ingress
```

The command below is especially important for the evaluation:

```bash
sudo k3s kubectl get ingress -o wide
```

Example output:

```text
NAME           CLASS     HOSTS               ADDRESS          PORTS
apps-ingress   traefik   app1.com,app2.com   192.168.56.110   80
```

`app3.com` may not appear in the `HOSTS` column. This is normal in this configuration because `app-three` is configured as the default catch-all route. It receives requests when the host is not `app1.com` or `app2.com`.

To prove this clearly, use:

```bash
sudo k3s kubectl describe ingress apps-ingress
```

You should see:

```text
app1.com  /  app-one:80
app2.com  /  app-two:80
*         /  app-three:80
```

## Configure hostnames on the Linux host

On the real Linux host machine, not inside the VM, edit `/etc/hosts`:

```bash
sudo nano /etc/hosts
```

Add:

```text
192.168.56.110 app1.com
192.168.56.110 app2.com
192.168.56.110 app3.com
```

Save and exit.

## Test from the host machine

From the real Linux host machine, run:

```bash
curl http://app1.com
curl http://app2.com
curl http://app3.com
```

Expected result:

```text
Hello from app1.
Hello from app2.
Hello from app3.
```

You can also test directly with the IP address and the `Host` header:

```bash
curl -H "Host: app1.com" http://192.168.56.110
curl -H "Host: app2.com" http://192.168.56.110
curl -H "Host: app3.com" http://192.168.56.110
curl -H "Host: anything.com" http://192.168.56.110
curl http://192.168.56.110
```

Expected routing:

| Command | Expected application |
|---|---|
| `curl -H "Host: app1.com" http://192.168.56.110` | `app-one` |
| `curl -H "Host: app2.com" http://192.168.56.110` | `app-two` |
| `curl -H "Host: app3.com" http://192.168.56.110` | `app-three` |
| `curl -H "Host: anything.com" http://192.168.56.110` | `app-three` |
| `curl http://192.168.56.110` | `app-three` |

## Useful commands for evaluation

Run these commands inside the VM:

```bash
hostname
ip -br a
sudo systemctl status k3s --no-pager
sudo k3s kubectl get nodes -o wide
sudo k3s kubectl get pods -o wide
sudo k3s kubectl get deployments -o wide
sudo k3s kubectl get svc -o wide
sudo k3s kubectl get ingress -o wide
sudo k3s kubectl describe ingress apps-ingress
sudo k3s kubectl get all -o wide
```

Run these commands on the host machine:

```bash
curl http://app1.com
curl http://app2.com
curl http://app3.com
curl -H "Host: app1.com" http://192.168.56.110
curl -H "Host: app2.com" http://192.168.56.110
curl -H "Host: anything.com" http://192.168.56.110
```

## Explanation of the Ingress

The Ingress uses Traefik, which is installed by default with K3s.

The routing logic is:

```text
Host app1.com        -> app-one
Host app2.com        -> app-two
Any other host       -> app-three
```

In `apps.yaml`, the first two rules use explicit hostnames:

```yaml
- host: app1.com
- host: app2.com
```

The last rule has no `host` field:

```yaml
- http:
    paths:
      - path: /
```

This makes it a catch-all rule. Therefore, `app3.com`, `anything.com`, or a direct request to `192.168.56.110` can be routed to `app-three`.

## Why app2 has three pods

In `apps.yaml`, the `app-two` deployment contains:

```yaml
spec:
  replicas: 3
```

This means Kubernetes must keep three running pods for `app-two`.

You can check it with:

```bash
sudo k3s kubectl get deployment app-two
sudo k3s kubectl get pods | grep app-two
```

Expected idea:

```text
app-two   3/3
app-two-xxxxx   Running
app-two-xxxxx   Running
app-two-xxxxx   Running
```

## Troubleshooting

### `vagrant ssh` does not work

Start the VM first:

```bash
vagrant up --provider=libvirt
vagrant ssh
```

### `K3s` is not installed

Check:

```bash
sudo systemctl status k3s --no-pager
```

If the service does not exist, install K3s:

```bash
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server --write-kubeconfig-mode=644" sh -
```

### `/vagrant/apps.yaml` does not exist

Check that the file is present in the synced folder:

```bash
ls /vagrant
```

If `apps.yaml` is not there, make sure you started Vagrant from the directory containing `apps.yaml`.

### `curl http://app1.com` shows Nginx instead of app1

This usually means that `app1.com` is not pointing to the VM IP address.

Check `/etc/hosts` on the host machine:

```bash
grep app1.com /etc/hosts
```

Expected:

```text
192.168.56.110 app1.com
```

You can bypass DNS and test directly with:

```bash
curl -H "Host: app1.com" http://192.168.56.110
```

### `app3.com` does not appear in `kubectl get ingress`

This is normal for the current configuration.

`app-three` is configured as the default route with no explicit host. Therefore, `kubectl get ingress` shows only:

```text
app1.com,app2.com
```

The default route is visible with:

```bash
sudo k3s kubectl describe ingress apps-ingress
```

### Port 80 does not answer

Check Traefik:

```bash
sudo k3s kubectl get pods -A | grep traefik
sudo ss -ltnp '( sport = :80 )'
```

### Clean reset of the VM

From the host machine:

```bash
vagrant halt
vagrant destroy -f
vagrant up --provider=libvirt
vagrant ssh
```

Then reinstall K3s and reapply `apps.yaml`.

## Main evaluation commands

Use these commands during defense:

```bash
sudo k3s kubectl get nodes -o wide
sudo k3s kubectl get all -o wide
sudo k3s kubectl get ingress -o wide
sudo k3s kubectl describe ingress apps-ingress
curl -H "Host: app1.com" http://192.168.56.110
curl -H "Host: app2.com" http://192.168.56.110
curl -H "Host: anything.com" http://192.168.56.110
```


## 3️⃣ Exercise 3: k3d and GitOps with ArgoCD

**The Goal:** Configure a local Kubernetes cluster using k3d, deploy a simple application, and keep it updated automatically using ArgoCD (GitOps).

For this exercise, we will take the concept a bit further and use **k3d**.
>[!NOTE]
> k3d is a very lightweight wrapper that runs k3s inside Docker containers. It allows you to deploy one or more local clusters in seconds without needing to manage VMs with Vagrant.
> To install it use : curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash

Here are the steps to configure a cluster via k3d and continuously deploy our application with ArgoCD:

### Working Files Architecture
Here is how our files are structured for this deployment (based on the `p3/` folder):
```text
p3/
 ├── Makefile                    # Automated useful commands
 └── confs/
     ├── cluster-config.yaml     # k3d cluster configuration
     └── argocd/                 # ArgoCD installation and configuration
         ├── application.yaml
         ├── ingress.yaml
         └── install-argocd.yaml
```

## 1. Create a local Kubernetes cluster with k3d
To provision the cluster, we use the command:
```bash
k3d cluster create --config confs/cluster-config.yaml
```

Example of a k3d config file (`confs/cluster-config.yaml`):
```yaml
apiVersion: k3d.io/v1alpha5
kind: Simple 
metadata:
  name: p3-cluster
servers: 1      # Control Plane Node
agents: 2       # Worker Node(s)
image: rancher/k3s:v1.27.4-k3s1
ports:
  - port: 8080:80
    nodeFilters:
      - loadbalancer
```

## 2. Create a namespace and install ArgoCD

Let's create the dedicated namespace:
```bash
kubectl create namespace argocd
```

Then, let's apply the official installation manifest:
```bash
kubectl apply -n argocd -f confs/argocd/install-argocd.yaml
```

*Note: For security reasons and to better manipulate Kubernetes, we chose not to expose ArgoCD outside the base cluster via a public Ingress, but to use `kubectl port-forward` to connect to it.*

## 3. Ask ArgoCD to monitor and deploy the application
We apply the `application.yaml` file which will configure ArgoCD to point to the source code of our repository (GitOps):
```bash
kubectl apply -n argocd -f confs/argocd/application.yaml
```

>[!NOTE]
> Example of a file describing how the ArgoCD Application (`application.yaml`) synchronizes our folder:
>```yaml
>apiVersion: argoproj.io/v1alpha1
>kind: Application
>metadata:
>  name: app
>  namespace: argocd
>spec:
>  project: default
>  source:
>    repoURL: 'https://github.com/monsieurCanard/antgabri'
>    targetRevision: HEAD
>    path: 'app'
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

## 4. Monitored application manifests (in `app-config/`)
Once ArgoCD is in place, it will read the configurations below to deploy and expose the Pods. We use a textbook case here with the name `wil` and an `nginx` image.

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

### `ingress.yaml` (HTTPS/HTTP routing management)
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
> In this example, all requests arriving at the root (`/`) of the cluster via the gateway are redirected to the service named `wil-service`, which in turn distributes the traffic to the `wil-deployment` pods.

### 💡 Justification of certain strategic choices:
- **Security and network:** As mentioned, I chose not to expose ArgoCD directly via Ingress. This forces the learning of `port-forwarding` with `kubectl` to access the application dashboard securely without exposing it to the internet. As for secrets, passwords and configurations are not hardcoded to encourage best practices. (Ultimately, for grading/correction, I preferred to run ArgoCD with `--insecure` to avoid blocking my terminal and the complications related to self-signed certificate management, but in a real project, this should be handled properly).
- **Automation via Makefile:** I provided a `Makefile` to group recurring bash commands (k3d deployment, applying Kustomizations, namespaces, passwords). It is much more robust. Furthermore, if we want to automate this in a VM via Vagrant, the "shell provisioner" only has to execute a single command (e.g., `make all`).

## Testing the application
For launching the application
```bash
make
```

To get the argocd default password and access the ArgoCD dashboard, run:
```bash
make argocd-password
```

Now you can use your favorite web browser to access the ArgoCD dashboard at `http://argocd.localhost:8080` and log in with:
- **Username:** admin
- **Password:** (the one retrieved with `make argocd-password`)
---

## 🦊 Bonus: Local GitLab

**The Goal:** Push the GitOps concept to the limit by hosting your own Git server locally.

In the previous exercises, ArgoCD monitored a repository hosted on GitHub (an external service). For this bonus, the goal is to replace GitHub with our own **GitLab** instance, completely deployed on our Kubernetes cluster.

### ❓ Why deploy GitLab locally?
- **Total independence:** We no longer depend on an internet connection or Microsoft/GitHub servers. Everything works in a closed loop.
- **Infrastructure control and security:** This simulates an ultra-secure "air-gapped" (isolated) enterprise environment where the source code never leaves the company's internal network (or the cluster).

## ⁉️ Why use Helm for this deployment?
- **Handling complexity:** GitLab is a huge solution composed of many components (PostgreSQL database, Redis, web servers, Sidekiq workers, Gitaly...). Creating and maintaining each YAML resource manually would be extremely cumbersome and error-prone.
- **The Chart concept:** Helm allows you to "package" this entire infrastructure in the form of a **Kubernetes Chart** (`templates/` files coupled with a `Chart.yaml`).
- **Centralized configuration (`values.yaml`):** Instead of modifying your information (passwords, ingress, certificates) file by file, Helm allows you to inject dynamic variables from a single `values.yaml` file.

## Bonus Part Architecture
All files are located in the `bonus/` folder. It contains the configurations for *k3d* and *ArgoCD*, but with one major addition: the `gitlab/` folder.
```text
bonus/
 ├── Makefile
 └── confs/
     ├── cluster-config.yaml
     ├── app/                    # The application to deploy
     │   ├── deployment.yaml
     │   ├── ingress.yaml
     │   └── service.yaml
     ├── argocd/                 # Modified ArgoCD configuration
     │   ├── application.yaml
     │   ├── ingress.yaml
     │   └── install-argocd.yaml
     └── gitlab/                 # GitLab deployment files
         ├── Chart.yaml          # Helm chart structure
         ├── values.yaml
         └── templates/
             ├── deployment.yaml
             ├── ingress.yaml
             └── service.yaml
```

## How does it work?
1. **GitLab Deployment:** We use the manifests provided in `bonus/gitlab/` (shaped as a Helm Chart to facilitate configuration with `values.yaml`) to launch the GitLab server within our cluster. A local domain name (via Ingress) is assigned for browser access.
2. **Local Repository Setup:** Once our GitLab instance is functional, we connect to it, create our project, and use `git push` to push the `confs/app/` folder directly to GitLab.
3. **ArgoCD modification:** The `argocd/application.yaml` file is updated. Instead of pointing to a web address like `https://github.com/...`, the `repoURL` variable will now point to our own internal instance.

>[!NOTE]
> **Example of a modification on the ArgoCD Application resource:**
>```yaml
>spec:
>  source:
>    # Previously: repoURL: 'https://github.com/monsieurCanard/antgabri.git'
>    repoURL: 'http://my-gitlab-gitlab.gitlab.svc.cluster.local/root/wil-app.git'
>    # The above URL allows ArgoCD to contact the GitLab service directly via the internal Kubernetes DNS.
>    targetRevision: HEAD
>    path: 'app-config'
>```

With this architecture, the **GitOps** loop is completed fully autonomously: the developer pushes to the internal GitLab, ArgoCD detects it, and automatically updates the application pods on the same cluster.


## Testing the application
For launching the application
```bash
make
```

To get the argocd default password and access the ArgoCD dashboard, run:
```bash
make argocd-password
```

Now you can use your favorite web browser to access the ArgoCD dashboard at `http://argocd.localhost:8080` and log in with:
- **Username:** admin
- **Password:** (the one retrieved with `make argocd-password`)
---

To access to the GitLab dashboard, run:
```bash
make gitlab-password
```

Then, open your web browser and go to `http://gitlab.localhost:8080`. Log in with:
- **Username:** root
- **Password:** (the one retrieved with `make gitlab-password`)

> [!WARNING]
> Gitlab is a very heavy application. It may take several minutes to start and be fully functional. Be patient after the deployment before trying to access the dashboard or pushing code

When you are in GitLab, create a new project (e.g., "mon-projet") and push the `app-config/` folder to it.
Add the remote URL of your gitlab project to argocd/application.yaml and apply the configuration again:
```bash
kubectl apply -n argocd -f argocd/application.yaml
```
Now, ArgoCD will monitor your local GitLab repository. Any change you push to it will trigger an automatic update of your application on the cluster. This is the essence of GitOps: using Git as the single source of truth for your infrastructure and applications.
