---
title: "D\xE9tection de fonctions d\u2019identit\xE9 dans Flambda"
description: "Au cours de discussions parmi les d\xE9veloppeurs OCaml sur le type
  vide (PR#9459), certains caressaient l\u2019id\xE9e d\u2019annoter des fonctions
  avec un attribut indiquant au compilateur que la fonction devrait \xEAtre triviale,
  et toujours renvoyer une valeur strictement \xE9quivalente \xE0 son argument. Nous
  ..."
url: https://ocamlpro.com/blog/2021_07_15_fr_detection_de_fonctions_didentite_dans_flambda
date: 2021-07-15T13:31:53-00:00
preview_image: https://ocamlpro.com/assets/img/og_image_ocp_the_art_of_prog.png
authors:
- "\n    Leo Boitel\n  "
source:
---

<blockquote>
<p>Au cours de discussions parmi les développeurs OCaml sur le type vide (<a href="https://github.com/ocaml/ocaml/issues/9459">PR#9459</a>), certains caressaient l’idée d’annoter des fonctions avec un attribut indiquant au compilateur que la fonction devrait être triviale, et toujours renvoyer une valeur strictement équivalente à son argument.
Nous étions curieux de voir si l’implémentation d’une telle fonctionnalité serait possible et nous avons publié une offre de stage pour explorer ce sujet.
L’équipe Compilation d’OCamlPro a ainsi accueilli Léo Boitel durant trois mois pour se consacrer à ce sujet, avec Vincent Laviron pour encadrant. Nous sommes fiers des résultats auxquels Léo a abouti !</p>
<p>Voici ce que Léo en a écrit 🙂</p>
</blockquote>
<h3>Description du problème</h3>
<p>Le typage fort d’OCaml est un de ses grands avantages : il permet d’écrire du code plus sûr grâce à la capacité d’abstraction qu’il offre. La plupart des erreurs de conception se traduiront directement en erreur de typage, et l’utilisateur ne peut pas faire d’erreur avec la manipulation de la mémoire puisqu’elle est entièrement gérée par le compilateur.</p>
<p>Cependant, ces avantages empêchent l’utilisateur de faire certaines optimisations lui-même, en particulier celles liées aux représentations mémoires puisqu’il n’y accède pas directement.</p>
<p>Un cas classique serait le suivant :</p>
<pre><code class="language-Ocaml">type return = Ok of int | Failure
let id = function
| Some x -&gt; Ok x
| None -&gt; Failure
</code></pre>
<p>Cette fonction est une identité, car la représentation mémoire de <code>Some x</code> et de <code>Ok x</code> est la même (idem pour <code>None</code> et <code>Failure</code>). Cependant, l’utilisateur ne le voit pas, et même s’il le voyait, il aurait besoin de cette fonction pour conserver un typage correct.</p>
<p>Un autre exemple serait le suivant:
Another good example would be this one:</p>
<pre><code class="language-Ocaml">type record = { a:int; b:int }
let id (x,y) = { a = x; b = y }
</code></pre>
<p>Même si ces fonctions sont des identités, elles ont un coût : en plus de nous coûter un appel, elles réallouent le résultat au lieu de nous retourner leur argument directement. C’est pourquoi leur détection permettrait des optimisations intéressantes.</p>
<h3>Difficultés</h3>
<p>Si on veut pouvoir détecter les identités, on se heurte rapidement au problème des fonctions récursives : comment définir l’identité pour ces dernières ? Est-ce qu’une fonction peut-être l’identité si elle ne termine pas toujours, voire jamais ?</p>
<p>Une fois qu’on a défini l’identité, le problème est la preuve qu’une fonction est bien l’identité. En effet, on veut garantir à l’utilisateur que cette optimisation ne changera pas le comportement observable du programme.</p>
<p>On veut aussi éviter d’ajouter des failles de sûreté au typage. Par exemple, si on a une fonction de la forme suivante:</p>
<pre><code class="language-Ocaml">let rec fake_id = function
| [] -&gt; 0
| t::q -&gt; fake_id (t::q)
</code></pre>
<p>Une preuve naïve par induction nous ferait remplacer cette fonction par l’identité, car <code>[]</code> et <code>0</code> ont la même représentation mémoire. C’est dangereux car le résultat d’une application à une liste non-vide sera une liste alors qu’il est typé comme un entier (voir exemples plus bas).</p>
<p>Pour résoudre ces problèmes, nous avons commencé par une partie théorique qui a occupé les trois quarts du stage, pour finir par une partie pratique d’implémentation dans Flambda.</p>
<h3>Résultats théoriques</h3>
<p>Pour cette partie, nous avons travaillé sur des extensions de lambda-calcul, implémentées en OCaml, pour pouvoir tester nos idées au fur et à mesure dans un cadre plus simple que Flambda.</p>
<h4>Paires</h4>
<p>Nous avons commencé par un lambda calcul auquel on ajoute seulement des paires. Pour effectuer nos preuves, on annote toutes les fonctions comme des identités ou non. On prouve ensuite ces annotations en β-réduisant le corps des fonctions. Après chaque réduction récursive, on applique une règle qui dit qu’une paire composée des deux projections d’une variable est égale à la variable. On ne réduit pas les applications, mais on les remplace par l’argument si la fonction est annotée comme une identité.</p>
<p>On garde ainsi une complexité raisonnable par rapport à une β-réduction complète qui serait évidemment irréaliste pour de gros programmes.</p>
<p>On passe ensuite à l’ordre supérieur en permettant des annotations de la forme <code>Annotation → Annotation</code>. Les fonctions comme <code>List.map</code> peuvent donc être représentées comme <code>Id → Id</code>. Bien que cette solution ne soit pas complète, elle couvre la grande majorité des cas d’utilisation.</p>
<h4>Reconstruction de tuples</h4>
<p>On passe ensuite des paires aux tuples de taille arbitraire. Cela complexifie le problème : si on construit une paire à partir des projections des deux premiers champs d’une variable, ce n’est pas forcément la variable, puisqu’elle peut avoir plus de champs.</p>
<p>On a alors deux solutions : tout d’abord, on peut annoter les projections avec la taille du tuple pour savoir si on reconstruit la variable en entier. Par exemple, si on reconstruit une paire avec deux projections d’un triplet, on sait qu’on ne peut pas simplifier cette reconstruction.</p>
<p>L’autre solution, plus ambitieuse, est d’adopter une définition moins stricte de l’égalité, et de dire qu’on peut remplacer, par exemple, <code>(x,y)</code> par <code>(x,y,z)</code>. En effet, si la variable a été typée comme une paire, on a la garantie qu’on accédera jamais au champ <code>z</code> de toute façon. Le comportement du programme sera donc le même si on étend la variable avec des champs supplémentaires.</p>
<p>Utiliser l’égalité observationnelle permet d’éviter beaucoup d’allocations, mais elle peut utiliser plus de mémoire dans certains cas : si le triplet cesse d’être utilisé, il ne sera pas désalloué par le Garbage Collector (GC), et le champ <code>z</code> restera donc en mémoire pour rien tant que <code>(x,y)</code> est utilisé.</p>
<p>Cette approche reste intéressante, au moins si on donne la possibilité à l’utilisateur de l’activer manuellement pour certains blocs.</p>
<h4>Récursion</h4>
<p>On ajoute maintenant les définitions récursives à notre langage, par le biais d’un opérateur de point fixe.</p>
<p>Pour prouver qu’une fonction récursive est l’identité, on doit procéder par induction. La difficulté est alors de prouver que la fonction termine, pour que l’induction soit correcte.</p>
<p>On peut distinguer trois niveaux de preuve : la première option est de ne pas prouver la terminaison, et de laisser l’utilisateur choisir les fonctions dont il est sûr qu’elles terminent. On suppose donc que la fonction est l’identité, et on simplifie son corps avec cette hypothèse. Cette approche est suffisante pour la plupart des cas pratiques, mais son problème principal est qu’elle autorise à écrire du code qui casse la sûreté du typage, comme discuté ci-dessus.</p>
<p>La seconde option est de faire notre hypothèse d’induction uniquement sur des applications de la fonction sur des éléments plus “petits” que l’argument. Un élément est défini comme tel s’il est une projection de l’argument, ou une projection d’un élément plus petit. Cela n’est pas suffisant pour prouver que la fonction termine (par exemple si l’argument est cyclique), mais c’est assez pour avoir un typage sûr. En effet, cela implique que toutes les valeurs de retour possibles de la fonction sont construites (puisqu’elles ne peuvent provenir directement d’un appel récursif), et ont donc un type défini. Le typage échouerait donc si la fonction pouvait renvoyer une valeur qui n’est pas identifiable à son argument.</p>
<p>Finalement, on peut vouloir une équivalence observationnelle parfaite entre la fonction et l’identité pour la simplifier. Dans ce cas, la solution que nous proposons est de créer une annotation spéciale pour les fonctions qui sont l’identité quand elles sont appliquées à un objet non cyclique. On peut prouver qu’elles ont cette propriété avec l’induction décrite ci-dessus. La difficulté est ensuite de faire la simplification sur les bonnes applications : si un objet est immutable, n’est pas défini récursivement, et que tous ses sous-objets satisfont cette propriété, on le dit inductif et on peut simplifier les applications sur lui. On propage le statut inductif des objets lors de notre passe récursive d’optimisation.</p>
<p>###Reconstruction de blocs</p>
<p>La représentation des blocs dans Flambda pose des problèmes intéressants pour détecter leur égalité, ce qui est souvent nécessaire pour prouver une identité. En effet, il est difficile de détecter la reconstruction d’un bloc à l’identique.</p>
<h4>Blocs dans Flambda</h4>
<h5>Variants</h5>
<p>The blocks in Flambda come from the existence of variants in OCaml: one type may have several different constructors, as we can see in</p>
<pre><code class="language-Ocaml">type choice = A of int | B of int
</code></pre>
<p>Quand OCaml est compilé vers Flambda, l’information du constructeur utilisé par un objet est perdue, et est remplacée par un tag. Le tag est un nombre contenu dans un entête de la représentation mémoire de l’objet, et est un nombre entre <code>0</code> et <code>255</code> représentant le constructeur de l’objet. Par exemple, un objet de type choice aurait le tag <code>0</code> si c’est un <code>A</code> et <code>1</code> si c’est un <code>B</code>.</p>
<p>Le tag est ainsi présent dans la mémoire à l’exécution, ce qui permet par exemple d’implémenter le pattern matching de OCaml comme un switch en Flambda, qui fait de simples comparaisons sur le tag pour décider quelle branche prendre.</p>
<p>Ce système nous complique la tâche puisque le typage de Flambda ne nous dit pas quel type de constructeur contient un variant, et empêche donc de décider facilement si deux variants sont égaux.</p>
<h5>Généralisation des tags</h5>
<p>Pour plus de complexité, les tags sont en faits utilisés pour tous les blocs, c’est à dire les tuples, les modules, les fonctions (en fait presque toutes les valeurs sauf les entiers et les constructeurs constants). Quand l’objet n’est pas un variant, on lui donne généralement un tag 0. Ce tag n’est donc jamais lu par la suite (puisqu’on ne fait pas de match sur l’objet), mais nous empêche de comparer simplement deux tuples, puisqu’on verra simplement deux objets de tag inconnu en Flambda.</p>
<h5>Inlining</h5>
<p>Enfin, on optimise ce système en inlinant les tuples : si on a un variant de type <code>Pair of int*int</code>, au lieu d’être représenté comme le tag de Pair et une adresse mémoire pointant vers un couple (donc un tag 0 et les deux entiers), le couple est inliné et l’objet est de la forme <code>(tag Pair, entier, entier)</code>.</p>
<p>Cela implique que les variants sont de taille arbitraire, qui est aussi inconnue dans Flambda.</p>
<h4>Approche existante</h4>
<p>Une solution partielle au problème existait déjà dans une Pull Request (PR) disponible <a href="https://github.com/ocaml/ocaml/pull/8958">ici</a>.</p>
<p>L’approche qui y est adoptée est naturelle : on y utilise les switchs pour gagner de l’information sur le tag d’un bloc, en fonction de la branche prise. La PR permet aussi de connaître la mutabilité et la taille du bloc dans chaque branche, en partant de OCaml (où l’information est connue puisque le constructeur est explicite dans le match), et propageant l’information jusqu’à Flambda.</p>
<p>Cela permet d’enregistrer tous les blocs sur lesquels on a fait un switch dans l’environnement, avec leur tag, taille et mutabilité. On peut ensuite détecter si on reconstruit l’un d’entre eux avec la primitive <code>Pmakeblock</code>.</p>
<p>Cette approche est malheureusement limitée puisqu’ils existe de nombreux cas où on pourrait connaître le tag et la taille du bloc sans faire de switch dessus. Par exemple, on ne pourra jamais simplifier une reconstruction de tuple avec cette solution.</p>
<h4>Nouvelle approche</h4>
<p>Notre nouvelle approche commence donc par propager plus d’information depuis OCaml. La propagation est fondée sur deux PR qui existaient sur Flambda 2, et qui annotent dans lambda chaque projection (<code>Pfiel</code>) avec des informations dérivées du typage OCaml. Une ajoute la <a href="https://github.com/ocaml-flambda/ocaml/commit/fa5de9e64ff1ef04b596270a8107d1f9dac9fb2d">mutabilité du bloc</a> et l’autre <a href="https://github.com/ocaml-flambda/ocaml/pull/53">son tag et enfin sa taille</a>.</p>
<p>Notre première contribution a été d’adapter ces PRs à Flambda 1 et de les propager de lambda à Flambda correctement.</p>
<p>Nous avons ensuite les informations nécessaires pour détecter les reconstructions de blocs : en plus d’avoir une liste de blocs sur lesquels on a switché, on crée une liste de blocs partiellement immutables, c’est à dire dont on sait que certains champs sont immutables.</p>
<p>On l’utilise ainsi :</p>
<h6>Découverte de blocs</h6>
<p>Dès qu’on voit une projection, on regarde si elle est faite sur un bloc immutable de taille connue. Si c’est le cas, on ajoute le bloc correspondant aux blocs partiels. On vérifie que l’information qu’on a sur le tag et la taille est compatible avec celle des projections de ce bloc vues précédemment. Si on connaît maintenant tous les champs du bloc, on l’ajoute à notre liste de blocs connus sur lesquels on peut faire des simplifications.</p>
<p>On garde aussi les informations sur les blocs qu’on connaît grâce aux switchs.</p>
<h6>Simplification</h6>
<p>Cette partie est similaire à celle de la PR originale : quand on construit un bloc immutable, on vérifie si on le connaît, et le cas échéant on ne le réalloue pas.</p>
<p>Par rapport à l’approche originale, nous avons aussi réduit la complexité de la PR originale (de quadratique à linéaire), en enregistrant l’association de chaque variable de projection à son index et bloc original. Nous avons aussi modifié des détails de l’implémentation originale qui auraient pu créer un bug lorsque associés à notre PR.</p>
<h4>Exemple</h4>
<p>Considérons cette fonction:</p>
<pre><code class="language-Ocaml">type typ1 = A of int | B of int * int
type typ2 = C of int | D of {x:int; y:int}
let id = function
  | A n -&gt; C n
  | B (x,y) -&gt; D {x; y}
</code></pre>
<p>Le compilateur actuel produirait le Flambda suivant:</p>
<pre><code>End of middle end:
let_symbol
  (camlTest__id_21
    (Set_of_closures (
      (set_of_closures id=Test.8
        (id/5 = fun param/7 -&gt;
          (switch*(0,2) param/7
           case tag 0:
            (let
              (Pmakeblock_arg/11 (field 0&lt;{../../test.ml:4,4-7}&gt; param/7)
               Pmakeblock/12
                 (makeblock 0 (int)&lt;{../../test.ml:4,11-14}&gt;
                   Pmakeblock_arg/11))
              Pmakeblock/12)
           case tag 1:
            (let
              (Pmakeblock_arg/15 (field 1&lt;{../../test.ml:5,4-11}&gt; param/7)
               Pmakeblock_arg/16 (field 0&lt;{../../test.ml:5,4-11}&gt; param/7)
               Pmakeblock/17
                 (makeblock 1 (int,int)&lt;{../../test.ml:5,17-23}&gt;
                   Pmakeblock_arg/16 Pmakeblock_arg/15))
              Pmakeblock/17)))
         free_vars={ } specialised_args={}) direct_call_surrogates={ }
        set_of_closures_origin=Test.1])))
  (camlTest__id_5_closure (Project_closure (camlTest__id_21, id/5)))
  (camlTest (Block (tag 0,  camlTest__id_5_closure)))
End camlTest
</code></pre>
<p>Notre amélioration permet de détecter que cette fonction reconstruit des blocs similaires et donc la simplifie:</p>
<pre><code>End of middle end:
let_symbol
  (camlTest__id_21
    (Set_of_closures (
      (set_of_closures id=Test.7
        (id/5 = fun param/7 -&gt;
          (switch*(0,2) param/7
           case tag 0 (1): param/7
           case tag 1 (2): param/7))
         free_vars={ } specialised_args={}) direct_call_surrogates={ }
        set_of_closures_origin=Test.1])))
  (camlTest__id_5_closure (Project_closure (camlTest__id_21, id/5)))
  (camlTest (Block (tag 0,  camlTest__id_5_closure)))
End camlTest
</code></pre>
<h4>Pistes d’amélioration</h4>
<h5>Relâchement de l’égalité</h5>
<p>On peut utiliser l’égalité observationnelle étudiée dans la partie théorique pour l’égalité de blocs, afin d’éviter plus d’allocations. L’implémentation est simple :</p>
<p>Quand on crée un bloc, pour voir si il est alloué, l’approche normale est de regarder si chacun de ses champs est une projection connue d’un autre bloc, a le même index et si les deux blocs sont de même taille. On peut simplement supprimer la dernière vérification.</p>
<p>L’implémentation a été un peu plus difficile que prévu à cause de détails pratiques. Tout d’abord, on veut appliquer cette optimisation uniquement sur certains blocs annotés par l’utilisateur. Il faut donc propager l’annotation jusqu’à Flambda.</p>
<p>De plus, si on se contente d’implémenter l’optimisation, beaucoup de cas seront ignorés car les variables inutilisées sont simplifiées avant notre passe. Par exemple, prenons une fonction de la forme suivante :</p>
<pre><code class="language-Ocaml">let loose_id (a,b,c) = (a,b)
</code></pre>
<p>La variable <code>c</code> sera simplifiée avant d’atteindre Flambda, et on ne pourra donc plus prouver que <code>(a,b,c)</code> est immutable car son troisième champ pourrait ne pas l’être. Ce problème est en passe d’être résolu sur Flambda 2 grâce à une PR qui propage l’information de mutabilité pour tous les blocs, mais nous n’avons pas eu le temps nécessaire pour l’adapter à Flambda 1.</p>
<h3>Détection d’identités récursives</h3>
<p>Maintenant que nous pouvons détecter les reconstructions de blocs, reste à résoudre le problème des fonctions récursives.</p>
<h4>Approche sans garanties</h4>
<p>Nous avons commencé par implémenter une approche qui ne comporte pas de preuve de terminaison. L’idée est de rajouter la preuve ensuite, ou d’autoriser les fonctions qui ne terminent pas toujours à être simplifiées à condition qu’elles soient correctes au niveau du typage (voir section 7 dans la partie théorique).</p>
<p>Ici, on fait confiance à l’utilisateur pour vérifier ces propriétés manuellement.</p>
<p>Nous avons donc modifié la simplification de fonction : quand on simplifie une fonction à un seul argument, on commence par supposer que cette fonction est l’identité avant de simplifier son corps. On vérifie ensuite si le résultat est équivalent à une identité en le parcourant récursivement, pour couvrir le plus de cas possible (par exemple les branchements conditionnels). Si c’est le cas, la fonction est remplacée par l’identité ; sinon, on revient à une simplification classique, sans hypothèse d’induction.</p>
<h4>Propagation de constantes</h4>
<p>Nous avons ensuite amélioré notre fonction qui détermine si le corps d’une fonction est une identité ou non, pour gérer les constantes. Il propage les informations d’égalité qu’on gagne sur l’argument lors des branchements conditionnels.</p>
<p>Ainsi, si on a une fonction de la forme</p>
<pre><code class="language-Ocaml">type truc = A | B | C
let id = function
  | A -&gt; A
  | B -&gt; B
  | C -&gt; C
</code></pre>
<p>ou même</p>
<pre><code class="language-Ocaml">let id x = if x=0 then 0 else x
</code></pre>
<p>on détectera bien que c’est l’identité.</p>
<h4>Exemples</h4>
<h5>Fonctions récursives</h5>
<p>Nous pouvons maintenant détecter les identités récursives :</p>
<pre><code class="language-Ocaml">let rec listid = function
  | t::q -&gt; t::(listid q)
  | [] -&gt; []
</code></pre>
<p>compilait avant ainsi:</p>
<pre><code>End of middle end:
let_rec_symbol
  (camlTest__listid_5_closure
    (Project_closure (camlTest__set_of_closures_20, listid/5)))
  (camlTest__set_of_closures_20
    (Set_of_closures (
      (set_of_closures id=Test.11
        (listid/5 = fun param/7 -&gt;
          (if param/7 then begin
            (let
              (apply_arg/13 (field 1&lt;{../../test.ml:9,4-8}&gt; param/7)
               apply_funct/14 camlTest__listid_5_closure
               Pmakeblock_arg/15
                 *(apply*&amp;#091;listid/5]&lt;{../../test.ml:9,15-25}&gt; apply_funct/14
                    apply_arg/13)
               Pmakeblock_arg/16 (field 0&lt;{../../test.ml:9,4-8}&gt; param/7)
               Pmakeblock/17
                 (makeblock 0&lt;{../../test.ml:9,12-25}&gt; Pmakeblock_arg/16
                   Pmakeblock_arg/15))
              Pmakeblock/17)
            end else begin
            (let (const_ptr_zero/27 Const(0a)) const_ptr_zero/27) end))
         free_vars={ } specialised_args={}) direct_call_surrogates={ }
        set_of_closures_origin=Test.1])))
let_symbol (camlTest (Block (tag 0,  camlTest__listid_5_closure)))
End camlTest
</code></pre>
<p>On détecte maintenant que c’est l’identité :</p>
<pre><code>End of middle end:
let_symbol
  (camlTest__set_of_closures_20
    (Set_of_closures (
      (set_of_closures id=Test.13 (listid/5 = fun param/7 -&gt; param/7)
        free_vars={ } specialised_args={}) direct_call_surrogates={ }
        set_of_closures_origin=Test.1])))
  (camlTest__listid_5_closure
    (Project_closure (camlTest__set_of_closures_20, listid/5)))
  (camlTest (Block (tag 0,  camlTest__listid_5_closure)))
End camlTest
</code></pre>
<h5>Exemple non sûr</h5>
<p>En revanche, on peut profiter de l’absence de garanties pour contourner le typage, et accéder à une adresse mémoire comme à un entier :</p>
<pre><code class="language-Ocaml">type bugg = A of int*int | B of int
let rec bug = function
  | A (a,b) -&gt; (a,b)
  | B x -&gt; bug (B x)
  
let (a,b) = (bug (B 42))
let _ = print_int b
</code></pre>
<p>Cette fonction va être simplifiée vers l’identité alors que le type <code>bugg</code> n’est pas compatible avec le type tuple ; quand on essaie de projeter sur le second champ du variant <code>b</code>, on accède à une partie de la mémoire indéfinie :</p>
<pre><code>$ ./unsafe.out
47423997875612
</code></pre>
<h4>Pistes d’améliorations – court terme</h4>
<h5>Annotation des fonctions</h5>
<p>Une amélioration simple en théorie, serait de laisser le choix à l’utilisateur des fonctions sur lesquelles il veut appliquer ces optimisations qui ne sont pas toujours correctes. Nous n’avons pas eu le temps de faire le travail de propagation de l’information jusqu’à Flambda, mais il ne devrait pas y avoir de difficultés d’implémentation.</p>
<h5>Ordre sur les arguments</h5>
<p>Pour avoir une optimisation plus sûre, on voudrait pouvoir utiliser l’idée développée dans la partie théorique, qui rend l’optimisation correcte sur les objets non cycliques, et surtout qui nous redonne les garanties du typage pour éviter le problème vu dans l’exemple ci-dessus.</p>
<p>Afin d’avoir cette garantie, on veut changer la passe de simplification pour que son environnement contienne une option de couple fonction – argument. Quand cette option existe, le couple indique que nous sommes dans le corps d’une fonction, en train de la simplifier, et donc que les applications de la fonction sur des éléments plus petits que l’argument peuvent être simplifiés en une identité. Bien sûr, on devrait aussi modifier la passe pour se rappeler des éléments qui ne sont pas plus petits que l’argument.</p>
<h4>Pistes d’améliorations – long terme</h4>
<h5>Exclusion des objets cycliques</h5>
<p>Comme décrit dans la partie théorique, on pourrait déduire récursivement quels objets sont cycliques et tenter de les exclure de notre optimisation. Le problème est alors qu’au lieu de remplacer les fonctions par l’identité, on doit avoir une annotation spéciale qui représente <code>IdRec</code>.</p>
<p>Cela devient bien plus complexe à implémenter quand on compile entre plusieurs fichiers, puisqu’on doit alors avoir cette information dans l’interface des fichiers déjà compilés pour pouvoir faire l’optimisation quand c’est nécessaire.</p>
<p>Une piste serait d’utiliser les fichiers .cmx pour enregistrer cette information quand on compile un fichier, mais ce genre d’implémentation était trop longue pour être réalisée pendant le stage. De plus, il n’est même pas évident qu’elle soit un bon choix pratique : elle complexifierait beaucoup l’optimisation pour un avantage faible par rapport à une version correcte sur les objets non cycliques et activée par une annotation de l’utilisateur.</p>

