---
title: '[Interview] Sylvain Conchon rejoint OCamlPro'
description: "Sylvain Conchon vient de rejoindre OCamlPro en tant que Chief Scientific
  Officer M\xE9thodes Formelles. Professeur \xE0 l\u2019Universit\xE9 Paris-Saclay,
  il travaille dans le domaine de la d\xE9monstration automatique pour la preuve de
  programmes et le model checking pour syst\xE8mes param\xE9tr\xE9s. Il est aussi
  ..."
url: https://ocamlpro.com/blog/2020_06_05_fr_interview_sylvain_conchon_rejoint_ocamlpro
date: 2020-06-05T13:31:53-00:00
preview_image: https://ocamlpro.com/assets/img/og_image_ocp_the_art_of_prog.png
authors:
- "\n    Aurore Dombry\n  "
source:
---

<p><img src="https://ocamlpro.com/blog/assets/img/picture_sylvainconchon.jpg" alt=""></p>
<blockquote>
<p>Sylvain Conchon vient de rejoindre OCamlPro en tant que Chief Scientific Officer Méthodes Formelles. Professeur à l’Université Paris-Saclay, il travaille dans le domaine de la démonstration automatique pour la preuve de programmes et le model checking pour systèmes paramétrés. Il est aussi le co-créateur d’Alt-Ergo.</p>
</blockquote>
<h3>Recherche et industrie</h3>
<p><strong>Sylvain, tu fréquentes de longue date le monde industriel,
que penses-tu des interactions entre les industriels et les laboratoires
de recherche ?</strong></p>
<p>J’ai toujours trouvé très enrichissantes les interactions avec les
industriels. Pendant mes études, j’ai travaillé plusieurs années en
SSII, et je suis mes étudiants en stage ou en apprentissage dans des
sociétés technologiques ou chez de grands industriels. Je participe
également à des projets de recherche qui impliquent des industriels,et
j’ai passé quelques temps chez Intel à Portland, ce qui m’a permis de
découvrir l’industrie du hardware.</p>
<p><strong>Comment parvenir à établir des relations fructueuses entre le monde académique et les industriels ?</strong></p>
<p>C’est beaucoup une histoire de rencontre. On le voit lors des
montages de projets de recherche collaboratifs qui réunissent
académiques et industriels. Les outils issus de la recherche, quels
qu’ils soient, doivent avant tout répondre à un besoin réel des
industriels. Si c’est le cas, il faut aussi que le logiciel soit
utilisable par des ingénieurs du métier sans qu’il leur soit nécessaire
de comprendre son fonctionnement interne (par exemple, pour positionner
les 50 options nécessaires à son utilisation, interpréter ses résultats
ou ses absences de résultats!). Cela nécessite à l’évidence un travail
d’ingénierie important, tourné vers l’utilisateur final et souvent
éloigné des activités des chercheurs. Il faut donc comprendre les
problèmes et les besoins des industriels, et ensuite déterminer si les
technologies et les outils que l’on maîtrise peuvent être adaptés ou
utilisés pour réaliser un prototype qui réponde à certains de ces
besoins.</p>
<p><strong>Tu viens de rejoindre OCamlPro, quelles sont tes premières impressions ?</strong></p>
<p>Je suis heureux d’avoir rejoint une entreprise très dynamique, pleine
de gens talentueux, motivés et sympathiques, où l’on fait à la fois de
l’ingénierie de haut niveau et de la recherche de qualité !</p>
<blockquote>
<p><em>“ Les outils issus de la recherche, quels qu’ils soient, doivent avant tout répondre à un besoin réel des industriels.”</em></p>
</blockquote>
<h3>OCaml, un langage de pointe</h3>
<p><strong>Tu es connu dans la communauté OCaml, et certains de
tes étudiants sont devenus des fans d’OCaml (et de ton enseignement)…
que dis-tu à tes étudiants qui découvrent OCaml ?</strong></p>
<p>J’ai tendance à résumer en disant ceci : <em>« avec OCaml, vous n’apprenez pas la programmation des 10 dernières années, mais celle des 10 prochaines années »</em>. Cette affirmation s’est toujours vérifiée car bon nombre de traits du langage OCaml se sont retrouvés dans les langages <em>mainstream</em>, avec plusieurs années de décalage. Cela dit, mes années d’expérience dans l’enseignement de ce langage me laissent penser que quelques modifications dans sa syntaxe permettraient une approche plus aisée pour certains débutants.</p>
<p><strong>Et toi, comment as-tu découvert OCaml ?</strong></p>
<p>Pendant mes études à l’Université lors de mon projet de fin de
maîtrise : un de mes enseignants m’avait orienté vers ce langage pour
m’aider à réaliser un compilateur pour un langage de programmation
concurrente. J’ai donc découvert ce langage par moi-même, en lisant le
manuel et les exemples. Ce n’est que pendant mon DEA que j’ai découvert
les fondements théoriques de ce beau langage (sémantique, typage,
compilation).</p>
<p><strong>OCaml, un langage industriel ou pas encore ?</strong></p>
<p>Il convient de préciser la question : qu’est-ce qu’un langage
industriel ? Si c’est un langage utilisé par les industriels, alors
OCaml n’est hélas pas encore suffisamment utilisé dans l’industrie pour
être qualifié ainsi. Si la question est de savoir s’il a le niveau des
langages utilisés dans l’industrie, alors la réponse est oui, sans
hésiter. Mais peut-être la question porte-t-elle davantage sur
l’écosystème OCaml et la maturité de l’outillage: il y a sûrement des
progrès à faire pour atteindre le niveau d’un langage très répandu dans
l’industrie, mais c’est en bonne voie, en particulier grâce à des
entreprises telles qu’OCamlPro.</p>
<h3>Les méthodes formelles comme technique industrielle, et l’exemple du solveur Alt-Ergo</h3>
<p><strong>Les méthodes formelles sont l’un des domaines d’expertise d’OCamlPro, en quoi penses-tu qu’OCaml est adapté au domaine des SMT ?</strong></p>
<p>Les outils comme les solveurs SMT sont principalement des logiciels
de manipulation symbolique des données qui permettent d’analyser, de
transformer et de raisonner sur des formules logiques. OCaml est fait
pour ce genre de traitements. Il y a aussi une partie plus «
calculatoire » dans ces outils qui nécessite une programmation fine des
structures de données ainsi qu’une gestion efficace de la mémoire. OCaml
est particulièrement adapté pour ce genre de développements, surtout
avec son ramasse-miettes (GC) extrêmement performant. Enfin, les
solveurs SMT sont des outils qui doivent avoir un grand niveau de
fiabilité car les erreurs dans ces logiciels sont difficiles à trouver
et leur présence peut être très préjudiciable. Le système de types
d’OCaml contribue à la fiabilité de ces outils.</p>
<blockquote>
<p><em>“Les solveurs SMT sont aujourd’hui incontournables dans le domaine de l’ingénierie du logiciel.”</em></p>
</blockquote>
<p><strong>Peux-tu nous parler d’Alt-Ergo en quelques mots ?</strong></p>
<p>C’est un logiciel utilisé pour prouver automatiquement (sans
intervention humaine) des formules logiques, c’est-à-dire savoir si ces
formules sont vraies ou fausses. Alt-Ergo appartient à une famille de
démonstrateurs automatiques appelée SMT (pour Satisfiabilité Modulo
Théories). Il a été conçu pour être intégré dans des plate-formes de
vérification de programmes. Ces outils (comme Why3, Frama-C, Spark,…)
génèrent des formules logiques qu’il est nécessaire de prouver afin de
garantir qu’un programme est sûr. Faire la preuve de ces formules à la
main serait très fastidieux (il y a parfois plusieurs dizaines de
milliers de formules à prouver). Un solveur SMT comme Alt-Ergo est là
pour faire ce travail, de manière complètement automatique. C’est ce qui
permet à ces plateformes de vérification d’être utilisables au niveau
industriel.</p>
<p><strong>En quoi le développement d’Alt-Ergo en OCaml peut-il être un avantage par rapport aux concurrents ?</strong></p>
<p>Cela lui confère une plus grande sûreté, car un solveur SMT, comme
n’importe quel programme peut aussi avoir des bugs. La plus grande
partie d’Alt-Ergo est programmée dans un style purement fonctionnel,
c’est-à-dire uniquement avec l’utilisation de structures de données
immuables. L’un des avantages de ce style de programmation est qu’il
nous a permis de prouver formellement ses principaux composants (par
exemple, son noyau a été formalisé à l’aide de l’assistant à la preuve
Coq, ce qui serait impossible à faire dans un langage comme C++), sans
sacrifier son efficacité grâce au très bon ramasse-miettes et à la
bibliothèque de structures de données persistantes très performantes
d’OCaml. Enfin, nous avons largement bénéficié du système de modules
d’OCaml, en particulier les foncteurs et les modules récursifs, pour
concevoir un code très modulaire, maintenable et facilement extensible.
Au final, OCaml nous a permis de concevoir un solveur SMT aussi
performant que CVC4 ou Z3 pour la preuve de programmes, mais avec un
nombre de lignes de code divisé par trois ou quatre. Bien sûr, cela ne
garantit pas que Alt-Ergo ait zéro bugs, mais cela nous aide beaucoup à
mettre le doigt dessus quand quelqu’un en trouve.</p>
<p><em>“OCaml nous a permis de concevoir un solveur SMT aussi performant que CVC4 ou Z3 pour la
preuve de programmes, mais avec un nombre de lignes de code divisé par
trois ou quatre.“</em></p>
<p><strong>Quel est ton avis sur les solveurs SMT et l’état de l’art SMT actuel ?</strong></p>
<p>Les solveurs SMT sont aujourd’hui incontournables dans le domaine de
l’ingénierie du logiciel. On les trouve aussi bien dans des outils de
preuve, de test, de model checking, d’interprétation abstraite ou encore
de typage. La principale raison de ce succès est qu’ils sont de plus en
plus efficaces et les théories sous-jacentes sont très expressives.
C’est un domaine de recherche très concurrentiel entre les meilleures
universités ou laboratoires du monde et de grandes entreprises en
informatique. Mais la marge de progression de ces outils est encore très
grande, en particulier dans le domaine de l’arithmétique non linéaire
où la demande des utilisateurs est de plus en plus forte. Pour le
moment, un de mes objectifs en recherche est de combiner les outils de
Model Checking avec ceux de preuve de programmes. Ces deux familles
d’outils reposent sur les SMT et elles devraient se compléter pour
offrir des outils de vérification encore plus automatiques.</p>
<p><strong>Quelles applications les techniques SMT et Alt-Ergo peuvent-elles avoir dans l’industrie ?</strong></p>
<p>Les techniques SMT peuvent être utilisées partout où les méthodes
formelles peuvent être utiles. Par exemple (mais cette liste est loin
d’être exhaustive), pour vérifier la sûreté de logiciels critiques dans
le domaine de l’embarqué, pour trouver des failles de sécurité dans les
systèmes informatiques ou pour résoudre des problèmes de planification.
On les trouve également dans le domaine de l’intelligence artificielle
où il est crucial de garantir la stabilité des réseaux de neurones mais
aussi de produire des explications formelles sur leurs résultats.</p>
<p><strong>Tu as été amené à travailler sur le Model Checking, peux-tu
nous parler des liens entre Model Checking et SMT et de son utilisation
actuelle ?</strong></p>
<p>Le Model Checking consiste à vérifier que tous les états possibles
d’un système respectent bien certaines propriétés, et ce quelles que
soient les données en entrée. C’est un problème difficile car certains
systèmes (microprocesseurs par ex.) peuvent avoir des centaines de
millions d’états. Pour passer à l’échelle, les model checkers
implémentent des algorithmes très perfectionnés pour visiter ces états
rapidement, en les stockant d’une manière très compacte. Cependant,
cette technique atteint ses limites quand les valeurs prises en entrée
sont non bornées ou quand le nombre de composants du système n’est pas
connu. Pensez aux algorithmes de routage d’Internet où on ne connaît pas
le nombre de machines sur le réseau, ces algorithmes doivent être
corrects, quel que soit ce nombre de machines. C’est là que les solveurs
SMT entrent en jeu. En utilisant des formules logiques, on peut
représenter des ensembles d’états de taille arbitraire. Visiter les
états d’un système consiste alors à calculer les formules qui
représentent ces états. Vérifier que les états respectent une propriété
revient à prouver que les formules qui représentent des états impliquent
la propriété voulue, etc. Tout dans le Model Checking repose donc sur
des formules logiques et les solveurs SMT sont évidemment là pour
raisonner sur ces formules.</p>

