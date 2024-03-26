# Définir des alertes avec Alert Manager

## Objectifs

* Configurer les alertes avec Alert Manager
* Vérifier le déclenchement des alertes

## Pré-requis

- Avoir Prometheus configuré et installé

## Installation et configuration de AlertManager

* Ajouter dans le fichier docker-compose.yml le service alertmanager suivants :
```
alertmanager:
        image: prom/alertmanager:latest
        restart: unless-stopped
        ports:
        - "9093:9093"
        volumes:
        - "./alertmanager:/config"
        - alertmanager-data:/data
        command: --config.file=/config/alertmanager.yml --log.level=debug
```
* Ajouter dans la partie volumes du docker-compose.yml la ligne suivantes :
```
    alertmanager-data: {}
```
* Enfin dans le même fichier, dans le service prometheus remplacer la ligne de partage du fichier my-recording-rules.yml par la ligne suivante
```
      - ./rules:/etc/prometheus/rules:ro
```
* Créer un dossier rules et déplacer votre fichiers my-recording-rules.yml dans ce dossier
* Créer à la racine un fichier alertmanager.yml (qui va contenir la configuration d'alertmanager).
* Ajouter dans ce fichier les lignes suivantes :
```
route:
  receiver: 'mail'
  repeat_interval: 4h
  group_by: [ alertname ]


receivers:
  - name: 'mail'
    email_configs:
      - smarthost: 'smtp.gmail.com:465'
        auth_username: 'your_mail@gmail.com'
        auth_password: ""
        from: 'your_mail@gmail.com'
        to: 'some_mail@gmail.com'
```
* Ensuite il nous reste à rédémarrer notre environnement pour prendre en compte les modifications
```
docker compose down
docker compose up -d
```
* Une fois les conteneurs démarré, vous devriez accéder à l'interface d'alertmanager sur l'adresse : http://localhost:9093


## Définition des alertes

* Dans le fichier de configuration de prometheus, on vérifie la configuration d'Alert Manager qui doit correspondre à la suivante :
```
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - localhost:9093
    scheme: http
    timeout: 10s

rule_files:
  - 'rules/*'
```
* Puis on définit des règles d'alertes dans un fichier (à créer) alerts.yml
```
groups:
  - name: toto
    rules:
    - alert: InstanceDown
      expr: up == 0
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: "L'instance {{ $labels.instance }} est down"
        description: "Le Job: {{ $labels.job }} signale que {{ $labels.instance}} est down depuis deja 1 minute. Tu attends quoi ?"
    - alert: DiskFull
      expr: node_filesystem_free_bytes{mountpoint ="/stockage",instance="serveurparticulier:9100"} / 1024 / 1024 / 1024 < 20
      for: 1m
      labels:
        severity: warning
      annotations:
        summary: "Et zut, on arrive a moins de 20Go sur {{ $labels.instance }}"
        description: "Il reste precisement {{ $value }}"
```
* Une fois le fichier enregistrer on peut vérifier la syntaxe de celui-ci avec la commande :
```
promtool check rules /etc/prometheus/rules/alerts.yml
```

## Vérifier le déclenchement des alertes

* Pour tester le déclenchement de l'alerte, on peut arrêter le service node exporter
* Une fois le service arrêté, on pourra voir dans l'interface graphique de Prometheus, l'alerte passé en orange, puis au bout de quelques minutes, passé en rouge.

## Pour aller plus loin

* Quels seraient les alertes pertinentes à mettre en place sur vos propres projets ?
* Définir les règles d'alertes pertinentes et leur règles et vérifier leur fonctionnement.
