# Troobleshooting DNS (situation 1)

- Auteur(s) :  Victor Somers
- Date : 27/11/2025
- Usage des IAGs : / 


## 1. Bug Report

Il y a un bug lorsque de la machine client 'direction' je réalise un ping vers www.google.com. Effectivement
je reçois le message suivant:

<img width="809" height="576" alt="Capture d&#39;écran 2025-11-27 173637" src="https://github.com/user-attachments/assets/f65209a8-7c10-4ef7-a5f3-4e94c1af9b59" />

Alors que si le serveur DNS étatit bien configuré, la requête ping devrait fonctionner.


## 2. Collecte des symptômes

En regardant la trace wireshark de cette échange je constate bien qu'il y a un refus du resolver lors de la requête dns du client 'direction'.

<img width="1853" height="899" alt="Capture d&#39;écran 2025-11-27 174112" src="https://github.com/user-attachments/assets/fdc70e26-b64b-4602-97a3-f5d4ce474f10" />

<img width="1024" height="455" alt="Capture d&#39;écran 2025-11-27 174521" src="https://github.com/user-attachments/assets/21e77196-4b1b-4710-aaff-182e929b7900" />





Effectivement quand je réalise un dig depuis la machine client 'direction' vers l'adresse ip du resolver (192.168.0.2), je me rend compte que je reçois un refus.





<img width="1199" height="626" alt="Capture d&#39;écran 2025-12-23 174545" src="https://github.com/user-attachments/assets/d756e90f-399b-4872-a671-ea7b42f7b11c" />



### Liste des outils :

Comme outils j'ai utilisé wireshark, dig et ping.

## 3. Identification et description du problème 

Sur base de mes indices, je pense que le problème vient du fichier named du resolver qui serait mal configuré.
- Mon bug est que la requête ping venant du client 'direction' pour joindre www.google.com ne fonctionne pas.
- Mon indice est que sur la trace wireshark, je constate que le resolver dns refuse ma requête.
- Un autre indice est que lorsque je réalise un dig depuis la machine client 'direction' vers l'ip du resolver (192.168.0.2), je constate que je reçois un refus à ma requête avec comme warning : "recursion requested but not available".
- Hypothèse : je pense que le fichier named du resolver est mal configuré au niveau des recursions.

 

## 4. Proposition de solution 
 
<img width="1205" height="737" alt="Capture d&#39;écran 2025-11-27 175306" src="https://github.com/user-attachments/assets/9ce7e06e-025e-42c1-bdb7-8eea8cdb21af" />

- Je constate qu'il y a bien une erreur au niveau du allow-recursion, effectivement l'ip du network est 192.168.0.0/24 et ici il est indiqué 192.168.1.0/24.

  
<img width="1389" height="839" alt="Capture d&#39;écran 2025-12-23 175420" src="https://github.com/user-attachments/assets/bfeb5961-2dc2-4341-93d0-f2d73149997e" />

- Voici le fichier named.conf avec la modification au niveau du allow-recursion.

  
<img width="639" height="320" alt="Capture d&#39;écran 2025-12-23 175543" src="https://github.com/user-attachments/assets/ce079390-f5c8-42cf-8944-d3109303214b" />

- Ensuite je réalise un reload afin de relancer le serveur et de pouvoir tester si la modification a résolu le problème.


  
<img width="1079" height="356" alt="Capture d&#39;écran 2025-12-23 181131" src="https://github.com/user-attachments/assets/2740a188-1d58-47d5-a89f-3410ec770ae5" />
- Je constate que mon dig depuis la machine client 'direction' vers l'ip du resolver (192.168.0.2) fonctionne bien et que maintenant le status est "NOERROR".




<img width="1404" height="205" alt="Capture d&#39;écran 2025-12-23 181407" src="https://github.com/user-attachments/assets/c1592136-ae38-412f-bea4-e39253788c63" />
- Je constate aussi que le ping www.google.com fonctionne parfaitement.


