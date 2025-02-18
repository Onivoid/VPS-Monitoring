# 📊 Monitoring & Gestion Docker avec Grafana, Prometheus, cAdvisor & Portainer

Ce projet permet de **monitorer et gérer ses conteneurs Docker** avec une stack complète basée sur **Grafana, Prometheus, cAdvisor, Caddy et Portainer**.

---

## 🚀 Fonctionnalités
- **Monitoring en temps réel** 📊 : Suivi de l'utilisation CPU, RAM, réseau des conteneurs Docker avec **Grafana + Prometheus + cAdvisor**.
- **Gestion des conteneurs** 🛠️ : Interface web pour gérer Docker avec **Portainer**.
- **Accès sécurisé** 🔒 : Reverse Proxy avec **Caddy** pour une connexion HTTPS.

---

## 📂 Structure du projet
```
.
├── Caddyfile            # Configuration de Caddy (Reverse Proxy)
├── docker-compose.yml  # Stack Docker principale
├── grafana/            # Configuration pour Grafana
│   └── provisioning/
│       └── datasources/
│           └── datasources.yml # Configuration de Prometheus pour Grafana
└── prometheus.yml      # Configuration de Prometheus
```

---

## 📦 Prérequis
- **Un serveur avec Docker & Docker Compose**
  ```bash
  sudo apt update && sudo apt install -y docker docker-compose
  ```
- **Un nom de domaine pointant vers le serveur** (ex: `monitor.example.com`)

---

## 🛠️ Installation & Déploiement

### **1️⃣ Cloner le repo et se positionner dans le dossier**
```bash
git clone https://github.com/votre-repo.git monitoring-stack
cd monitoring-stack
```

### **2️⃣ Vérifier et modifier les fichiers de configuration**

#### 📝 **Configurer `prometheus.yml` (Prometheus -> cAdvisor)**
Le fichier `prometheus.yml` contient la configuration de Prometheus pour récupérer les données de cAdvisor.
Vérifiez qu'il est bien présent et qu'il contient :
```yaml
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['prometheus:9090']

  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']
```

#### 📝 **Configurer `datasources.yml` (Grafana -> Prometheus)**
Le fichier `grafana/provisioning/datasources/datasources.yml` doit être présent et contenir :
```yaml
apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
```

#### 📝 **Configurer le `Caddyfile` pour HTTPS**
Le fichier `Caddyfile` doit contenir :
```caddyfile
https://domain.fr {
  handle_path /portainer/* {
    reverse_proxy portainer:9000
  }

  handle_path /cadvisor/* {
    reverse_proxy cadvisor:8080
  }

  handle_path /prometheus/* {
    reverse_proxy prometheus:9090
  }

  handle /* {
    reverse_proxy grafana:3000
  }
}
```
📌 **Remplace `domain.fr` par ton propre domaine !**

#### 📝 **(Optionnel) Importer un Dashboard Docker dans Grafana**
Si vous souhaitez importer un dashboard Docker dans Grafana, suivez ces étapes :
```bash
mkdir -p grafana/provisioning/dashboards
nano grafana/provisioning/dashboards/dashboard.yml
```
Ajoutez :
```yaml
apiVersion: 1
providers:
  - name: "default"
    folder: ""
    type: file
    options:
      path: /var/lib/grafana/dashboards
```
Ensuite, créez le dossier des dashboards et téléchargez un exemple :
```bash
mkdir -p grafana/dashboards
curl -o grafana/dashboards/docker.json https://grafana.com/api/dashboards/193/revisions/latest/download
```
Ajoutez le volume correspondant dans `docker-compose.yml` :
```yaml
services:
  grafana:
    volumes:
      - ./grafana/dashboards:/var/lib/grafana/dashboards
```

---

### **3️⃣ Lancer la stack Docker**
```bash
docker compose up -d
```

📌 **Les services démarreront et seront accessibles aux URLs suivantes :**
- **Grafana** → `https://domain.fr`
- **Prometheus** → `https://domain.fr/prometheus`
- **cAdvisor** → `https://domain.fr/cadvisor`
- **Portainer** → `https://domain.fr/portainer`

---

## 🔄 Maintenance & Gestion

### **Démarrer/Arrêter les services**
```bash
docker compose stop  # Arrêter la stack
docker compose start # Redémarrer la stack
docker compose down  # Supprimer la stack (sans effacer les volumes)
```

### **Vérifier les logs**
```bash
docker logs -f prometheus  # Voir les logs de Prometheus
docker logs -f grafana     # Voir les logs de Grafana
docker logs -f caddy       # Voir les logs de Caddy
```

### **Mettre à jour la stack**
```bash
docker compose pull  # Met à jour les images Docker
docker compose up -d --force-recreate  # Redéploie les services
```

---

## 🛠️ Contribuer
Si vous souhaitez contribuer à ce projet, n’hésitez pas à proposer une **issue** ou une **pull request** !

---

## 📜 License
MIT License - Vous êtes libre d’utiliser et de modifier ce projet.

---

🎉 **Et voilà, votre stack Docker est prête à l’emploi !** 🚀😃

