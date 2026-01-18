---
title: "Curse of the Pharaoh"
date: 2025-01-01
draft: false
categories: ["writeup"]
tags: ["ctf", "game"]
---
<html>

<head>
<meta http-equiv=Content-Type content="text/html; charset=utf-8">
<meta name=Generator content="Microsoft Word 15 (filtered)">
<style>
<!--
 /* Font Definitions */
 @font-face
	{font-family:Wingdings;
	panose-1:5 0 0 0 0 0 0 0 0 0;}
@font-face
	{font-family:"Cambria Math";
	panose-1:2 4 5 3 5 4 6 3 2 4;}
@font-face
	{font-family:"Segoe UI Emoji";
	panose-1:2 11 5 2 4 2 4 2 2 3;}
@font-face
	{font-family:Aptos;
	panose-1:2 11 0 4 2 2 2 2 2 4;}
@font-face
	{font-family:"Calisto MT";
	panose-1:2 4 6 3 5 5 5 3 3 4;}
 /* Style Definitions */
 p.MsoNormal, li.MsoNormal, div.MsoNormal
	{margin-top:0in;
	margin-right:0in;
	margin-bottom:8.0pt;
	margin-left:0in;
	line-height:115%;
	font-size:12.0pt;
	font-family:"Calisto MT",serif;}
h2
	{mso-style-link:"Heading 2 Char";
	margin-top:8.0pt;
	margin-right:0in;
	margin-bottom:4.0pt;
	margin-left:0in;
	line-height:115%;
	page-break-after:avoid;
	font-size:16.0pt;
	font-family:"Calisto MT",serif;
	color:#8C3314;
	font-weight:normal;}
p.MsoListParagraph, li.MsoListParagraph, div.MsoListParagraph
	{margin-top:0in;
	margin-right:0in;
	margin-bottom:8.0pt;
	margin-left:.5in;
	line-height:115%;
	font-size:12.0pt;
	font-family:"Calisto MT",serif;}
p.MsoListParagraphCxSpFirst, li.MsoListParagraphCxSpFirst, div.MsoListParagraphCxSpFirst
	{margin-top:0in;
	margin-right:0in;
	margin-bottom:0in;
	margin-left:.5in;
	line-height:115%;
	font-size:12.0pt;
	font-family:"Calisto MT",serif;}
p.MsoListParagraphCxSpMiddle, li.MsoListParagraphCxSpMiddle, div.MsoListParagraphCxSpMiddle
	{margin-top:0in;
	margin-right:0in;
	margin-bottom:0in;
	margin-left:.5in;
	line-height:115%;
	font-size:12.0pt;
	font-family:"Calisto MT",serif;}
p.MsoListParagraphCxSpLast, li.MsoListParagraphCxSpLast, div.MsoListParagraphCxSpLast
	{margin-top:0in;
	margin-right:0in;
	margin-bottom:8.0pt;
	margin-left:.5in;
	line-height:115%;
	font-size:12.0pt;
	font-family:"Calisto MT",serif;}
span.Heading2Char
	{mso-style-name:"Heading 2 Char";
	mso-style-link:"Heading 2";
	font-family:"Calisto MT",serif;
	color:#8C3314;}
.MsoChpDefault
	{font-size:12.0pt;
	font-family:"Calisto MT",serif;}
.MsoPapDefault
	{margin-bottom:8.0pt;
	line-height:115%;}
@page WordSection1
	{size:8.5in 11.0in;
	margin:1.0in 1.25in 1.0in 1.25in;}
div.WordSection1
	{page:WordSection1;}
 /* List Definitions */
 ol
	{margin-bottom:0in;}
ul
	{margin-bottom:0in;}
-->
</style>

</head>

<body bgcolor="#272727" lang=EN-US link="#E98052" vlink="#F4B69B"
style='word-wrap:break-word'>

<div class=WordSection1>

<h2><b><span style='font-size:20.0pt;line-height:115%'>Writeup – Curse of the
Pharaoh </span></b><b><span style='font-size:20.0pt;line-height:115%;
font-family:"Segoe UI Emoji",sans-serif'>&#127994;&#128511;&#128043;</span></b></h2>

<h2><span lang=FR>Curse of the Pharaoh 1</span><span lang=FR style='font-size:
12.0pt;line-height:115%;color:windowtext'> </span><span lang=FR>– <i>La boule
mystère</i></span></h2>

<p class=MsoNormal><span lang=FR>Description (français)</span></p>

<p class=MsoNormal><span lang=FR>C'est un départ pour l'Égypte, tu t'étais mis
en tête d'aller visiter les pyramides, mais dès que tu t'es approché de la
première des caractères étranges sont apparus sur ta peau, je me demande ce
qu'ils signifient...</span></p>

<p class=MsoNormal><span lang=FR>Voir l'indice</span></p>

<p class=MsoNormal><span lang=FR>Je me demande s'il y a une façon de regarder
les textures de plus près... / I wonder if there's a way to take a closer look
at the textures...</span></p>

<p class=MsoNormal><span lang=FR>&nbsp;</span></p>

<p class=MsoNormal><span lang=FR>Quand on lance le jeu, on découvre qu’on <b>incarne
une boule</b>.<br>
On peut se déplacer un peu dans le décor et, en observant notre surface, on
remarque qu’il y a bien quelque chose écrit dessus… mais impossible de le lire
correctement. <img id="Image 3"
src="images/image001.png"
alt="Une image contenant cercle, balle&#10;&#10;Le contenu généré par l’IA peut être incorrect."></span></p>

<p class=MsoNormal><span lang=FR>Pas besoin d’un doctorat en archéologie pour
deviner que le flag est caché dans cette texture.<br>
La question devient donc : comment voir cette texture <i>en dehors du jeu</i> ?</span></p>

<p class=MsoNormal><span lang=FR>Pour ça, on va utiliser <b>AssetRipper</b>.<br>
C’est un logiciel qui permet d’extraire toutes les ressources d’un jeu Unity
(textures, modèles 3D, sons, etc.).<br>
En gros, il va nous donner accès à ce que le jeu utilise en arrière-plan.</span></p>

<p class=MsoNormal><img
src="images/image002.png"
hspace=12><span lang=FR>Voici les étapes</span></p>

<p class=MsoNormal><span lang=FR>On lance AssetRipper.exe.Une page web devrait
alors s’ouvrir.</span></p>

<p class=MsoNormal><span lang=FR>En haut à gauche </span><span lang=FR
style='font-family:"Times New Roman",serif'>→</span><span lang=FR> <b>File </b></span><b><span
lang=FR style='font-family:"Times New Roman",serif'>→</span></b><b><span
lang=FR> Load Folder</span></b><span lang=FR> </span><span lang=FR
style='font-family:"Times New Roman",serif'>→</span><span lang=FR> on choisit
le dossier CurseOfThePharaoh.</span></p>

<p class=MsoNormal><span lang=FR>Ensuite </span><span lang=FR style='font-family:
"Times New Roman",serif'>→</span><span lang=FR> <b>Export </b></span><b><span
lang=FR style='font-family:"Times New Roman",serif'>→</span></b><b><span
lang=FR> Export All Files</span></b><span lang=FR>.</span></p>

<p class=MsoNormal><span lang=FR>On coche <i>Create subfolder</i> et on
sélectionne notre dossier de travail (par ex. CurseOfThePharaoh_Writeup).</span></p>

<p class=MsoNormal><span lang=FR>On clique sur <b>Primary Content</b> </span><span
lang=FR style='font-family:"Times New Roman",serif'>→</span><span lang=FR> et
on laisse tourner quelques secondes.</span></p>

<p class=MsoNormal><span lang=FR>AssetRipper génère alors un nouveau dossier
AssetRipper_export.</span></p>

<p class=MsoNormal><b><span lang=FR>Trouver le flag</span></b></p>

<p class=MsoNormal><img
src="images/image003.gif"
hspace=12
alt="Une image contenant capture d’écran, Caractère coloré, pixel, Graphique&#10;&#10;Le contenu généré par l’IA peut être incorrect."><span
lang=FR>On ouvre ce dossier, on explore un peu et on se dirige vers :<br>
Assets </span><span lang=FR style='font-family:"Times New Roman",serif'>→</span><span
lang=FR> Texture2D.</span></p>

<p class=MsoNormal><span lang=FR>Et là, on tombe sur l’image qui avait sur
notre boule.<br>
En l’ouvrant, on voit enfin clairement le flag qui était écrit dessus.</span></p>

<p class=MsoNormal><span lang=FR>Premier flag trouvé, on est prêt à attaquer le
deuxième challenge.</span></p>

<h2>Curse of the Pharaoh 2</h2>

<p class=MsoNormal>Description (français)</p>

<p class=MsoNormal><span lang=FR>Les hiéroglyphes ont parlé, mais un second
message reste enfoui où seulement les créateurs de cet endroit peuvent le
trouver...</span></p>

<p class=MsoNormal><span lang=FR>Indice implicite : on va probablement devoir
regarder <b>le code du jeu</b>.</span></p>

<p class=MsoNormal><span lang=FR>En lisant l’énoncé, je me suis dit tout de
suite : “créateurs” = développeurs, donc il faut aller voir directement dans le
code du jeu. Comme le jeu est fait avec Unity et compilé en C#, on peut
utiliser un outil appelé <b>dnSpy</b>. C’est un décompilateur qui permet
d’ouvrir et de lire le contenu des fichiers .dll (les librairies compilées du
jeu) comme si on avait accès au code source.</span></p>

<p class=MsoNormal><img 
src="images/image004.png"
hspace=12
alt="Une image contenant texte, capture d’écran, affichage, ordinateur&#10;&#10;Le contenu généré par l’IA peut être incorrect."><span
lang=FR>Je lance donc <b>dnSpy</b>, je clique sur <i>File </i></span><i><span
lang=FR style='font-family:"Times New Roman",serif'>→</span></i><i><span
lang=FR> Open</span></i><span lang=FR> et j’ouvre le fichier <b>Assembly-CSharp.dll</b>,
qu’on retrouve dans le dossier CurseOfThePharaoh_Data/Managed. Une fois ouvert,
dnSpy affiche une vue en arbre avec tous les modules et classes du jeu. Les
parties en jaune attirent l’attention : ce sont souvent des endroits importants
à examiner. </span></p>

<p class=MsoNormal><span lang=FR>&nbsp;</span></p>

<p class=MsoNormal><span lang=FR>En parcourant les fichiers, je finis par
tomber sur une fonction qui s’appelle <b>FlagPrintSystem</b>. Le nom est assez
explicite, ça donne tout de suite envie d’aller voir ce qu’il y a dedans.<img
id="Image 7"
src="images/image005.png"
alt="Une image contenant texte, capture d’écran, logiciel, affichage&#10;&#10;Le contenu généré par l’IA peut être incorrect."></span></p>

<p class=MsoNormal><span lang=FR>Et effectivement, à l’intérieur de cette
fonction, on découvre une chaîne de caractères étrange :</span></p>

<p class=MsoNormal><span lang=FR>ZmxhZy1IMHdfZDFEX3kwdV9HMzdfSDNyRT8=</span></p>

<p class=MsoNormal><span lang=FR>Elle ressemble clairement à du <b>Base64</b>. Pour vérifier, il suffit de la
décoder. On peut le faire de plusieurs manières : soit dans un terminal avec la
commande base64 -d, soit directement dans un site comme <b>CyberChef</b>, qui
est bien pratique pour tester différents types d’encodage. D’ailleurs,
CyberChef a une fonction magique : quand il détecte le type d’encodage, il
propose une petite baguette magique. Il suffit de cliquer dessus pour que le
décodage se fasse automatiquement.</span></p>

<p class=MsoNormal><span lang=FR><img id="Image 8"
src="images/image006.png"
alt="Une image contenant texte, capture d’écran, logiciel, affichage&#10;&#10;Le contenu généré par l’IA peut être incorrect."></span><span
lang=FR> </span></p>

<p class=MsoNormal><b><span lang=FR style='font-size:16.0pt;line-height:115%'>Version
alternative&nbsp;: </span></b></p>

<p class=MsoNormal><span lang=FR>Il existe une autre méthode, encore plus
rapide. Dans certains jeux Unity, un fichier de log est généré automatiquement
sur votre machine. Ici, on peut aller regarder dans:</span></p>

<p class=MsoNormal><span lang=FR>%USERPROFILE%\AppData\LocalLow\DefaultCompany\UnitedCTF\Player.log</span></p>

<p class=MsoNormal><span lang=FR>Dans ce Player.log, on retrouve aussi la
chaîne Base64 ZmxhZy1IMHdfZDFEX3kwdV9HMzdfSDNyRT8=. En la copiant et en la décodant,
on obtient le même résultat sans même avoir besoin d’ouvrir dnSpy.</span></p>

<p class=MsoNormal><img 
src="images/image007.gif" 
hspace=12
alt="Une image contenant dessin humoristique, dessin, illustration, peinture&#10;&#10;Le contenu généré par l’IA peut être incorrect."><span
lang=FR>Bravo, on a donc récupéré le deuxième flag ! Cette fois-ci en passant
par une analyse du code (ou par les logs si on veut la méthode rapide). On est
maintenant prêts à s’attaquer au troisième challenge. </span></p>

<h2><span lang=FR>Curse of the Pharaoh 3</span></h2>

<p class=MsoNormal>Description (français)</p>

<p class=MsoNormal><span lang=FR>Mais comment passer ce garde??</span></p>

<p class=MsoNormal><span lang=FR>On peut relancer le jeu depuis dnSpy (bouton <i>Start</i>
</span><span lang=FR style='font-family:"Times New Roman",serif'>→</span><span
lang=FR> choisir l</span><span lang=FR>’</span><span lang=FR>ex</span><span
lang=FR>é</span><span lang=FR>cutable CurseOfThePharaoh.exe) ; en se déplaçant on
croise un garde qui bloque la route vers la droite, qu’il demande un
pot-de-vin, et surtout qu’un <b>mur invisible</b> empêche le passage. L’énigme
sent la bidouille de code.</span></p>

<p class=MsoNormal><b><span lang=FR>Exploration et premières idées</span></b></p>

<p class=MsoNormal><span lang=FR>J’ouvre dnSpy et je commence à chercher où la
logique du pot-de-vin est implémentée. Après un peu d’exploration, on tombe sur
le module <b>BribingSystem / BribeLogic</b> qui gère tout l’échange. Le nom
AcceptBribe fait immédiatement penser qu’on a trouvé la bonne cible.</span></p>

<p class=MsoNormal><span lang=FR><img  id="Image 10"
src="images/image008.png"
alt="Une image contenant texte, affichage, capture d’écran, logiciel&#10;&#10;Le contenu généré par l’IA peut être incorrect."></span></p>

<p class=MsoNormal><span lang=FR><img  id="Image 11"
src="images/image009.png"
alt="Une image contenant texte, capture d’écran&#10;&#10;Le contenu généré par l’IA peut être incorrect."></span></p>

<p class=MsoNormal><span lang=FR>Dans AcceptBribe on repère une comparaison du
genre</span></p>

<p class=MsoNormal><span lang=FR>item.ValueRO.Value &gt;= item2.ValueRO.Value</span></p>

<p class=MsoNormal><span lang=FR>L’idée évidente est de forcer la condition
pour que même un petit montant passe — par exemple changer &gt;= en &lt;=. Je
tente donc d’éditer la méthode avec <b>Edit Method (C#)</b> dans dnSpy et de
recompiler.</span></p>

<p class=MsoNormal><img 
src="images/image010.png" 
hspace=12><span lang=FR>Après la modification, dnSpy refuse de compiler :
erreur de compilation. J’essaie de faire la modif la plus simple possible (ne
rien changer) et la compilation plante encore — signe que ce n’est pas la
logique en elle-même qui pose problème, mais la façon dont la méthode est
attachée au système ECS (Entity Component System) ou des dépendances que cette
DLL attend.</span></p>

<p class=MsoNormal><span lang=FR>En pratique, Unity + ECS introduisent parfois
des types ou des appels qui ne sont pas trivialement recompilables depuis dnSpy
(problèmes de signatures, d’attributs, ou d’accès à certains types liés au runtime).
Plutôt que d’acharner AcceptBribe, je parcours d’autres fonctions du même
module pour trouver quelque chose qui <b>compile proprement</b> si je l’édite.
Finalement, je découvre que <b>OnBribe()</b> se recompilent sans erreur. Bingo
: il faut donc agir via OnBribe() plutôt que AcceptBribe.</span></p>

<p class=MsoNormal><b><span lang=FR>Lecture de OnBribe() et stratégie finale</span></b></p>

<p class=MsoNormal><span lang=FR>Voici la fonction OnBribe() telle qu’on la
trouve dans le code :</span></p>

<p class=MsoNormal><img id="Image 5"
src="images/image011.png"
alt="Une image contenant Logiciel multimédia, logiciel, texte, capture d’écran&#10;&#10;Le contenu généré par l’IA peut être incorrect."></p>

<p class=MsoNormal><span lang=FR>En lisant ça et le reste du module, on
comprend plusieurs choses utiles :</span></p>

<ul style='margin-top:0in' type=disc>
 <li class=MsoNormal><span lang=FR>OnBribe() est appelé quand on parle au garde
     (ou quand l’événement de don est déclenché).</span></li>
 <li class=MsoNormal><span lang=FR>Le système utilise <b>ECS</b> : il interroge
     un singleton (via EntityQueryBuilder) pour récupérer un
     BribeRequestComponent.</span></li>
 <li class=MsoNormal><span lang=FR>Si on parvient à récupérer ou créer/modifier
     le composant qui contient les informations sur le mur (BribeComponent /
     champ .Wall), on pourra reproduire ce que fait AcceptBribe (mettre le
     résultat à true et détruire le mur) <b>sans toucher à AcceptBribe</b>.</span></li>
</ul>

<p class=MsoNormal><span lang=FR>Donc la stratégie : au lieu d’altérer la
comparaison d’AcceptBribe, on modifie OnBribe() (qui compile) pour <b>récupérer
le BribeComponent singleton</b> et effectuer directement l’action qui supprime
le mur / marque la requête comme acceptée.</span></p>

<p class=MsoNormal><span lang=FR>Concrètement, j’ai ajouté la récupération d’un
BribeComponent par un EntityQueryBuilder similaire à ce qui est fait dans
OnBribe() mais avec WithAll&lt;BribeComponent&gt;(), puis j’ai lu/écrit les
champs nécessaires (activer le flag de suppression du mur, ou détruire l’entité
du mur selon l’implémentation). Après recompilation, OnBribe() s’exécute en
jeu, le mur disparaît et le garde laisse passer.</span></p>

<p class=MsoNormal><span lang=FR><img  id="Image 6"
src="images/image012.png"
alt="Une image contenant texte, logiciel, capture d’écran&#10;&#10;Le contenu généré par l’IA peut être incorrect."></span></p>

<p class=MsoNormal><span lang=FR>On arrive ensuite dans la zone où le flag est
affiché.</span></p>

<p class=MsoNormal><b><span lang=FR>Remarques pédagogiques (pour les débutants)</span></b></p>

<ul style='margin-top:0in' type=disc>
 <li class=MsoNormal><b><span lang=FR>Pourquoi AcceptBribe a planté alors que
     OnBribe() passait ?</span></b><span lang=FR> Modifier une méthode qui
     dépend fortement de types/constructeurs du runtime ECS peut provoquer des
     erreurs de signature à la recompilation. Cherche une méthode équivalente
     dans le même module qui compile proprement : patcher une méthode « plus haut
     niveau » peut suffire.</span></li>
 <li class=MsoNormal><img 
     src="images/image013.gif"
     align=left hspace=12
     alt="Une image contenant Dessin animé, Animation, illustration, dessin humoristique&#10;&#10;Le contenu généré par l’IA peut être incorrect."><b><span
     lang=FR>ECS résumé rapide :</span></b><span lang=FR> au lieu d’objets
     Unity classiques, le jeu manipule des <i>entities</i> et <i>components</i>
     via un EntityManager. Pour modifier le comportement, on cherche souvent le
     composant singleton qui contient l’état (ici BribeComponent) et on le
     change.</span></li>
 <li class=MsoNormal><b><span lang=FR>Débogage itératif :</span></b><span
     lang=FR> si une modif ne compile pas, ne t’obstine pas : explore d’autres
     fonctions proches et teste des modifications minimales qui compilent.
     C’est souvent plus rapide.</span></li>
</ul>

<p class=MsoNormal><span lang=FR>Bravo — troisième flag en poche. On avance
maintenant vers le dernier défi de la track.</span></p>

<h2>Curse of the Pharoah 4 </h2>

<p class=MsoNormal>Description (français)</p>

<p class=MsoNormal><span lang=FR>On dirait que le pharaon a truqué le jeu en sa
faveur... Mais comment le battre?</span></p>

<p class=MsoNormal><span lang=FR>Note : Lorsque vous l'aurez battu, le flag
apparaîtra à sa gauche.</span></p>

<p class=MsoNormal><span lang=FR>Voir l'indice</span></p>

<p class=MsoListParagraphCxSpFirst style='text-indent:-.25in'><span lang=FR
style='font-family:"Aptos",sans-serif'>-<span style='font:7.0pt "Times New Roman"'>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
</span></span><span lang=FR>Et si on injectait un système ECS dans le jeu...? /
What about injecting an ECS system into the game...?</span></p>

<p class=MsoListParagraphCxSpLast style='text-indent:-.25in'><span lang=FR
style='font-family:"Aptos",sans-serif'>-<span style='font:7.0pt "Times New Roman"'>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
</span></span><span lang=FR>L'ordre d'exécution des systèmes est important. /
The order of execution of systems is important.</span></p>

<p class=MsoNormal><i><span lang=FR>
Note : La solution présentée pour ce challenge repose sur une approche alternative qui n’était pas la méthode initialement prévue par la créatrice. Bien que non intentionnelle, cette solution permettait néanmoins de valider le défi lors de la compétition.
</span></i></p>

<p class=MsoNormal><span lang=FR>Au départ, le pharaon n’a pas l’air bien
méchant. On s’approche, on essaie de lui parler… et résultat : il nous expulse
directement de la pyramide. Visiblement, il ne va pas nous donner son flag
gratuitement.<img id="Image 19"
src="images/image014.png"
alt="Une image contenant art, statue, musée&#10;&#10;Le contenu généré par l’IA peut être incorrect."></span></p>

<span lang=FR>Direction <b>dnSpy</b> pour voir ce qu’il fait
exactement. En fouillant dans le module <b>BossLogic</b>, on remarque deux
parties intéressantes : un <b>BossUISystem</b> et un <b>HealthSystem</b>. En
lisant rapidement les deux, on découvre qu’il existe un <b>FlagContainer</b>
lié à l’état du boss. Avec un peu de chance, ce container contient bien le
flag, il ne resterait qu’à le forcer à s’afficher.</span></p>

<p class=MsoNormal><span lang=FR> </span></p>

<p class=MsoNormal><span lang=FR>L’idée est donc de reproduire la même logique
que dans le challenge précédent (quand on a supprimé le mur du garde). Cette
fois-ci, on va devoir manipuler une entité contrôlée par le <b>HealthSystem</b>.
Si on réussit à récupérer cette entité et ses données, on peut modifier
directement le composant BossDefeatedComponent pour activer le FlagContainer et
l’afficher.</span></p>

<p class=MsoNormal><span lang=FR>Concrètement, on crée une nouvelle requête
avec un EntityQueryBuilder, comme on l’avait fait pour le garde. Mais au lieu
de cibler PlayerTag ou BribeRequestComponent, on cible <b>BossDefeatedComponent</b>.
On stocke le résultat dans une entité (par exemple entity2). Ensuite, on
récupère les données du composant :</span></p>

<p class=MsoNormal>BossDefeatedComponent componentData =
entityManager.GetComponentData&lt;BossDefeatedComponent&gt;(entity2);</p>

<p class=MsoNormal><img 
src="images/image016.png" 
hspace=12
alt="Une image contenant capture d’écran, Logiciel multimédia, logiciel&#10;&#10;Le contenu généré par l’IA peut être incorrect."><span
lang=FR>On ajoute ensuite une petite vérification pour s’assurer que l’entité
n’est pas vide et qu’elle existe bien. Enfin, on modifie la valeur du
FlagContainer afin de la passer à true.</span></p>

<p class=MsoNormal><img 
src="images/image017.gif"
hspace=12
alt="Une image contenant dessin humoristique, Dessin d’enfant, dessin, art&#10;&#10;Le contenu généré par l’IA peut être incorrect."></p>

<p class=MsoNormal><span lang=FR>&nbsp;</span></p>

<p class=MsoNormal><span lang=FR>&nbsp;</span></p>

<p class=MsoNormal><span lang=FR>&nbsp;</span></p>

<p class=MsoNormal><span lang=FR style='font-size:16.0pt;line-height:115%'>Parfait
allons voir si le flag est présent. </span></p>

<p class=MsoNormal><span lang=FR>&nbsp;</span></p>

<p class=MsoNormal><span lang=FR>Bravo ! Vous avez complété la track <b>Curse
of the Pharaoh</b>.<img  id="Image 24"
src="images/image018.gif"
alt="Une image contenant Dessin d’enfant, dessin, art, dessin humoristique&#10;&#10;Le contenu généré par l’IA peut être incorrect."></span></p>

<p class=MsoNormal><span lang=FR>&nbsp;</span></p>

</div>

</body>

</html>
