# **Attribution dynamique de VLAN** 

![Figure 1 : Schéma structurel du Lab EVE-NG](.attachments.652/Capture%20d%27%C3%A9cran%202024-01-10%20162931.png)

# 

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

> ## **Objectif**

Dans ce projet mené sur la plateforme EVE-NG, l'objectif est de créer un environnement réseau virtuel représentatif d'une infrastructure d'entreprise. Deux VLAN distincts seront configurés pour symboliser deux services distincts au sein de l'organisation. L'accent sera mis sur l'attribution de deux utilisateurs Windows 10 à ces VLAN respectifs, chacun appartenant à des groupes distincts dans le cadre d'Active Directory.

La finalité de cette démarche est de démontrer la segmentation efficace du réseau grâce aux VLAN. Cette segmentation permettra un contrôle précis du trafic, garantissant une meilleure gestion des ressources. En associant ces VLAN à des groupes spécifiques d'Active Directory, des politiques pourront être appliquées afin de définir les droits d'accès propres à chaque service.

L'objectif global est d'illustrer comment la gestion des VLAN, conjointement avec l'intégration d'Active Directory, peut refléter la structure organisationnelle d'une entreprise. Cette approche permettra également de contrôler plus rigoureusement le trafic réseau, assurant ainsi une sécurité renforcée et une gestion optimale des données au sein du réseau. 

Ce laboratoire virtuel sur EVE-NG permettra de visualiser concrètement comment ces mesures peuvent être mises en place pour répondre aux besoins spécifiques d'une organisation.

![Figure 2 : Attribution VLAN](.attachments.652/Capture%20d%27%C3%A9cran%202024-01-11%20101138.png)

> ## **NPS Policy**

![Figure 3 : Stratégie de demande](.attachments.652/Capture%20d%27%C3%A9cran%202024-01-10%20172014.png)

![Figure 4 : Stratégie réseau PAPA](.attachments.652/Capture%20d%27%C3%A9cran%202024-01-10%20172131.png)

![Figure 5 : Stratégie réseau FILS](.attachments.652/Capture%20d%27%C3%A9cran%202024-01-10%20172203.png)

> ## **Switch Vios Configuration** 

### **Configuration Switch, attribution automatique de VLAN**

```
#### IP du switch

enable
conf t
hostname SW1
interface vlan 1
ip address 10.10.10.1 255.255.255.0
no shut
end


#### Création des vlan sur le switch


configure terminal
vlan 100
name PAPA
vlan 200
name FILS
end


#### Port en mode trunk

configure terminal
interface GigabitEthernet 0/0
no shut
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan al
end


#### Activation radius

configure terminal
aaa new-model
aaa authentication dot1x default group radiusaaa authorization network default group radiusdot1x system-auth-control
radius server RADIUS_SERVER
address ipv4 10.10.10.10 auth-port 1812 acct-port 1813
key Test123
end


#### Configuration des ports DOT1X

conf t
radius-server dead-criteria time 3 tries 3
radius-server timeout 3

interface GigabitEthernet 0/2
no shut
switchport mode access
dot1x pae authenticator
dot1x port-control auto
authentication event server dead action authorize
authentication event server alive action reinitialize 
spanning-tree portfast
end


conf t
interface GigabitEthernet 1/0
no shut
switchport mode access
dot1x pae authenticator
dot1x port-control auto
spanning-tree portfast
authentication event server dead action authorize
authentication event server alive action reinitialize
end 
wr
```

### **Configuration attribution VLAN 100 sur le port Gi0/2** 

```
#### IP du switch

enable
conf t
hostname SW1
interface vlan 1
ip address 10.10.10.1 255.255.255.0
no shut
end

#### Port en mode trunk

configure terminal
interface GigabitEthernet 0/0
no shut
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan all
end

#### Activation radius

conf t
aaa new-model
radius server AD1
address ipv4 10.10.10.10 auth-port 1812 acct-port 1813
key Test123
end

conf t
aaa group server radius nps-servers
server name AD1
ip radius source-interface Vlan1
domain-stripping
dot1x system-auth-control
dot1x logging verbose
aaa authentication dot1x default group nps-servers
aaa authorization network default group nps-servers
end

#### Configuration du port Gi0/2 pour VLAN 100

conf t
interface GigabitEthernet 0/2
no shut
switchport mode access
switchport access vlan 100
authentication port-control auto
dot1x pae authenticator
end 
wr
```

> ## **Source**

- [**__https://ipcisco.com/lesson/cisco-802-1x-configuration/__**](https://ipcisco.com/lesson/cisco-802-1x-configuration/)
- [**__https://mikepembo.wordpress.com/2016/11/07/dynamic-vlan-assignment-cisco-and-nps/__**](https://mikepembo.wordpress.com/2016/11/07/dynamic-vlan-assignment-cisco-and-nps/)
- [**__https://www.expertnetworkconsultant.com/installing-and-configuring-network-devices/ieee-802-1x-authentication-and-dynamic-vlan-assignment-with-nps-radius-server/__**](https://www.expertnetworkconsultant.com/installing-and-configuring-network-devices/ieee-802-1x-authentication-and-dynamic-vlan-assignment-with-nps-radius-server/)

- [**__https://sharifulhoque.blogspot.com/2019/08/8021x-wired-authentication-with-cisco.html__**](https://sharifulhoque.blogspot.com/2019/08/8021x-wired-authentication-with-cisco.html)
