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

### 🌐 3.2 Utiliser un nom de domaine dynamique (DuckDNS)

Pour éviter les problèmes liés au changement d’IP publique (côté box), tu peux :

1. Créer un nom de domaine gratuit avec [DuckDNS](https://www.duckdns.org)
2. Mettre en place un script de mise à jour automatique (exécuté toutes les 5 min via `cron`)
3. Rediriger les ports nécessaires dans ta box (port 22 pour SSH)

> Exemple de connexion avec DuckDNS :
```bash
ssh pi@monpi.duckdns.org
```


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
