# Troobleshooting DNS (situation 1)

- Auteur(s) :  Victor Somers
- Date : 27/11/2025
- Usage des IAGs : 

Note : Si vous utilisez une IAG pour autre chose que la correction orthographique ou la formulation (ex : rédaction, aide à la résolution de problème, ...), vous devez insérer à la fin de votre rapport un compte-rendu de cet usage (ex : prompts + réponses).  


## 1. Bug Report

Il y a un bug lorsque de la machine clients 'direction' je réalise un ping vers www.google.com. Effectivement
je reçois le message suivant:

<img width="809" height="576" alt="Capture d&#39;écran 2025-11-27 173637" src="https://github.com/user-attachments/assets/f65209a8-7c10-4ef7-a5f3-4e94c1af9b59" />

Alors que si le serveur DNS étatit bien configuré, la requête ping devrait fonctionner.


## 2. Collecte des symptômes

En regardant la trace wireshark de cette échange je constate bien qu'il y a un refus du resolver lors de la requête dns du client 'direction'.

<img width="1853" height="899" alt="Capture d&#39;écran 2025-11-27 174112" src="https://github.com/user-attachments/assets/fdc70e26-b64b-4602-97a3-f5d4ce474f10" />

<img width="1024" height="455" alt="Capture d&#39;écran 2025-11-27 174521" src="https://github.com/user-attachments/assets/21e77196-4b1b-4710-aaff-182e929b7900" />


### Liste des outils :

Comme outils j'ai utilisé wireshark et ping.

## 3. Identification et description du problème 

Sur base de mes indices, je pense que le problème vient du fichier named du resolver qui serait mal configuré.
- Mon bug est que la requête ping venant du client 'direction' pour joindre www.google.com ne fonctionne pas.
- Mon indice est que sur la trace wireshark, je constate que le resolver dns refuse ma requête.
- Hypothèse : je pense que le fichier named du resolver est mal configuré.

 

## 4. Proposition de solution 
 
<img width="1205" height="737" alt="Capture d&#39;écran 2025-11-27 175306" src="https://github.com/user-attachments/assets/9ce7e06e-025e-42c1-bdb7-8eea8cdb21af" />

- Je pense que un des problèmes vient du options. Où il y a un allow-recursion qui est sur le network 192.168.1.0/24 alors que le network global est le 192.168.0.0/24.
- Je pense aussi qu'il y a peut-être un  problème dans les zones avec les type.

--> Problème pas résolu
