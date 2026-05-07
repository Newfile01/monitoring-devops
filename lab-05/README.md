# LAB-05_06 - Ajout scraping apiserver + kube-state-metrics

---

## Scraping api-server

Pour pouvoir scraper l'apiserver et d'autres composants du control-plane il faut définir unnouveau job :

```bash
- job_name: 'kubernetes-apiservers'
        kubernetes_sd_configs:
        - role: endpoints
        scheme: https
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        relabel_configs:
        - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
          action: keep
          regex: default;kubernetes;https
```

Il permet de :
- Utiliser les labels et relabels de kubernetes
- Interroger les APIs en https sur la base du certificats et du token associé au serviceAccount 'default'
- Récupérer certaines sources de données selon une regex précise : namespace=default,  service=kubernetes, port_name=https

Les données sont gérables à cette adresse : https://prometheus.io/docs/prometheus/latest/configuration/configuration/#kubernetes_sd_config

> [!WARNING]
> Bien vérifier que le clusterRole.yaml se trouvant dans les sources/prometheus a été appliqué
> Si ce n'est pas le cas les requêtes de scrping seront rejetées.
>
> Vérifications possibles :
> 1. Le target 'kubernetes-apiservers' ne remonte pas sur l'UI prometheus
> 2. Les logs avec `kubectl logs -n monitoring deploy/prometheus-deployment --tail=200` indiquent
>       GET .../api/v1/endpoints?... → 403 Forbidden
>       GET .../api/v1/services?... → 403 Forbidden
>       GET .../api/v1/pods?... → 403 Forbidden
> 3. Vérifier certains types de requêtes (list, get ,watch) avec un serviceAccount donné :
> ```bash
> kubectl auth can-i list endpoints --as=system:serviceaccount:monitoring:default
> kubectl auth can-i list services --as=system:serviceaccount:monitoring:default
> ```
> Si le retour donne 'no' >> pas d'accès, vérifier l'existence des compte.
> On devrait trouver dans notre cas un serviceAccount "prometheus"
>
> `kubectl get clusterrole prometheus` ou `kubectl get clusterrolebinding prometheus`
>
> Si non présent >> appliquer à nouveau le clusterRole.yaml se trouvant dans `sources/prometheus`

**Note* : Le fichier clusterRole.yaml a été modifier de la manière suivante :

- Api de ClusterRole & ClusterRoleBinding passé de `rbac.authorization.k8s.io/v1beta` à `rbac.authorization.k8s.io/v1`
- Le bloc
```
- apiGroups:
  - extensions
```
est remplacé par `- apiGroups: ["networking.k8s.io"]`

## Scraping kube-state-metrics

Source: https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-state-metrics
Installé en tant que Helm chart depuis le repository `prometheus-community`

```bash
helm install kube-state-metrics oci://ghcr.io/prometheus-community/charts/kube-state-metrics
# ou
helm install kube-state-metrics prometheus-community/charts/kube-state-metrics
```

Celui-ci permet de récupérer un certain nombre de données sur l'ensemble du cluster comme le nombre de ressource d'une certaine sorte créées, le nombre de crash, etc... il vient collecter directement ses données auprès de l'apiserver puis les expose à Prometheus

## Méthode de modification / actualisation de la configMap et du deployment

Pour simplifier les MàJ de la configMap et du deployment de prometheus il est possible d'utiliser les commandes suivantes :

```bash
# Depuis le dossier ou elle se trouve :
kubectl apply -f config-map.yaml
# 'apply' met à jour automatiquement la ressource si elle existe déjà

# Idem, depuis l'emplacement du deployment :
kubectl rollout restart deployment/prometheus-deployment -n monitoring

# Ne pas oublier de couper et exposer à novueau les ports du service prometheus
kubectl port-forward --address 0.0.0.0 svc/prometheus-service 8090:8090 -n monitoring

# RAPPEL pour Grafana :
kubectl port-forward --address 0.0.0.0 svc/grafana-dashboard 9000:9000
```
