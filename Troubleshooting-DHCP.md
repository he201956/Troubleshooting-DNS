# Troubleshooting DHCP

- Auteur(s) :  V. Van den Schrieck
- Date : 12 novembre 2025
- Usage des IAG : Aucun. 


## 1. Bug Report

- Retour utilisateurs : des soucis de "consultation d'Internet" ont été signalés.  Les rapports d'intervention indiquent que le dernier changement effectué dans l'infrastructure concerne le service DHCP.  

- Le bug est reproductible en utilisant la simulation GNS3 fournie.  Pour cela : 
    - Démarrer le logiciel GNS3 la VM fournie, en s'assurant que cette dernière dispose d'une connectivité Internet. 
    - Démarrer les deux postes clients et les deux serveurs.  
    - Sur le serveur DHCP : lancer la commande "dhcpd -d" dans le répertoire /etc/dhcp/. 
    - Sur le résolveur DNS : lancer la commande "named -g" dans le répertoire /etc/bind/.  

Voici le schéma représentant l'infrastructure analysée, avec son adressage IP : 
<img width="811" height="547" alt="Capture d’écran 2025-11-12 à 21 10 30" src="https://github.com/user-attachments/assets/6d31dbda-eaa3-4d72-ba8a-4f146cafbffc" />

- Pour mettre en évidence le bug, ouvrir une console sur le poste "direction" et simuler une navigation Web via un "ping www.google.com" : Le ping échoue.   

## 2. Collecte des symptômes

Pour les symptômes, nous allons procéder couche par couche : Couche réseau, couche transport puis couche applicative.  

### Connectivité réseau

Pour vérifier la connectivité Internet, on effectue un ping vers un hôte distant. 

=> Résultat attendu : Réception de messages ICMP Echo Reply.  

=> Résultat obtenu : La machine distante (IP 8.8.8.8) a bien répondu aux sondes Ping, cfr ci-dessous. 

```root@direction:/# ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=127 time=31.0 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=127 time=18.3 ms
^C
--- 8.8.8.8 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1003ms
rtt min/avg/max/mdev = 18.282/24.620/30.958/6.338 ms
```

**Conclusion : les postes client disposent bien d'une connectivité IP.**

### Couche Transport : 

Comme nous soupçonnons que le problème se situe au niveau du DHCP, on va s'assurer que le serveur DHCP répond bien au niveau de la couche transport.  Pour cela : 
- On vérifie sur le serveur si le logiciel `dhcpd` est bien à l'écoute sur le port UDP 67, avec la commande `netstat` : 

```
root@dhcp:/etc/dhcp# netstat -nltpu
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
udp        0      0 0.0.0.0:67              0.0.0.0:*                           527/dhcpd         
```

Le serveur DHCP (processus `dhcpd` de pid 527) est bien à l'écoute sur le port UDP 67, sur toutes ses interfaces (Local Address 0.0.0.0:67).  Il accepte les requêtes depuis toutes les adresses externes (0.0.0.0:*).  Il devrait donc être joignable par les postes clients.    

- On peut également observer, sur une trace Wireshark, si la communication UDP s'effectue correctement entre client et serveur.  Pour cela, on lance une capture sur le lien vers le serveur DHCP avant de démarrer un client, et on observe si un échange DHCP (DORA) a bien lieu.  Voici ce qui est obtenu : 
<img width="1016" height="124" alt="Capture d’écran 2025-11-12 à 21 12 25" src="https://github.com/user-attachments/assets/fed2b5af-e945-4cd7-a1ba-43731f466ded" />

Les échanges DHCP sont visibles, ce qui indique que : 
- Le processus DHCP est bien démarré (nous le savions déjà via le netstat)
- Le serveur DHCP est accessible en UDP sur le port 67 (nous le savions déjà via le netstat)
- Puisque un échange DORA complet est effectué, la communication transport fonctionne bien.  

**Conclusion : la couche transport semble fonctionner comme attendu.**

### Couche applicative

Puisque les couches 3 et 4 semblent fonctionnelles, le problème est probablement au niveau applicatif.  Puisque la connectivité IP fonctionne, l'échec du ping `www.google.com` doit avoir une autre cause.  Reprenons l'output de cette commande : 
```
root@direction:/# ping www.google.com
ping: www.google.com: Temporary failure in name resolution
```
#### Vérification du fonctionnement du résolveur DNS
Le message d'erreur parle de panne dans la résolution de noms.  Nous pouvons soupçonner qu'il y a un problème au niveau du DNS.  Dans ce labo, le DNS est assuré par le résolveur. Pour s'assurer qu'il fonctionne, nous pouvons le tester en faisant une requête via dig directement vers lui, depuis le client :

```
root@direction:/# dig @192.168.0.2 www.google.com

; <<>> DiG 9.18.28-0ubuntu0.20.04.1-Ubuntu <<>> @192.168.0.2 www.google.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 35963
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: a32452cde5c78512010000006914ec0e854c64d6d46c32f3 (good)
;; QUESTION SECTION:
;www.google.com.			IN	A

;; ANSWER SECTION:
www.google.com.		300	IN	A	142.251.39.132

;; Query time: 188 msec
;; SERVER: 192.168.0.2#53(192.168.0.2) (UDP)
;; WHEN: Wed Nov 12 20:20:30 UTC 2025
;; MSG SIZE  rcvd: 87
```
**Conclusion : Le résolveur a bien répondu en donnant l'adresse IP demandée.  Il semble donc fonctionner correctement.**

#### Vérification de la configuration du client

Pour comprendre pourquoi notre poste client ne sait pas obtenir de réponse DNS alors que le résolveur est pourtant fonctionnel, nous pouvons observer s'il y a des requêtes DNS qui sortent de ce poste client.  Une capture sur le lien entre le switch et le poste de direction nous donne le résultat suivant, après application d'un filtre sur le trafic DNS : 
<img width="1173" height="274" alt="Capture d’écran 2025-11-12 à 21 24 46" src="https://github.com/user-attachments/assets/0f94e72f-18e6-4d5d-9a29-4c57193f62c6" />
Cette capture nous montre : 
- des requêtes DNS sortantes pour demander l'adresse IP de www.google.com (RR de type A)
- aucune réponse DNS en retour
- une réponse ICMP Port Unreachable, relative aux requêtes DNS. 

On en déduit que le serveur ayant reçu la requête DNS n'a pas de processus à l'écoute sur le port TCP/UDP 53.  Or, le résolveur l'était bel est bien, nous avons pu l'observer plus haut grâce à l'outil ```dig```! 

En observant les IPs, on remarque cependant que l'IP de destination n'est pas celle du résolveur : la requête est envoyée à l'adresse 192.168.0.1 au lieu de 192.168.0.2 ! Le client contacte donc le mauvais serveur pour la résolution de noms.  Il est possible de vérifier cette hypothèse en allant voir quelle est l'adresse du résolveur configuré pour le poste de directeur : 

```
root@direction:/# cat /etc/resolv.conf
search woodytoys.lab
nameserver 192.168.0.1
```
Le fichier contient bien l'adresse identifiée dans la trace Wireshark.  

**Conclusion : Le client est configuré avec une mauvaise adresse pour le résolveur. Cette adresse est normalement fournie par le serveur DHCP : c'est probablement lui qui est à l'origine de l'erreur.**

#### Vérification des options envoyées par le serveur DHCP

Pour vérifier les informations envoyées par le DHCP, ré-examinons la trace Wireshark avec l'échange DHCP DORA, et plus particulièrement le datagramme contenant le DHCP ACK : c'est lui qui contient les informations optionnelles telles que l'IP du résolveur DHCP.  

<img width="608" height="527" alt="Capture d’écran 2025-11-12 à 21 31 14" src="https://github.com/user-attachments/assets/bfc9f1ee-77d3-4409-a7c3-b3ca7374913c" />
On constate effectivement que le serveur DHCP a bien annoncé l'adresse IP 192.168.0.1, qui est sa propre IP et non celle du résolveur. 

**Conclusion : Le serveur DHCP envoie une mauvaise IP pour le résolveur DNS.**


## 3. Identification et description du problème 

Faisons une synthèse des étapes de l'analyse et leurs conclusions : 
- La connectivité réseau et le fonctionnement au niveau de la couche transport sont correct : le problème est à la couche applicative.  
- L'output du ```ping www.google.com``` nous indique un problème de résolution de noms.  Or, nous avons vérifié que le résolveur fonctionne correctement
- Puisque le résolveur fonctionne, nous avons regardé le client de plus près : une trace Wireshark nous a indiqué qu'il contacte le mauvais résolveur.  Or, l'IP du résolveur lui est fournie par le serveur DHCP
- En analysant l'échange DORA, l'hypothèse d'une information erronée transmise par le DHCP concernant le résolveur est confirmée. 
  
En consultant le fichier de configuration ```/etc/dhcp/dhcpd.conf```, on trouve ceci : 

```
authoritative;
default-lease-time 7200;
max-lease-time 7200;
subnet 192.168.0.0 netmask 255.255.255.0 {
        option routers 192.168.0.254;
        option subnet-mask 255.255.255.0;
        range 192.168.0.10 192.168.0.100;
        option domain-name "woodytoys.lab";
        option domain-name-servers 192.168.0.1; #MAUVAISE ADRESSE IP ! 
}
```
La dernière ligne concerne l'IP du résolveur, et indique effectivement une adresse erronée.  L'origine du bug est donc identifiée. 


## 4. Proposition de solution 


Le problème peut se solutionner rapidement en remplaçant cette adresse par celle du résolveur (192.168.0.2) et en redémarrant le serveur.  

Pour vérifier que le bug est bien corrigé, il suffit de reproduire le ping qui mettait le problème en évidence : 
```
root@direction:/# ping www.google.com
PING www.google.com (142.251.39.132) 56(84) bytes of data.
64 bytes from 142.251.39.132: icmp_seq=1 ttl=127 time=18.3 ms
64 bytes from 142.251.39.132: icmp_seq=2 ttl=127 time=28.4 ms
^C
--- www.google.com ping statistics ---
3 packets transmitted, 2 received, 33.3333% packet loss, time 2001ms
rtt min/avg/max/mdev = 18.287/23.332/28.377/5.045 ms

``` 
On peut cette fois constater que l'adresse `www.google.com` est bien résolue en IP, et le ping passe.  Les utilisateurs vont pouvoir à nouveau naviguer sur Internet.  