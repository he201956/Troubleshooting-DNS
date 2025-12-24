# Trobleshooting Web (situation 1)

- Auteur(s) :  Victor Somers
- Date : 24/12/2025
- Usage des IAGs : /

## 1. Bug Report

Un bug report permet à l'utilisateur de documenter un problème afin que ceux qui s'occuperont de le résoudre disposent de suffisamment d'informations pour pouvoir reproduire le bug dans leur propre environnement de test.  


Il doit donc contenir les informations suivantes : 

- Description du système sur lequel le bug s'est produit (quel OS dans quelle version, quel logiciel dans quelle version, etc.). 
- Les circonstances dans lesquelles le bug s'est produit (quelles étapes pour arriver au bug?  Quels inputs?)
- Le bug observé : description du résultat obtenu par rapport au résultat normalement attendu.









  Situation : "L'entreprise Woodytoys vous contacte car son serveur web semble capricieux depuis la dernière maintenance, effectuée par un stagiaire.  Le serveur gère deux sites : le site www.woodytoys.lab et le site blog.woodytoys.lab.  Actuellement, l'adresse blog.woodytoys.lab affiche le site web principal au lieu du blog lui-même! . Pourriez-vous jeter un coup d'œil afin de trouver une solution au problème?"

- Effectivement lorsque je réalise un links http://blog.woodytoys.lab j'arrive sur le site web principal (wwww) et pas sur le blog.
  <img width="1025" height="118" alt="Capture d&#39;écran 2025-12-24 123007" src="https://github.com/user-attachments/assets/de0e4c41-8284-41da-870d-711e0bc4d009" />
<img width="1746" height="678" alt="Capture d&#39;écran 2025-12-24 122947" src="https://github.com/user-attachments/assets/4fb2faa5-8a78-4f9a-818b-140faa23fceb" />

  
 
## 2. Collecte des symptômes

Après la description du bug vient la phase d'enquête.  Vous devez récolter un maximum d'indices pour faire la lumière sur le problème rencontré.  Pour cela, à vous d'utiliser les bons outils ! 

Dans ce rapport, pour chaque outil utilisé : 

- Nommez-le
- Indiquez sur quelle machine (ou lien) vous l'utilisez, et avec quels paramètres 
- Indiquez quel serait le résultat que vous obtiendriez en situation normale (= _output attendu_)
- Donnez l'_output obtenu_ (screenshot ou output textuel selon les circonstances)
- Indiquez ce que vous déduisez de cet output (comparaison output attendu/output obtenu + analyse)


- Je réalise un ping depuis la machine client 'direction' vers blog.woodytoys.lab et je constate que le ping passe bien, ce qui était attendu.
<img width="1256" height="332" alt="Capture d&#39;écran 2025-12-24 123756" src="https://github.com/user-attachments/assets/78efbbb3-3b19-4a73-b8dc-e45fb89c6e76" />

- Je réalise un dig depuis la machine client 'direction' vers blog.woodytoys.lab et je constate qu'il n'y a pas d'erreur, ce qui était attendu.
<img width="1110" height="793" alt="Capture d&#39;écran 2025-12-24 123730" src="https://github.com/user-attachments/assets/82e1104e-fb40-4b7f-9faf-6c5e3cc5c06a" />

- Je réalise une trace wireshark entre la machine client 'direction' et le switch. Je constate qu'il n'y a pas d'erreur au niveau de ma requête dig http://blog.woodytoys.lab mais je constate que la réponse du serveur est bien le contenu de la page wwww et non le contenu du blog.
<img width="1814" height="817" alt="Capture d&#39;écran 2025-12-24 124627" src="https://github.com/user-attachments/assets/d196a2c9-9c76-4dfb-8db2-45452bfd1d67" />

- Sur le serveur wwww, je réalise un configtest pour voir si le problème vient d'apache mais tout semble ok.
<img width="583" height="274" alt="Capture d&#39;écran 2025-12-24 124942" src="https://github.com/user-attachments/assets/d02738cf-8a18-458c-af6f-863ed402af08" />

- Sur le serveur, je réalise un apache2ctl -S et je constate que que le virtualhost du blog est bien présent à côté de celui de www.
<img width="1510" height="591" alt="Capture d&#39;écran 2025-12-24 125408" src="https://github.com/user-attachments/assets/cb05658e-4d2e-43de-ba0e-04721b4d8aa1" />

- Sur le serveur, je réalise un netstat et je constate que les ports 80 et 8000 sont bien en listen pour apache.
<img width="1532" height="256" alt="Capture d&#39;écran 2025-12-24 125740" src="https://github.com/user-attachments/assets/feb73354-11cf-4c68-81ce-336572f40eb5" />

- Sur le serveur, je constate que malgré mes requêtes links vers blog.woodytoys.lab il n'y a rien dans les fichiers acces.log et error.log du blog. Mais je constate que toutes mes requêtes se retrouvent dans le fichiers access.log de wwww et rien dans error.log de www.
<img width="1804" height="141" alt="Capture d&#39;écran 2025-12-24 130523" src="https://github.com/user-attachments/assets/f7920312-b5ec-4121-975d-39f1992b5870" />






### Liste des outils :
Comme outils j'ai utilisé : 
- Wireshark
- netstat
- dig
- ping
- les logs des services wwww et blog
- apache2ctl -S
- apache2ctl configtest









Dans le cadre de ce cours, vous devez autant que possible utiliser systématiquement :
- Wireshark,
- netstat,
- les logs du/des services concernés 

Selon les cas, vous pourriez aussi utiliser : 
- ping, 
- traceroute, 
- ip a, 
- dig/nslookup, 
- affichage de fichiers systèmes, 
- nmap, ... 

Note : Au stade actuel, vous **ne devez toujours pas** aller **examiner les fichiers de configuration**. Ce sont vos déductions qui vous indiqueront où regarder dans les configs.  

## 3. Identification et description du problème 

Sur base de vos indices, vous devriez à présent arriver à comprendre le problème.  

1. Décrivez et expliquez, sur base d'une synthèse des indices récoltés et des conclusions que vous en avez tiré, une hypothèse expliquant le bug observé. Il doit y avoir un lien clair entre les indices et l'hypothèse.

     
1. * Mon bug est que depuis les machines clients, lorsque j'essaie de joindre l'adresse blog.woodytoys.lab, j'obtiens le contenu de l'adresse wwww.
   * Après investiguations je me rends compte qu'il n'y a pas de problèmes de connexions entre les machines clients et le serveur wwww et que sur le serveur wwww tout semble fonctionner normalement.
   * Hypothèse : je pense qu'il doit y avoir un problème au niveau de la configuration du <Directory> du blog ou du <virtualhost> du blog dans le fichier blog-woodytoys-lab.conf. Et que donc il est impossible de transmettre le fichier index.html du blog, et que le serveur transmet le fichier index.html de www par défault.


2. 

Exemple fictif et non technique :  
   * Mon "bug" est la disparition du chocolat de l'armoire à friandise alors qu'il y avait une tablette complète une heure plus tôt.   
   * J'investigue et récolte un indice : mon fils a du chocolat sur le visage
   * Hypothèse : mon fils a mangé le chocolat.  
Ici, le lien entre l'indice observé et l'hypothèse est assez évident. 


2. Votre hypothèse devrait vous indiquer une erreur probable dans les configurations.  Allez à présent vérifier dans les fichiers de configuration si elle est correcte.  

## 4. Proposition de solution 
Une fois le problème identifié, il faut le corriger.  

- Expliquez les changements effectués pour rétablir l'état attendu du système. 
- Validez votre résolution :
  * donnez les commandes à utiliser pour prouver que le bug est résolu. 
  * montrer une preuve (screenshot) que c'est bien le cas
