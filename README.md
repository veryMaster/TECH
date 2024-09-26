-basics

Objectif :

Déployer Prometheus (outil de monitoring) sur une instance EC2.
Déployer Grafana (outil de visualisation) sur une autre instance EC2.
Prometheus collectera les métriques système, et Grafana utilisera Prometheus comme source de données pour afficher des graphiques et tableaux de bord.
Étapes de l'exercice :

1. Prérequis :

Deux Instances EC2 :
Instance EC2 Prometheus : Instance dédiée pour héberger Prometheus.
Instance EC2 Grafana : Instance dédiée pour héberger Grafana.
Docker installé sur chaque EC2 : Utilise la procédure suivante pour installer Docker :

```
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo usermod -aG docker ubuntu
```
2. Déploiement de Prometheus sur EC2 Prometheus

Créer un fichier de configuration pour Prometheus :

Sur l'instance EC2 dédiée à Prometheus, crée un répertoire pour Prometheus :
```
mkdir prometheus && cd prometheus
```
Crée un fichier prometheus.yml avec le contenu suivant :
```
yaml
Copier le code
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```
Ce fichier indique à Prometheus de surveiller lui-même comme un point de départ.

Créer un fichier Dockerfile pour Prometheus :

Crée un fichier Dockerfile dans le même répertoire :
```
FROM prom/prometheus:v2.37.0

COPY prometheus.yml /etc/prometheus/prometheus.yml
```
Build et Run du conteneur Docker pour Prometheus :

Construis l’image Docker :

```
sudo docker build -t prometheus-custom .
```

Lancer le conteneur Docker en exposant le port 9090 :
```
sudo docker run -d -p 9090:9090 prometheus-custom
```
Vérification :

Accède à Prometheus via l'IP publique de l'instance EC2 Prometheus :
```
http://<EC2-Prometheus-IP>:9090
```
3. Déploiement de Grafana sur EC2 Grafana

Run du conteneur Grafana :

Sur l'instance EC2 dédiée à Grafana, exécute le conteneur Grafana avec Docker :
```
sudo docker run -d -p 3000:3000 --name=grafana grafana/grafana
```
Vérification :

Accède à Grafana via l'IP publique de l'instance EC2 Grafana :
```
http://<EC2-Grafana-IP>:3000
```
Par défaut, le nom d'utilisateur et le mot de passe sont admin / admin.

4. Configuration de Grafana pour utiliser Prometheus comme source de données

Ajout de Prometheus comme source de données :

Dans l'interface Grafana, connecte-toi et suis ces étapes :
Clique sur Configuration (icône d'engrenage) > Data Sources.
Clique sur Add data source.
Choisis Prometheus.
Dans le champ URL, entre l’URL de Prometheus (IP publique de l’instance EC2 Prometheus avec le port) :
```
http://<EC2-Prometheus-IP>:9090
```
Clique sur Save & Test pour vérifier la connexion.

Créer un tableau de bord Grafana :

Crée un nouveau tableau de bord pour visualiser les métriques :
Clique sur + dans la barre latérale gauche > Create > Dashboard.
Ajoute un graphique en cliquant sur Add new panel.
Dans la section Query, choisis Prometheus comme source de données.
Utilise la requête suivante pour afficher le taux d'utilisation CPU de Prometheus :
scss
```
rate(process_cpu_seconds_total[1m])
Sauvegarde et affiche le graphique.
```

Explication des Concepts Microservices :

Architecture distribuée :

Prometheus et Grafana sont déployés sur des instances EC2 distinctes, chacun dans son propre conteneur Docker.
Ces deux services indépendants fonctionnent ensemble pour réaliser une solution de monitoring.
Modularité :

Prometheus est un service de collecte de métriques. Il peut être remplacé ou mis à l'échelle indépendamment de Grafana.
Grafana est une interface de visualisation des métriques, ce qui permet une indépendance fonctionnelle et une meilleure évolutivité.
Communication via des APIs :

Grafana communique avec Prometheus en utilisant l'API HTTP de Prometheus, un aspect clé des microservices où chaque service expose une API pour la communication.
Isoler les services :

Chaque service est dans un conteneur Docker, facilitant l’isolation et le déploiement séparé sur des infrastructures différentes, tout en restant connectés via des réseaux.
