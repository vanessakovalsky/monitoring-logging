# Définir des règles d'enregistrement

## Objectifs :

- Découvrir la syntaxe des règles d'enregistrement
- Mettre en place des règles d'enregistrement sur Prometheus

## Pré-requis

* Avoir un prometheus installé qui monitore des conteneurs docker
* Avoir accès aux fichiers de configuration de prometheus

## Mise en place des règles

* Depuis le fichier de configuration principal de prometheus, modifier la ligne contenant `rules_files` pour lui indiquer le chemin du fichier de règles
```
# Rule files specifies a list of globs. Rules and alerts are read from
# all matching files.
rule_files:
  - ./my-recording-rules.yml
```
* Enregistrer le fichier de configuration et fermer le
* Créer un fichier my-recording-ryles.yml et ajouter le contenu suivant
```
groups:
- name: my-recording-rules
  rules:
  - record: instance_mode:node_cpu:rate1m # The new output metric name.
    expr: sum without(cpu) (rate(node_cpu_seconds_total{job="node"}[1m]))
    labels:
      my_label: my_value
```
* Nous avons ajouter deux règles d'enregistrement
* Vérifier dans Prometheus que vos metriques `instance_mode:node_cpu:rate1m` sont disponibles.

## Pour aller plus loin

* Définir vos propres règles d'enregistrement sur les métriques qui vous semblent pertinent
* Les ajouter dans le fichier de configuration des règles d'enregistrement
* Utiliser ces nouvelles métriques dans vos requêtes et/ou visualisation
