# MàJ du repository de chart

> [!WARNING] 
> Suite à une migration des charts de Bitnami sur un nouveau repository en 2025 il convient de modifier le repository où se trouve le chart Odoo à utiliser

Nous suiverons donc les instructions stipulées à cette adresse : https://artifacthub.io/packages/helm/bitnami/odoo

```bash
helm repo add oci://registry-1.docker.io/bitnamicharts
helm install my-release oci://registry-1.docker.io/bitnamicharts/odoo -f values.yaml
```