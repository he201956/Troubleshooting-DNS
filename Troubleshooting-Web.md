# < Titre du travail >

- Auteur(s) :  Victor Somers
- Date : 23/12/2025
- Usage des IAGs : /



## 1. Bug Report

Un bug report permet à l'utilisateur de documenter un problème afin que ceux qui s'occuperont de le résoudre disposent de suffisamment d'informations pour pouvoir reproduire le bug dans leur propre environnement de test.  


Il doit donc contenir les informations suivantes : 

- Description du système sur lequel le bug s'est produit (quel OS dans quelle version, quel logiciel dans quelle version, etc.). 
- Les circonstances dans lesquelles le bug s'est produit (quelles étapes pour arriver au bug?  Quels inputs?)
- Le bug observé : description du résultat obtenu par rapport au résultat normalement attendu.

- En démarrant le serveur www, je constate qu'il y a une erreur de configuration de apache2.
- Il m'indique aussi que l'essais de démarrer le serveur www a échoué.
- Il me conseil de consulter le fichier error log de apache pour avoir plus d'information su la panne.
<img width="847" height="305" alt="Capture d&#39;écran 2025-12-23 185454" src="https://github.com/user-attachments/assets/8fc0f25e-59ea-45f8-a09c-7c1e1859dd18" />


- Lorsque je fais un links http://blog.woodytoys.lab/ une erreur s'affiche et je n'arrive pas à accéder au site web principal.
<img width="927" height="130" alt="Capture d&#39;écran 2025-12-23 192102" src="https://github.com/user-attachments/assets/af5ce847-1e79-44a9-ad5c-cd9baae6c4f8" />
<img width="1084" height="508" alt="Capture d&#39;écran 2025-12-23 192112" src="https://github.com/user-attachments/assets/01381994-f8e1-46d4-b51e-3e8737e8e5cc" />


 
## 2. Collecte des symptômes

Après la description du bug vient la phase d'enquête.  Vous devez récolter un maximum d'indices pour faire la lumière sur le problème rencontré.  Pour cela, à vous d'utiliser les bons outils ! 

Dans ce rapport, pour chaque outil utilisé : 

- Nommez-le
- Indiquez sur quelle machine (ou lien) vous l'utilisez, et avec quels paramètres 
- Indiquez quel serait le résultat que vous obtiendriez en situation normale (= _output attendu_)
- Donnez l'_output obtenu_ (screenshot ou output textuel selon les circonstances)
- Indiquez ce que vous déduisez de cet output (comparaison output attendu/output obtenu + analyse) 

### Liste des outils :

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
