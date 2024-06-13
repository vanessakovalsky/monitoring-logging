# Lab - Installer et configurer Prometheus

## Objectif

Ce lab vous permet de : 

* savoir installer prometheus sur un cluster Kubernetes ou via un docker compose
* Vérifier que Prometheus fonctionne et accéder à son interface web
* Avoir Helm installé

## Pré-requis 

* Avoir un cluster Kubernetes installer et kubectl configuré
* ou : avoir docker et docker compose installé et fonctionnel

## Installation de Prometheus

### Avec Kubernetes

* Commencer par récupérer le chart helm, créer un namespace et installer le helm chart
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
kubectl create namespace monitoring
helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring
```
* Puis exposer le service Prometheus pour pouvoir y accéder 
```
kubectl port-forward --address 0.0.0.0 svc/prometheus-kube-prometheus-prometheus -n monitoring 9090 
```
* /!\ Laisser tourner pour accéder à Prometheus :)

### Avec docker compose

* Récupérer dans le dépôt le fichier docker-compose.yml
* Récupérer à cette adresse le fichier de configuration prometheus : https://github.com/vanessakovalsky/grafana-training/blob/main/infra/prometheus.yml
* Mettre les deux fichiers dans le même dossier et ouvrir une invite de commande ou un terminal dans ce dossier, puis faire la commande :
```
docker compose up -d
```
* Les conteneurs vont se lancer, pour vérifier que tout est ok, vous pouvez utilisez la commande :
```
docker compose ps 
```
* Les conteneurs doivent alors tous être au status `up`

## Accéder à Prometheus

* Prometheus fournit une interface web exposer par defaut sur le port 9090
* Avec minikube utiliser localhost:9090 (ou récupérer l'URL correct avec minikube service nomduservice --url )
* Avec docker compose utiliser localhost:9091 

* Vous accéder alors à une interface ressemblant à celle-ci

![](img/Console-Prometheus.jpg)

* Certaines métriques de bases sont déjà surveillées et accessible, entrer dans le champs de recherche le nom `machine_memory_bytes`, passez en vue graphique et cliquer sur Exécuter 
* Vous obtiendrez alors

![](img/La-chaine-metrique-de-Prometheus.jpg)

* Les conteneurs sont également déjà monitorés, par exemple utiliser l'expression `rate(container_cpu_usage_seconds_total{}[1m])`pour afficher le taux d'utilisation du CPU pour le conteneur Prometheus 

* -> Votre installation de Prometheus est fonctionnelle
