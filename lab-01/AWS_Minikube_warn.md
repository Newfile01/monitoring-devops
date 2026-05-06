# Exposition de Minikube dans AWS

Pour pouvoir accéder à un service depuis l'extérieur ceux-ci doivent être de type NodePort ou LoardBalancer.
Cependant, l'accès n'est donné qu'à la première couche externe au cluster. Dans un environnement cloud comme AWS ce n'est pas pareil :

AWS (Accès externe) [ VPC (Réseau privé dans le Cloud) [ EKS (Minikube) ]]

Il faut donc imposer à minikube l'exposition du service pour toutes les adresses extérieures et sur un port précis afin de pouvoir faire du port-forwarding :

```bash
# Obtenir le détail sur le service exposant Odoo (ici dans le namespace default)
kubectl get svc

# On forward le port 80, interne au service Odoo, vers le port 8069, exposé à l'extérieur du cluster
kubectl port-forward --address 0.0.0.0 svc/odoo 8069:80

# Configurer ensuite les règles de pare-feu pour autoriser les connexions entrantes (Inbound) sur le port 8069 dans le groupe de sécurité associé au VPC
#    [Type] Custom TCP   [Protocol] TCP    [Port range] 8069   [Source] Custom  0.0.0.0/0   [Desc] ...
#
# On se connecte ensuite en http au port 8069 avec l'IP public de la VM EC2 hébergeant le cluster (même id/pwd que fournis lors de l'installation du chart Odoo)
```