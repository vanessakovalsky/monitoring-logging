# Lab - Découverte des métriques

## Objectifs

* Découvrir les signaux d'or
* Faire ses premières requêtes PromQL

## Pré-requis

* Avoir prometheus installer sur un cluster K8s

## Découverte des signaux d'or

* Une des premières question que l'on se pose lorsque l'on fait du monitoring est de savoir qu'est ce qui va être surveillé, quelles sont les métriques importantes à surveiller.
* Pour cela, on peut s'aider de document comme celui de Google [Monitoring distributed system](https://sre.google/sre-book/monitoring-distributed-systems/)

* On distingue quatre signaux d'or pour le monitoring :
    * le trafic : on mesure la "demande imposée" au système
    * la latence : temps écoulé entre la réception de la requête et la réponse envoyée
    * les erreurs : taux d'erreurs des requêtes
    * la saturation : réseaux, disque et mémoire ont un point de saturation à partir duquel la demande dépasse la limite de performance d'un service

* Vous pouvez lire également : https://www.lemagit.fr/conseil/Les-quatre-Golden-Signals-et-comment-les-mettre-en-pratique 

## Découvertes des requêtes PromQL

* Nous voulons définir des requêtes pour les élements suivants 
- Taux de l'utilisation du CPU pour chaque conteneur : 
```
sum(rate(container_cpu_usage_seconds_total{instance=~".*"}[5m])) by (pod) *100
```
- Ainsi que l'utilisation de la mémoire
```
sum(container_memory_rss{instance=~".*",name=~".*",name=~".+"}) by (name)
```
- Et celle du trafic réseau reçu : 
```
sum(rate(container_network_receive_bytes_total{instance=~".*",name=~".*",name=~".+"}[5m])) by (name)
```
- Et le trafic sortant :
```
sum(rate(container_network_transmit_bytes_total{instance=~".*",name=~".*",name=~".+"}[5m])) by (name)
```

* Analyser ses requêtes et définissez vos propres requêtes en fonction des métriques disponibles et de ce que vous souhaitez surveiller
