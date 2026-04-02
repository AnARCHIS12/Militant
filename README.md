# MILITANT

<div align="center">
  <img src="assets/logo.svg" alt="Militant" width="200"/>
  <br/><br/>
  <img src="assets/badges/anarchist.svg" alt="Anarchist" height="30"/>
  <img src="assets/badges/antifa.svg" alt="Antifa" height="30"/>
  <img src="assets/badges/militant.svg" alt="Militant" height="30"/>
  <br/><br/>
</div>

Réseau social militant autogéré - v2.2

## Architecture Technique

MILITANT est conçu pour être léger, rapide et ultra-sécurisé.
- Backend : PHP 8.2 (Vanilla)
- Frontend : Vanilla HTML/JS, Bootstrap 5.3
- Base de données : MySQL 8.0 / MariaDB
- Networking : Apache 2.4 avec support mod_remoteip
- Conteneurisation : Docker & Docker Compose
- CI/CD : GitLab CI (Auto-build & Push vers GitLab Registry)

## Guide du Mainteneur : Build & Distribution

### Système de Versioning Automatique

MILITANT utilise GitLab CI pour builder et publier automatiquement les images Docker.

**Version déterminée par :** `config/version.php`

#### Créer une nouvelle version

```bash
# Utiliser le script automatique
bash release.sh 2.2.

# Publier sur GitLab
git push origin main
git push origin v2.2
```

Le script `release.sh` fait automatiquement :
- Met à jour `config/version.php`
- Crée un commit de release
- Crée un tag Git `v2.2`

#### Ce qui se passe sur GitLab

Quand vous poussez un tag (ex: `v2.2`), GitLab CI automatiquement :
1. Build l'image Docker
2. La publie avec les tags :
   - `registry.gitlab.com/militant1/millitant:2.3.0`
   - `registry.gitlab.com/militant1/millitant:latest`

#### Vérifier les versions disponibles

Sur GitLab : **Packages & Registries** → **Container Registry**

Vous verrez toutes les versions : `latest`, `2.2.0`, `2.3.0`, etc.

#### Build manuel (si nécessaire)

```bash
# Build local
docker build -t militant:2.3.0 .

# Tag et push vers GitLab
docker tag militant:2.2. registry.gitlab.com/militant1/millitant:2.2
docker tag militant:2.2 registry.gitlab.com/militant1/millitant:latest
docker push registry.gitlab.com/militant1/millitant:2.2
docker push registry.gitlab.com/militant1/millitant:latest
```

### Mise à jour à chaud (Sans redémarrage)

MILITANT supporte les mises à jour à chaud via GitLab :

**Via l'interface Admin :**
1. Admin → Mises à jour
2. Vérifier les mises à jour
3. Cliquer sur "Mettre à jour"
4. Recharger après 10 secondes

**Via script :**
```bash
bash hot_update.sh
```

**Avantages :**
- Zéro downtime
- Pas de redémarrage
- Mise à jour en 10 secondes
- Connexions maintenues

## Installation & Déploiement

### Développement (Local)

Pour contribuer ou tester avec les outils de dev (Mailpit, Hot-reload...) :

```bash
git clone https://gitlab.com/anarchymedialibertaire-group/millitant.git
cd millitant
docker compose up -d --build
```

- Site web : http://localhost:9000
- Emails (Mailpit) : http://localhost:8025
- Admin : http://localhost:9000/setup_admin.php

### Production (Serveur)

Pour déployer sur un serveur en ligne (VPS, dédié...) :

1. Télécharger le fichier de production :
```bash
curl -O https://gitlab.com/anarchymedialibertaire-group/millitant/-/raw/main/docker-compose.prod.yml
```

2. Configurer l'environnement :
Créez un fichier .env à côté :
```env
DB_PASS=votre_mot_de_passe_sql
SMTP_HOST=smtp.votre-fournisseur.com
SMTP_USER=contact@votre-domaine.com
SMTP_PASS=votre_mot_de_passe_smtp
APP_URL=https://votre-domaine.com
```

3. Lancer :
```bash
docker compose -f docker-compose.prod.yml up -d
```

### Auto-hébergement Professionnel (Recommandé)

Cette méthode utilise l'image pré-construite et est la plus simple à maintenir.

1. Télécharger les fichiers nécessaires :
```bash
curl -O https://gitlab.com/militant1/millitant/-/raw/main/docker-compose.hosting.yml
curl -O https://gitlab.com/militant1/millitant/-/raw/main/.env.template
```

2. Configuration :
```bash
cp .env.template .env
nano .env # Remplissez vos tokens et mots de passe
```

3. Lancement :
```bash
docker compose -f docker-compose.hosting.yml up -d
```

### Utiliser l'image Docker toute prête (GitLab Registry)

Si vous ne voulez pas builder l'image vous-même, vous pouvez utiliser l'image officielle générée automatiquement par notre GitLab :

1. Se connecter au registre GitLab (nécessaire une seule fois) :
```bash
docker login registry.gitlab.com
```

2. Utiliser l'image dans votre docker-compose.yml :
```yaml
services:
  app:
    image: registry.gitlab.com/militant1/millitant:latest
    # ... reste de la config
```

### Installation via CasaOS

1. Ouvrez App Center puis cliquez sur Custom Install (en haut à droite).
2. Cliquez sur l'icône Docker Compose (en haut à droite de la modale).
3. Collez le contenu du fichier docker-compose.casaos.yml.
4. Cliquez sur Install.
5. Accédez à http://IP-CASAOS:8080
6. Créer le compte admin : Allez sur http://IP-CASAOS:8080/setup_admin.php

Configuration requise :
- ~512 Mo RAM minimum
- ~5 Go disque (selon uploads)

Fonctionnalités incluses :
- Fail2Ban intégré (protection anti-bruteforce)
- Auto-migration de la base de données
- Volumes persistants pour uploads et logs
- Cloudflare Ready : Détection automatique des IP réelles

## Networking & Reverse Proxy

MILITANT est optimisé pour tourner derrière un reverse proxy (Cloudflare, Pangolin, Nginx, Traefik...).

### Real IP (Apache mod_remoteip)
Le système est configuré pour extraire l'IP réelle de vos visiteurs. Cela permet à Fail2Ban de bannir les attaquants et non vos serveurs internes.

Configuration par défaut : X-Forwarded-For (compatible avec Pangolin, Nginx, Traefik)

Pour Cloudflare : Modifier fail2ban/remoteip.conf :
```apache
RemoteIPHeader CF-Connecting-IP
```

Pour Pangolin/Nginx/Traefik (par défaut) :
```apache
RemoteIPHeader X-Forwarded-For
```

### Appliquer la configuration RemoteIP (sans redémarrer)

Étape 1 : Créer le fichier de configuration
```bash
docker exec <nom_conteneur> bash -c 'echo "RemoteIPHeader X-Forwarded-For
RemoteIPTrustedProxy 10.0.0.0/8
RemoteIPTrustedProxy 172.16.0.0/12
RemoteIPTrustedProxy 192.168.0.0/16" > /etc/apache2/conf-available/remoteip.conf'
```

Étape 2 : Activer le module et la configuration
```bash
docker exec <nom_conteneur> bash -c 'a2enmod remoteip && a2enconf remoteip && apache2ctl graceful'
```

Commande complète (tout en une fois) :
```bash
docker exec <nom_conteneur> bash -c 'echo "RemoteIPHeader X-Forwarded-For
RemoteIPTrustedProxy 10.0.0.0/8
RemoteIPTrustedProxy 172.16.0.0/12
RemoteIPTrustedProxy 192.168.0.0/16" > /etc/apache2/conf-available/remoteip.conf && a2enmod remoteip && a2enconf remoteip && apache2ctl graceful'
```

Vérifier que ça fonctionne :
```bash
# Vérifier que le module est chargé
docker exec <nom_conteneur> apache2ctl -M | grep remoteip

# Créer un fichier de test
docker exec <nom_conteneur> bash -c 'echo "<?php echo \"IP: \" . \$_SERVER[\"REMOTE_ADDR\"]; ?>" > /var/www/html/test-ip.php'

# Accéder à https://votre-domaine.com/test-ip.php
# Vous devriez voir votre IP publique (pas 192.168.x.x)
```

Important : 
- Remplacez <nom_conteneur> par le nom de votre conteneur (ex: militant-app-militant-app-1)
- Trouvez le nom avec : docker ps | grep militant
- Cette modification est temporaire. Pour la rendre permanente, utilisez le Dockerfile ou sauvegardez avec docker commit

Proxies de confiance : Les réseaux Docker internes (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16) sont automatiquement approuvés.

## Protection Fail2Ban

Fail2Ban est activé par défaut sur CasaOS pour protéger contre les attaques par force brute.

Voir aussi DEPLOY.md pour les instructions de déploiement et mise à jour.

### Vérifier le statut

```bash
# Voir si Fail2Ban tourne
docker ps | grep fail2ban

# Statut des jails
docker exec militant-fail2ban fail2ban-client status

# Détails de la jail (IPs bannies, tentatives)
docker exec militant-fail2ban fail2ban-client status militant-auth
```

### Activer Fail2Ban (si non activé)

Si docker ps | grep fail2ban ne retourne rien, il faut l'ajouter au docker-compose :

1. Créer les fichiers de configuration :
```bash
sudo mkdir -p /DATA/AppData/militant/fail2ban/filter.d

# Créer jail.local
sudo tee /DATA/AppData/militant/fail2ban/jail.local << 'EOF'
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 5
ignoreip = 127.0.0.1/8

[militant-auth]
enabled = true
filter = militant-auth
logpath = /var/log/militant/security.log
maxretry = 5
bantime = 3600
findtime = 300
EOF

# Créer le filtre
sudo tee /DATA/AppData/militant/fail2ban/filter.d/militant-auth.conf << 'EOF'
[Definition]
failregex = ^\[.*\] FAILED_LOGIN .* IP:<HOST>
ignoreregex =
EOF
```

2. Ajouter le service au docker-compose :
```bash
sudo nano /var/lib/casaos/apps/militant/docker-compose.yml
```

Ajouter ce bloc dans la section services: :
```yaml
  fail2ban:
    container_name: militant-fail2ban
    image: crazymax/fail2ban:latest
    volumes:
      - /DATA/AppData/militant/fail2ban/jail.local:/etc/fail2ban/jail.local:ro
      - /DATA/AppData/militant/fail2ban/filter.d:/etc/fail2ban/filter.d:ro
      - /DATA/AppData/militant/logs:/var/log/militant:ro
    cap_add:
      - NET_ADMIN
      - NET_RAW
    network_mode: host
    restart: unless-stopped
```

3. Démarrer Fail2Ban :
```bash
cd /var/lib/casaos/apps/militant && sudo docker compose up -d fail2ban
```

### Configuration par défaut

| Paramètre | Valeur |
|-----------|--------|
| Tentatives max | 5 |
| Durée du ban | 1 heure |
| Fenêtre de détection | 5 minutes |

### Débannir une IP

```bash
docker exec militant-fail2ban fail2ban-client set militant-auth unbanip 192.168.1.100
```

### Logs de sécurité

Les tentatives suspectes sont enregistrées dans :
```
/DATA/AppData/militant/logs/security.log
```

Consultables aussi depuis l'interface admin : /update.php?tab=logs

## Backup automatique vers Docker Hub

### Configuration du backup automatique

1. Créer le script de backup :
```bash
sudo tee /usr/local/bin/militant-backup.sh << 'EOF'
#!/bin/bash
DATE=$(date +%Y%m%d)

# Utiliser les credentials Docker de l'utilisateur
export DOCKER_CONFIG=/home/anar/.docker

docker commit militant-app liberchat/militant:backup-$DATE
docker push liberchat/militant:backup-$DATE
docker tag liberchat/militant:backup-$DATE liberchat/militant:latest-backup
docker push liberchat/militant:latest-backup
EOF

sudo chmod +x /usr/local/bin/militant-backup.sh
```

2. Configurer le cron (backup hebdomadaire le dimanche à 3h) :
```bash
(crontab -l 2>/dev/null; echo "0 3 * * 0 /usr/local/bin/militant-backup.sh >> /var/log/militant-backup.log 2>&1") | crontab -
```

3. Vérifier la configuration :
```bash
crontab -l
```

### Lancer un backup manuel

```bash
sudo /usr/local/bin/militant-backup.sh
```

### Restaurer depuis un backup

```bash
# Voir les backups disponibles
docker search liberchat/militant

# Restaurer un backup spécifique
sudo sed -i 's|liberchat/militant:[^"]*|liberchat/militant:backup-20260111|' /var/lib/casaos/apps/militant/docker-compose.yml
cd /var/lib/casaos/apps/militant && sudo docker compose pull && sudo docker compose up -d
```

## Fonctionnalités

### Sécurité & Compte
- Authentification double facteur (2FA) via TOTP (Google Authenticator, Authy...)
- Passkeys (WebAuthn) : Connexion biométrique (empreinte, visage)
- Mots de passe hachés avec Argon2ID
- Protection CSRF, XSS, et Rate Limiting stricts

### Publications
- Posts avec photos/vidéos (jusqu'à 500MB)
- Rich Link Previews (aperçus automatiques des liens)
- Réactions emoji
- Commentaires imbriqués avec réponses
- @mentions avec notifications

### Social
- Groupes de messagerie style Messenger avec partage média (images, vidéos, audio)
- Messages vocaux en direct avec enregistrement
- Messages éphémères avec auto-suppression configurable (1 min à 7 jours)
- Badges militants (Antifa, CNT-AIT, FA, UCL, OCL, CGA, Anarchiste, MILITANT, SLM, FLL)
- Messages privés avec chiffrement de base
- Stories éphémères (24h)
- Fil "Pour toi" / "Abonnements"
- Système d'amis et followers
- Recherche globale unifiée

### Groupes & Collectifs
- Gouvernance horizontale : Tous les membres sont admins (gestion horizontale)
- Groupes de messagerie : Conversations de groupe avec demandes d'adhésion
- Groupes privés : Questionnaire d'entrée, validation par admins
- Pages pour organisations/collectifs
- Pages privées : Contenu réservé aux abonnés
- Gestion d'équipe (admins, éditeurs)
- Gestion des abonnés (voir/retirer)
- Événements publics et privés

### API REST Complète
- 27 catégories d'endpoints
- Authentification Bearer token (30 jours)
- Rate limiting par endpoint
- SDKs Python et JavaScript
- Panneau admin auto-hébergé
- Protection complète contre les attaques (SQL injection, XSS, brute force, DoS)
- Headers de sécurité (HSTS, CSP, X-Frame-Options)
- Logging de sécurité complet
- **Déploiement séparé requis** : L'API doit être installée sur son propre conteneur
- Dépôt séparé : https://gitlab.com/militant1/militant-api
- Wiki complet : /api/WIKI.md
- Documentation : /api/README.md
- Sécurité : /api/SECURITY.md

## Outils de Développement

L'environnement local (docker-compose.yml) inclut :

### Emails (Mailpit)
En dev, les emails sont interceptés par Mailpit pour éviter le spam.
Accédez à l'interface : http://localhost:8025

### Base de données & Migrations
Le système gère automatiquement le schéma :
- Installation fraîche : Utilise database/schema_full.sql.
- Mise à jour : Utilise cli-migrate.php au démarrage.

### Testeur SMTP
Pour diagnostiquer les problèmes d'envoi d'emails (ex: OVH, Gmail) :
- Accédez à : http://votre-url/mail-test.php (réservé aux admins).
- Cet outil affiche le log détaillé de la session SMTP pour identifier précisément où le blocage se situe (connexion, TLS ou authentification).

## Configuration Complète (.env)

| Variable | Description | Défaut (Dev) |
|----------|-------------|--------------|
| DB_HOST | Hôte MySQL | db |
| DB_NAME | Nom de la base | militant |
| DB_USER | Utilisateur MySQL | militant |
| DB_PASS | Mot de passe MySQL | militant123 |
| SMTP_HOST | Serveur SMTP | mailpit |
| SMTP_PORT | Port SMTP | 1025 |
| APP_URL | URL publique | http://localhost:9000 |
| HCAPTCHA_SITE_KEY | Clé site hCaptcha | - |
| HCAPTCHA_SECRET_KEY | Clé secrète hCaptcha | - |

## Mise à jour

### Mise à jour à chaud (Recommandé - Sans redémarrage)

MILITANT supporte les mises à jour à chaud via GitLab sans aucun redémarrage.

**Méthode 1 : Via l'interface Admin**
1. Connectez-vous en tant qu'admin
2. Allez dans **Admin** → **Mises à jour**
3. Cliquez sur **"Vérifier les mises à jour"**
4. Si disponible, cliquez sur **"Mettre à jour"**
5. Rechargez la page après 10 secondes

**Méthode 2 : Via script**
```bash
bash hot_update.sh
```

**Avantages :**
- Zéro downtime (aucune interruption)
- Pas de redémarrage du conteneur
- Mise à jour en 10 secondes
- Connexions utilisateurs maintenues
- WebSocket non interrompu

**Comment ça marche :**
1. Récupère les dernières modifications depuis GitLab
2. Copie les fichiers dans le conteneur en cours
3. Exécute les migrations de base de données
4. Recharge Apache sans interruption (graceful restart)

### Configuration GitLab (Optionnel)

Pour les dépôts privés ou éviter les limites de taux, ajoutez dans `.env` :

```env
GITLAB_PROJECT=militant1/millitant
GITLAB_TOKEN=votre_token_gitlab
```

**Obtenir un token GitLab :**
1. GitLab.com → Settings → Access Tokens
2. Créer un token avec le scope `read_repository`
3. Ajouter dans `.env` : `GITLAB_TOKEN=glpat-xxxxx`

### Mise à jour manuelle (Ancienne méthode)

Si vous préférez la méthode traditionnelle avec redémarrage :

```bash
docker compose pull
docker compose up -d
```

**Note :** Cette méthode cause 30-60 secondes de downtime.

## Maintenance manuelle (Avancé)

Si vous avez besoin d'intervenir manuellement à l'intérieur du conteneur.

### Accéder au terminal du conteneur
```bash
docker exec -it militant-app bash
```

### Exécuter les migrations manuellement
```bash
docker exec -it militant-app php /var/www/html/cli-migrate.php
```

Note : Privilégiez toujours ./hot_update.sh pour les mises à jour régulières.

## Maintenance & FAQ

### Voir les logs en direct
```bash
# Logs Apache (Accès et erreurs)
docker logs -f militant-app

# Logs de sécurité (Fail2Ban/Bad logins)
tail -f ./logs/security.log
```

### Réparer les permissions
Si vous avez des erreurs d'upload :
```bash
sudo ./hot_update.sh
```

### Pourquoi mon IP est 192.168.x.x ?
Si vous voyez des IP internes dans vos paramètres, déconnectez-vous et reconnectez-vous. Le système enregistre l'IP au moment du login. Toute nouvelle connexion après la mise à jour affichera votre IP publique.

## Modération communautaire

Pas de hiérarchie. Décisions par consensus.

- Signalements traités par vote communautaire
- 70% de consensus requis
- Modérateurs élus et révocables par tous

## Langues

Disponible en : Français, English, Español, Esperanto

Traduction automatique des posts et messages intégrée.

Ajouter une langue : créer lang/xx.php

## PWA

Application installable sur mobile et desktop. Fonctionne hors ligne.

## Application Mobile (Flutter)

Une application mobile native est disponible pour Android et iOS.

### Fonctionnalités Mobiles
- **Support Multi-Serveurs** : L'utilisateur peut se connecter à n'importe quelle instance auto-hébergée simplement en entrant l'URL.
- **Notifications Push Dynamiques** : Support natif de OneSignal. L'application récupère automatiquement la configuration du serveur (App ID) au démarrage.
- **Trilingue** : Français, Anglais, Espagnol.
- **Confidentialité** : Respect total des paramètres de compte privé et permissions de messages.

### Installation
1.  Téléchargez l'APK ou installez via F-Droid (bientôt).
2.  Entrez l'URL de votre instance (ex: `https://militant.revlibertaire.com`).
3.  Connectez-vous avec vos identifiants habituels.

## Page de Status (jointomilitant)

Une page de status publique est disponible pour surveiller l'état des services MILITANT en temps réel.

### Accès
- **Local (Docker)** : http://localhost:9001
- **Production** : Déployez sur votre domaine (ex: status.militant.com)

### Fonctionnalités
- Surveillance en temps réel des services (Web + API)
- Historique des incidents sur 7 jours
- Barres d'uptime visuelles (40 derniers jours)
- Interface multilingue (FR, EN, ES, EO)
- Panneau admin pour gérer les incidents

### Déploiement Docker

```bash
cd jointomilitant
docker-compose up -d
```

Le site sera accessible sur http://localhost:9001

### Configuration Admin

**Mot de passe par défaut** : `StatusAdmin2024`

**Changer le mot de passe** :
1. Créer un fichier `.env` dans `jointomilitant/` :
```env
STATUS_ADMIN_PASSWORD=VotreNouveauMotDePasse
```

2. Rebuild le container :
```bash
cd jointomilitant
docker-compose down
docker-compose up -d --build
```

### Accès Admin

- **URL** : http://localhost:9001/status-admin.php
- **Fonctionnalités** :
  - Ajouter des incidents
  - Supprimer des incidents
  - Gérer l'historique
  - Données stockées dans `incidents.json`

### Endpoints Health Check

Le système vérifie automatiquement :
- **Web Platform** : https://militant.revlibertaire.com/health-public.php
- **API v1** : https://api.militant.revlibertaire.com/health-public.php

**Format de réponse** :
```json
{
  "status": "ok",
  "timestamp": 1771204891
}
```

**Statuts possibles** :
- `ok` : Service opérationnel
- `degraded` : Service dégradé
- `partial_outage` : Panne partielle

### Personnalisation

Modifier les URLs des services dans les fichiers `status.html` (4 langues) :
```javascript
const services = [
    { id: 'web', name: 'Plateforme Web', url: 'https://votre-domaine.com/health-public.php' },
    { id: 'api', name: 'API v1', url: 'https://api.votre-domaine.com/health-public.php' }
];
```

## Liens

- GitLab Militant : https://gitlab.com/militant1/millitant
- GitLab API : https://gitlab.com/militant1/militant-api
- Wiki API : /api/WIKI.md
- Documentation API : /api/README.md
- Sécurité API : /api/SECURITY.md
- Guide CasaOS API : /api/CASAOS.md
- Guide Docker API : /api/DOCKER.md
- Guide rapide API : /api/DEMARRAGE_RAPIDE.md
- Index API : /api/v1/index.php
- Page Status : /jointomilitant/

## API REST

L'API est dans `/api/` et **doit être déployée séparément** sur son propre conteneur.

**Dépôt séparé :** https://gitlab.com/militant1/militant-api

**Configuration requise :** Créer `/api/.env` avec les credentials de votre instance Militant

```env
API_DB_HOST=militant-db-host
API_DB_NAME=militant
API_DB_USER=militant
API_DB_PASS=votre_mot_de_passe
```

**Sécurité :**
- Protection anti brute-force (10 tentatives/15min)
- Rate limiting (60 req/min par IP)
- Protection SQL injection, XSS, path traversal
- Headers de sécurité (HSTS, CSP, X-Frame-Options)
- Tokens SHA-256 sécurisés
- Logging de tous les événements de sécurité
- Validation stricte de toutes les entrées

### Tester l'API

```bash
curl https://votre-api.com/api/v1/index.php
```

### Se connecter

```bash
curl -X POST https://votre-api.com/api/v1/auth.php \
  -H "Content-Type: application/json" \
  -d '{"username": "votre_username", "password": "votre_password"}'
```

### Déployer l'API avec Docker

```bash
cd api/
cp .env.example .env
nano .env  # Configurer : API_DB_HOST, API_DB_USER, API_DB_PASS
docker-compose up -d
```

L'API sera accessible sur http://localhost:8081

**CasaOS** : Voir /api/CASAOS.md pour l'installation sur CasaOS

Voir : /api/DOCKER.md pour toutes les options.

**Documentation complète :** /api/README.md 
