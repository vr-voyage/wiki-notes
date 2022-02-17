---
title: Les bases du protocole OSC avec VRchat
description: Les bases du protocole OSC, et l'utilisation de protocole avec VRChat
published: true
date: 2022-02-17T23:13:14.821Z
tags: vrchat, osc
editor: markdown
dateCreated: 2022-02-17T23:13:14.821Z
---

# Protocole OSC : Bases et utilisation avec VRChat

## Le protocole OSC

### Explications rapides

#### Utilisation

VRChat peut envoyer et recevoir des **messages OSC** via UDP.

Les ports UDP utilisés par défaut sont :
* Le port `9000` pour la réception de données (**input**)
* Le port `9001` pour l'envoi de données (**output**)

Lorsque configuré pour envoyer des données, VRChat enverra les données à l'adresse `127.0.0.1:9001` par défaut.

Il est possible de passer le paramètre `--osc=PortEntreeN:AdresseDestinataire:PortSortieN` à l'exécutable pour modifier ces valeurs.  
En sachant que :
* `PortEntreeN` est le port que VRChat doit utiliser pour recevoir des données OSC (**input**)
* `AdresseDestinataire` est l'adresse à laquelle VRChat doit envoyer toute donnée OSC (**output**)
* `PortSortieN` est le port à laquelle VRChat doit envoyer toute donnée OSC (**output**)

#### Messages OSC

Les messages OSC sont formattés de la manière suivante :

* (Chaîne ASCII) Point d'entrée, commençant par `/`
* (Chaîne ASCII) Format des données envoyées, commençant par `,`
* (Encodage binaire) Données à envoyer

Toute chaîne de caractères ASCII doit être :
* encodée en ASCII,
* terminée par des caractères NULL (`\0`),
* rembourrée avec des caractères NULL pour être alignés sur 4 octets.

Les formats sont encodées sous forme de chaître de caractère ASCII, démarrant par une virgule, et continuant avec un caractère de format par donnée envoyée :
* `i` pour des nombres entiers 32-bits
* `f` pour des nombres à virgules flottantes 32-bits

Le format `bool` de VRChat est tout simplement un entier 32-bits ayant pour valeur **0** ou **1**.

#### Liste de points d'entrée par défaut

Une liste de points d'entrée par défaut, permettant de joeur avec les contrôles, est disponible ici :  
https://docs.vrchat.com/v2022.1.1/docs/osc-as-input-controller

Il est aussi possible de contrôler les paramètres de l'avatar.  
Cela sera traité dans une autre page.

#### Exemple

Envoi de messages OSC pour avancer :
* en envoyant au point d'entrée `input/Vertical`
* la valeur flottante (`f`) : **1.0**

Le tout en Ruby (n'importe quel langage de programma fera l'affaire, cela étant) :

```ruby
require 'socket'
s = UDPSocket.new

s.send(
  mesg="/input/Vertical\0,f\0\0\x3F\x80\x00\x00",
  flags=0,
  host=127.0.0.1, 
  port=9000)
```

##### Explications
###### Point d'entrée

Le point d'entrée est `/input/Vertical`.  
C'est une chaîne de caractère ASCII, elle doit être terminée par un '\0', et rembourée par des '\0' afin qu'elle soit alignée sur 4 octets.

Résultat : 
* `/input/Vertical` (15 octets) devient
* `/input/Vertical\0` (16 octets)

###### Format

On envoi qu'un seul nombre à virgule flottante.  
Le format sera donc `,f`.
C'est une chaîne de caractère ASCII, elle doit être terminée par un '\0', et rembourée par des '\0' afin qu'elle soit alignée sur 4 octets.

Résultat : 
* `,f` (2 octets) devient
* `,f\0\0` (4 octets)

###### Données

On envoie le nombre flottant 32-bits **1.0**.  
L'encodage binaire est `3F 80 00 00` (`\x3F\x80\x00\x00` dans une chaîne de caractère Python/Ruby).

Résultat : 
* **1.0** devient 
* `\x3F\x80\x00\x00`

###### Message

Le message complet n'est autre que la concaténation des champs ci-dessus :

`/input/Vertical\0,f\0\0\x3F\x80\x00\x00`

###### Communication

Par défaut, VRChat écoute à l'adresse `127.0.0.1:9000` pour recevoir des données UDP (**input**).  
On envoie donc à cette adresse, les messages OSC encapsulées dans des paquets UDP.
