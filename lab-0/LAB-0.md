# Mise en place de l'environnement de labs

## Installation Kubernetes

Ici nous utiliserons la distrubution [Minikube](https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fbinary+download).

Sa vocation est de permettre toutes les possibilités de Kubernetes sur un cluster mono ou multi-noeuds même avec un seul hôte.
L'architecture du cluster dépendra des paramètres fournis à la création du cluster et notamment du driver choisi :
- Docker
- KVM
- VirtualBox
- ...

## CLI Kubectl

Il nous faudra l'interpréteur de commande Kubernetes [Kubectl](https://kubernetes.io/fr/docs/tasks/tools/install-kubectl/)
Il permettra d'interagir avec les APIs du cluster

L'auto-complétion est un plus à avoir afin de faciliter l'utilsiation
L'emploi d'alias peut apporter aussi un certain confort en usage parallèle à Minikube

##  ![HELM](https://helm.sh/img/helm.svg) Installation Helm
Le gestionnaire de paquets Kubernetes Helm constitue un moyen de templatiser les manifests Kubernetes.
Il nous faudra installer sa version actuelle (au moment du lab) v4.1.4 (Major.Minor.Patch) : [Install HELM](https://helm.sh/docs/intro/install/)

De notre côté nous avons utilisé la méthode *[From Script](https://helm.sh/docs/intro/install/#from-script)* permettant de télécharger et exécuter le script d'installation de Helm.