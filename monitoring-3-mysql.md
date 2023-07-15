# Lab - Surveiller des BDD MySQL avec son exporter

## Déploiement de mysql et de mysql exporter pour collecter les métriques dans prometheus 

- Récuper les deux fichier yaml dans le dossier mysql 
- Créer le secret et deployer les pods
```
kubectl create secret generic mysecret --from-literal=ROOT_PASSWORD=demo -n monitoring
kubectl apply -f mysql-statefulset.yaml
```

- Déployer prometheus et le scrapper 
```
helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring -f values.yaml
```

## Exploitation des métriques de l'exporter

* Nous allons définir quelques requêtes à partir des métriques disponibles
    * Uptime : mysql_global_status_uptime{}
    * Current QPS : rate(mysql_global_status_queries{}[$interval]) or irate(mysql_global_status_queries{}[5m])
    * InnodDB Buffer Pool Size : mysql_global_variables_innodb_buffer_pool_size{}
    * Buffer Pool Size of Total RAM : (mysql_global_variables_innodb_buffer_pool_size{} * 100) / on (instance) node_memory_MemTotal_bytes{}
* A quoi corresponde ces métriques et requêtes ?
* Quels sont les autres métriques intéressantes à observer ?
