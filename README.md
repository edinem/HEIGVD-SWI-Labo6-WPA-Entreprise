- [Livrables](#livrables)

- [Échéance](#%c3%89ch%c3%a9ance)

- [Quelques éléments à considérer](#quelques-%c3%a9l%c3%a9ments-%c3%a0-consid%c3%a9rer-pour-les-parties-2-et-3)

- [Travail à réaliser](#travail-%c3%a0-r%c3%a9aliser)

# Sécurité des réseaux sans fil

## Laboratoire 802.11 Sécurité WPA Entreprise

__A faire en équipes de deux personnes__

### Objectif :

1.	Analyser les étapes d’une connexion WPA Entreprise avec une capture Wireshark
2.	__(optionnel)__ Implémenter une attaque WPE (Wireless Pwnage Edition) contre un réseau WPA Entreprise
1.  __(optionnel)__ Implémenter une attaque GTC Dowgrade contre un réseau WPA Entreprise


## Quelques éléments à considérer pour les parties 2 et 3 :

Les parties 2 et 3 sont optionnelles puisque vous ne disposez pas forcement du matériel nécessaire pour les réaliser.

En principe, il devrait être possible de démarrer vos machines en Kali natif (à partir d'une clé USB, avec une distro live par exemple) ou d'employer une autre version de Linux. Si vous n'avez pas une interface WiFi USB externe, __vous ne pouvez pas faire ces parties dans une VM Linux__. 

Dans le cas où vous arriverais à tout faire pour démarrer un Linux natif, il existe toujours la possibilité que votre interface WiFi ne puisse pas être configurée en mode AP, ce qui à nouveau empêche le déroulement des parties 2 e 3.

Ces deux parties sont vraiment intéressantes et __je vous encourage à essayer de les faire__, si vous avez les ressources. Malheureusement je ne peux pas vous proposer un bonus si vous les faites, puisqu'il risque d'y avoir des personnes qui n'auront pas la possibilité de les réaliser pour les raisons déjà expliquées.

Si toutes les équipes rendent le labo complet, il sera donc corrigé entièrement et les parties 2 et 3 seront considérées pour la note.

Si vous vous lancez dans ces deux parties, voici quelques informations qui peuvent vous aider :

- Solution à l’erreur éventuelle « ```Could not configure driver mode``` » :

```
nmcli radio wifi off
rfkill unblock wlan
```
-	Pour pouvoir capturer une authentification complète, il faut se déconnecter d’un réseau et attendre 1 minute (timeout pour que l’AP « oublie » le client) 
-	Les échanges d’authentification entreprise peuvent être facilement trouvés utilisant le filtre d’affichage « ```eap``` » dans Wireshark


## Travail à réaliser

### 1. Analyse d’une authentification WPA Entreprise

Dans cette première partie, vous allez analyser [une connexion WPA Entreprise](files/auth.pcap) avec Wireshark et fournir des captures d’écran indiquant dans chaque capture les données demandées.

- Comparer [la capture](files/auth.pcap) au processus d’authentification donné en théorie (n’oubliez pas les captures d'écran pour illustrer vos comparaisons !). En particulier, identifier les étapes suivantes :

  - Requête et réponse d’authentification système ouvert

    Ci-dessous, une capture d'écran présentant l'étape *Open System Authentification* présentée dans le cours. 

    ![](./images/ex1/authentification.png)

   	- Requête et réponse d’association (ou reassociation)
  	
  	Ci-dessous, une capture d'écran présentant l'étape *Association* présentée dans le cours. 
  	
  	![](./images/ex1/association.png)
  	
  - Négociation de la méthode d’authentification entreprise

    Ci-dessous, une capture d'écran présentant l'étape de la négociation de la méthode d'authentification entre le client et le serveur. 

    ![](./images/ex1/methode_auth.png)

  - Phase d’initiation. Arrivez-vous à voir l’identité du client ?

    Comme on peut le voir ci-dessous, l'identité du client qui souhaite se connecté est "*einet\joel.gonina*". 

    ![](./images/ex1/methode_auth2.png)

  - Phase hello :

    Ci-dessous, deux captures d'écran présentant les informations de la phase Hello du client et du serveur. On discutera plus bas les informations trouvées dans ces dernières.

    **Client : **

    ![](./images/ex1/client_hello.png)

    **Server :**

    ![](./images/ex1/server_hello.png)

    - Version TLS

      La version de TLS utilisée est la 1.0 mais le client demande de passer sur la version 1.2. Cependant, nous pouvons remarquer que le serveur indique au client que la version utilisée pour les futurs échanges sera la 1.0.

    - Suites cryptographiques et méthodes de compression proposées par le client et acceptées par l’AP

      Ci-dessous, une capture d'écran présentant les ciphersuites supportées par le client : 

      ![](./images/ex1/ciphersuites_client.png)

      Le serveur a indiqué, au travers du *Server Hello* que la ciphersuite utilisée sera *TLS_RSA_WITH_AES_256_CBC_SHA*.

    - Nonces

      Les nonces peuvent se trouver sous les champs "Random" du *Client Hello* et *Server Hello*.

    - Session ID

      Les sessions ID peuvent se trouver sous les champs "Session ID" du *Client Hello* et *Server Hello*.

  - Phase de transmission de certificats
      - Echanges des certificats

      Ci-dessous, on peut voir que le serveur envoie trois certificats au client : 

      ![](./images/certificats.png)

    - Change cipher spec

      Ci-dessous, une capture présentant la capture d'écran du paquet "*Change Cipher Spec*". Il permet de repréciser l'algorithme à utiliser pour les futurs messages chiffrés et contient un *Handshake* chiffré qui est le hash de tous les messages précédemment échangés entre le client et le serveur durant la négociation TLS. 

      ![](./images/ex1/cipher_spec.png)

  - Authentification interne et transmission de la clé WPA (échange chiffré, vu comme « Application data »)

    Ci-dessous, la capture d'écran présentant l'authentification interne et la transmission de la clé WPA.

    ![](./images/ex1/application_data.png)

  - 4-way handshake

    Ci-dessous, une capture d'écran présentant le 4-way handshake entre le client et le serveur.

    ![](./images/ex1/4-way-handshake.png)

### Répondez aux questions suivantes :

> **_Question :_** Quelle ou quelles méthode(s) d’authentification est/sont proposé(s) au client ?
>
> **_Réponse :_** Lors du premier échange, l'AP propose la méthode d'authentification EAP-TLS. Cependant, le client va refuser cette méthode et demander d'utiliser la méthode d'authentification EAP-PEAP.
>
> Ci-dessous, une capture d'écran présentant le paquet où l'AP demande d'utiliser EAP-TLS : 
>
> ![](./images/eap-tls.png)
>
> Ci-dessous, une capture d'écran présentant le paquet où le client demande d'utiliser EAP-PEAP : 
>
> ![](./images/request-eap-peap.png)

---

> **_Question:_** Quelle méthode d’authentification est finalement utilisée ?
>
> **_Réponse:_** Comme mentionné précédemment, le client a demandé d'utiliser EAP-PEAP et c'est cette méthode qui sera utilisée.
>
> ![](./images/eap-peap-request.png)

---

> **_Question:_** Lors de l’échange de certificats entre le serveur d’authentification et le client :
>
> - a. Le serveur envoie-t-il un certificat au client ? Pourquoi oui ou non ?
>
> **_Réponse:_** Oui, il en envoit même trois : le sien, et deux autres certificats de l'issuer de celui du serveur. Ainsi, le client pourra valider le certificat reçu. Le serveur envoit son certificat afin de pouvoir valider son identité auprès du client. Cela permet d'éviter des attaques de type MITM par exemple. 
>
>   Ci-dessous, la capture d'écran présentant le paquet contenant les trois certificats :
>
> ![](./images/certificats.png)
>
> 
>
> - b. Le client envoie-t-il un certificat au serveur ? Pourquoi oui ou non ?
>
> **_Réponse:_** Non le client n'envoit pas de certificat au serveur car cette méthode d'authentification repose sur des identifiants (uesrname:password). Contrairement à EAP-TLS qui repose sur des certificats afin d'authentifier les clients et le serveur.  

---

### 2. (__Optionnel__) Attaque WPA Entreprise (hostapd)

Les réseaux utilisant une authentification WPA Entreprise sont considérés aujourd’hui comme étant très surs. En effet, puisque la Master Key utilisée pour la dérivation des clés WPA est générée de manière aléatoire dans le processus d’authentification, les attaques par dictionnaire ou brute-force utilisés sur WPA Personnel ne sont plus applicables. 

Il existe pourtant d’autres moyens pour attaquer les réseaux Entreprise, se basant sur une mauvaise configuration d’un client WiFi. En effet, on peut proposer un « evil twin » à la victime pour l’attirer à se connecter à un faux réseau qui nous permette de capturer le processus d’authentification interne. Une attaque par brute-force peut être faite sur cette capture, beaucoup plus vulnérable d’être craquée qu’une clé WPA à 256 bits, car elle est effectuée sur le compte d’un utilisateur.

Pour faire fonctionner cette attaque, il est impératif que la victime soit configurée pour ignorer les problèmes de certificats ou que l’utilisateur accepte un nouveau certificat lors d’une connexion.

Pour implémenter l’attaque :

- Installer ```hostapd-wpe``` (il existe des versions modifiées qui peuvent peut-être faciliter la tâche... je ne les connais pas. Dans le doute, utiliser la version originale). Lire la documentation du site de l’outil ou d’autres ressources sur Internet pour comprendre son utilisation
- Modifier la configuration de ```hostapd-wpe``` pour proposer un réseau semblable au réseau de l’école ou le réseau de votre préférence, sachant que dans le cas d'une attaque réelle, il faudrait utiliser le vrai SSI du réseau de la cible
- Lancer une capture Wireshark
- Tenter une connexion au réseau (ne pas utiliser vos identifiants réels)
- Utiliser un outil de brute-force (```john```, par exemple) pour attaquer le hash capturé (utiliser un mot de passe assez simple pour minimiser le temps)

### Répondez aux questions suivantes :

> **_Question :_** Quelles modifications sont nécessaires dans la configuration de hostapd-wpe pour cette attaque ?
> 
> **_Réponse :_** 

---

> **_Question:_** Quel type de hash doit-on indiquer à john pour craquer le handshake ?
> 
> **_Réponse:_** 

---

> **_Question:_** Quelles méthodes d’authentification sont supportées par hostapd-wpe ?
> 
> **_Réponse:_**


### 3. (__Optionnel__) GTC Downgrade Attack avec [EAPHammer](https://github.com/s0lst1c3/eaphammer) 

[EAPHammer](https://github.com/s0lst1c3/eaphammer) est un outil de nouvelle génération pour les attaques WPA Entreprise. Il peut en particulier faire une attaque de downgrade GTC, pour tenter de capturer les identifiants du client en clair, ce qui évite le besoin de l'attaque par dictionnaire.

- Installer ```EAPHammer```. Lire la documentation du site de l’outil ou d’autres ressources sur Internet pour comprendre son utilisation
- Modifier la configuration de ```EAPHammer``` pour proposer un réseau semblable au réseau de l’école ou le réseau de votre préférence. Le but est de réaliser une GTC Downgrade Attack.
- Lancer une capture Wireshark
- Tenter une connexion au réseau


### Répondez aux questions suivantes :

> **_Question :_** Expliquez en quelques mots l'attaque GTC Downgrade
> 
> **_Réponse :_** 

---

> **_Question:_** Quelles sont vos conclusions et réflexions par rapport à la méthode hostapd-wpe ?
> 
> **_Réponse:_** 


## Livrables

Un fork du repo original . Puis, un Pull Request contenant :

-	Captures d’écran + commentaires
-	Réponses aux questions

## Échéance

Le 1 juin 2020 à 23h59