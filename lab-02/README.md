# Installation de prometheus

On commence par récupérer les sources sur le repository Github : https://github.com/eazytrainingfr/prometheus-training/tree/main#

Pour plus de lisibilité on place les sources dans à la racine de notre repository.

Les éléments à déployer dans l'ordre sont :

- clustRole.yaml
- config-map.yaml
- prometheus-deployement.yaml
- prometheus-service.yaml

Le fichier config-map représentera toutes les sources de données scrapées par Prometheus

## Exposition du service Prometheus

Pour permettre l'accès à l'UI Prometheus, un service NodePort est créé et pointe vers le 9090.
Ce service est exposé sur le port 8090, il faut donc port-forward le 8090 (service) vers l'extérieur.
Ici on choisira de l'exposer sur le 8090 aussi (à l'extérieur du cluster) :

```bash
# CTRL-C pour stopper
kubectl port-forward --address 0.0.0.0 svc/prometheus-service 8090:8090 -n monitoring
```

## Installation de Grafana

Pour installer Grafana nous suiverons les instructions données ici : https://github.com/grafana-community/helm-charts/tree/main


```bash
helm repo add grafana-community https://grafana-community.github.io/helm-charts
helm repo update
```

Nous utiliserons les `values.yaml` définies dans les sources/ (après vérification)
Utiliser les commandes suivantes après installation pour récupérer l'url de connexion :

```bash
 export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services grafana-dashboard)
     export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
     echo http://$NODE_IP:$NODE_PORT
```

Pour l'utilisateur `admin` et pour le mdp utiliser la commande suivante :

```bash
kubectl get secret --namespace default grafana-dashboard -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```
Il faudra ensuite à novueau exposer le service de Grafana cette fois-ci (se trouvant dans le namespace default) afin d'y accéder :

```bash
kubectl port-forward --address 0.0.0.0 svc/grafana-dashboard 9000:9000
```

> [!WARNING]
> Sur un cluster local, il n'y aura pas besoin d'IP particulière, le port-forward vers l'adresse 0.0.0.0 permet d'atteindre le service au travers du localhost. Ici l'adresse sera donc http://localhost:9000

## Ajout de Promethues à Grafana en tant que source de donnée

Pour lier Prometheus et Grafana il faut ajouter une source de donnée en choisissant "Prometheus" dans le type Time series puis ajouter l'url du **SERVICE** prometheus permettant de récupérer l'ensemble de ses données : http://prometheus-service.monitoring.svc:8090

Cliquez ensuite sur "Save & Test" qui indiquera directement si la connexion est bonne

## Intégration d'un dashboard adapté

La communauté propose un grand nombre de dashboard pré-construits pour divers usages à l'url : https://grafana.com/grafana/dashboards/

Dans notre cas, nous utiliserons le dashbard 3662 pour représenter les données de Prometheus
 : https://grafana.com/grafana/dashboards/3662-prometheus-2-0-overview/

Il suffira ensuite de faire "+" "Import" et d'entrer l'ID du dashboard avant d'apply (choisir la source de donnée du dashboard : Prometheus)


