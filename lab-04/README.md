# LAB-04

--- 

## Monitoring des hôtes à l'aide de node-exporter

Exporter : Outils qui permet à Prometheus d'aller lire le composant à monitorer. Quand Prometheus voudras la donnée il interrogera l'exporter. Il les exposera sous forme de endpoints.


Pour installer le node-exporter nous utiliserons un Helm chart se trouvant sur le repo : https://github.com/prometheus-community/helm-charts

```bash
# Ajout du repo helm
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
# Trouver les charts correspondant
helm search repo node-exporter
# Installation
helm install node-exporter oci://ghcr.io/prometheus-community/charts/prometheus-node-exporter
```

Exposition & accès au `node-exporter`

```bash
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=prometheus-node-exporter,app.kubernetes.io/instance=node-exporter" -o jsonpath="{.items[0].metadata.name}")
  echo "Visit http://127.0.0.1:9100 to use your application"
  kubectl port-forward --namespace default $POD_NAME 9100
```

## Connexion Prometheus-Node-exporter

On commence par modifier la configmap.yaml dans `sources/prometheus`

Ajout du job
```bash
# ...
- job_name: 'node-exporter'
  static_configs:
    - targets: ['node-exporter-prometheus-node-exporter.default.svc:9100']
# ...
```

On doit ensuite supprimer puis recréer `configmap.yaml`et `prometheus-deployment.yaml` pour prendre en compte les modifications :

```bash
# Suppression ConfigMap
kubectl delete configmaps prometheus-server-conf -n monitoring
# Suppression deployment
kubectl delete deployment prometheus-deployment -n monitoring
# Création nouvelle CM et nouveau DP (depuis la racine de ce repository)
kubectl apply -f lab-04/config-map.yaml
kubectl apply -f sources/prometheus/prometheus-deployment.yaml -n monitoring
```

## Choix du Dashboard Grafana pour le Node-Exporter

Contrairement à ce qui est indiqué dans le lab-04 Eazytraining, nous utiliserons le dashboard suivant : https://grafana.com/grafana/dashboards/7249-kubernetes-cluster/

Adapté au cluster Kubernetes# Suppression ConfigMap
