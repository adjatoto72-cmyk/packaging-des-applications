# packaging-des-applications
Déploiement d’application avec PowerShell Remoting (Azure)
# 🧪 Lab – Déploiement d’application avec PowerShell Remoting (Azure)

## 🎯 Objectif

Mettre en place un environnement de test dans Azure pour simuler le déploiement d’une application Windows sur un poste distant via PowerShell Remoting (WinRM).

---

## 🧱 Architecture

* 2 machines virtuelles Windows 10 (22H2) sur Microsoft Azure :

  * **PACKAGER** : machine d’administration
  * **CLIENT** : machine cible (poste utilisateur)

* Les deux VM sont :

  * dans le même Virtual Network
  * accessibles en RDP(3389)

---

## ⚙️ Étapes réalisées

### 1. Préparation de l’environnement

* Création de deux VM Azure
* Configuration réseau (VNet, NSG)
* Test de connectivité (ping)
<img width="1285" height="765" alt="Capture d&#39;écran 2026-04-13 203614" src="https://github.com/user-attachments/assets/f9dbf6cd-b45e-4d3f-a7ae-b0202ff71247" />
Le ping se fait après avoir demarrer les deux machines virtuelles et il s'agit de pping les adresses Privées
Assurez-vous de bien pinger l'adresse IP privée et non l'IP publique.

Pinger de 10.1.0.4 vers 10.1.0.5 (et inversement).
<img width="1203" height="793" alt="Capture d&#39;écran 2026-04-13 201756" src="https://github.com/user-attachments/assets/a398b685-8fe1-4b03-9d26-f95d58336bb0" />
Le ping entre IP publiques sur Azure échoue presque toujours sans une configuration spécifique de Load Balancer ou de NAT.
<img width="1479" height="800" alt="Capture d&#39;écran 2026-04-13 204626" src="https://github.com/user-attachments/assets/f79890ca-25ec-452e-83a1-bda69003029a" />





### 2. Configuration des rôles

* Renommage des machines (PACKAGER / CLIENT)
Sur VM 1 :
Rename-Computer -NewName "PACKAGER" -Restart

* Sur VM 2 :
Rename-Computer -NewName "CLIENT" -Restart

Par défaut, Windows bloque les requêtes ICMP (le protocole utilisé par le ping). Même si Azure autorise le passage, la machine de destination rejette le paquet elle-même.

La solution : Sur les deux VM, ouvrez un terminal PowerShell en tant qu'administrateur et exécutez cette commande :

PowerShell
New-NetFirewallRule -DisplayName "Allow ICMPv4-In" -Protocol ICMPv4
Ou, plus simplement pour tester, désactivez temporairement le pare-feu public/privé dans le Panneau de Configuration.
* Vérification des comptes utilisateurs locaux




### 3. Activation de PowerShell Remoting

* Activation de WinRM sur les deux machines :

```powershell
Enable-PSRemoting -Force
```

* Configuration des TrustedHosts (en environnement hors domaine)

### 4. Connexion à distance

* Utilisation de :

```powershell
Enter-PSSession -ComputerName CLIENT -Credential CLIENT\username
```
pour connaitre le username il faut en mode powershell taoper la commande""WHOAMI"
<img width="1333" height="783" alt="Capture d&#39;écran 2026-04-13 205612" src="https://github.com/user-attachments/assets/263b3b56-e41e-4f70-a8d8-0926ea19f4ab" />



### 5. Préparation du package

* Téléchargement de Google Chrome Enterprise (format MSI)
* Test d’installation silencieuse :

```powershell
msiexec /i chrome.msi /qn
```

### 6. Transfert du fichier

* Montage d’un lecteur réseau avec credentials :
Le fichier telechargé se met dans le C://Temp
💡 Si le dossier n’existe pas :

```powershell
New-Item -ItemType Directory -Path C:\Temp
```
```powershell
New-PSDrive -Name Z -PSProvider FileSystem -Root "\\CLIENT\C$" -Credential (Get-Credential)
```

* Copie du fichier MSI vers la machine CLIENT
<img width="939" height="149" alt="Capture d&#39;écran 2026-04-13 211125" src="https://github.com/user-attachments/assets/94ebd911-fb4e-499d-ba06-c8ce5752091f" />

### 7. Déploiement à distance

```powershell
Invoke-Command -ComputerName CLIENT -ScriptBlock {
    Start-Process msiexec.exe -ArgumentList "/i C:\Temp\chrome.msi /qn" -Wait
} -Credential CLIENT\username
```
<img width="940" height="724" alt="Capture d&#39;écran 2026-04-13 213808" src="https://github.com/user-attachments/assets/3c05c294-2134-404f-8c43-abd2fc728ae4" />


### 8. Vérification

```powershell
Test-Path "C:\Program Files\Google\Chrome\Application\chrome.exe"
```

---

## ✅ Résultat
🎯 Résultat

👉 Si tu vois :
```powershell
  True
```
<img width="1520" height="780" alt="Capture d&#39;écran 2026-04-13 221044" src="https://github.com/user-attachments/assets/bd53111f-c3ed-4ab2-896b-47672218c759" />

Chrome est installé à distance = SUCCÈS TOTAL

* Application installée à distance sans interaction utilisateur
* Déploiement automatisé validé
* Communication entre machines fonctionnelle

---

## 🧠 Compétences mises en pratique

* PowerShell Remoting (WinRM)
* Administration système Windows
* Déploiement applicatif
* Gestion réseau Azure (VNet, NSG)
* Installation silencieuse (.msi)

---

## 🚀 Améliorations possibles

* Intégration avec Microsoft Intune
* Utilisation de PSAppDeployToolkit
* Ajout de scripts de détection et de désinstallation

---

## 📌 Conclusion

Ce lab permet de simuler un scénario réel de déploiement d’application en entreprise et constitue une base solide pour évoluer vers des solutions de gestion centralisée comme Intune.
