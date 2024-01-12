# **Attribution dynamique de VLAN** 

<figure>
  <img src="https://github.com/ApprentiBash/VLAN_Assignement/blob/main/allScreen/screen1.png?raw=true" alt="Schéma structurel du Lab EVE-NG">
  <figcaption>Figure 1 : Schéma structurel du Lab EVE-NG</figcaption>
</figure>

____
> ## **Contexte** 

Notre entreprise se prépare à instaurer une méthode dynamique pour la gestion des VLAN au sein de notre infrastructure réseau. Cette approche permettra une segmentation automatisée des périphériques connectés, basée sur des critères tels que les départements, les équipes ou les exigences de sécurité spécifiques.

En adoptant cette stratégie, nous visons à simplifier la gestion du réseau en automatisant l'attribution des VLAN. Par exemple, les employés seraient placés dans des réseaux virtuels dédiés à leur département ou à leur équipe, tandis que les dispositifs IoT ou les invités seraient isolés dans des VLAN distincts pour des raisons de sécurité.

En intégrant des mécanismes automatisés, nous réduirons la charge de configuration manuelle, tout en assurant une meilleure flexibilité pour adapter dynamiquement les changements dans la structure du réseau. Cette évolution vers des VLAN attribués dynamiquement renforcera la sécurité tout en optimisant la gestion et les performances globales de notre réseau.

### Prérequis 

**Environnement :** 

Nous utiliserons l'outil EVE-NG, un outil de virtualisation réseau. 

En utilisant cette plateforme pour créer un environnement de test, nous pourrons expérimenter et évaluer la configuration des VLAN. Cette étape nous permettra de vérifier leur fonctionnement avant de les déployer sur notre réseau en conditions réelles.

**Machine a virtualisé :** 

- 1x Pfsense (VLAN, DHCP)
- 1x Windows serveur 2016 (AD, NPS)
- 1x Switch Ciscos Vios (Relais)
- 2x Windows 10 (Client)

### Adressage IP des machines 

|  | **PFSENSE** |
:--------------:|:---------------:|
| **WAN** | 192\.168.1.100 /24 |
| **LAN** | 10\.10.10.2 /24 |
| **VLAN 100** | 10\.10.100.2 /24 |
| **VLAN 200** | 10\.10.200.2 ./24 |



| **Windows server** | 10\.10.10.10 /24 |
|:--------------:|:---------------:|
| **Switch Vios** | 10\.10.10.1 /24 |
| **Windows 10** | DHCP |


### Création domaine et utilisateur dans l'AD

<figure>
  <img src="https://github.com/ApprentiBash/VLAN_Assignement/blob/main/allScreen/screen6.png?raw=true" alt="Groupe et Utilisateur AD">
  <figcaption>Figure 2 : Groupe et Utilisateur AD</figcaption>
</figure>

____
> ## **Objectif**

Dans ce projet mené sur la plateforme EVE-NG, l'objectif est de créer un environnement réseau virtuel représentatif d'une infrastructure d'entreprise. Deux VLAN distincts seront configurés pour symboliser deux services distincts au sein de l'organisation. L'accent sera mis sur l'attribution de deux utilisateurs Windows 10 à ces VLAN respectifs, chacun appartenant à des groupes distincts dans le cadre d'Active Directory.

La finalité de cette démarche est de démontrer la segmentation efficace du réseau grâce aux VLAN. Cette segmentation permettra un contrôle précis du trafic, garantissant une meilleure gestion des ressources. En associant ces VLAN à des groupes spécifiques d'Active Directory, des politiques pourront être appliquées afin de définir les droits d'accès propres à chaque service.

L'objectif global est d'illustrer comment la gestion des VLAN, conjointement avec l'intégration d'Active Directory, peut refléter la structure organisationnelle d'une entreprise. Cette approche permettra également de contrôler plus rigoureusement le trafic réseau, assurant ainsi une sécurité renforcée et une gestion optimale des données au sein du réseau. 

Ce laboratoire virtuel sur EVE-NG permettra de visualiser concrètement comment ces mesures peuvent être mises en place pour répondre aux besoins spécifiques d'une organisation.

<figure>
  <img src="https://github.com/ApprentiBash/VLAN_Assignement/blob/main/allScreen/screen5.png?raw=true" alt="Attribution VLAN">
  <figcaption>Figure 3 : Attribution VLAN</figcaption>
</figure>

____
> ## **NPS Policy**

<figure>
  <img src="https://github.com/ApprentiBash/VLAN_Assignement/blob/main/allScreen/screen2.png?raw=true" alt="Stratégie de demande">
  <figcaption>Figure 4 : Stratégie de demande</figcaption>
</figure>

<figure>
  <img src="https://github.com/ApprentiBash/VLAN_Assignement/blob/main/allScreen/screen3.png?raw=true" alt="Stratégie réseau PAPA">
  <figcaption>Figure 5 : Stratégie réseau PAPA</figcaption>
</figure>

<figure>
  <img src="https://github.com/ApprentiBash/VLAN_Assignement/blob/main/allScreen/screen4.png?raw=true" alt="Stratégie réseau FILS">
  <figcaption>Figure 6 : Stratégie réseau FILS</figcaption>
</figure>


____
> ## **Switch Vios Configuration** 

### **Configuration Switch, attribution automatique de VLAN**

```
# Configuration de l'adresse IP du switch
enable                    # Accéder au mode d'administration privilegié
conf t                    # Accéder au mode de configuration globale
hostname SW1              # Définir le nom d'hôte du switch comme SW1
interface vlan 1          # Accéder à l'interface de gestion du VLAN 1
ip address 10.10.10.1 255.255.255.0  # Définir l'adresse IP et le masque de sous-réseau pour le VLAN 1
no shut                   # Activer l'interface
end                       # Quitter le mode de configuration

# Création des VLAN sur le switch
configure terminal        # Accéder au mode de configuration globale
vlan 100                  # Créer le VLAN 100
name PAPA                 # Nommer le VLAN 100 comme "PAPA"
vlan 200                  # Créer le VLAN 200
name FILS                 # Nommer le VLAN 200 comme "FILS"
end                       # Quitter le mode de configuration

# Configuration du port en mode trunk
configure terminal        # Accéder au mode de configuration globale
interface GigabitEthernet 0/0  # Accéder à l'interface GigabitEthernet 0/0
no shut                   # Activer le port
switchport trunk encapsulation dot1q  # Configurer l'encapsulation trunk en mode dot1q
switchport mode trunk     # Définir le mode du port en trunk
switchport trunk allowed vlan all  # Autoriser tous les VLAN sur le trunk
end                       # Quitter le mode de configuration

# Activation de l'authentification RADIUS
configure terminal        # Accéder au mode de configuration globale
aaa new-model             # Activer le modèle d'authentification, d'autorisation et d'accounting (AAA)
aaa authentication dot1x default group radius  # Définir le groupe RADIUS par défaut pour l'authentification 802.1X
aaa authorization network default group radius  # Définir le groupe RADIUS par défaut pour l'autorisation réseau
dot1x system-auth-control # Activer le contrôle d'authentification pour le protocole 802.1X
radius server RADIUS_SERVER  # Accéder à la configuration du serveur RADIUS nommé RADIUS_SERVER
address ipv4 10.10.10.10 auth-port 1812 acct-port 1813  # Configurer l'adresse IP du serveur RADIUS et les ports d'authentification et d'accounting
key Test123               # Définir la clé partagée pour l'authentification avec le serveur RADIUS
end                       # Quitter le mode de configuration

# Configuration des ports en mode DOT1X
conf t                    # Accéder au mode de configuration globale
radius-server dead-criteria time 3 tries 3  # Définir les critères de détection de l'état mort du serveur RADIUS
radius-server timeout 3   # Définir le délai d'attente pour le serveur RADIUS

interface GigabitEthernet 0/2  # Accéder à l'interface GigabitEthernet 0/2
no shut                   # Activer le port
switchport mode access    # Définir le mode du port en mode access
dot1x pae authenticator  # Configurer le port comme authenticateur 802.1X
dot1x port-control auto   # Configurer le contrôle de port automatique pour l'authentification 802.1X
authentication event server dead action authorize  # Définir l'action en cas de serveur RADIUS mort
authentication event server alive action reinitialize  # Définir l'action en cas de reprise du serveur RADIUS
spanning-tree portfast    # Activer le mode PortFast pour le port
end                       # Quitter le mode de configuration

conf t                    # Accéder au mode de configuration globale
interface GigabitEthernet 1/0  # Accéder à l'interface GigabitEthernet 1/0
no shut                   # Activer le port
switchport mode access    # Définir le mode du port en mode access
dot1x pae authenticator  # Configurer le port comme authenticateur 802.1X
dot1x port-control auto   # Configurer le contrôle de port automatique pour l'authentification 802.1X
spanning-tree portfast    # Activer le mode PortFast pour le port
authentication event server dead action authorize  # Définir l'action en cas de serveur RADIUS mort
authentication event server alive action reinitialize  # Définir l'action en cas de reprise du serveur RADIUS
end                       # Quitter le mode de configuration
write memory              # Sauvegarder la configuration dans la mémoire persistante

```

### **Configuration attribution VLAN 100 sur le port Gi0/2** 

```
# Configuration du nom d'hôte du switch
enable                    # Accéder au mode d'administration privilegié
conf t                    # Accéder au mode de configuration globale
hostname SW1              # Définir le nom d'hôte du switch comme SW1
interface vlan 1          # Accéder à l'interface de gestion du VLAN 1
ip address 10.10.10.1 255.255.255.0  # Définir l'adresse IP et le masque de sous-réseau pour le VLAN 1
no shut                   # Activer l'interface
end                       # Quitter le mode de configuration

# Configuration du port en mode trunk
configure terminal        # Accéder au mode de configuration globale
interface GigabitEthernet 0/0  # Accéder à l'interface GigabitEthernet 0/0
no shut                   # Activer le port
switchport trunk encapsulation dot1q  # Configurer l'encapsulation trunk en mode dot1q
switchport mode trunk     # Définir le mode du port en trunk
switchport trunk allowed vlan all  # Autoriser tous les VLAN sur le trunk
end                       # Quitter le mode de configuration

# Activation de l'authentification RADIUS
conf t                    # Accéder au mode de configuration globale
aaa new-model             # Activer le modèle d'authentification, d'autorisation et d'accounting (AAA)
radius server AD1         # Accéder à la configuration du serveur RADIUS nommé AD1
address ipv4 10.10.10.10 auth-port 1812 acct-port 1813  # Configurer l'adresse IP du serveur RADIUS et les ports d'authentification et d'accounting
key Test123               # Définir la clé partagée pour l'authentification avec le serveur RADIUS
end                       # Quitter le mode de configuration

conf t                    # Accéder au mode de configuration globale
aaa group server radius nps-servers  # Créer un groupe RADIUS nommé nps-servers
server name AD1           # Ajouter le serveur RADIUS AD1 au groupe
ip radius source-interface Vlan1  # Spécifier l'interface source pour les communications RADIUS
domain-stripping          # Activer le "domain stripping" pour les noms d'utilisateur
dot1x system-auth-control # Activer le contrôle d'authentification pour le protocole 802.1X
dot1x logging verbose     # Activer les journaux détaillés pour l'authentification 802.1X
aaa authentication dot1x default group nps-servers  # Définir le groupe RADIUS par défaut pour l'authentification 802.1X
aaa authorization network default group nps-servers  # Définir le groupe RADIUS par défaut pour l'autorisation réseau
end                       # Quitter le mode de configuration

# Configuration du port GigabitEthernet 0/2 pour le VLAN 100
conf t                    # Accéder au mode de configuration globale
interface GigabitEthernet 0/2  # Accéder à l'interface GigabitEthernet 0/2
no shut                   # Activer le port
switchport mode access    # Définir le mode du port en mode access
switchport access vlan 100 # Assigner le port au VLAN 100
authentication port-control auto  # Configurer le contrôle d'authentification automatique
dot1x pae authenticator  # Configurer le port comme authenticateur 802.1X
end                       # Quitter le mode de configuration
write memory              # Sauvegarder la configuration dans la mémoire persistante
```

____
> ## **Result** 

<figure>
  <img src="https://github.com/ApprentiBash/VLAN_Assignement/blob/main/allScreen/screen7.png?raw=true" alt="Resultat atterndu">
  <figcaption>Figure 7 : Resultat attendu</figcaption>
</figure>

____
> ## **Source**

- [**__https://ipcisco.com/lesson/cisco-802-1x-configuration/__**](https://ipcisco.com/lesson/cisco-802-1x-configuration/)
- [**__https://mikepembo.wordpress.com/2016/11/07/dynamic-vlan-assignment-cisco-and-nps/__**](https://mikepembo.wordpress.com/2016/11/07/dynamic-vlan-assignment-cisco-and-nps/)
- [**__https://www.expertnetworkconsultant.com/installing-and-configuring-network-devices/ieee-802-1x-authentication-and-dynamic-vlan-assignment-with-nps-radius-server/__**](https://www.expertnetworkconsultant.com/installing-and-configuring-network-devices/ieee-802-1x-authentication-and-dynamic-vlan-assignment-with-nps-radius-server/)
- [**__https://sharifulhoque.blogspot.com/2019/08/8021x-wired-authentication-with-cisco.html__**](https://sharifulhoque.blogspot.com/2019/08/8021x-wired-authentication-with-cisco.html)

### Note

- Vous pouvez deployé votre service DHCP sur Windows serv ou Pfsense peut importe, personnelement je l'ai fait sur mon Pfsense.
- Je n'ai pas détaillé le deploiment d'un serveur Active directory ni la configuration du serveur NPS, ni la mise en place d'un Pfsense. J'estime qu'il faut savoir au minimum faire cela pour la mise en place de ce lab et ce ne pas des informations compliquées à trouver.
