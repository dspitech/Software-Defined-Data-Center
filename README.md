# TP - 17 : Lab global Datacenter

## Nom : Lo
## Prénom : Pape

## Introduction

Ce TP  intitulé « **Lab global Datacenter** » a pour but de regrouper et réutiliser les principales fonctionnalités étudiées dans les **TP 1 à 7**, en mettant l’accent sur la configuration automatisée via **PowerCLI**.

## Objectifs

Les objectifs de ce TP sont les suivants :

- Concevoir et mettre en place un datacenter virtualisé complet.
- Créer et configurer un cluster de virtualisation.
- Ajouter et gérer plusieurs hôtes au sein du cluster.
- Configurer et optimiser le fonctionnement du cluster.
- Déployer, cloner et administrer des machines virtuelles.
- Mettre en œuvre les fonctionnalités avancées telles que la haute disponibilité (HA), le Distributed Resource Scheduler (DRS) et la migration à chaud (vMotion).
- Assurer la sauvegarde des machines virtuelles à l’aide de backups et de snapshots.
- Mettre en place des mécanismes de protection et de sécurisation des machines virtuelles.
- Gérer les ressources de stockage, y compris les datastores et les solutions de stockage distribué comme vSAN.
- Configurer et administrer le réseau virtuel (switchs virtuels, VLANs, segmentation réseau).
- Mettre en place des politiques de gestion des ressources (CPU, RAM, stockage).
- Superviser l’infrastructure afin de garantir la performance et la disponibilité des services.
- Tester les scénarios de tolérance aux pannes et de reprise après incident.
- Optimiser les performances globales du datacenter.

---

## Déploiement 

### 1. Création d'un datacenter (PowerCLI)

- Lancer PowerCLI
![image](https://hackmd.io/_uploads/r1QLvIi2bl.png)
- Modifier la stratégie d'exécution PowerShell
```PowerShell
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser
```
![image](https://hackmd.io/_uploads/rJoqO8onZl.png)
- Importer les modules
```PowerShell
Import-Module -Name VMware.VimAutomation.Storage.psd1
Import-Module -Name VMware.VimAutomation.Storage.psm1
Import-Module -Name "C:\Labfiles\HOL-2530\VMware.VMEncryption.psd1"
Import-Module -Name "C:\Labfiles\HOL-2530\VMware.VMEncryption.psm1"
```
![image](https://hackmd.io/_uploads/ryxoK8i2Zg.png)
- Se connecter à vSphere
```PowerShell
Connect-VIServer -server vcsa-01a.vcf.sddc.lab -user administrator@vsphere.local -password VMware123!
```
![image](https://hackmd.io/_uploads/HkOptIshZg.png)
![image](https://hackmd.io/_uploads/ryky5Ii3bg.png)
- Voir la location > Emplacement
```PowerShell
Get-Folder
```
![image](https://hackmd.io/_uploads/SJBXoIonbx.png)

- Création du Datacenter nommé `ESTIAM-Data-01`
```PowerShell
New-Datacenter -Name "ESTIAM-Data-01" -Location (Get-Folder -Name "Datacenters")
```
![image](https://hackmd.io/_uploads/rkzIjUsh-e.png)
- Voir le résultat
```PowerShell
Get-Datacenter -Name "ESTIAM-Data-01"
```
![image](https://hackmd.io/_uploads/H1k2iIj3-x.png)
![image](https://hackmd.io/_uploads/ryK6jIo3bx.png)
- Voir plus de détails
```PowerShell
Get-Datacenter -Name "ESTIAM-Data-01" | Format-List *
```
![image](https://hackmd.io/_uploads/BJKN28inZl.png)

---

### 2. Création d'un cluster 
Dans cette partie on va créer 3 clusters qui sont :` ESTIAM-Paris`, `ESTIAM-Lyon` & `ESTIAM-Geneve` et ne pas activer le `HA `et `DRS`.
- Cluster ESTIAM-Paris
```PowerShell
New-Cluster -Name "ESTIAM-Paris" -Location (Get-Datacenter "ESTIAM-Data-01")
```
![image](https://hackmd.io/_uploads/ryGhT8ihZe.png)
- Cluster ESTIAM-Lyon
```PowerShell
New-Cluster -Name "ESTIAM-Lyon" -Location (Get-Datacenter "ESTIAM-Data-01")
```
![image](https://hackmd.io/_uploads/BJWR6Ijn-g.png)
- Cluster ESTIAM-Geneve
```PowerShell
New-Cluster -Name "ESTIAM-Geneve" -Location (Get-Datacenter "ESTIAM-Data-01")
```
![image](https://hackmd.io/_uploads/BkA1A8ihZx.png)
- Vérification des clusters
```PowerShell
Get-Datacenter "ESTIAM-Data-01" | Get-Cluster
```
![image](https://hackmd.io/_uploads/ByGICUj3We.png)
![image](https://hackmd.io/_uploads/ryatA8o2-l.png)

---

### 3. Ajout ESXi au cluster
Dans cette partie; on ajoute un hôte au cluster `ESTIAM-Paris`.
- Ajout ESXi au cluster `ESTIAM-Paris`
Host 1 : `esx-05a.vcf.sddc.lab`
```PowerShell
Add-VMHost -Name "esx-05a.vcf.sddc.lab" -Location "ESTIAM-Paris" -User "root" -Password "VMware123!"
```
![image](https://hackmd.io/_uploads/rk-_ZvshWl.png)

- Vérification 
![image](https://hackmd.io/_uploads/SJ1LHDs3We.png)

---
### 4. Configuration du cluster
Maintenant que le cluster a été créé et que les hôtes ESXi ont été ajoutés, on passe à l’étape finale du processus.
Dans cette phase, on utilise vCenter QuickStart afin de finaliser la configuration du cluster de manière simplifiée et automatisée.

- Vérification du NTP configuré au niveau de l'hôte
```PowerShell
Get-VMHost "esx-05a.vcf.sddc.lab" | Get-VMHostNtpServer
```
![image](https://hackmd.io/_uploads/Bk60ZOsh-l.png)
- Vérification RDMA (réseau avancé)
```PowerShell
Get-VMHost "esx-05a.vcf.sddc.lab" | Get-VMHostNetworkAdapter | Select Name, RDMAEnabled
```
![image](https://hackmd.io/_uploads/SyPrGus2We.png)

- Vérifier Lockdown Mode

**Sert à sécuriser l’accès à l’ESXi**
* Disabled → accès normal (admin direct possible)
* Normal/Strict → accès limité via vCenter

#### Pourquoi c’est important ?
- sécurité du datacenter
- éviter accès direct non autorisé
- standard en production
```PowerShell
Get-VMHost "esx-05a.vcf.sddc.lab" | Select Name, LockdownMode, ConnectionState, PowerState
```
![image](https://hackmd.io/_uploads/B1Sgmuo3We.png)

| Vérification  | Rôle               | Impact              |
| ------------- | ------------------ | ------------------- |
| Lockdown Mode | Sécurité           | contrôle accès ESXi |
| NTP           | Temps              | stabilité cluster   |
| EVC           | CPU compatibilité  | vMotion possible    |
| RDMA          | Performance réseau | stockage rapide     |

---

### 5. Création d’une Image (Cluster Image)

Maintenant que notre cluster a été configuré, nous allons passer à l’étape de création d’une **image de cluster**.

Lors de l’étape précédente, cette configuration avait été volontairement ignorée afin de s’y attarder plus en détail ici.

L’objectif est de créer et d’attacher une **image de cluster** afin de :

- Garantir une configuration **homogène** sur tous les hôtes ESXi
- Simplifier la gestion et les mises à jour
- Réduire les écarts de configuration (*drift*)

Une **Cluster Image** est un modèle standard qui définit :

- La version d’**ESXi**
- Les **drivers** (pilotes)
- Le **firmware** (si intégré)
- Les **add-ons fournisseurs**

Tous les hôtes du cluster doivent se conformer à cette image.

#### 5.1 Configuration
- Créer le cluster nommé `ESTIAM-Paris-SDDC`
![image](https://hackmd.io/_uploads/BkkeodohZe.png)
![image](https://hackmd.io/_uploads/SkQWjOj3-x.png)
![image](https://hackmd.io/_uploads/rkSXj_j2Zl.png)
- Désactiver HA / DRS / vSAN
![image](https://hackmd.io/_uploads/ryIBsOi3Wx.png)
- Assurez-vous que l'option « `Gérer tous les hôtes du cluster avec une seule image` » est cochée.
![image](https://hackmd.io/_uploads/B1Rtiui2bx.png)
- Cocher « `Importer une image à partir d'un hôte existant dans l'inventaire vCenter` »
![image](https://hackmd.io/_uploads/ry6noOs3bl.png)
- Sélectionner l'image concernée
![image](https://hackmd.io/_uploads/rkIl2_i3-x.png)
![image](https://hackmd.io/_uploads/S1GWndihZx.png)
- Récapitulatif
![image](https://hackmd.io/_uploads/Hkazn_ohbl.png)
- Vérification
![image](https://hackmd.io/_uploads/Hk_3h_ihWg.png)
![image](https://hackmd.io/_uploads/Syjanui2-x.png)
![image](https://hackmd.io/_uploads/ryMyTdsnbx.png)

---

### 6. vSphere Cluster Services (vCLS)
Les vCLS assurent la disponibilité et la continuité des services du cluster, même en cas de défaillance partielle de l’infrastructure.

Les vSphere Cluster Services (vCLS) sont activés par défaut dans tous les clusters vSphere.
Ils assurent que les services du cluster restent disponibles même si le vCenter Server est indisponible, garantissant la santé et la disponibilité des workloads.
vCenter reste nécessaire pour configurer et exécuter DRS et HA.

#### 6.1 vCLS VM
Une **vCLS VM** (vSphere Cluster Services Virtual Machine) est une petite machine virtuelle système déployée automatiquement dans chaque cluster **vSphere**.  
Sa fonction principale est de garantir que les **services du cluster**—comme **DRS (Distributed Resource Scheduler)** et **HA (High Availability)**—restent opérationnels même si le **vCenter Server** devient indisponible.

#### Points clés :

- Les vCLS VMs sont **créées automatiquement** lors de la création d’un cluster.  
- Elles ne sont pas destinées à exécuter des workloads utilisateurs, mais gèrent les services du cluster.  
- Chaque cluster possède généralement **2 ou 3 vCLS VMs**, selon la version de vSphere et le niveau de redondance souhaité.  
- Elles permettent à **DRS de fonctionner** sans vCenter, assurant l’équilibrage des ressources et la santé du cluster.  
- Depuis **vSphere 8.0 U3**, les vCLS VMs sont **intégrées (Embedded)** et utilisent la technologie **vSphere Pods (PodCRX)** au lieu de VM complètes avec Photon OS.

#### 6.1.1 Configuration
- Voir les VMs VCLS du cluster
```PowerShell
Get-Cluster "RegionA01-COMP01" | Get-VM | Where-Object {$_.Name -like "vCLS*"} | Select Name, PowerState, VMHost
```
![image](https://hackmd.io/_uploads/rkXJgYsnbg.png)
![image](https://hackmd.io/_uploads/SJsxeYo2Zg.png)
![image](https://hackmd.io/_uploads/rko4lFon-e.png)

#### 6.2 Datastore vCLS

Le **datastore vCLS** est l’emplacement de stockage où résident les vCLS VMs.

#### Points clés :

- Les vCLS VMs nécessitent un **datastore accessible par tous les hôtes** du cluster.  
- Il est recommandé d’utiliser un **stockage partagé** : vSAN, NFS ou VMFS.  
- Évitez les datastores locaux, car si l’hôte tombe en panne, la vCLS VM ne pourra pas fonctionner ailleurs.  
- Un placement correct du datastore garantit la **disponibilité, la résilience et la performance** des services du cluster.

#### 6.2.1 Configuration
- Voir les vCLS Datastore
```PowerShell
Get-Cluster "RegionA01-COMP01" | Get-VM |
Where-Object {$_.Name -like "vCLS*"} |
Select Name, VMHost, @{N="Datastore";E={$_.ExtensionData.LayoutEx.File[0].Name}}
```
![image](https://hackmd.io/_uploads/ByYTlFo3bl.png)
#### 6.2.1.1 Ajouter un nouveau Datastore pour vCLS
![image](https://hackmd.io/_uploads/rkOhQKo2bl.png)
![image](https://hackmd.io/_uploads/BycTmti3Wg.png)
![image](https://hackmd.io/_uploads/Sk60mKj3Zl.png)

---

### 7. Création d'une machine virtuelle
#### OBJECTIF
Créer une VM :
- Nom : `Photon-VM01`
- Cluster : `RegionA01-COMP01`
- Datastore : `RegionA01-ISCSI01-COMP01`
- Réseau : `VM-RegionA01-vDS-COMP`
- OS : `Linux (Photon OS)`
- Sans template (VM vide + ISO ensuite)

#### 7.1 Configuration
- Variables
```PowerShell
$cluster = Get-Cluster "RegionA01-COMP01"
$ds = Get-Datastore "RegionA01-ISCSI01-COMP01"
$network = Get-VDPortgroup -Name "VM-RegionA01-vDS-COMP"
```
![image](https://hackmd.io/_uploads/Hkcz_Fi2Zl.png)

- Création de la VM
```PowerShell
New-VM -Name "Photon-VM01" `
       -ResourcePool $cluster `
       -Datastore $ds `
       -NumCPU 2 `
       -MemoryGB 4 `
       -NetworkName $network.Name `
       -GuestId "other3xLinux64Guest"
```
![image](https://hackmd.io/_uploads/HJeGKFi2Ze.png)
![image](https://hackmd.io/_uploads/rJgUFti2-e.png)

- Ajouter le lecteur CD/DVD à la VM
```PowerShell
Get-VM "Photon-VM01" | New-CDDrive
```
![image](https://hackmd.io/_uploads/B1E55Fo3Ze.png)
![image](https://hackmd.io/_uploads/HJjo5Yonbl.png)
- Voir tous les périphériques 
Cette commande liste tout ce qui est attaché à la VM (contrôleurs, RAM, CPU, disques, lecteurs CD) 
```PowerShell
(Get-VM "Photon-VM01").ExtensionData.Config.Hardware.Device | Select-Object @{N="DeviceType";E={$_.GetType().Name}}, Key, UnitNumber
```
![image](https://hackmd.io/_uploads/rJHBoYsnbg.png)
- Affiche les infos essentielles : CPU, RAM, réseau, IP, hôte ESXi et état de l'alimentation.
```PowerShell
Get-VM "Photon-VM01" | Select-Object Name, PowerState, NumCpu, MemoryGB, VMHost, UsedSpaceGB, ProvisionedSpaceGB | Format-List
```
![image](https://hackmd.io/_uploads/SkRCoYjhWg.png)
- Voir TOUTES les propriétés
```PowerShell
Get-VM "Photon-VM01" | Format-List *
```
![image](https://hackmd.io/_uploads/BkmH3Fon-e.png)
![image](https://hackmd.io/_uploads/Syx83ts3bl.png)
- Créer un dossier sur le datastore, puis y transférer l'ISO.

```PowerShell
$localPath = "C:\labfiles\HOL-2530\ISO\tiny_micro.iso"
$ds = Get-Datastore "RegionA01-ISCSI01-COMP01"

## Créer le dossier nommé "estiam" et copier l'ISO
Copy-DatastoreItem -Item $localPath -Destination "$($ds.DatastoreBrowserPath)\estiam\" -Force
```
![image](https://hackmd.io/_uploads/SJHeW5i3Wx.png)

- Vérifier que l'ISO est bien dans le nouveau dossier
```PowerShell
Get-ChildItem -Path "$($ds.DatastoreBrowserPath)\estiam\"
```
![image](https://hackmd.io/_uploads/HkMNb9s2bg.png)
![image](https://hackmd.io/_uploads/B1icb9j2bx.png)
![image](https://hackmd.io/_uploads/r1shb9shWe.png)
- Monter l'ISO sur votre VM "**Photon-VM01**"
Maintenant que l'ISO est rangé dans le dossier estiam du datastore, on peut l'attacher à la VM.
```PowerShell
$cd = Get-VM "Photon-VM01" | Get-CDDrive
Set-CDDrive -CD $cd -IsoPath "[RegionA01-ISCSI01-COMP01] estiam/tiny_micro.iso" -StartConnected $true -Confirm:$false
```
![image](https://hackmd.io/_uploads/B14_McjhZx.png)
- vérification pour être sûr que la VM est prête à booter correctement
```PowerShell
Get-VM "Photon-VM01" | Get-CDDrive | Select-Object Name, IsoPath, ConnectionState, StartConnected
```
![image](https://hackmd.io/_uploads/r1yJX5o2bl.png)
- Démarrer la VM
```PowerShell
Start-VM -VM "Photon-VM01"
```
![image](https://hackmd.io/_uploads/HkCNQ5onWl.png)
- Déploiement terminé
![image](https://hackmd.io/_uploads/ryxKXqjnbl.png)
- Eteindre la VM
```PowerShell
Stop-VM -VM "Photon-VM01" -Kill -Confirm:$false
```
![image](https://hackmd.io/_uploads/SyDGEcshWl.png)
![image](https://hackmd.io/_uploads/HyYXNco2bx.png)

---

### 8. Assigner un Tag à une VM dans vSphere
- Créer une Catégorie
```PowerShell
New-TagCategory -Name "VM-SDDC" -Description "Tags pour les ressources ESTIAM" -Cardinality Single
```
![image](https://hackmd.io/_uploads/rJPkD9j3Zg.png)

**Note** : `-Cardinality Single` signifie qu'une VM ne peut avoir qu'un seul tag de cette catégorie à la fois.
- Créer le Tag
```PowerShell
# Création du tag spécifique
New-Tag -Name "ESTIAM-Paris-VMs" -Category "VM-SDDC"
```
![image](https://hackmd.io/_uploads/ByS-v9o2Wg.png)

- Assigner le Tag à la VM
```PowerShell
# On récupère la VM et le Tag, puis on les lie
$vm = Get-VM "Photon-VM01"
$tag = Get-Tag -Name "ESTIAM-Paris-VMs"
New-TagAssignment -Entity $vm -Tag $tag
```
![image](https://hackmd.io/_uploads/B1yND5snZl.png)

- Vérifier l'assignation
```PowerShell
Get-TagAssignment -Entity "Photon-VM01"
```
![image](https://hackmd.io/_uploads/H1_Sv9jhbl.png)
![image](https://hackmd.io/_uploads/ByvPwcjnWg.png)

Une fois les VM taguées, vous peut faire des recherches ultra-rapides. Par exemple, pour lister toutes les VM qui font partie du `ESTIAM-Paris-VMs`, peu importe leur nom.
```PowerShell
Get-VM -Tag "ESTIAM-Paris-VMs"
```
![image](https://hackmd.io/_uploads/BJiyO5i3We.png)

---

### 9. Ajouter une deuxième carte réseau à une machine virtuelle existante
- voir la carte réseau de la VM
```PowerShell
Get-VM "Photon-VM01" | Get-NetworkAdapter | Select-Object Name, NetworkName, Type, MacAddress, ConnectionState
```
![image](https://hackmd.io/_uploads/HyYkF5sh-x.png)
![image](https://hackmd.io/_uploads/Hk7XY9snZl.png)

- Pour savoir sur quel réseau vous pouvez brancher la deuxième carte
![image](https://hackmd.io/_uploads/BJwBF9o3bl.png)
- Ajout de la 2ème carte réseau
On va utiliser le réseau ESXi-RegionA01-vDS-COMP pour cet exemple (puisqu'il est disponible dans ta liste).
```PowerShell
# On définit la VM et le réseau cible
$vm = Get-VM "Photon-VM01"
$targetNetwork = "ESXi-RegionA01-vDS-COMP"

# On ajoute la nouvelle carte de type Vmxnet3
New-NetworkAdapter -VM $vm -NetworkName $targetNetwork -Type "Vmxnet3" -StartConnected:$true
```
![image](https://hackmd.io/_uploads/B1L-qqj2Wl.png)
![image](https://hackmd.io/_uploads/BJRtcqi3Wg.png)

---

### 10. Déconnecter l’ISO de la machine virtuelle
#### Pourquoi ?
- Si l’ISO reste attaché, à chaque redémarrage, la VM peut tenter de booter à nouveau depuis le CD/DVD virtuel, ce qui pourrait relancer l’installation.
- Déconnecter l’ISO permet à la VM de démarrer directement sur le disque système.
- Une ISO montée consomme un peu de mémoire et d’E/S sur le datastore.
- La déconnecter allège la charge sur le stockage et le réseau.
- Certaines opérations, comme les snapshots ou les migrations vMotion, peuvent générer des avertissements si un CD/DVD virtuel est encore connecté.
- Déconnecter l’ISO évite les messages d’erreur ou les interruptions.
![image](https://hackmd.io/_uploads/H1rBs9ohbl.png)
```PowerShell
Get-VM "Photon-VM01" | Get-CDDrive | Set-CDDrive -NoMedia -StartConnected $false -Confirm:$false
```
![image](https://hackmd.io/_uploads/H1J3jqihbl.png)
![image](https://hackmd.io/_uploads/rk7Cicj2Zg.png)
- Vérification
```powerShell
Get-VM "Photon-VM01" | Get-CDDrive | Select-Object Name, IsoPath
```
![image](https://hackmd.io/_uploads/Hy1zn9s3be.png)

---

### 12. Clonage - Template
#### 12.1 Clonage
Une machine virtuelle existante peut servir de base pour créer d’autres machines virtuelles. Cloner une VM permet de gagner beaucoup de temps lorsqu’il s’agit de déployer plusieurs machines similaires. Plutôt que de créer et configurer chaque VM individuellement, vous pouvez configurer une seule machine virtuelle, installer les logiciels nécessaires, puis la cloner autant de fois que nécessaire.

Le clonage crée une copie identique de ta VM `Photon-VM01` vers une nouvelle VM appelée `Photon-VM02`.
- Création
```PowerShell
# On définit la source et la destination
$vmSource = Get-VM "Photon-VM01"
$vmDestName = "Photon-VM02"
$datastore = Get-Datastore "RegionA01-ISCSI01-COMP01"
$vmHost = Get-VMHost "esx-01a.vcf.sddc.lab" 

# Lancement du clonage
New-VM -Name $vmDestName -VM $vmSource -Datastore $datastore -VMHost $vmHost
```
![image](https://hackmd.io/_uploads/r1-f6csh-e.png)
![image](https://hackmd.io/_uploads/HkLXTci3Zl.png)
#### 12.2 Template
- On transforme la VM en modèle (Template)
```PowerShell
Get-VM "Photon-VM01" | Set-VM -ToTemplate -Confirm:$false
```
![image](https://hackmd.io/_uploads/BJrJ05jhWe.png)
![image](https://hackmd.io/_uploads/BJ1-Rcjn-l.png)
Une fois cette commande lancée, la VM disparaît de la liste `Get-VM`. C'est normal, elle change de statut.
- Vérifier que le Template existe
```PowerShell
Get-Template -Name "Photon-VM01"
```
![image](https://hackmd.io/_uploads/BkKHAqsnbg.png)
- Déployer une nouvelle VM à partir de ce Template
```PowerShell
# 1. On prépare les cibles
$template = Get-Template -Name "Photon-VM01"
$vmHost = Get-VMHost "esx-01a.vcf.sddc.lab"
$datastore = Get-Datastore "RegionA01-ISCSI01-COMP01"

# 2. On déploie
New-VM -Name "Photon-Clone" -Template $template -VMHost $vmHost -Datastore $datastore
```
![image](https://hackmd.io/_uploads/BkK905ohWg.png)
![image](https://hackmd.io/_uploads/BkSZ1si2We.png)

#### Différence entre Clone et Template

Il est important de bien comprendre la nuance pour ton lab :

| Caractéristique | Clone (VM vers VM) | Template (VM vers Modèle) |
|----------------|-------------------|---------------------------|
| État final | Une deuxième VM prête à démarrer | Un modèle (template) non démarrable |
| Utilisation | Copie ponctuelle d’une VM existante | Déploiements massifs et standardisés |
| Modification | Modifiable à tout moment | Nécessite reconversion en VM pour modification |

- Démarrer la VM
```PowerShell
Get-VM "Photon-Clone" | Start-VM
```
![image](https://hackmd.io/_uploads/ry38yoj3Zl.png)
![image](https://hackmd.io/_uploads/SJV_JiinZl.png)

---

### 13. Haute Disponibilité vSphere - HA

La HA vSphere fournit une protection pour les machines virtuelles en regroupant celles-ci et leurs hôtes au sein d’un cluster.
- Les hôtes sont constamment surveillés.
- En cas de panne d’un hôte, ses machines virtuelles sont redémarrées automatiquement sur les autres hôtes disponibles.

Lors de la création d’un cluster vSphere HA :
- Un hôte principal est automatiquement élu.
- Cet hôte communique avec vCenter Server et surveille l’état des machines virtuelles protégées et des hôtes secondaires.
- Il doit détecter différents types de pannes et faire la distinction entre un hôte réellement défaillant et un hôte isolé du réseau.
- Pour ce faire, l’hôte principal utilise les pulsations réseau et les signaux de la banque de données afin de déterminer le type de panne.

**NB** : vSphere HA fonctionne au niveau de l’hôte, ce qui signifie qu’aucune dépendance à vCenter n’est nécessaire pour que le basculement des machines virtuelles vers d’autres hôtes s’effectue correctement.

#### 13.1 Configuration
- Identifier le Cluster concerné
```PowerShell
Get-Cluster
```
![image](https://hackmd.io/_uploads/S19O7is2Ze.png)
- Vérifier si HA est activé
![image](https://hackmd.io/_uploads/Bku3msjnbl.png)
Dans cette capture on remarque HA n'est pas activé.
- Activer vSphere HA
```PowerShell
$clusterName = "RegionA01-COMP01" # Remplace par ton nom de cluster
Set-Cluster -Cluster $clusterName -HAEnabled $true -Confirm:$false
```
![image](https://hackmd.io/_uploads/HyAWVjs2Wg.png)
![image](https://hackmd.io/_uploads/HkrN4iinZl.png)
- Configurer les options avancées (Admission Control)
L'Admission Control est crucial : il réserve de la RAM et du CPU pour s'assurer que si un hôte tombe, les autres ont assez de place pour accueillir les VM rescapées.
```PowerShell
# On configure pour tolérer la panne de 1 hôte (Host failure tolerance)
Set-Cluster -Cluster $clusterName -HAAdmissionControlEnabled $true -HAFailoverLevel 1
```
![image](https://hackmd.io/_uploads/BJETNjin-e.png)
![image](https://hackmd.io/_uploads/HJ71Sjsnbg.png)
- Configurer la réponse à l'isolement (Isolation Response)
Si un hôte est vivant mais ne capte plus le réseau, que doit-il faire de ses VM ? Généralement, on choisit de les éteindre pour qu'elles redémarrent sur un hôte qui a du réseau.
![image](https://hackmd.io/_uploads/B1oQBji3We.png)
```PowerShell
Set-Cluster -Cluster $clusterName -HAIsolationResponse PowerOff
```
![image](https://hackmd.io/_uploads/HJ7uSjinbe.png)
![image](https://hackmd.io/_uploads/BJjKHishZx.png)
- Vérifier l'état de santé du HA
Une fois configuré, vSphere va installer des agents `(FDM - Fault Domain Manager)` sur chaque ESXi. On peut vérifier si tout est "Green".
```PowerShell
Get-Cluster $clusterName | Select-Object Name, HAEnabled, HAFailoverLevel, HAAdmissionControlEnabled
```
![image](https://hackmd.io/_uploads/ByyMLojh-x.png)
- Activer "VM Monitoring" : "Guest not heartbeating"
Si on l'active, vSphere HA va surveiller les VMware Tools à l'intérieur de la VM. Si Windows ou Linux plante `(Blue Screen / Kernel Panic)` et que les VMware Tools ne répondent plus, HA va redémarrer la VM automatiquement, même si le serveur ESXi, lui, fonctionne très bien.
![image](https://hackmd.io/_uploads/HJfxDsoh-e.png)
Pour que cela fonctionne, il faut impérativement que les VMware Tools soient installés et en cours d'exécution dans la VM.

```PowerShell
# On crée l'objet de configuration de base
$spec = New-Object VMware.Vim.ClusterConfigSpec
$spec.DasConfig = New-Object VMware.Vim.ClusterDasConfigInfo

# On définit le monitoring sur VM uniquement
$spec.DasConfig.VmMonitoring = "vmMonitoringOnly"

# On applique la configuration au cluster
(Get-Cluster "RegionA01-COMP01").ExtensionData.ReconfigureCluster($spec, $true)
```
![image](https://hackmd.io/_uploads/HyMNdss3-g.png)
![image](https://hackmd.io/_uploads/BJwS_soh-l.png)
![image](https://hackmd.io/_uploads/S1wK_iihWx.png)

- Vérification finale du HA
```PowerShell
$cluster = Get-Cluster "RegionA01-COMP01"
$das = $cluster.ExtensionData.Configuration.DasConfig

[PSCustomObject]@{
    "HA_Active"            = $cluster.HAEnabled
    "Monitoring_VM"        = $das.VmMonitoring
    "Admission_Control"    = $cluster.HAAdmissionControlEnabled
    "Failover_Level"       = $cluster.HAFailoverLevel
    "Isolation_Response"   = $das.DefaultVmSettings.IsolationResponse
    "Datastore_Heartbeat"  = $das.Option | Where-Object {$_.Key -eq "das.heartbeatDsPerHost"} | Select -ExpandProperty Value
} | Format-List
```
![image](https://hackmd.io/_uploads/BkhzKjj3Zl.png)

- Correction erreur `The number of vSphere HA heartbeat datastores for this host is 1, which is less than required: 2`
C'est un avertissement classique ! vSphere HA préfère avoir deux banques de données (datastores) pour les pulsations (heartbeats). Cela permet de s'assurer que si un datastore tombe en panne, HA peut toujours vérifier si un hôte est "vivant" via le deuxième.
- Ignorer l'avertissement
```PowerShell
$cluster = Get-Cluster "RegionA01-COMP01"
New-AdvancedSetting -Entity $cluster -Name "das.ignoreInsufficientHbDatastore" -Value "true" -Type "ClusterHA" -Confirm:$false -Force:$true
```
![image](https://hackmd.io/_uploads/BkEZoiin-l.png)
- Forcer le cluster à lire sa nouvelle configuration.
```PowerShell
(Get-Cluster "RegionA01-COMP01").ExtensionData.ReconfigureCluster((New-Object VMware.Vim.ClusterConfigSpec), $true)
```
![image](https://hackmd.io/_uploads/rkT4osshWl.png)

---

### 14. DRS
- Activer le DRS en mode "Fully Automated"
```PowerShell
$clusterName = "RegionA01-COMP01"
Set-Cluster -Cluster $clusterName -DrsEnabled $true -DrsAutomationLevel "FullyAutomated" -Confirm:$false
```
![image](https://hackmd.io/_uploads/BywWJnsn-g.png)
![image](https://hackmd.io/_uploads/S1G412i2We.png)
- Ajuster le seuil de migration `(Migration Threshold)`
Le curseur de `"Migration Threshold"` (de 1 à 5) définit si le DRS est agressif ou conservateur :
    - 1 (Conservateur) : Ne déplace les VM que pour respecter les règles ou sortir d'un hôte en maintenance.
    - 3 (Milieu) : C'est le réglage standard (recommandé).
    - 5 (Agressif) : Déplace les VM au moindre petit déséquilibre de charge.
![image](https://hackmd.io/_uploads/rkUT12onbg.png)

- Vérification finale du DRS
```PowerShell
$cluster = Get-Cluster "RegionA01-COMP01"
$cluster.ExtensionData.Configuration.DrsConfig | Select-Object Enabled, DefaultVmBehavior, VmotionRate
```
![image](https://hackmd.io/_uploads/rJbD-ni2We.png)

---

### 15. Surveillance des événements et création d'alarmes
#### 15.1 Alarme 1 : CPU Hôte > 80% >  Mode Maintenance
Lorsqu'un hôte atteint ou dépasse 80 % d'utilisation du processeur pendant plus de 5 minutes, une alarme d'avertissement est déclenchée et l'hôte passe en mode maintenance.

##### 15.1.1 Configuration
![image](https://hackmd.io/_uploads/HJTIw3onZl.png)
![image](https://hackmd.io/_uploads/HJK0D2jh-x.png)
![image](https://hackmd.io/_uploads/ryAetninZx.png)
![image](https://hackmd.io/_uploads/SkwMYhi3Ze.png)
![image](https://hackmd.io/_uploads/H18XKns3-g.png)
![image](https://hackmd.io/_uploads/SJIuKnj3Wl.png)
![image](https://hackmd.io/_uploads/SJNFFns3Wg.png)


#### 15.2 Alarme 2 : CPU Ready > 8000ms > Migration VM
On crée une alarme qui migrera une machine virtuelle si le temps d'attente du processeur dépasse une moyenne de 8000 ms sur une période de 5 minutes.

![image](https://hackmd.io/_uploads/rkoot2i3Wg.png)
![image](https://hackmd.io/_uploads/SyM153o2-e.png)
![image](https://hackmd.io/_uploads/rkyYqhihWe.png)
![image](https://hackmd.io/_uploads/SyqF9njn-x.png)
![image](https://hackmd.io/_uploads/HkH5q3i3Zg.png)
![image](https://hackmd.io/_uploads/ryooq2onbl.png)
![image](https://hackmd.io/_uploads/By3252jn-g.png)

#### 15.3 Vérification des deux alarmes
- vérification
```PowerShell
Get-AlarmDefinition | Where-Object {$_.Name -like "ALM*"} | Select-Object Name, Description, Enabled
```
![image](https://hackmd.io/_uploads/rJcdohj3-g.png)
- voir si ce sont des Hôtes ou des VM et vérifier le seuil de 80% (8000)
```PowerShell
Get-AlarmDefinition | Where-Object {$_.Name -like "ALM*"} | Select-Object Name, `
    @{N="TargetType"; E={$_.ExtensionData.Info.Expression.Expression[0].Type}}, `
    @{N="YellowThreshold"; E={$_.ExtensionData.Info.Expression.Expression[0].Yellow}} | Format-Table -AutoSize
```
![image](https://hackmd.io/_uploads/rygG0s2s3Zl.png)

---

### 16. Les partages et les ressources
Les partages définissent l'importance relative d'une machine virtuelle (ou d'un pool de ressources). Si une machine virtuelle possède deux fois plus de partages d'une ressource qu'une autre, elle est autorisée à consommer deux fois plus de cette ressource lorsque ces deux machines virtuelles sont en concurrence pour l'accès aux ressources. 

Les partages sont généralement définis comme étant de priorité élevée, normale ou faible.

Les limites empêchent une machine virtuelle d'utiliser plus de ressources que la limite définie. Les réservations garantissent une quantité minimale de ressources disponible pour la machine virtuelle.

- Vérification détaillée (CPU & Mémoire) pour core-A
```PowerShell
Get-VM "core-A" | Select-Object Name,
    @{N="CPU_Shares_Level"; E={$_.ExtensionData.Config.CpuAllocation.Shares.Level}},
    @{N="CPU_Reservation_MHz"; E={$_.ExtensionData.Config.CpuAllocation.Reservation}},
    @{N="CPU_Limit_MHz"; E={if($_.ExtensionData.Config.CpuAllocation.Limit -eq -1){"Unlimited"}else{$_.ExtensionData.Config.CpuAllocation.Limit}}},
    @{N="Mem_Shares_Level"; E={$_.ExtensionData.Config.MemoryAllocation.Shares.Level}},
    @{N="Mem_Reservation_MB"; E={$_.ExtensionData.Config.MemoryAllocation.Reservation}},
    @{N="Mem_Limit_MB"; E={if($_.ExtensionData.Config.MemoryAllocation.Limit -eq -1){"Unlimited"}else{$_.ExtensionData.Config.MemoryAllocation.Limit}}} |
    Format-List
```
![image](https://hackmd.io/_uploads/HyZ2pnohZe.png)
![image](https://hackmd.io/_uploads/r1tUChi3-l.png)

**Shares Level (Normal)** : En cas de surcharge de l'hôte, `core-A` a la même priorité que n'importe quelle autre VM configurée par défaut. Elle recevra sa part équitable, mais ne sera pas privilégiée.

**Limit (Unlimited)** : C'est l'idéal. La VM peut consommer autant de ressources physiques que l'hôte peut lui en donner, sans plafond artificiel.

**Reservation (0 / Vide)** : Il n'y a aucune garantie matérielle. Si l'hôte manque de RAM,` core-A` pourrait voir ses performances chuter car vSphere ne lui a `"bloqué"` aucune ressource prioritaire.

- Booster `core-A`
Transformer `core-A` en VM Prioritaire. Nous allons lui donner deux fois plus de poids que les autres `(High)` et lui garantir un minimum de ressources.

```PowerShell
# On reste cohérent avec la taille de la VM (256Mo total)
Get-VM "core-A" | Get-VMResourceConfiguration | Set-VMResourceConfiguration `
    -CpuSharesLevel High `
    -MemSharesLevel High `
    -MemReservationMB 128
```
![image](https://hackmd.io/_uploads/B1MMkTj2Wl.png)
- Vérification 
```PowerShell
Get-VM "core-A" | Select-Object Name,
    @{N="CPU_Shares"; E={$_.ExtensionData.Config.CpuAllocation.Shares.Level}},
    @{N="Mem_Shares"; E={$_.ExtensionData.Config.MemoryAllocation.Shares.Level}},
    @{N="Mem_Reservation_MB"; E={$_.ExtensionData.Config.MemoryAllocation.Reservation}},
    @{N="Configured_Memory_MB"; E={$_.MemoryMB}} |
    Format-Table -AutoSize
```
![image](https://hackmd.io/_uploads/S1IYkajhZe.png)

**CPU_Shares** : Doit être passé à High.
**Mem_Shares** : Doit être passé à High.
**Mem_Reservation_MB** : Doit afficher 128.
**Configured_Memory_MB** : on verra ton 256, confirmant que la réservation de 128 est bien valide (car $128 \le 256$).

En passant en `High`, on double le poids de `core-A` par rapport aux autres VMs `"Normal"` en cas de saturation du CPU. Et avec les 128 Mo de réservation, on crée une `"bulle de sécurité"` : même si l'hôte est en panique de mémoire, il ne touchera jamais à ces `128 Mo` appartenant à `core-A`.
**La VM core-A est Prioritaire dans ce cluster.**

---

### 17. Migration de machines virtuelles avec VMware vMotion
Les interruptions de service planifiées représentent généralement plus de 80 % des indisponibilités des centres de données. La maintenance matérielle, la migration des serveurs et les mises à jour du firmware nécessitent toutes des interruptions de service pour les serveurs physiques. Afin de minimiser l'impact de ces interruptions, les entreprises sont contraintes de reporter la maintenance jusqu'à des périodes d'indisponibilité difficiles à planifier et peu pratiques.

La fonctionnalité vMotion de vSphere permet aux entreprises de réduire les interruptions de service planifiées, car les charges de travail dans un environnement VMware peuvent être déplacées dynamiquement vers différents serveurs physiques sans interruption de service. Les administrateurs peuvent ainsi effectuer des opérations de maintenance plus rapides et totalement transparentes, sans être obligés de planifier des fenêtres de maintenance inopportunes. Avec vSphere vMotion, les entreprises peuvent :

- Éliminer les interruptions de service liées aux opérations de maintenance courantes.
- Éliminer les fenêtres de maintenance planifiées.
- Effectuer la maintenance à tout moment sans perturber les utilisateurs ni les services. Une autre fonctionnalité de vSphere, Storage vMotion, permet de migrer une machine virtuelle vers différents périphériques de stockage sans interruption de service.

#### 17.1 - Configuration
Migrer toutes les machines virtuelles hébergées sur `esx-01a.vcf.sddc.lab` vers `esx-02a.vcf.sddc.lab`.

- Désactiver DRS
Pour éviter que le DRS ne tente de replacer les machines ailleurs pendant qu'on les déplaces manuellement, on le passe en mode Manuel (ou on le désactive).
![image](https://hackmd.io/_uploads/HJfUGasnbl.png)
```PowerShell
Set-Cluster -Cluster "RegionA01-COMP01" -DrsEnabled $false -Confirm:$false
```
![image](https://hackmd.io/_uploads/SJfyQ6shZg.png)
![image](https://hackmd.io/_uploads/HySlmTihWl.png)
- Inventaire des VMs sur l'hôte source
```PowerShell
$source = "esx-01a.vcf.sddc.lab"
Get-VMHost $source | Get-VM | Select-Object Name, PowerState, NumCpu, MemoryGB | Format-Table -AutoSize
```
![image](https://hackmd.io/_uploads/Sk61NpihZg.png)

- Lancement de la migration (vMotion)
```PowerShell
$vms = Get-VM -Location "esx-01a.vcf.sddc.lab"
$dest = Get-VMHost "esx-02a.vcf.sddc.lab"

write-host "Début de la migration de $($vms.Count) machines virtuelles..." -ForegroundColor Cyan

$vms | Move-VM -Destination $dest -Confirm:$false
```
![image](https://hackmd.io/_uploads/SJHjVao2bl.png)

- Vérification finale
Une fois que PowerCLI a fini de te rendre la main, on vérifie que l'hôte source est bien vide.
```PowerShell
Get-VMHost "esx-01a.vcf.sddc.lab" | Select-Object Name, @{N="VM_Count"; E={(Get-VM -Location $_).Count}}
```
![image](https://hackmd.io/_uploads/SkV-Bai3bx.png)
![image](https://hackmd.io/_uploads/BkJQSTo3bg.png)

- Vérifier le contenu de l'hôte de destination
```PowerShell
Get-VMHost "esx-02a.vcf.sddc.lab" | Get-VM | Select-Object Name, PowerState, NumCpu, MemoryGB | Format-Table -AutoSize
```
![image](https://hackmd.io/_uploads/B1R5S6snWl.png)
![image](https://hackmd.io/_uploads/H1raS6o3bg.png)

- Voir où se trouve la VM core-A actuellement
```PowerShell
Get-VM "core-A" | Select-Object Name, VMHost, PowerState
```
![image](https://hackmd.io/_uploads/r1fv8Tsn-l.png)

---

### 18. Surveillance et performances de vSphere - Monitoring
VMware propose plusieurs outils pour vous aider à surveiller votre environnement virtuel et à identifier la source des problèmes potentiels et actuels.
![image](https://hackmd.io/_uploads/SkS8DainWg.png)

- Voir les derniers événements globaux
```PowerShell
Get-VIEvent -MaxSamples 20 | Select-Object CreatedTime, UserName, FullFormattedMessage | Format-Table -AutoSize
```
![image](https://hackmd.io/_uploads/ByD9waj2Ze.png)

- Monitoring spécifique à un Hôte ou une VM
```PowerShell
Get-VIEvent -Entity "esx-02a.vcf.sddc.lab" -MaxSamples 20 | Select-Object CreatedTime, FullFormattedMessage
```
![image](https://hackmd.io/_uploads/r1uzd6jhZg.png)
- Surveillance des performances en temps réel
```PowerShell
Get-Stat -Entity "core-A" -Stat "cpu.usage.average" -Realtime -MaxSamples 5 | Select-Object Timestamp, Value, Unit
```
![image](https://hackmd.io/_uploads/Hkr8uaj3Ze.png)
- Exporter les logs
```PowerShell
Get-VIEvent -MaxSamples 500 | Export-Csv -Path "C:\Users\Administrator\Desktop\vCenter_Logs.csv" -NoTypeInformation
```
![image](https://hackmd.io/_uploads/BkenuTs2Ze.png)
![image](https://hackmd.io/_uploads/SyYyKpj3Wx.png)
![image](https://hackmd.io/_uploads/Sy5-K6i2Zx.png)

---

### 19. Compute Policy dans vSphere
Une Compute Policy permet de définir des règles et des contraintes pour les machines virtuelles ou les clusters afin de garantir que les ressources (CPU, mémoire, stockage, etc.) sont utilisées conformément aux bonnes pratiques et aux besoins de l’entreprise.
![image](https://hackmd.io/_uploads/H1nscajhZe.png)

L'idée est : on met un "badge" (Tag) sur la VM, un autre sur l'Hôte, et on crée une politique qui dit "Les VMs avec ce badge doivent aller sur les Hôtes avec ce badge".

- Créer les Catégories et les Tags

```PowerShell
# 1. Créer les catégories 
New-TagCategory -Name "AppType" -Description "Type d'application"
New-TagCategory -Name "HostGroup" -Description "Groupement d'hôtes"

# 2. Créer les Tags
$tagVM = New-Tag -Name "Core-App" -Category "AppType"
$tagHost = New-Tag -Name "Production-Hosts" -Category "HostGroup"
```
![image](https://hackmd.io/_uploads/Bkn3sponbe.png)

- Assigner les Tags
On va marquer la VM `core-A` et l'hôte `esx-02a`.
```PowerShell
# 1. On récupère d'abord l'objet Tag et l'objet VM pour être sûr
$tag = Get-Tag -Name "Core-App"
$vm = Get-VM "core-A"

# 2. On crée l'assignation (le lien entre les deux)
New-TagAssignment -Tag $tag -Entity $vm
```
![image](https://hackmd.io/_uploads/B1zQ2Toh-e.png)

- vérifier que le tag est bien mis ?
```PowerShell
Get-TagAssignment -Entity "core-A" | Select-Object Tag, Entity
```
![image](https://hackmd.io/_uploads/B19LhTshWx.png)
Ensuite créer la règle d'affinité pour forcer core-A à rester sur un groupe d'hôtes spécifique.
- Crée les groupes
**Groupe 1 : type** `VM Group`
![image](https://hackmd.io/_uploads/SyeExCihZg.png)
![image](https://hackmd.io/_uploads/BJgwxAs3-x.png)
![image](https://hackmd.io/_uploads/Synug0j2-l.png)
![image](https://hackmd.io/_uploads/H19tgRi2Wx.png)
![image](https://hackmd.io/_uploads/S1ToeCsnbe.png)

**Groupe 1 : type** `Host Group`
![image](https://hackmd.io/_uploads/B1FZb0jhWe.png)
![image](https://hackmd.io/_uploads/SJiQWRshZx.png)
![image](https://hackmd.io/_uploads/SJpVbAjhWg.png)
![image](https://hackmd.io/_uploads/B1JL-CjhWe.png)
![image](https://hackmd.io/_uploads/rJzD-CjnWl.png)

- Créer la règle
![image](https://hackmd.io/_uploads/Bk85-Aoh-l.png)
![image](https://hackmd.io/_uploads/HJu6bAohWg.png)
![image](https://hackmd.io/_uploads/ByRCWRs3Ze.png)

- Vérifier 
```PowerShell
$cluster.ExtensionData.Configuration.Rule | Select-Object Name, Enabled, Mandatory, @{N="Type"; E={$_.gettype().Name}} | Format-Table -AutoSize
```
![image](https://hackmd.io/_uploads/SJC0MCjhWe.png)

- Test
Essaie d'envoyer `core-A` vers l'hôte (`esx-02a.vcf.sddc.lab`).
```PowerShell
$vm = Get-VM "core-A"
$destinationInvalide = Get-VMHost "esx-02a.vcf.sddc.lab"

# Cette commande devrait générer une erreur ou un avertissement de compatibilité
Move-VM -VM $vm -Destination $destinationInvalide
```
![image](https://hackmd.io/_uploads/BJI270jnWl.png)
Le vCenter a fait une vérification de pré-vol avant de lancer le vMotion. Il a vu que tu essayais d'envoyer la VM sur l'hôte `02a`, alors que ta règle dit : "Doit être sur le groupe `01a"`. Il a donc stoppé l'opération pour garantir la conformité.

---

### 20. Le Desired State (état désiré) 
la configuration idéale qu'on veut pour un système.
#### On définit :
- version ESXi
- configuration réseau
- drivers
- firmware
- paramètres cluster
- et vSphere s’assure que tous les hôtes respectent exactement cet état

#### 20.1 Configuration
![image](https://hackmd.io/_uploads/By9lIAsnbx.png)
- Nouvelle configuration
![image](https://hackmd.io/_uploads/Sk5BIRjhZg.png)
![image](https://hackmd.io/_uploads/B128L0i3bx.png)
- importer l'hpote de référence
![image](https://hackmd.io/_uploads/BJ6oLCinbl.png)
![image](https://hackmd.io/_uploads/rJhhIAo3bg.png)
![image](https://hackmd.io/_uploads/Bk8680inWg.png)
![image](https://hackmd.io/_uploads/SJxA80o3be.png)
![image](https://hackmd.io/_uploads/Sy2lPAihZe.png)
![image](https://hackmd.io/_uploads/H1K-v0s2-l.png)
![image](https://hackmd.io/_uploads/B1SMP0o2-x.png)
![image](https://hackmd.io/_uploads/SJR7wRi2We.png)
![image](https://hackmd.io/_uploads/S1nNwAi2We.png)
![image](https://hackmd.io/_uploads/SydHPRi2be.png)
![image](https://hackmd.io/_uploads/SJzDvRonbx.png)
![image](https://hackmd.io/_uploads/rkRDPCi2bg.png)
![image](https://hackmd.io/_uploads/rk5uw0onWx.png)
![image](https://hackmd.io/_uploads/HkrKDRjn-l.png)

#### 20.1.1 Vérification de la conformité des hôtes
Une fois l’état souhaité appliqué, vérifiez la compliance des hôtes :
- **Compliant** : l’hôte respecte l’état souhaité
- **Non Compliant** : l’hôte dévie de l’état souhaité
Utiliser vSphere Client pour obtenir un rapport clair sur l’état des hôtes.
![image](https://hackmd.io/_uploads/Bk-wORsn-l.png)

### 21. Drift

- **Configuration Drift** : déviation ou écart de la configuration d’un hôte par rapport à l’état souhaité.  
- Causes fréquentes :
  - Mise à jour manuelle d’un hôte
  - Modification de paramètres réseau ou stockage
  - Installation de drivers non standard

#### 21.1 Configuration : Modification des paramètres NTP
Pour bien tester le Desired State changeaons l'ip NTP de l'hôte 1.
- Afficher la configuration NTP actuelle
```PowerShell
$host1 = Get-VMHost "esx-01a.vcf.sddc.lab"
Get-VMHostNtpServer -VMHost $host1
```
![image](https://hackmd.io/_uploads/Byx0FRonWe.png)
![image](https://hackmd.io/_uploads/H1RMqRsnZx.png)
- Changer l'IP NTP pour créer le Drift
```PowerShell
# Supprimer les serveurs NTP existants pour cet hôte
$oldNtp = Get-VMHostNtpServer -VMHost $host1
Remove-VMHostNtpServer -VMHost $host1 -NtpServer $oldNtp

# Ajouter l'IP erronée
Add-VMHostNtpServer -VMHost $host1 -NtpServer "192.168.1.1"

# Redémarrer le service NTP pour appliquer le changement
Get-VMHostService -VMHost $host1 | Where-Object {$_.Key -eq "ntpd"} | Restart-VMHostService -Confirm:$false
```
![image](https://hackmd.io/_uploads/ry4n50s3bx.png)
![image](https://hackmd.io/_uploads/HJpa50sn-x.png)
- Après qu’un drift se produit, les hôtes affectés seront marqués Non Compliant.
- Vérifiez les rapports pour identifier quels hôtes sont affectés et quelles modifications ont été détectées.
![image](https://hackmd.io/_uploads/S16zsAonWl.png)
![image](https://hackmd.io/_uploads/By9QoRihbg.png)
![image](https://hackmd.io/_uploads/S1grjAj2We.png)

#### Résultat

    - Desired State a détecté un hôte non conforme.
    - Sélectionnez l'hôte non conforme, `esx.01a.vcf.sddc.lab`
    - il semble que la valeur de NTP soit differente de celle attendue par nos autres hotes du cluster.
    - L'ip du NTP à été moédifé 

- Remédiation 
Après une modification du NTP sur ce ESXI, on rétablie la conformité.
![image](https://hackmd.io/_uploads/HkdrnRoh-l.png)
![image](https://hackmd.io/_uploads/r1fUn0jn-l.png)
![image](https://hackmd.io/_uploads/H1QD30i2Zg.png)
![image](https://hackmd.io/_uploads/HJ0P3Rj3bg.png)
![image](https://hackmd.io/_uploads/BJgYhRihbg.png)
![image](https://hackmd.io/_uploads/Hyoc20o3bx.png)
![image](https://hackmd.io/_uploads/Bylh30i2Zg.png)

---

### 22. Retreat Mode 

## 1. Introduction

Le **Retreat Mode** est une fonctionnalité qui permet de placer **tous les hôtes d’un cluster** en mode maintenance simultanément.  
Cela est utile pour effectuer des **opérations de maintenance à grande échelle**, comme les mises à jour, les migrations de workloads ou la vérification des configurations.
- Mode par défaut
![image](https://hackmd.io/_uploads/BJL7a0i2be.png)
![image](https://hackmd.io/_uploads/ryCEaRj3Zx.png)
- Activer le mode retreat
```PowerShell
$cluster = Get-Cluster "RegionA01-COMP01"
# On récupère l'ID interne du cluster (ex: domain-c8)
$clusterId = $cluster.ExtensionData.MoRef.Value

# On définit le nom du paramètre spécifique
$settingName = "config.vcls.clusters.$clusterId.enabled"

# On applique le mode Retreat (False = Désactivé)
Get-AdvancedSetting -Entity $global:DefaultVIServer -Name $settingName | Set-AdvancedSetting -Value $False -Confirm:$false
```
![image](https://hackmd.io/_uploads/H1l7R0j3Wg.png)
![image](https://hackmd.io/_uploads/r1IVCAi2bl.png)

- Repasser en mode `"System Managed"`

```PowerShell
$cluster = Get-Cluster "RegionA01-COMP01"
$clusterId = $cluster.ExtensionData.MoRef.Value
$settingName = "config.vcls.clusters.$clusterId.enabled"

# On repasse la valeur à True pour réactiver les services
Get-AdvancedSetting -Entity $global:DefaultVIServer -Name $settingName | Set-AdvancedSetting -Value $True -Confirm:$false

Write-Host "Le mode System Managed a été réactivé. vCenter va redéployer les VMs vCLS..." -ForegroundColor Cyan
```
![image](https://hackmd.io/_uploads/HkiUJy2nWl.png)
![image](https://hackmd.io/_uploads/HJpDy13hZl.png)

---

### 23. vSphere Standard Switch
#### 23.1 Ajouter un réseau
Cette action permet de configurer et ajouter des connexions réseau pour les hôtes ESXi ou les machines virtuelles, incluant la création de commutateurs virtuels (vSwitch) et la gestion des adaptateurs réseau.
![image](https://hackmd.io/_uploads/H1dT-J23be.png)

Nous allons créer un nouveau commutateur virtuel et y ajouter un groupe de ports (Port Group) pour le trafic des machines virtuelles.
- Créer un nouveau vSwitch (vSwitch1)
Nous allons d'abord créer un nouveau commutateur virtuel indépendant du `vSwitch0` (qui gère généralement le Management).
```PowerShell
$esxi = Get-VMHost "esx-01a.vcf.sddc.lab"

# Créer le vSwitch sans adaptateur physique pour l'instant (Internal only)
New-VirtualSwitch -VMHost $esxi -Name "vSwitch1" -NumPorts 120
```
![image](https://hackmd.io/_uploads/B1VBMy22bx.png)
![image](https://hackmd.io/_uploads/Syh8M1n2Wg.png)

- Ajouter un Port Group (Réseau VM)
Un vSwitch seul ne sert à rien sans un groupe de ports pour y connecter des objets. Créons un réseau nommé `"Production-Net"`.
```PowerShell
# Ajouter le groupe de ports au vSwitch
New-VirtualPortGroup -VirtualSwitch (Get-VirtualSwitch -VMHost $esxi -Name "vSwitch1") -Name "Production-Net" -VLanId 10
```
![image](https://hackmd.io/_uploads/H1jjGk3hZl.png)
![image](https://hackmd.io/_uploads/S10hzkn2Ze.png)
- Identifier les cartes réseau physiques libres
```PowerShell
$esxi = Get-VMHost "esx-01a.vcf.sddc.lab"

# Lister les cartes physiques et le switch auquel elles sont liées
$esxi | Get-VMHostNetworkAdapter -Physical | Select-Object Name, DeviceName, VirtualSwitch, LinkSpeedMb, Status | Format-Table -AutoSize
```
![image](https://hackmd.io/_uploads/HJoBXJ3hWx.png)
![image](https://hackmd.io/_uploads/S18dmk22Wg.png)
- Assigner une carte réseau physique` (Uplink)`
Pour que ce réseau sorte de l'hôte ESXi vers le monde physique, il faut lui assigner une carte réseau `(vmnic`).

```PowerShell
# On récupère l'adaptateur physique vmnic1
$nic = Get-VMHostNetworkAdapter -VMHost $esxi -Name "vmnic1"

# On l'ajoute au vSwitch1
Add-VirtualSwitchPhysicalNetworkAdapter -VirtualSwitch $vSwitch1 -VMHostPhysicalNic $nic -Confirm:$false
```
![image](https://hackmd.io/_uploads/rJyL41hhbe.png)
![image](https://hackmd.io/_uploads/SyBvV1n2bx.png)
- Vérification
```PowerShell
$vSwitch1 | Select-Object Name, Nic
```
![image](https://hackmd.io/_uploads/HJ3c4J2nZg.png)
**On y voit très clairement** :
- Le Standard Switch: vSwitch1 est bien actif.
- Le Port Group "Production-Net" (VLAN 10) est créé.
- L'adaptateur physique vmnic1 est bien rattaché en tant qu'Uplink avec une vitesse de 10000 Full (10 Gbps).

Puisque le  commutateur est prêt, nous allons maintenant y brancher une machine virtuelle pour valider que le trafic peut passer.

Nous allons utiliser la VM `core-A`.
- Basculer son interface réseau sur le nouveau port group
```PowerShell
# On récupère l'adaptateur réseau de la VM core-A
$vm = Get-VM "core-A"
$nic = Get-NetworkAdapter -VM $vm

# On la connecte au nouveau Port Group "Production-Net"
Set-NetworkAdapter -NetworkAdapter $nic -PortGroup "Production-Net" -Confirm:$false
```
![image](https://hackmd.io/_uploads/SJt-vJ2h-x.png)
![image](https://hackmd.io/_uploads/B1IUv13nbx.png)

- vérifie que l'adaptateur est bien `Connected`
```PowerShell
Get-NetworkAdapter -VM "core-A" | Select-Object Parent, NetworkName, ConnectionState
```
![image](https://hackmd.io/_uploads/S1rjv123bl.png)
![image](https://hackmd.io/_uploads/BJh3Pk3hZg.png)

---

### 24. vSphere Distributed Switch (vDS) 
vSphere Distributed Switch (vDS)  est l'étape logique pour centraliser ta gestion réseau. 
Contrairement au Standard Switch, le vDS se configure au niveau du Datacenter et s'applique uniformément à tous les hôtes du cluster.

#### 24.1 Configuration
- Créer le vSphere Distributed Switch
Nous allons créer un vDS nommé `RegionA01-vDS-PROD`
```PowerShell
$datacenter = Get-Datacenter "RegionA01"

# Création du switch distribué
New-VDSwitch -Name "RegionA01-vDS-PROD" -Location $datacenter -NumUplinkPorts 2
```
![image](https://hackmd.io/_uploads/HyhyYkh3-l.png)
![image](https://hackmd.io/_uploads/ryobty23bl.png)
- Créer les Port Groups avec VLANs
C'est ici que nous segmentons le trafic. Nous allons créer deux réseaux distincts sur ce même switch.
```PowerShell
$vds = Get-VDSwitch -Name "RegionA01-vDS-PROD"

# Port Group pour la Production (VLAN 20)
New-VDPortgroup -VDSwitch $vds -Name "vDS-Prod-Net" -VLanId 20

# Port Group pour le Backup (VLAN 30)
New-VDPortgroup -VDSwitch $vds -Name "vDS-Backup-Net" -VLanId 30
```
![image](https://hackmd.io/_uploads/HJODFy2hbg.png)
![image](https://hackmd.io/_uploads/SyE5Kk23Zg.png)
- Ajouter les hôtes au vDS
Le `switch` existe dans le vCenter, mais il doit être "étendu" sur les hôtes physiques pour être utilisable. Nous allons utiliser la `vmnic1` de chaque hôte (celle qui est libre).
```PowerShell
$hosts = Get-Cluster "RegionA01-COMP01" | Get-VMHost
$vds = Get-VDSwitch -Name "RegionA01-vDS-PROD"

foreach ($esxi in $hosts) {
    Write-Host "Traitement de l'hôte : $($esxi.Name)" -ForegroundColor Cyan
    
    # ÉTAPE A : Ajouter l'hôte au vDS (obligatoire avant de manipuler les nics)
    Write-Host "  -> Ajout de l'hôte au vDS..."
    Add-VDSwitchVMHost -VDSwitch $vds -VMHost $esxi -Confirm:$false
    
    # ÉTAPE B : Récupérer et lier la vmnic1
    $nic = Get-VMHostNetworkAdapter -VMHost $esxi -Name "vmnic1"
    Write-Host "  -> Liaison de la vmnic1..."
    Add-VDSwitchPhysicalNetworkAdapter -DistributedSwitch $vds -VMHostPhysicalNic $nic -Confirm:$false
}
```
![image](https://hackmd.io/_uploads/rk99q13n-x.png)
![image](https://hackmd.io/_uploads/H1y1oJ3nZx.png)
![image](https://hackmd.io/_uploads/rJwlsJnnbe.png)
![image](https://hackmd.io/_uploads/rJEbi132Wx.png)
- Vérification
```PowerShell
Get-VDSwitch -Name "RegionA01-vDS-PROD" | Get-VMHost | Select-Object Name
```
![image](https://hackmd.io/_uploads/SJ4Eiyh2-g.png)

#### 24.2 Migration du Management vers le vDS
Le but est de déplacer l'interface vmk0 (Management) du vSwitch0 vers un nouveau Port Group distribué sur notre `vDS-PROD`.
- Créer le Port Group pour le Management
D'abord, il nous faut une "cible" sur le switch distribué pour accueillir le trafic de gestion.
```PowerShell
$vds = Get-VDSwitch -Name "RegionA01-vDS-PROD"

# Créer un Port Group dédié au Management (souvent VLAN 0 ou le VLAN de gestion)
New-VDPortgroup -VDSwitch $vds -Name "vDS-MGMT-Net" -VLanId 0
```
![image](https://hackmd.io/_uploads/B1zLhynh-e.png)
- Migrer l'interface VMkernel `(vmk0)`
Nous allons dire à l'hôte : "`Prends ton IP de gestion et bascule-la sur le switch distribué".`
```PowerShell
foreach ($esxi in $hosts) {
    Write-Host "Migration du Management pour : $($esxi.Name)" -ForegroundColor Yellow
    
    # 1. Récupérer l'interface de management actuelle (vmk0)
    $vmk = Get-VMHostNetworkAdapter -VMHost $esxi -Name "vmk0"
    
    # 2. Récupérer le nouveau port group cible
    $targetPortGroup = Get-VDPortgroup -Name "vDS-MGMT-Net"
    
    # 3. Basculer l'interface
    Set-VMHostNetworkAdapter -VirtualNic $vmk -PortGroup $targetPortGroup -Confirm:$false
}
```
![image](https://hackmd.io/_uploads/ryAC2JhnWg.png)
Succès total ! les logs confirment que les trois interfaces VMkernel `(vmk0)` ont bien conservé leurs adresses IP respectives `(10.0.0.51, .52, .53)` tout en migrant vers le nouveau Port Group distribué.
![image](https://hackmd.io/_uploads/r1mIakh2-e.png)
![image](https://hackmd.io/_uploads/S1wPp1hnbg.png)
![image](https://hackmd.io/_uploads/rksdTJh3Wg.png)

- Vérification
```PowerShell
Get-VMHostNetworkAdapter -VMHost $hosts -Name "vmk0" | Select-Object VMHost, Name, PortGroupName, IP
```
![image](https://hackmd.io/_uploads/BJCsTJ2hWl.png)

#### Nettoyage : Que faire du vieux vSwitch0 ?
Maintenant que le Management est sur le vDS, le vSwitch0 (Standard) est probablement vide ou ne contient plus que des choses secondaires; une fois qu'on a migré tous les VMkernels et les VMs vers le vDS, on finit par supprimer le switch standard pour ne pas laisser de configuration "fantôme".
- Vérifions d'abord ce qu'il reste comme switches standards sur tes hôtes
```PowerShell
Get-VirtualSwitch -VMHost $hosts | Where-Object { $_.gettype().Name -eq "VirtualSwitchImpl" } | Select-Object VMHost, Name, Nic
```
![image](https://hackmd.io/_uploads/Sk1CAk2hbg.png)
- Déplacer les VMs vers le vDS 
Nous allons chercher toutes les VMs connectées à Production-Net et les basculer sur `vDS-Prod-Net`
```PowerShell
# Trouver les VMs sur l'ancien réseau et les basculer
Get-VM | Get-NetworkAdapter | Where-Object {$_.NetworkName -eq "Production-Net"} | Set-NetworkAdapter -PortGroup (Get-VDPortgroup -Name "vDS-Prod-Net") -Confirm:$false
```
![image](https://hackmd.io/_uploads/BJH2yx23We.png)
![image](https://hackmd.io/_uploads/S1spkx32bx.png)
- Supprimer le Port Group et le vSwitch 
```PowerShell
# Suppression du Port Group
Get-VirtualPortGroup -VMHost "esx-01a.vcf.sddc.lab" -Name "Production-Net" | Remove-VirtualPortGroup -Confirm:$false

# Suppression du vSwitch1
Get-VirtualSwitch -VMHost "esx-01a.vcf.sddc.lab" -Name "vSwitch1" | Remove-VirtualSwitch -Confirm:$false
```
![image](https://hackmd.io/_uploads/rkefxe23-l.png)

- Lier les cartes aux Uplinks
Nous allons forcer l'association des cartes physiques aux ports d'Uplink du vDS pour les 3 hôtes.
![image](https://hackmd.io/_uploads/rJ2crl22bg.png)
![image](https://hackmd.io/_uploads/Skv2re32Wx.png)
![image](https://hackmd.io/_uploads/HyRfLx23-e.png)
![image](https://hackmd.io/_uploads/HyCNIghnZx.png)
![image](https://hackmd.io/_uploads/By18Ign3We.png)
![image](https://hackmd.io/_uploads/HJCD8enhZg.png)
- Augmenter le nombre d'Uplinks du vDS
On va passer le nombre d'Uplinks à 2 (ou plus) pour permettre la redondance.
```PowerShell
$vds = Get-VDSwitch -Name "RegionA01-vDS-PROD"

# On monte à 2 Uplinks pour autoriser vmnic0 et vmnic1
Set-VDSwitch -VDSwitch $vds -NumUplinkPorts 2 -Confirm:$false
```
![image](https://hackmd.io/_uploads/SJlfzlhh-x.png)
- Activer la Redondance
Une fois vmnic1 libérée sur `esx-01a`,on ajoute les deux cartes `(vmnic0 et vmnic1)` au vDS pour tous les hôtes.
```PowerShell
$vds = Get-VDSwitch -Name "RegionA01-vDS-PROD"
$hosts = Get-Cluster "RegionA01-COMP01" | Get-VMHost

foreach ($esxi in $hosts) {
    $nics = Get-VMHostNetworkAdapter -VMHost $esxi | Where-Object { $_.Name -match "vmnic[0-1]" }
    Add-VDSwitchPhysicalNetworkAdapter -DistributedSwitch $vds -VMHostPhysicalNic $nics -Confirm:$false -ErrorAction SilentlyContinue
}
```
![image](https://hackmd.io/_uploads/rJH5egnh-l.png)
- Activer HA et DRS
```PowerShell
$cluster = Get-Cluster -Name "RegionA01-COMP01"

Write-Host "Activation de HA et DRS sur $($cluster.Name)..." -ForegroundColor Cyan

Set-Cluster -Cluster $cluster `
    -DrsEnabled $true `
    -DrsAutomationLevel "FullyAutomated" `
    -HAEnabled $true `
    -HAAdmissionControlEnabled $true `
    -Confirm:$false
```
![image](https://hackmd.io/_uploads/HkbVPgh2-l.png)
- Activer le Health Check
Le vSphere DLDS Health Check est un filet de sécurité génial : il vérifie en temps réel s'il y a des erreurs de configuration sur tes VLANs, le MTU (pour le Jumbo Frames) ou le Teaming.
![image](https://hackmd.io/_uploads/Hk2cdln2-x.png)
![image](https://hackmd.io/_uploads/SJ3oug3h-l.png)
![image](https://hackmd.io/_uploads/rJkeYg32Zx.png)
![image](https://hackmd.io/_uploads/HkJbYxh2-l.png)

--- 

### 25. stockage vSphere
- Ajout hôte
```PowerShell
# 1. On récupère l'objet Datacenter
$datacenter = Get-Datacenter -Name "RegionA01"

# 2. On ajoute l'hôte avec son IP réelle
Add-VMHost -Name "10.0.0.53" -Location $datacenter -User "root" -Password "VMware123!" -Force -Confirm:$false
```
![image](https://hackmd.io/_uploads/SkPjrz22Wx.png)
![image](https://hackmd.io/_uploads/Sy4pSz23-g.png)
- Mode maintenance
```PowerShell
Set-VMHost -VMHost "10.0.0.53" -State "Maintenance"
```
![image](https://hackmd.io/_uploads/rJi-Ifnn-x.png)
- Montage du Datastore NFS sur l'hôte 3
```PowerShell
$h3 = Get-VMHost -Name "10.0.0.53"

Write-Host "Montage du datastore ds-nfs02 sur l'hôte 3..." -ForegroundColor Cyan

New-Datastore -VMHost $h3 -Name "ds-nfs02" -Nfs -NfsHost "10.10.20.60" -Path "/mnt/NFS02" -ReadOnly:$false
```
![image](https://hackmd.io/_uploads/rkDUPzn3bl.png)
![image](https://hackmd.io/_uploads/HJTKwz32Ze.png)
- Créer un datastore iSCSI vSphere
![image](https://hackmd.io/_uploads/HJdUtf33-x.png)
![image](https://hackmd.io/_uploads/HJDDYM33Wx.png)
![image](https://hackmd.io/_uploads/H1CFtf23be.png)
![image](https://hackmd.io/_uploads/SJ2atMh3-x.png)
![image](https://hackmd.io/_uploads/BktCtzhnZx.png)
![image](https://hackmd.io/_uploads/Syg-qfh2bl.png)
![image](https://hackmd.io/_uploads/BkjZqf22-l.png)
![image](https://hackmd.io/_uploads/HyTzqG22Zl.png)
- Monter un datastore NFS sur un nouvel hôte – Assistant
Dans cette étape, nous utilisons l’assistant pour monter un datastore NFS (`ds-nfs01 datastore`) sur un nouvel hôte ESXi.  
Cela permet de connecter l’hôte à un stockage réseau distant (serveur NFS) afin qu’il puisse accéder aux machines virtuelles et aux fichiers partagés déjà existants.
```PowerShell
New-Datastore -VMHost $h3 -Name "ds-nfs01" -Nfs -NfsHost "10.10.20.60" -Path "/mnt/NFS01" -ReadOnly:$false
```
![image](https://hackmd.io/_uploads/Sy1kjGh2-l.png)
![image](https://hackmd.io/_uploads/BJZroz3h-x.png)
- Ajouter un serveur SendTarget"
Cette option permet à un hôte ESXi de spécifier un serveur iSCSI cible pour découvrir automatiquement les LUNs disponibles sur ce serveur.
![image](https://hackmd.io/_uploads/r1ZChM22bl.png)
![image](https://hackmd.io/_uploads/BkkN6z3h-x.png)
![image](https://hackmd.io/_uploads/BkpUaz23We.png)
- Relancer l’analyse de l’adaptateur de stockage iSCSI
Cette opération permet à l’hôte ESXi de rechercher de nouveaux LUNs ou modifications sur la cible iSCSI après l’ajout ou la modification d’une cible.
![image](https://hackmd.io/_uploads/HJqcTMh3bg.png)
![image](https://hackmd.io/_uploads/rkIi6M2h-x.png)
- Déplacer dans le cluster
Cette action consiste à intégrer un hôte ESXi existant dan# Vérifier l'état du service SSH sur tous les hôtes
Get-VMHost | Get-VMHostService | Where-Object {$_.Key -eq "TSM-SSH"}

# Désactiver et arrêter le SSH (Hardening)
Get-VMHost | Get-VMHostService | Where-Object {$_.Key -eq "TSM-SSH"} | Set-VMHostService -Policy Off
Get-VMHost | Get-VMHostService | Where-Object {$_.Key -eq "TSM-SSH"} | Stop-VMHostService -Confirm:$false

# Réactiver et démarrer le SSH
Get-VMHost | Get-VMHostService | Where-Object {$_.Key -eq "TSM-SSH"} | Set-VMHostService -Policy On
Get-VMHost | Get-VMHostService | Where-Object {$_.Key -eq "TSM-SSH"} | Start-VMHostService -Confirm:$falses un cluster, afin qu’il puisse participer aux fonctionnalités partagées comme vMotion, HA et DRS.
```PowerShell
$h3 = Get-VMHost -Name "10.0.0.53"
$cluster = Get-Cluster -Name "RegionA01-COMP01"

# Déplacement de l'hôte dans le cluster
Move-VMHost -VMHost $h3 -Destination $cluster -Confirm:$false
```
![image](https://hackmd.io/_uploads/HyXoAfn2be.png)
![image](https://hackmd.io/_uploads/ryC3CMh3-g.png)

#### 25.1 Storage vMotion
- Migration de la VM (TinyLinux)
- 
```PowerShell
# Déplacer le stockage de TinyLinux vers le datastore NFS
Move-VM -VM "TinyLinux" -Datastore "ds-nfs01" -RunAsync
```
![image](https://hackmd.io/_uploads/BJz2JXh2Ze.png)
![image](https://hackmd.io/_uploads/HyPC1XnhZx.png)

---

### 26. Gestion des instantanés des machines virtuelles
- Ajouter un nouveau disque virtuel à une machine virtuelle existante (5 GB)
```PowerShell
# 1. Sélectionner la VM
$vm = Get-VM -Name "TinyLinux"

# 2. Ajouter le nouveau disque
# On spécifie Thin pour économiser l'espace du lab
New-HardDisk -VM $vm -CapacityGB 5 -StorageFormat Thin -Persistence Persistent

Write-Host "Le disque de 5 GB a été ajouté avec succès à $($vm.Name)" -ForegroundColor Green
```
![image](https://hackmd.io/_uploads/rJtxr7n2Zx.png)
- Vérification
```PowerShell
Get-VM -Name "TinyLinux" | Get-HardDisk | Select-Object Name, CapacityGB, StorageFormat
```
![image](https://hackmd.io/_uploads/HkCIr72nbl.png)
- Étendre le disque original de la machine virtuelle pour augmenter sa capacité
```PowerShell
# 1. On récupère le premier disque de la VM
$vm = Get-VM -Name "TinyLinux"
$disk = Get-HardDisk -VM $vm | Where-Object {$_.Name -eq "Hard disk 1"}

# 2. On modifie la capacité vers 32 GB
Set-HardDisk -HardDisk $disk -CapacityGB 32 -Confirm:$false

Write-Host "Le disque 1 de $($vm.Name) est passé de 30 GB à 32 GB." -ForegroundColor Green
```
![image](https://hackmd.io/_uploads/BJsb8Q3h-g.png)
![image](https://hackmd.io/_uploads/r1lSImh3-g.png)

---

### 27. Gestion des instantanés des machines virtuelles
- Créer 1 Snapshot
```PowerShell
# 1. Identifier la VM
$vmWin = Get-VM -Name "Windows10"

Write-Host "Création du snapshot pour $($vmWin.Name)..." -ForegroundColor Cyan

# 2. Créer le snapshot
# On inclut la mémoire pour pouvoir revenir à l'état "Allumé" exactement là où on était
New-Snapshot -VM $vmWin -Name "Snapshot_Avant_Modif" -Description "Snapshot de sécurité fait pendant le TP" -Memory -Quiesce
```
![image](https://hackmd.io/_uploads/SJLMwX22-e.png)
![image](https://hackmd.io/_uploads/BkgEP7nhWl.png)
- Changer la taille de la mémoire du VM de 2 à 4 GB
![image](https://hackmd.io/_uploads/HkvTvm22bg.png)
```PowerShell
# 1. Éteindre la VM proprement (Guest OS)
Stop-VMGuest -VM $vmWin -Confirm:$false

# 2. Changer la RAM à 4 GB maintenant qu'elle est hors ligne
Set-VM -VM $vmWin -MemoryGB 4 -Confirm:$false

# 3. Rallumer la VM
Start-VM -VM $vmWin -Confirm:$false

```
![image](https://hackmd.io/_uploads/S1aSOXh2Zl.png)
![image](https://hackmd.io/_uploads/B1ODumnnbl.png)
- Restauration
Une fois les 4 GB, on teste la restauration pour revenir à tes 2 GB (l'état du snapshot)
```PowerShell
# Restaurer le snapshot pour annuler la modif de RAM
$snap = Get-Snapshot -VM $vmWin | Sort-Object Created -Descending | Select-Object -First 1
Set-VM -VM $vmWin -Snapshot $snap -Confirm:$false
```
![image](https://hackmd.io/_uploads/SkER_X33Ze.png)
![image](https://hackmd.io/_uploads/HJBJKX2h-x.png)
- Supprimer le Snapshot
![image](https://hackmd.io/_uploads/Sy65Y72nWl.png)

---

### 28. Protection VM
#### 28.1  SSH
- Service SSH
```PowerShell
# Vérifier l'état du service SSH sur tous les hôtes
Get-VMHost | Get-VMHostService | Where-Object {$_.Key -eq "TSM-SSH"}

# Désactiver et arrêter le SSH (Hardening)
Get-VMHost | Get-VMHostService | Where-Object {$_.Key -eq "TSM-SSH"} | Set-VMHostService -Policy Off
Get-VMHost | Get-VMHostService | Where-Object {$_.Key -eq "TSM-SSH"} | Stop-VMHostService -Confirm:$false

# Réactiver et démarrer le SSH
Get-VMHost | Get-VMHostService | Where-Object {$_.Key -eq "TSM-SSH"} | Set-VMHostService -Policy On
Get-VMHost | Get-VMHostService | Where-Object {$_.Key -eq "TSM-SSH"} | Start-VMHostService -Confirm:$false
```
![image](https://hackmd.io/_uploads/Hy2QeN2hbx.png)
![image](https://hackmd.io/_uploads/BkYElNn3Zx.png)

- Charger le module de sécurité
```PowerShell
Import-Module VMware.VimAutomation.Security

# Vérification
Get-Command -Module VMware.VimAutomation.Security
```
![image](https://hackmd.io/_uploads/S1yrM4nhbe.png)

#### 28.2 NKP
- Configuration du Native Key Provider (NKP)
Avant de chiffrer, vCenter doit avoir un fournisseur de clés actif.
![image](https://hackmd.io/_uploads/rkMUEV2n-x.png)
![image](https://hackmd.io/_uploads/SkPXcEn3Ze.png)

![image](https://hackmd.io/_uploads/SytY4N2hWl.png)
- Effectuer un sauvegarde
![image](https://hackmd.io/_uploads/BkdiV4nhZx.png)
![image](https://hackmd.io/_uploads/ByL3V4h2Wl.png)
![image](https://hackmd.io/_uploads/Sya6VN2nWg.png)

#### 28.3 Configuration d'une machine virtuelle en mode vMotion chiffré
- Arrêt de la VM core-A
Pour appliquer une politique de chiffrement, la VM doit être hors tension car vSphere doit modifier le format des fichiers sur le disque.
```PowerShell
Stop-VMGuest -VM "core-A" -Confirm:$false
```
![image](https://hackmd.io/_uploads/BkS6UVhhZx.png)
![image](https://hackmd.io/_uploads/By80U43nbg.png)

- Application du chiffrement : Chiffrer la machine virtuelle
![image](https://hackmd.io/_uploads/HkfK-fr2We.png)
![image](https://hackmd.io/_uploads/HJj9-GH3bx.png)
![image](https://hackmd.io/_uploads/B1vjZzr3Wl.png)
![image](https://hackmd.io/_uploads/SJmhZzBhZg.png)
![image](https://hackmd.io/_uploads/ByqA-zH3Ze.png)
- Migration
![image](https://hackmd.io/_uploads/B1TsGGHhZl.png)
![image](https://hackmd.io/_uploads/BJu6GzHhbx.png)
![image](https://hackmd.io/_uploads/BJxJQfr3-x.png)
![image](https://hackmd.io/_uploads/SyMe7GB3Zg.png)
![image](https://hackmd.io/_uploads/BJ6g7zH3Ze.png)

#### 28.4 Exploiter PowerCLI
- Afficher l'état de chiffrement des VMs
![image](https://hackmd.io/_uploads/Hk0qNfr3bl.png)
**Remarque** : aucune des Vms listées n'est actuellement chiffré.
- Supprimer les Snapshots
Supprimer les snapshot avant le chiffrement.
![image](https://hackmd.io/_uploads/B11CLGShbe.png)
![image](https://hackmd.io/_uploads/HyXJwzrhWg.png)
![image](https://hackmd.io/_uploads/rJvxvMHnWx.png)
![image](https://hackmd.io/_uploads/rkIbvMH2Zx.png)
- Chiffrer la Vm `Core-A`
![image](https://hackmd.io/_uploads/SkALDzH2Wx.png)
- Vérifier si le chiffrement est bien passé
![image](https://hackmd.io/_uploads/H1WhDfSnbg.png)
- Voir la stratégie de stockage de la VM chiffré
![image](https://hackmd.io/_uploads/rkdeOfS3Zx.png)
- Déchiffrer la VM `Core-A`
![image](https://hackmd.io/_uploads/HykVOfr3bg.png)
- Vérification > Déchiffrement
![image](https://hackmd.io/_uploads/rkKvOfrnWe.png)
![image](https://hackmd.io/_uploads/HyD5uzBn-g.png)
- Pour savoir par quel serveur KMS les machines virtuelles chiffrées ont été chiffrées
![image](https://hackmd.io/_uploads/SkitKGH2Ze.png)
![image](https://hackmd.io/_uploads/Hywk5zB2We.png)
- Pour voir quel cluster KMS est celui par defaut
![image](https://hackmd.io/_uploads/r1vScGHhWg.png)

#### 28.5 Commandes PowerCLI

- Affiche le cluster KMS actuel  
```powershell
Get-KmsCluster
```

- Définit un nouveau cluster KMS

```PowerShell
Set-DefaultKmsCluster -KmsCluster "Nouveau_Nom_Cluster_KMS"
```

- Confirme le nouveau KMS

```PowerShell
Get-KmsCluster
```

- Définit un nouveau cluster KMS par défaut

```PowerShell
Set-DefaultKmsCluster -KmsCluster "Nouveau_Nom_Cluster_KMS"
```

- Supprime un cluster KMS existant

```PowerShell
Remove-KmsCluster -KmsCluster "Nom_Cluster_KMS"
```

- Ajoute un cluster KMS à vCenter

```PowerShell
Add-KmsCluster -Name "Nom_Cluster_KMS" -Server "Adresse_KMS"
```

- Active le chiffrement d’une machine virtuelle

```PowerShell
Get-VM -Name "VM-Name" | Enable-VMEncryption
```

- Désactive le chiffrement d’une machine virtuelle

```PowerShell
Get-VM -Name "VM-Name" | Disable-VMEncryption
```

- Affiche si une VM est chiffrée ou non

```PowerShell
Get-VM -Name "VM-Name" | Select Name,EncryptionEnabled
```

#### 28.6 VTPM

TPM (Trusted Platform Module) : puce physique sur un ordinateur qui stocke en toute sécurité les clés de chiffrement et garantit l’intégrité du système.
vTPM : module TPM émulé dans une machine virtuelle pour fournir les mêmes fonctions de sécurité qu’un TPM physique.

Le vTPM permet de :
- Stocker de manière sécurisée les clés de chiffrement des VM
- Utiliser BitLocker ou d’autres solutions de chiffrement dans la VM
- Vérifier l’intégrité de la VM (protection contre le démarrage non autorisé)
- Fonctionner avec le chiffrement des machines virtuelles vSphere

#### 28.6.1 Configuration
- Se connecter 
![image](https://hackmd.io/_uploads/H1tdm7Sh-g.png)
- Importer les modules 
![image](https://hackmd.io/_uploads/BJtHVQB2Ze.png)

- Ajouter le vTPM à la VM 
![image](https://hackmd.io/_uploads/rkgPNmH3Wx.png)

- Eteindre la VM
![image](https://hackmd.io/_uploads/ryw61Qr2Wg.png)
- Activer le secure boot dans les paramètres
![image](https://hackmd.io/_uploads/Hkm24XH2-l.png)
- Redémarrer la VM
![image](https://hackmd.io/_uploads/By3CEQS2Zx.png)
- Vérification 
![image](https://hackmd.io/_uploads/r1FOUmH3bx.png)

---

### 29. Fédération d'identité dans vSphere
vCenter Server peut interagir avec **Active Directory**, **LDAP**, et dispose de son propre annuaire pour l'**authentification unique (SSO)** vSphere.

#### 29.1 Exigences pour le service AD FS
- **AD FS** : Windows Server 2016 R2 ou version ultérieure  
- AD FS doit être **connecté à Active Directory**  
- Un **groupe d'applications pour vCenter Server** doit être créé  
- Un **certificat de serveur AD FS** (ou certificat CA/intermédiaire ayant signé le certificat de serveur AD FS) doit être ajouté au **magasin de certificats racines de confiance**  
- Un **groupe `vCenter Server Administrators`** dans AD FS doit inclure les utilisateurs à qui vous souhaitez accorder des privilèges d’administrateur vCenter Server
- 
#### 29.2 Configuration requise pour vCenter Server
- **vSphere 7.0 ou version ultérieure**  
- vCenter doit pouvoir **se connecter au point de terminaison de découverte AD FS**  
- Privilège nécessaire : **VcldentityProviders.Manage**

#### 29.3 Configuration
- Se connecter 
![image](https://hackmd.io/_uploads/HkiBnmH2We.png)
![image](https://hackmd.io/_uploads/rJyY37B3bl.png)
- Ajouter un certificat racine de confiance
![image](https://hackmd.io/_uploads/Bkwa3mS3-e.png)
![image](https://hackmd.io/_uploads/r1CAnmS3bx.png)
![image](https://hackmd.io/_uploads/rJEe6Xr3Zg.png)
![image](https://hackmd.io/_uploads/SJ7W6QB2Zg.png)
- Importer le certficat
![image](https://hackmd.io/_uploads/rJpQT7Hn-x.png)
![image](https://hackmd.io/_uploads/BkEUTQr3Wl.png)
![image](https://hackmd.io/_uploads/rJSw67Hn-x.png)
![image](https://hackmd.io/_uploads/BJ8q67H3bl.png)
- Remplacer le fournisseur d'identité par défaut par Microsoft AD FS
![image](https://hackmd.io/_uploads/H1uzCmrh-l.png)
![image](https://hackmd.io/_uploads/H1pmRmBnZl.png)
- Voir les prérequis 
![image](https://hackmd.io/_uploads/B1A8CQrnbl.png)
![image](https://hackmd.io/_uploads/HyWOCmHhZx.png)
- Tester l'éligibilité
![image](https://hackmd.io/_uploads/Byes0QB2Wg.png)
![image](https://hackmd.io/_uploads/r1z207rh-x.png)
- Accepter les conditions
![image](https://hackmd.io/_uploads/BJ_6R7BnWl.png)
- Remplir les informations relative au groupe d'applications AD FS
![image](https://hackmd.io/_uploads/Skx1eVrhZx.png)
![image](https://hackmd.io/_uploads/B11qxVB2bg.png)
![image](https://hackmd.io/_uploads/r16qf4rhZx.png)
![image](https://hackmd.io/_uploads/Hkk8QVB3Wx.png)
- Ajout du group AD + Ajouter l'utilisateur` holdev`
![image](https://hackmd.io/_uploads/HJti7VBnZg.png)
![image](https://hackmd.io/_uploads/HkyRmEr2We.png)
![image](https://hackmd.io/_uploads/HyQNE4S2Zx.png)
![image](https://hackmd.io/_uploads/Hk4jEVBnWx.png)
- Se déconnecte de Vcenter et se reconnecter ave ce nouveau utilisateur ajouté
![image](https://hackmd.io/_uploads/rJ4yHVH2Wx.png)
![image](https://hackmd.io/_uploads/SyOEBVr2bl.png)
![image](https://hackmd.io/_uploads/SyZPHEB3bg.png)
![image](https://hackmd.io/_uploads/HkJtS4H2Wg.png)
- Désactiver le AD FS et rétablir celui de vCenter
![image](https://hackmd.io/_uploads/SJ76UEHnbl.png)
![image](https://hackmd.io/_uploads/HJNR8VBnWg.png)
![image](https://hackmd.io/_uploads/ByOkPVSh-g.png)

---

### 30. Rôle et permissions
#### 30.1 Utilisateur
Créer un utilisateur, lui attribuer le rôle de cryptographie, puis se connecter en tant que cet utilisateur et lui attribuer le rôle d’administrateur.
![image](https://hackmd.io/_uploads/ByK_CE33be.png)
![image](https://hackmd.io/_uploads/rJdoJB2nbe.png)
![image](https://hackmd.io/_uploads/S1l5pJSh2-l.png)
![image](https://hackmd.io/_uploads/SJoCJH2nbl.png)

#### 30.2 Rôle > permission
- Attribuer le rôle de Cryptographie à l'utilisateur `estiam`
```
# 1. On récupère le rôle par son nom exact
$role = Get-VIRole -Name "NoCryptoAdmin"

# 2. On l'assigne à estiam sur la racine (Datacenters)
New-VIPermission -Entity (Get-Folder -Name "Datacenters") -Principal "vsphere.local\estiam" -Role $role -Propagate $true

Write-Host "L'utilisateur estiam a maintenant le rôle NoCryptoAdmin." -ForegroundColor Cyan
```
![image](https://hackmd.io/_uploads/Sk3RxH32Zl.png)
- Se connecter en tant que `estiam`
```PowerShell
# Déconnecte tout proprement
Disconnect-VIServer * -Confirm:$false

# Teste la connexion avec le nom complet
$vc = "vcsa-01a.vcf.sddc.lab" 
Connect-VIServer -server vcsa-01a.vcf.sddc.lab -user estiam@vsphere.local -password VMware123!
```
![image](https://hackmd.io/_uploads/rku4GSh3Zl.png)
![image](https://hackmd.io/_uploads/HySUzBh3-e.png)

#### 30.3 Modifier la stratégie d'une VM
![image](https://hackmd.io/_uploads/H1z_jVBn-e.png)
![image](https://hackmd.io/_uploads/BkbqsNrh-l.png)
![image](https://hackmd.io/_uploads/BJpssEB2bx.png)

---

### 31. VBS POUR MACHINES VIRTUEĻLES ET DÉMARRAGE SÉCURISÉ 

#### 31.2 Qu’est-ce que VBS ?
- **VBS (Virtualization-Based Security)** utilise la **virtualisation pour isoler les composants critiques** du système d’exploitation.  
- Crée une **enclave sécurisée** pour protéger :
  - Clés de chiffrement  
  - Informations d’identification  
  - Processus critiques  


#### 31.2 Utilité dans vSphere
- Permet aux VM Windows 10/11 et Windows Server de bénéficier de fonctionnalités avancées :
  - **Credential Guard** : protection des mots de passe et jetons  
  - **Device Guard** : renforcement de la sécurité des applications  
- Fonctionne en combinaison avec le **vTPM** et le chiffrement des VM pour sécuriser les données sensibles.


#### 31.3 Configuration
Pour activer VBS dans une VM vSphere :
1. La VM doit utiliser **UEFI avec Secure Boot**  
2. Activer le **vTPM** dans la VM  
3. Activer VBS depuis les **paramètres Windows** de la VM

#### 31.4 Secure Boot
- Secure Boot est une fonctionnalité de UEFI (Unified Extensible Firmware Interface).
- Son rôle est de vérifier l’intégrité du système au démarrage pour s’assurer que seuls les logiciels et systèmes d’exploitation approuvés puissent s’exécuter.
- Cela empêche le chargement de malwares ou rootkits avant que le système d’exploitation ne démarre.

#### 31.5 Comment ça fonctionne ?
- Le firmware UEFI contient une liste de signatures approuvées (certificats).
- Au démarrage, chaque composant critique (bootloader, noyau OS) est vérifié par rapport à cette liste.
- Si une signature est valide → le système démarre normalement.
- Si une signature n’est pas reconnue → le démarrage est bloqué.

####  31.6 Configurer Windows 10 pour VBS
- Se connecter
![image](https://hackmd.io/_uploads/ryyGAES3Zl.png)
- Eteindre la VM `Win10`
![image](https://hackmd.io/_uploads/HyxdCNB3Wg.png)
![image](https://hackmd.io/_uploads/rJuF0NS3Zl.png)
![image](https://hackmd.io/_uploads/Hyi5AEHn-x.png)
- Modifier les paramètres de cette VM
![image](https://hackmd.io/_uploads/HytaCErhZx.png)
- Activier le secure boot
![image](https://hackmd.io/_uploads/r1aygrShZl.png)
- Redémarrer la VM
![image](https://hackmd.io/_uploads/HJjflHrnbl.png)
![image](https://hackmd.io/_uploads/SJHNgBHnbe.png)
- Voir la liste des VMs
![image](https://hackmd.io/_uploads/SyEceBH3-g.png)
- Au niveau des colonnes > Voir TPM & VBS
![image](https://hackmd.io/_uploads/rkL8ZrH3Wg.png)
![image](https://hackmd.io/_uploads/H1Lv-SH3Zx.png)
![image](https://hackmd.io/_uploads/rJjOZSS3bx.png)
- Se connecter à la VM et lancer PowerShell
![image](https://hackmd.io/_uploads/HySkzBH2-l.png)
![image](https://hackmd.io/_uploads/HJXgzrS2be.png)
![image](https://hackmd.io/_uploads/rksZGHr3be.png)
![image](https://hackmd.io/_uploads/HJHzfSB2Zx.png)
![image](https://hackmd.io/_uploads/rkfVGSr2bx.png)
- Définir la politique d'éxécution des scripts dans PowerShell
![image](https://hackmd.io/_uploads/HJ0iMSBn-x.png)
- Lancer le script  > TPM
![image](https://hackmd.io/_uploads/r1Tw7Hr2Wl.png)

#### 31.7 Installer un VIB non signé

Dans cette partie, nous allons :
1. Copier un fichier ZIP d’installation **vSphere (VIB) non signé** dans le répertoire de l’hôte ESXi (**esx-04b**) afin de préparer son installation  
2. Nous connecter ensuite à l’hôte ESXi pour exécuter la commande d’installation du VIB non signé  

##### 31.8 Configuration VIB
- Se connecter 
![image](https://hackmd.io/_uploads/ryuA4SHh-g.png)
- Eteindre et désactiver le secure boot au niveau de la machine `esx-04b`
![image](https://hackmd.io/_uploads/H118BHH2bx.png)
![image](https://hackmd.io/_uploads/Hk6UrSHhZg.png)
![image](https://hackmd.io/_uploads/ryF_SSrn-l.png)
- Redémarrer la VM
![image](https://hackmd.io/_uploads/SkeorHH3-e.png)
![image](https://hackmd.io/_uploads/B1DnBSS3Ze.png)

- Copier le fichier VIB (vSphere Installation Bundle) non signé sur la machine virtuelle `esx-04b` ; nous utiliserons l'application WinSCP pour ce faire.
    - Lancer WinSCP
![image](https://hackmd.io/_uploads/BJJS8HB2Zg.png)
    - Remplir les identifiants de connexion et se connecter
![image](https://hackmd.io/_uploads/SJvcUSrhZe.png)
![image](https://hackmd.io/_uploads/BJni8HHnZe.png)
    - Copier le Zip
![image](https://hackmd.io/_uploads/ByRGvHS3We.png)
![image](https://hackmd.io/_uploads/SyFNPrH3Ze.png)
![image](https://hackmd.io/_uploads/BJqHDBBnZl.png)
    - Fermer winSCP
![image](https://hackmd.io/_uploads/B17uPBHhZg.png)

- Nous allons maintenant utiliser Putty pour nous connecter à l'hôte virtuel et exécuter les commandes permettant d'installer le package d'installation vSphere (VIB) non signé.
![image](https://hackmd.io/_uploads/ByOhvBS3We.png)
![image](https://hackmd.io/_uploads/B1HXOBBhWg.png)
![image](https://hackmd.io/_uploads/HJFIdHSnWe.png)
- Arrêter le système d'exploitation invité sur la VM
![image](https://hackmd.io/_uploads/B1Tq_SBh-g.png)
![image](https://hackmd.io/_uploads/S1YiOrrnbx.png)

#### 31.9 Activer le démarrage sécurisé et mettre sous tension l'hôte
Avant de mettre le serveur sous tension, il est nécessaire d'activer le démarrage sécurisé dans ses paramètres. Pour un serveur physique classique, cette activation se fait dans le BIOS. L'emplacement et le paramètre exacts dans le BIOS varient généralement d'un fournisseur à l'autre. 

Une fois le démarrage terminé, nous verrons apparaître l' écran rose de la mort (PSOD) indiquant l'état suivant : Échec de la validation des niveaux d'acceptation : Niveau d'acceptation attendu (partenaire) trouvé (communauté).
Remarque : Ce comportement est normal, car nous avons précédemment installé un package d'installation vSphere (VIB) non signé sur l'hôte. L'activation du démarrage sécurisé vérifie la présence de packages d'installation vSphere (VIB) non signés et empêche le démarrage complet de l'hôte le cas échéant. Seuls les packages d'installation vSphere (VIB) signés sont autorisés à démarrer.

- Modifier les paramètres de la VM
![image](https://hackmd.io/_uploads/HJCoFSBhWg.png)
- Activer le secure boot
![image](https://hackmd.io/_uploads/r1SRYBr2-e.png)
![image](https://hackmd.io/_uploads/BkSkcrr2bg.png)
- Redémarrer la VM
![image](https://hackmd.io/_uploads/ryZfcrrn-l.png)
![image](https://hackmd.io/_uploads/Bkom5rHhZe.png)
![image](https://hackmd.io/_uploads/rkVi9Hr2Wl.png)
- Maintenant pour que la Vm fonctionne à nouveau : désactive rle secure boot
![image](https://hackmd.io/_uploads/H1JGjHS2-g.png)
![image](https://hackmd.io/_uploads/r1GmjSB2-e.png)
![image](https://hackmd.io/_uploads/BJCQjSHh-l.png)
![image](https://hackmd.io/_uploads/HkZujBBhbg.png)
On remarque bien que la VM a démarré avec succès. 
- Supprimer le Zip maintenant via une connexion PuTTY
![image](https://hackmd.io/_uploads/SkQx2rrnbx.png)

```Bash
esxcli software vib remove -n net-tulip
```
![image](https://hackmd.io/_uploads/rk_w2SS3Ze.png)
