# 📘 Guide Complet : Gérer un Raspberry Pi

## 📑 Sommaire

1. [Préparer le matériel](#-1-préparer-le-matériel)
2. [Installer Raspberry Pi OS](#-2-installer-raspberry-pi-os)
3. [Connexion à distance (SSH)](#-3-connexion-à-distance-ssh)
    - [3.1 Comprendre l’adressage IP local](#-31-comprendre-ladressage-ip-local)
    - [3.2 Utiliser un nom de domaine dynamique (DuckDNS)](#-32-utiliser-un-nom-de-domaine-dynamique-duckdns)
4. [Installer Docker](#-4-installer-docker)
5. [Installer Docker Compose](#-5-installer-docker-compose)
6. [Organiser les services](#-6-organiser-les-services)
7. [Mises à jour et maintenance](#-7-mises-à-jour-et-maintenance)
8. [Sécuriser le Raspberry Pi](#-8-sécuriser-le-raspberry-pi)
9. [Sauvegarder les données](#-9-sauvegarder-les-données)
10. [Surveiller le système](#-10-surveiller-le-système)
11. [Bonnes pratiques](#-11-bonnes-pratiques)
12. [Ressources utiles](#-12-ressources-utiles)

---

## 🧰 1. Préparer le matériel

Matériel nécessaire :
- Un Raspberry Pi (modèle 3, 4 ou 5 recommandé)
- Une carte microSD ≥ 16 Go (classe 10 minimum)
- Une alimentation compatible
- Un clavier, une souris et un écran (ou accès SSH)
- Une connexion Internet (Wi-Fi ou Ethernet)

---

## 💽 2. Installer Raspberry Pi OS

Télécharge le Raspberry Pi Imager :  
➡️ [https://www.raspberrypi.com/software/](https://www.raspberrypi.com/software/)

### Étapes :

1. Choisir l’exécutable adapté à ton système d’exploitation :
   ![Choix de l'OS](./img/rpi_custom_etape1.png)

2. Configurer les paramètres avant l’écriture (nom d’utilisateur, mot de passe, Wi-Fi, SSH) :
   ![Configuration utilisateur](./img/rpi_custom_jp.jpg)

3. Insérer la carte SD dans le PC **avant** de lancer le flashage :
   ![Flash de la carte](./img/rpi_cistom_lastpart.png)

4. Insérer la carte SD dans le Raspberry Pi et démarrer.

---

## 🔐 3. Connexion à distance (SSH)

Accéder à ton Raspberry Pi à distance est une étape clé. Voici tout ce qu’il faut comprendre et mettre en place pour une connexion stable, sécurisée et fiable.

---

### 🔍 3.1 Comprendre l’adressage IP local

- Lorsqu’il démarre, ton Raspberry Pi reçoit une **adresse IP locale temporaire** (ex. `192.168.1.54`) via **DHCP**.
- Cette IP peut **changer régulièrement** (souvent toutes les 24h), ce qui rend les connexions instables.
- Pour un usage régulier, tu peux :
  - Soit configurer une **IP statique** (voir section 4),
  - Soit utiliser un **nom de domaine dynamique**.

---

### 🌐 3.2 Utiliser un nom de domaine dynamique avec DuckDNS

Pour garantir un accès stable à ton Raspberry Pi depuis l’extérieur, même si ton IP publique change, DuckDNS est une solution gratuite et simple à mettre en place.

---

#### 🦆 Étape 1 – Créer un nom de domaine gratuit sur [DuckDNS.org](https://www.duckdns.org)

![duck dns](./img/Capture%20d’écran%202025-06-18%20182206.png)

Une fois inscrit, tu obtiendras :
- un **nom de domaine** du type `monpi.duckdns.org`
- un **token personnel** pour authentifier les mises à jour

---

#### 🛠️ Étape 2 – Créer un script de mise à jour automatique

Crée un fichier `duck.sh` dans `~/duckdns/` :

```bash
#!/bin/bash

# Paramètres
LOG_FILE=~/duckdns/duck.log
EMAIL="ton.email@exemple.com"
TOKEN="TON_TOKEN"
DOMAIN="ton-sous-domaine"

echo url="https://www.duckdns.org/update?domains=$DOMAIN&token=$TOKEN&ip=" | curl -k -o $LOG_FILE -K -

# Vérification de succès
if ! grep -q "OK" "$LOG_FILE"; then
    echo "Échec de mise à jour DuckDNS pour $DOMAIN à $(date)" | mail -s "[ALERTE] Échec DuckDNS" $EMAIL
fi
```

---

#### 🧠 Explication pédagogique du script

| Ligne | Rôle |
|-------|------|
| `#!/bin/bash` | Indique que le fichier est un script Bash. |
| `LOG_FILE=~/duckdns/duck.log` | Fichier où sera stockée la réponse de DuckDNS. |
| `EMAIL="..."` | Adresse email pour envoyer une alerte en cas d’échec. |
| `TOKEN="..."` | Token sécurisé fourni par DuckDNS. |
| `DOMAIN="..."` | Ton sous-domaine DuckDNS. |
| `echo url=...` | Envoie une requête HTTPS à DuckDNS pour mettre à jour l’IP publique. |
| `-o $LOG_FILE` | Enregistre la réponse dans un fichier log. |
| `-K -` | Utilise l’entrée standard comme source des options `curl`. |
| `grep -q "OK"` | Vérifie que la mise à jour a réussi. |
| `mail -s ...` | Envoie un email si la mise à jour échoue. |

> ℹ️ Laisser `ip=` vide permet à DuckDNS de détecter automatiquement l'IP publique.

---

#### 📅 Étape 3 – Automatiser la mise à jour toutes les 5 minutes

Rends ton script exécutable :

```bash
chmod +x ~/duckdns/duck.sh
```

Ajoute-le à la crontab :

```bash
crontab -e
```

Ajoute la ligne suivante :

```cron
*/5 * * * * ~/duckdns/duck.sh >/dev/null 2>&1
```

> ⏱️ Cette ligne exécute le script toutes les 5 minutes.  
> `>/dev/null 2>&1` supprime les messages de sortie pour garder les logs propres.

## 🐳 4. Installer Docker

```bash
curl -sSL https://get.docker.com | sh
sudo usermod -aG docker pi
```

Redémarrer ensuite :

```bash
sudo reboot
```

Tester :

```bash
docker run hello-world
```

---

## ⚙️ 5. Installer Docker Compose

```bash
sudo apt install docker-compose -y
```

---

## 📂 6. Organiser les services

Structure conseillée :

```bash
mkdir -p ~/services/mon-service
cd ~/services/mon-service
nano docker-compose.yml
```

---

## 🔄 7. Mises à jour et maintenance

```bash
sudo apt update && sudo apt full-upgrade -y
sudo apt autoremove -y
sudo apt clean
```

---

## 🔐 8. Sécuriser le Raspberry Pi

Changer mot de passe :

```bash
passwd
```

Installer fail2ban :

```bash
sudo apt install fail2ban -y
```

Pare-feu (UFW) :

```bash
sudo apt install ufw -y
sudo ufw allow ssh
sudo ufw enable
```

---

## 💾 9. Sauvegarder les données

### Avec `rsync` :

```bash
rsync -avz /home/pi user@ip_serveur:/sauvegarde/raspi/
```

### Image de la carte SD :

Depuis un PC :

```bash
sudo dd if=/dev/sdX of=raspberry-backup.img bs=4M status=progress
```

---

## 📊 10. Surveiller le système

Vérifier température :

```bash
vcgencmd measure_temp
```

RAM :

```bash
free -h
```

Processus :

```bash
htop
```

Installer `glances` :

```bash
sudo apt install glances -y
glances
```

---

## ✅ 11. Bonnes pratiques

- Ne pas utiliser le compte root directement
- Documenter chaque projet
- Sauvegarder régulièrement
- Redémarrer de temps en temps
- Surveiller espace disque : `df -h`

---

## 🔗 12. Ressources utiles

- Documentation Raspberry Pi : [raspberrypi.com/documentation](https://www.raspberrypi.com/documentation/)
- Forum d’entraide : [forums.raspberrypi.com](https://forums.raspberrypi.com)
- Tutoriels français : [framboise314.fr](https://www.framboise314.fr/)

---
