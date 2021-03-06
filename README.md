- [Livrables](https://github.com/arubinst/HEIGVD-SWI-Labo3-WPA-Entreprise#livrables)

- [Échéance](https://github.com/arubinst/HEIGVD-SWI-Labo3-WPA-Entreprise#échéance)

- [Quelques éléments à considérer](https://github.com/arubinst/HEIGVD-SWI-Labo3-WPA-Entreprise#quelques-éléments-à-considérer-)

- [Travail à réaliser](https://github.com/arubinst/HEIGVD-SWI-Labo3-WPA-Entreprise#travail-à-réaliser)

# Sécurité des réseaux sans fil

## Laboratoire 802.11 Sécurité WPA Entreprise

__A faire en équipes de deux personnes__

### Objectif :

1.	Analyser les étapes d’une connexion WPA Entreprise avec une capture Wireshark
2.	Implémenter une attaque WPE (Wireless Pwnage Edition) contre un réseau WPA Entreprise

__Il est fortement conseillé d'employer une distribution Kali__ (on ne pourra pas assurer le support avec d'autres distributions). __Si vous utilisez une VM, il vous faudra une interface WiFi usb, disponible sur demande__.

__ATTENTION :__ Il est __particulièrement important pour ce laboratoire__ de bien fixer le canal lors de vos captures et vos injections. Si vous en avez besoin, la méthode la plus sure est de lancer un terminal séparé, et d'ouvrir airodump-ng avec l'option :

```--channel```

Il faudra __garder la fenêtre d'airodump ouverte__ en permanence pendant que vos scripts tournent ou vos manipulations sont effectuées.

## Travail à réaliser

### 1. Capture et analyse d’une authentification WPA Entreprise

Dans cette première partie, vous allez capturer une connexion WPA Entreprise au réseau de l’école avec Wireshark et fournir des captures d’écran indiquant dans chaque capture les données demandées.

- Identifier le canal utilisé par l’AP dont la puissance est la plus élevée. Vous pouvez faire ceci avec ```airodump-ng```, par exemple
- Lancer une capture avec Wireshark
- Etablir une connexion depuis un poste de travail (PC), un smartphone ou une tablette. Attention, il est important que la connexion se fasse à 2.4 GHz pour pouvoir sniffer avec les interfaces Alfa.
- Comparer votre capture au processus d’authentification expliqué en classe (n’oubliez pas les captures !). En particulier, identifier les étapes suivantes :
	
	- Requête et réponse d’authentification système ouvert
	
	  Il s'agit des deux premiers paquets.
	
	  ![Open Request](img/openRequest.png)
	
	  ![Open Response](img/openResponse.png)
 	- Requête et réponse d’association
	- Sélection de la méthode d’authentification
	
	  La méthode d'authentification est demandée par l'AP en premier (paquets 3 à 5) :
	
	  ![authMethod](img/authMethod.png)
	
	  On peut voir que l'AP demande si on va se connecter avec EAP-TLS et le client répond par la négative dans le paquet suivant (numéro 4 sur l'image). En effet, en se connectant au wifi proposé par l'AP, le client choisit de se connecter avec EAP-PEAP donc il refuse l'autre méthode proposée par l'AP en précisant qu'il souhaite utiliser EAP-PEAP. L'AP propose donc EAP-PEAP et le client, satisfait, entame la phase de Hello et échange de nonces, mettant fin à la phase d'initiation.
	
	- Phase d’initiation. Arrivez-vous à voir l’identité du client ?
	
	  Oui, elle est présente dans le second paquet (cf. image n°2)
	
	- Phase hello :
		- Version TLS
		
		  Il s'agit de la version 1.2 (cf photo plus bas).
		
		- Suites cryptographiques et méthodes de compression proposées par le client et acceptées par l’AP
		
		  Ici les ciphersuites proposées par le client
		
		  ![ciphersuites](img/ciphersuites.png)
		
		  Et ci-après, celle choisie par le serveur :
		
		  ![serverHello](img/serverHello.png)
		
		- Nonces
		
		  Sur les captures d'écran ci-dessus, on peut voir les nonces qui sont les champs random.
		
		- Session ID
		
		  Le session ID est visible sur le printscreen du serveur Hello, juste au dessus. La valeur est à acb406e51958041ae8081e3ab3b18b86f0f83b9c553494f0db9c0c9568850fb1
		
	- Phase de transmission de certificats
	
	  - Certificat serveur
	
	    Les certificats sont transmis via les 3 paquets suivants : 7, 9 et 11 sur l'image ci-après.
	
	    ![certExchange](img/certExchange.png)
	
	    Il y a normalement un paquet pour transmettre le certificat TLS du serveur, un pour la clef du serveur et un « server hello done » pour indiquer que le serveur a fini. Wireshark nous regroupe ces 3 frames sur cette capture. On peut le voir via les 3 points à côté des paquets concernés dans la liste des paquets ou sur la première ligne du détail d'un paquet sur l'image : « 3 EAP-TLS Fragments ... #7, #9, #11 ».
	
	    Ces paquets sont entre-coupés de « ACK » du client
	
	  - Client key exchange
	
	    Le client envoie ensuite sa clef et annonce qu'il est prêt à passer en chiffré avec le « Change Cipher Spec »
	
	    ![clientKeyExchange](img/clientKeyExchange.png) 
	
	  - Change cipher spec
	
	    L'AP envoie ensuite un « Change Cipher Spec » pour annoncer qu'il est également prêt à transmettre de manière chiffrée. Le client envoie un « ACK » et on bascule sur une communication chiffrée : on ne voit que des « Application Data ».
	
	    ![changeCipherSpec](img/changeCipherSpec.png)
	
	- Authentification interne et transmission de la clé WPA (échange chiffré, vu comme « Application data »)
	![](img/appData.png)
	
	- 4-way hadshake
	![](img/4wayHandshake.png)

### Répondez aux questions suivantes :

> **_Question :_** Quelle ou quelles méthode(s) d’authentification est/sont proposé(s) au client ?
> 
> **_Réponse :_** EAP-TLS ainsi que PEAP (paquets 3 à 5)
![](img/authMethod.png)

---

> **_Question:_** Quelle méthode d’authentification est utilisée ?
> 
> **_Réponse:_** PEAP
![](img/peapChoosen.png)

---

> **_Question:_** Lors de l’échange de certificats entre le serveur d’authentification et le client :
> 
> - Le serveur envoie-t-il un certificat au client ? Pourquoi oui ou non ?
> 
> **_Réponse:_** Oui, le serveur envoie effectivement un certificat au client. C'est toujours le cas, peu importe la méthode d'authentification.
> 
> - b.	Le client envoie-t-il un certificat au serveur ? Pourquoi oui ou non ?
> 
> **_Réponse:_** Non, le client n'envoie pas de certificat au serveur. Il devrait le faire s'il s'authentifiait avec EAP-TLS mais ce n'est pas le cas. Dans ce cas, PEAP est utilisé. Seul le serveur présente donc son certificat.
> 

---

### 2. Attaque WPA Entreprise

Les réseaux utilisant une authentification WPA Entreprise sont considérés aujourd’hui comme étant très surs. En effet, puisque la Master Key utilisée pour la dérivation des clés WPA est générée de manière aléatoire dans le processus d’authentification, les attaques par dictionnaire ou brute-force utilisés sur WPA Personnel ne sont plus applicables. 

Il existe pourtant d’autres moyens pour attaquer les réseaux Entreprise, se basant sur une mauvaise configuration d’un client WiFi. En effet, on peut proposer un « evil twin » à la victime pour l’attirer à se connecter à un faux réseau qui nous permette de capturer le processus d’authentification interne. Une attaque par brute-force peut être faite sur cette capture, beaucoup plus vulnérable d’être craquée qu’une clé WPA à 256 bits, car elle est effectuée sur le compte d’un utilisateur.

Pour faire fonctionner cette attaque, il est impératif que la victime soit configurée pour ignorer les problèmes de certificats ou que l’utilisateur accepte un nouveau certificat lors d’une connexion.

Pour implémenter l’attaque :

- Installer ```hostapd-wpe``` (il existe des versions modifiées qui peuvent peut-être faciliter la tâche... je ne les connais pas. Dans le doute, utiliser la version originale). Lire la documentation du site de l’outil ou d’autres ressource sur Internet pour comprendre son utilisation
- Modifier la configuration de ```hostapd-wpe``` pour proposer un réseau semblable au réseau de l’école. Ça ne sera pas evident de vous connecter si vous utilisez le même nom HEIG-VD. L'option la plus sure c'est de proposer votre propre réseau avec un SSID comme par exemple, HEIG-VD-Faux (sachant que dans le cas d'une attaque réelle, il faudrait utiliser le vrai SSI) 
- Lancer une capture Wireshark
- Tenter une connexion au réseau (ne pas utiliser vos identifiants réels)
- Utiliser un outil de brute-force (```john```, par exemple) pour attaquer le hash capturé (utiliser un mot de passe assez petit pour minimiser le temps)

### Répondez aux questions suivantes :

> **_Question :_** 4. Quelles modifications sont nécessaires dans la configuration de hostapd-wpe pour cette attaque ?
> 
> **_Réponse :_** Il faut modifier le SSID du réseau, ainsi que le channel utilisé et le nom de l'interface utilisée.

---

> **_Question:_** 5. Quel type de hash doit-on indiquer à john pour craquer le handshake ?
> 
> **_Réponse:_** On voit que hostapd-wpe nous propose le challenge, la response ainsi qu'un hash prévu pour pour John et un autre prévu pour Hashcat. On peut donc lui fournir la ligne suivante : ``jtr NETNTLM:		abraham:$NETNTLM$891aa9bb12581837$7cda61ee18802a01b1c7ba5ca216e00a75c693bfa3a328ec``
![](img/hash1.png)
![](img/hash2.png)
![](img/john.png)

---

> **_Question:_** 6.	Quelles méthodes d’authentification sont supportées par hostapd-wpe ?
> 
> **_Réponse:_** Selon leur [documentation](https://github.com/OpenSecurityResearch/hostapd-wpe), Hostapd-wpe supporte les méthodes d'authentification suivantes :    
    1. EAP-FAST/MSCHAPv2 (Phase 0)    
    2. PEAP/MSCHAPv2    
    3. EAP-TTLS/MSCHAPv2    
    4. EAP-TTLS/MSCHAP    
    5. EAP-TTLS/CHAP    
    6. EAP-TTLS/PAP    


## Quelques éléments à considérer :

- Solution à l’erreur éventuelle « ```Could not configure driver mode``` » :

```
nmcli radio wifi off
rfkill unblock wlan
```
-	Pour pouvoir capturer une authentification complète, il faut se déconnecter d’un réseau et attendre 1 minute (timeout pour que l’AP « oublie » le client) 
-	Les échanges d’authentification entreprise peuvent être trouvés facilement utilisant le filtre d’affichage « ```eap``` » dans Wireshark


## Livrables

Un fork du repo original . Puis, un Pull Request contenant :

-	Captures d’écran + commentaires
-	Réponses aux questions


## Échéance

Le 2 juin 2019 à 23h00
