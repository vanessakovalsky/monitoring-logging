# Lab - Surveiller des BDD MySQL avec son exporter

## Déploiement de mysql et de mysql exporter pour collecter les métriques dans prometheus 

### Sur Kubernetes

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

### Avec docker compose

* Ouvrir le fichier docker-compose.yml et remplacer son contenu par le suivant :
```
version: '3.7'

services:
    app:
        image: 'vanessakovalsky/kingoludo-laravel'
        container_name: shouts-laravel-app
        ports:
            - "8090:80"
        networks:
            - laravel-shouts
        depends_on:
            - mysql
        command: sh -c "/usr/local/bin/initscript.sh && apache2-foreground"
    mysql:
        image: 'mariadb:latest'
        container_name: shouts-laravel-db
        restart: unless-stopped
        ports:
            - "3310:3306"
        environment:
            MARIADB_DATABASE: laravel
            MARIADB_ROOT_PASSWORD: rootpassword
            MARIADB_PASSWORD: larapass
            MARIADB_USER: laravel
        volumes:
            - dbdata:/var/lib/mysql
        networks:
            - laravel-shouts
    prometheus:
      image: prom/prometheus:latest
      container_name: prometheus
      user: root
      ports:
      - 9090:9090
      command:
      - --config.file=/etc/prometheus/prometheus.yml
      - --storage.tsdb.path=/prometheus'
      - --web.console.libraries=/etc/prometheus/console_libraries
      - --web.console.templates=/etc/prometheus/consoles
      - --web.enable-lifecycle
      volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus_data:/prometheus
      depends_on:
      - cadvisor
    node-exporter:
      image: prom/node-exporter:latest
      container_name: node-exporter
      restart: unless-stopped
      volumes:
        - /proc:/host/proc:ro
        - /sys:/host/sys:ro
        - /:/rootfs:ro
      command:
        - '--path.procfs=/host/proc'
        - '--path.rootfs=/rootfs'
        - '--path.sysfs=/host/sys'
        - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
      expose:
        - 9100
    cadvisor:
      image: gcr.io/cadvisor/cadvisor:latest
      container_name: cadvisor
      ports:
      - 8080:8080
      volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:rw
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro

    grafana:
        image: grafana/grafana-enterprise
        container_name: grafana
        restart: unless-stopped
        ports:
        - '3000:3000'
    
    mysql-exporter:
        image: quay.io/prometheus/mysqld-exporter
        container_name: mysqld-exporter
        command:
            - "--mysqld.username=exporter:exporter"
            - "--mysqld.address=mysql:3306"
        ports:
        - 9104:9104
        links:
        - mysql
        depends_on:
        - mysql

networks:
    laravel-shouts:
        driver: bridge

volumes:
    dbdata: {}
    prometheus_data: {}
```
* Enregistrer le fichier et fermer le
* Remplacer le contenu du fichier prometheus.yml par le suivant :
```
scrape_configs:
  - job_name: cadvisor
    scrape_interval: 5s
    static_configs:
    - targets:
      - cadvisor:8080

  - job_name: node
    static_configs:
    - targets:
      - node-exporter:9100
  
  - job_name: mysql
    static_configs:
      - targets:
        - mysql-exporter:9104
```
* Ensuite on va ajouter un utilisateur sur la base de données pour l'exporter, pour cela ouvrir un terminal et faire les commandes suivantes :
```
docker compose exec -it mysql /bin/bash
mysql -u root -p
(entrer le mot de passe préciser dans le docker compose)
CREATE USER 'exporter'@'localhost' IDENTIFIED BY 'exporter' WITH MAX_USER_CONNECTIONS 3;
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'%';
exit
exit
docker compose up -d
docker compose restart prometheus
```

* Accéder à prometheus et dans les targets vous devriez voir mysql apparaître.

## Exploitation des métriques de l'exporter

* Nous allons définir quelques requêtes à partir des métriques disponibles
    * Uptime : mysql_global_status_uptime{}
    * Current QPS : rate(mysql_global_status_queries{}[$interval]) or irate(mysql_global_status_queries{}[5m])
    * InnodDB Buffer Pool Size : mysql_global_variables_innodb_buffer_pool_size{}
    * Buffer Pool Size of Total RAM : (mysql_global_variables_innodb_buffer_pool_size{} * 100) / on (instance) node_memory_MemTotal_bytes{}
* A quoi corresponde ces métriques et requêtes ?
* Quels sont les autres métriques intéressantes à observer ?
