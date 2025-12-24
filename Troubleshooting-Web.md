## Consignes

Ce template est destiné à vous aider à la rédaction d'un rapport de troubleshooting.  La structure proposée vise à mettre en avant le **raisonnement** réalisé, et pas tant le résultat obtenu.  

En effet, l'important, dans l'apprentissage du troubleshooting, est d'appliquer une procédure systématique d'investigation reposant sur la collecte d'indices et sur la mise en avant des liens entre ces indices pour arriver à l'identification du problème.  Un bug n'est pas résolu par la disparition du problème, mais bien par la compréhension profonde de ce qui s'est passé afin d'appliquer les corrections adéquates. Pour travailler spécifiquement cette démarche de recherche, nous vous recommandons fortement de **ne pas consulter les fichiers de configuration** avant d'avoir trouvé le bug.  Cela vous permettra de vous focaliser sur les outils qui vous permettront d'analyser et observer le système.  


Pour utiliser ce template, vous pouvez soit "forker" le repository, soit copier/coller le contenu de cette page.  Retirez bien entendu les consignes et les textes explicatifs avant de vous lancer dans la rédaction de votre rapport. 

Un exemple de rapport de troubleshooting est disponible [[ici|Troubleshooting DHCP]]
***
# < Titre du travail >

- Auteur(s) :  
- Date : 
- Usage des IAGs : 

Note : Si vous utilisez une IAG pour autre chose que la correction orthographique ou la formulation (ex : rédaction, aide à la résolution de problème, ...), vous devez insérer à la fin de votre rapport un compte-rendu de cet usage (ex : prompts + réponses).  


## 1. Bug Report

Un bug report permet à l'utilisateur de documenter un problème afin que ceux qui s'occuperont de le résoudre disposent de suffisamment d'informations pour pouvoir reproduire le bug dans leur propre environnement de test.  


Il doit donc contenir les informations suivantes : 

- Description du système sur lequel le bug s'est produit (quel OS dans quelle version, quel logiciel dans quelle version, etc.). 
- Les circonstances dans lesquelles le bug s'est produit (quelles étapes pour arriver au bug?  Quels inputs?)
- Le bug observé : description du résultat obtenu par rapport au résultat normalement attendu.  
 
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
