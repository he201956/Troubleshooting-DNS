# Trobleshooting Web (situation 1)

- Auteur(s) :  Victor Somers
- Date : 24/12/2025
- Usage des IAGs : /

## 1. Bug Report


  Situation : "L'entreprise Woodytoys vous contacte car son serveur web semble capricieux depuis la dernière maintenance, effectuée par un stagiaire.  Le serveur gère deux sites : le site www.woodytoys.lab et le site blog.woodytoys.lab.  Actuellement, l'adresse blog.woodytoys.lab affiche le site web principal au lieu du blog lui-même! . Pourriez-vous jeter un coup d'œil afin de trouver une solution au problème?"

- Effectivement lorsque je réalise un links http://blog.woodytoys.lab j'arrive sur le site web principal (www) et pas sur le blog.
  <img width="1025" height="118" alt="Capture d&#39;écran 2025-12-24 123007" src="https://github.com/user-attachments/assets/de0e4c41-8284-41da-870d-711e0bc4d009" />
<img width="1746" height="678" alt="Capture d&#39;écran 2025-12-24 122947" src="https://github.com/user-attachments/assets/4fb2faa5-8a78-4f9a-818b-140faa23fceb" />

  
 
## 2. Collecte des symptômes


- Je réalise un ping depuis la machine client 'directeur' vers blog.woodytoys.lab et je constate que le ping passe bien, ce qui était attendu.
<img width="1256" height="332" alt="Capture d&#39;écran 2025-12-24 123756" src="https://github.com/user-attachments/assets/78efbbb3-3b19-4a73-b8dc-e45fb89c6e76" />

- Je réalise un dig depuis la machine client 'directeur' vers blog.woodytoys.lab et je constate qu'il n'y a pas d'erreur, ce qui était attendu.
<img width="1110" height="793" alt="Capture d&#39;écran 2025-12-24 123730" src="https://github.com/user-attachments/assets/82e1104e-fb40-4b7f-9faf-6c5e3cc5c06a" />

- Je réalise une trace wireshark entre la machine client 'directeur' et le switch. Je constate qu'il n'y a pas d'erreur au niveau de ma requête dig http://blog.woodytoys.lab mais je constate que la réponse du serveur est bien le contenu de la page www et non le contenu du blog.
<img width="1814" height="817" alt="Capture d&#39;écran 2025-12-24 124627" src="https://github.com/user-attachments/assets/d196a2c9-9c76-4dfb-8db2-45452bfd1d67" />

- Sur le serveur www, je réalise un configtest pour voir si le problème vient de la syntax d'apache mais tout semble ok.
<img width="583" height="274" alt="Capture d&#39;écran 2025-12-24 124942" src="https://github.com/user-attachments/assets/d02738cf-8a18-458c-af6f-863ed402af08" />

- Sur le serveur, je réalise un apache2ctl -S et je constate que le virtualhost du blog est présent à côté de celui de www mais qu'il est sur le port 8000.
<img width="1510" height="591" alt="Capture d&#39;écran 2025-12-24 125408" src="https://github.com/user-attachments/assets/cb05658e-4d2e-43de-ba0e-04721b4d8aa1" />

- Sur le serveur, je réalise un netstat et je constate aussi que le blog écoute sur le port 8000 et le www sur le port 80.
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


## 3. Identification et description du problème 
  
* Mon bug est que depuis les machines clients, lorsque j'essaie de joindre l'adresse blog.woodytoys.lab via un links, j'obtiens le contenu de l'adresse www.
* Après investiguations je me rends compte qu'il n'y a pas de problèmes de connexions entre les machines clients et le serveur www et que sur le serveur www tout semble fonctionner normalement sauf le fait que le blog écoute sur le port 8000.
* Hypothèse : je pense qu'il doit y avoir un problème au niveau de la configuration du port du virtualhost du blog dans le fichier blog-woodytoys-lab.conf. Et que donc il est impossible de transmettre le fichier index.html du blog, donc le serveur transmet le fichier index.html de www par défault.

  

## 4. Proposition de solution 

- Je constate dans le fichier blog-woodytoys-lab.conf que le VirutalHost est bien sur le port 8000 et non sur le port 80 alors que mes requêtes links http://blog.woodytoys.lab/ sont dirigées par défaut sur le port 80.
<img width="1385" height="854" alt="Capture d&#39;écran 2025-12-27 123418" src="https://github.com/user-attachments/assets/615e1dbd-fa07-4baa-8fb9-360d072ff8c9" />


- Je décide donc de modifier le port du VirtualHost de blog-woodytoys-lab.conf qui est 8000 en 80.
  <img width="1813" height="884" alt="Capture d&#39;écran 2025-12-27 124154" src="https://github.com/user-attachments/assets/2ded0f61-4712-4844-83f8-801ffc5ddc99" />

- Je vérifie que le VirtualHost de blog-woodytoys-lab.conf est bien assigné au port 80 comme celui de www. Ensuite je vérifie si tout est ok au niveau syntax avec apache2ctl configtest et je restart apache.
  <img width="1709" height="705" alt="Capture d&#39;écran 2025-12-27 123538" src="https://github.com/user-attachments/assets/24f0a27b-4b3c-4fd0-8aed-2a5adbc226d4" />

- Pour finir, je réalise ma requête links http://blog.woodytoys.lab/ depuis la machine client 'directeur' et je constate que la requête fonctionne après cette modification.
  <img width="1212" height="162" alt="Capture d&#39;écran 2025-12-27 124608" src="https://github.com/user-attachments/assets/fdd97ebe-1d92-40b6-9e0b-1a544195ef48" />
<img width="1667" height="735" alt="Capture d&#39;écran 2025-12-27 124619" src="https://github.com/user-attachments/assets/1b1788d9-415c-4dbe-b514-eb1aa041ed89" />

