# Préparation & installation Odoo via Helm
## Enoncé

- Installez helm 3 sur votre cluster
- Utilisez le **chart helm** de l’application **odoo** pour le deployer
- **Désactivez** toutes les options de **persistence** de données (`odoo` et `postgres`)
- **Exposez** l’application via un service de type **nodeport** (`30069`)
- Créez un fichier `values.yaml` contenant toutes les **variables** que vous avez **surchargées** et poussez le sur un git dans un dossier
- Vérifiez que l’application est bien accessible via le service créé

# Définir les variables à surcharger

- Service : type NodePort, valeur `30069`
- Persistance Odoo : `persistence: enabled: false`
- Persistance Postgre : `postgresql: ... persistence: enabled: false`

**Note*: On Conservel e paramètre `enabled: true`permettant d'autoriser si nécessaire l'usage d'une BDD externe

# Préparer l'usage du repo HELM

Pour utiliser un chart Helm il faut indiquer son emplacement : le repository qui le contient :

```bash
helm repo add stable https://charts.helm.sh/stable
# "stable" has been added to your repositories
```

Ensuite il est possible de parcourrir la liste des *charts* disponibles avec la commande :
```bash
# Mise à jour de la liste des repo disponibles
helm repo update

# Recherche dans la liste
helm search repo odoo

# ####
# NAME            CHART VERSION   APP VERSION     DESCRIPTION                                       
# stable/odoo     13.0.5          12.0.20200215   DEPRECATED A suite of web based open source bus...
```

**Note*:
Les *apiVersion: v1 Charts* sont installable par Helm v2 & v3 ais à partir de l'*apiVersion: v2* il faudra Helm v3 ou +
Les projets HELM sont généralement maintenus pour une durée de **1 an après leur déploiement**.

Pour ce le repository https://github.com/helm/charts/tree/master/stable il appartient au propriétaire de maintenir le charts en fonction des contribution de la communauté. Il leur est conseillé de n'accepter que des patchs de sécurité.

Ce repository est le dépôt par défaut pour Helm v2

---

Le repository maintenu se trouve à l'adresse : https://github.com/bitnami/charts/tree/main/bitnami/odoo

---

# Installation d'Odoo

Pour installer Odoo via Helm il suffit d'entrer la commande suivante
```bash
# Mise à jour de la liste des repo disponibles
helm install stable/odoo -f values.yaml --generate-name

# level=WARN msg="this chart is deprecated"
# NAME: odoo-1777985168
# LAST DEPLOYED: Tue May  5 14:46:09 2026
# NAMESPACE: default
# STATUS: deployed
# REVISION: 1
# DESCRIPTION: Install complete
# TEST SUITE: None

# ** Please be patient while the chart is being deployed **

# 1. Get the Odoo URL by runningg:

# export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services odoo-1777985168)
# export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
# echo "Odoo URL: http://$NODE_IP:$NODE_PORT/"

# 2. Login with the following credentials

#   echo Email   : user@example.com
#   echo Password: $(kubectl get secret --namespace default odoo-1777985168 -o jsonpath="{.data.odoo-password}" | base64 --decode)
```

En utilisant les instructions proposées en *1.* et *2.* on pourra obtenir l'url et s'y connecter

---

Pour mettre à jour vers la version maintenu il suffit de suivre les instructions suivantes :

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm upgrade my-release bitnami/<chart>
```