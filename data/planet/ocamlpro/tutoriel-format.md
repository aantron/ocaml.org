---
title: Tutoriel Format
description: "Article \xE9crit par Mattias. Le module Format d\u2019OCaml est un module
  extr\xEAmement puissant mais malheureusement tr\xE8s mal utilis\xE9. Il combine
  notamment deux \xE9l\xE9ments distincts : les bo\xEEtes d\u2019impression \xE9l\xE9gante\nles
  tags s\xE9mantiques Le pr\xE9sent article vise \xE0 d\xE9mystifier une grande partie
  ..."
url: https://ocamlpro.com/blog/2020_06_01_fr_tutoriel_format
date: 2020-06-01T13:31:53-00:00
preview_image: https://ocamlpro.com/assets/img/og_image_ocp_the_art_of_prog.png
authors:
- "\n    OCamlPro\n  "
source:
---

<p><em>Article écrit par Mattias.</em></p>
<p>Le module <a href="http://caml.inria.fr/pub/docs/manual-ocaml/libref/Format.html">Format</a> d’OCaml est un module extrêmement puissant mais malheureusement très mal utilisé. Il combine notamment deux éléments distincts :</p>
<ul>
<li>les boîtes d’impression élégante
</li>
<li>les tags sémantiques
</li>
</ul>
<p>Le présent article vise à démystifier une grande partie de ce module afin de découvrir l’ensemble des choses qu’il est possible de faire avec.</p>
<p>Si tout va bien vous devriez passer de</p>
<p><img src="https://ocamlpro.com/blog/assets/img/error1-output.png" alt="sortie triviale"></p>
<p>à</p>
<p><img src="https://ocamlpro.com/blog/assets/img/ocaml-output.png" alt="sortie OCaml"></p>
<p>(En réalité nous arriverons à un résultat légèrement différent car l’auteur de ce tutoriel n’aime pas tous les choix faits pour afficher les messages d’erreur en OCaml mais les différences n’auront pas de grande importance)</p>
<h2>I. Introduction générale : <code>fprintf fmt "%a" pp_error e</code></h2>
<p>Si vous ne comprenez pas ce que le code dans le titre doit faire, je
vous invite à lire attentivement ce qui va suivre. Sinon vous pouvez
directement sauter à la deuxième partie.</p>
<h3>I.1. Rappels sur <code>printf</code></h3>
<p>Pour rappel, la fonction <code>printf</code> est une fonction variadique (c’est-à-dire qu’elle peut prendre un nombre variable de paramètres).</p>
<ul>
<li>
<p>Le premier paramètre est une chaîne de formattage composée de caractères et de spécificateurs de format.</p>
<ul>
<li>Les <strong>caractères</strong> sont affichés tels quels. <code>printf "abc"</code> affichera <code>abc</code>.
</li>
<li>Les <strong>spécificateurs de caractère</strong> sont des caractères précédés du caractère <code>% </code>(syntaxe héritée du C). Ils sont remplacés à l’exécution par un des paramètres fournis après la chaîne de formattage à la fonction et servent à indiquer de quel type doit être la valeur qui sera affichée (ainsi que d’autres informations dont les détails peuvent être trouvés dans la documentation du module <a href="https://caml.inria.fr/pub/docs/manual-ocaml/libref/Printf.html">Printf</a>. <code>printf "Test: %d"</code> attend un entier signé et affichera <code>Test: &lt;d&gt;</code> avec <code>&lt;d&gt;</code> remplacé par l’entier fourni.
</li>
</ul>
</li>
<li>
<p>Les paramètres suivants sont les valeurs fournies à <code>printf</code> pour remplacer les spécificateurs de format</p>
<ul>
<li><code>printf "%d %s %c" 3 s 'a'</code> affichera l’entier signé 3, une espace insécable, le contenu de la variable <code>s</code> qui doit être une chaîne de caractères, une autre espace insécable et finalement le caractère ‘a’.
</li>
<li>On remarque aussi qu’ici le nombre de paramètres supplémentaires fournis en plus de la chaîne de formattage correspond au nombre de spécificateurs et que ceux-ci ne peuvent être intervertis. <code>printf "%d %c" 'a' 3</code> ne pourra pas être compilé/exécuté car <code>%d</code> attend un entier signé et le premier paramètre est un caractère. Les spécificateurs qui n’attendent qu’un argument sont des spécificateurs que j’appelle <strong>unaires</strong> et sont extrêmement faciles à utiliser, il faut seulement savoir quel caractère correspond à quel type et les donner dans le bon ordre comme
illustré dans la figure ci-dessous (le chevron représentant la sortie standard)
</li>
</ul>
</li>
</ul>
<p><img src="https://ocamlpro.com/blog/assets/img/printf-base-out-dark.png" alt="Fonctionnement basique de printf"></p>
<h3>I.2. Afficher un type défini par l’utilisateur</h3>
<p>Arrive alors ce moment où vous commencez à définir vos propres structures de données et, malheureusement, il n’y a aucun moyen d’afficher votre expression avec les spécificateurs par défaut (ce qui semble normal). Définissons donc notre propre type et affichons-le avec les techniques déjà vues :</p>
<pre><code class="language-OCaml">type error =
  | Type_Error of string * string
  | Apply_Non_Function of string

let pp_error = function
  | Type_Error (s1, s2) -&gt; printf "Type is %s instead of %s" s1 s2
  | Apply_Non_Function s -&gt; printf "Type is %s, this is not a function" s
</code></pre>
<p>Supposons maintenant que nous ayons une liste d’erreurs et que nous souhaitions les afficher en les séparant par une ligne horizontale. Une première solution serait la suivante :</p>
<pre><code class="language-OCaml">let pp_list l =
  List.iter (fun e -&gt;
      pp_error e;
      printf "\n"
    ) l
</code></pre>
<p>Cette façon de faire a plusieurs inconvénients (qui vont être magiquement réglés par la fonction du titre).</p>
<h3>I.3. Afficher sur un <code>formatter</code> abstrait</h3>
<p>Le premier inconvénient est que <code>printf</code> envoie son résultat vers la sortie standard alors qu’on peut vouloir l’envoyer vers un fichier ou vers la sortie d’erreur, par exemple.</p>
<p>La solution est <code>fprintf</code> (il serait de bon ton de feindre la surprise ici).</p>
<p><code>fprintf</code> prend un paramètre supplémentaire avant la chaîne de formattage appelé <strong><code>formatter</code> abstrait</strong>. Ce paramètre est du type <code>formatter</code> et représente un imprimeur élégant (ou <em>pretty-printer</em>)</p>
<p>c’est-à-dire l’objet vers lequel le résultat devra être envoyé. L’énorme avantage qui en découle est qu’on peut transformer beaucoup de choses en <code>formatter</code>. Un fichier, un buffer, la sortie standard etc. À vrai dire, <code>printf</code> est implémenté comme <code>let printf = fprintf std_formatter</code></p>
<p>Pour l’utiliser on va donc modifier <code>pp_error</code> et lui donner un paramètre supplémentaire :</p>
<pre><code class="language-OCaml">let pp_error fmt = function
  | Type_Error (s1, s2) -&gt; fprintf fmt "Type is %s instead of %s" s1 s2
  | Apply_Non_Function s -&gt; fprintf fmt "Type is %s, this is not a function" s
</code></pre>
<p>Puis on réécrit <code>pp_list</code> pour prendre cela en compte :</p>
<pre><code class="language-OCaml">let pp_list fmt l =
  List.iter (fun e -&gt;
      pp_error fmt e;
      fprintf fmt "\n"
    ) l
</code></pre>
<p>Comme on peut le voir dans la figure ci-dessous, <code>fprintf</code> imprime dans le <code>formatter</code> qui lui est fourni en paramètre et non plus sur la sortie standard.</p>
<p><img src="https://ocamlpro.com/blog/assets/img/fprintf-base-out-dark.png" alt="Fontionnement basique de fprintf"></p>
<p>Si on veut maintenant afficher le résultat sur la sortie standard il suffira simplement de donner <code>pp_list std_formatter</code> comme <code>formatter</code> à <code>fprintf</code>. Cette façon de faire n’a, en réalité, que des avantages, puisqu’elle permet d’être beaucoup plus fexible quant au <code>formatter</code> qui sera utilisé à l’exécution du programme.</p>
<h3>I.4. Afficher des types complexes avec <code>%a</code></h3>
<p>Le deuxième problème arrivera bien assez vite si nous continuons avec cette méthode. Pour bien le comprendre, reprenons <code>pp_error</code>. Dans le cas de <code>Type_error of string * string</code> on veut écrire <code>Type is s1 instead of s2</code> et on fournit donc à <code>fprintf</code> la chaîne de formattage <code>"Type is %s instead of %s"</code> avec <code>s1</code> et <code>s2</code> en paramètres supplémentaires. Comment devrions-nous faire si <code>s1</code> et <code>s2</code> étaient des types définis par l’utilisateur avec chacun leur fonction d’affichage <code>pp_s1`` : formatter -&gt; s1 -&gt; unit</code> et <code>pp_s2 : formatter -&gt; s2 -&gt; unit</code> ? En suivant la logique de notre solution jusqu’ici, nous écririons le code suivant :</p>
<pre><code class="language-OCaml">let pp_error fmt = function
  | Type_Error (s1, s2) -&gt;
    fprintf fmt "Type is ";
    pp_s1 fmt s1;
    fprintf fmt "instead of ";
    pp_s2 fmt s2
  | Apply_non_function s -&gt; fprintf fmt "Type is %s, this is not a function" s
</code></pre>
<p>Il est assez facile de se rendre compte rapidement que plus nous devrons manipuler des types complexes, plus cette syntaxe s’alourdira. Tout cela parce que les spécificateurs de caractère unaires ne permettent de manipuler que les types de base d’OCaml.</p>
<p>C’est là qu’entre en jeu <code>%a</code>. Ce spécificateur de caractère est, lui, binaire (ternaire en réalité mais un de ses paramètres est déjà fourni). Ses paramètres sont :</p>
<ul>
<li>Une fonction d’affichage de type <code>formatter -&gt; 'a -&gt; unit</code> (premier paramètre devant être fourni)
</li>
<li>Le <code>formatter</code> dans lequel il doit afficher son résultat (qui ne doit pas être fourni en plus)
</li>
<li>La valeur qu’on souhaite afficher
</li>
</ul>
<p>Il appliquera ensuite le <code>formatter</code> et la valeur à la fonction fournie comme premier argument et lui donner la main pour qu’elle affiche ce qu’elle doit dans le <code>formatter</code> qui lui a été fourni en paramètre. Lorsqu’elle aura terminé, l’impression continuera. L’exemple suivant montre le fonctionnement (avec une impression sur la sortie standard, <code>fmt</code> ayant été remplacé par <code>std-formatter</code></p>
<p><img src="https://ocamlpro.com/blog/assets/img/fprintfpa-base-out-dark.png" alt=""></p>
<p>Dans notre cas nous avions déjà transformé nos fonctions d’affichage pour qu’elles prennent un <code>formatter</code> abstrait et nous n’avons donc presque rien à modifier :</p>
<pre><code class="language-OCaml">let pp_error fmt = function
  | Type_Error (s1, s2) -&gt; fprintf fmt "Type is %s instead of %s" s1 s2
  | Apply_Non_Function s -&gt; fprintf fmt "Type is %s, this is not a function" s

let pp_list fmt l =
  List.iter (fun e -&gt;
      fprintf fmt "%a\n" pp_error e;
    ) l
</code></pre>
<p>Et, bien sûr, si <code>s1</code> et <code>s2</code> avaient eu leurs propres fonctions d’affichage :</p>
<pre><code class="language-ocaml">let pp_error fmt = function
  | Type_Error (s1, s2) -&gt; fprintf fmt "Type is %a instead of %a" pp_s1 s1 pp_s2 s2
  | Apply_Non_Function s -&gt; fprintf fmt "Type is %s, this is not a function" s
</code></pre>
<p>Arrivé-e-s ici vous devriez être à l’aise avec les notions de <code>formatter</code> abstrait et de spécificateur de caractère binaire et vous devriez donc pouvoir afficher n’importe quelle structure de donnée, même récursive, sans aucun soucis. Je recommande vivement cette façon de faire afin que tout changement qui devrait succéder ne nécessite pas de changer l’intégralité du code.</p>
<h2>II. Les boîtes d’impression élégante</h2>
<p>Et pour justement avoir des changements qui ne nécessitent pas de tout modifier, il va falloir s’intéresser un minimum aux boîtes d’impression élégante.</p>
<p>Aussi appelées <em>pretty-print boxes</em>, je les appellerai “boîtes” dorénavant, un <a href="https://ocaml.org/learn/tutorials/format.fr.html">tutoriel</a> existe déjà, fait par l’équipe de la bibliothèque standard.</p>
<p>L'idée derrière les boîtes est tout simple :</p>
<blockquote>
<p>À mon niveau je m’occupe correctement de comment afficher mes éléments et je n’impose rien au-dessus.</p>
</blockquote>
<p>Reprenons, par exemple, la fonction permettant d’afficher les <code>error</code>:</p>
<pre><code class="language-ocaml">let pp_error fmt = function
  | Type_Error (s1, s2) -&gt; fprintf fmt "Type is %s instead of %s" s1 s2
  | Apply_Non_Function s -&gt; fprintf fmt "Type is %s, this is not a function" s
</code></pre>
<p>Si on ajoutait un retour à la ligne on imposerait à toute fonction nous appelant ce saut de ligne or ce n’est pas à nous d’en décider. Cette fonction, en l’état, fait parfaitement ce qu’elle doit faire.</p>
<p>Regardons, par contre, la fonction affichant une liste d’erreur :</p>
<pre><code class="language-ocaml">let pp_list fmt l =
  List.iter (fun e -&gt;
      fprintf fmt "%a\n" pp_error e;
    ) l
</code></pre>
<p>A l’issue de celle-ci un saut à ligne provenant du dernier élément est forcé. Non seulement il n’est pas recommandé d’utiliser <code>n</code> (ou <code>@n</code> ou même <code>@.</code>) car ce ne sont pas à proprement parler des directives de <code>Format</code> mais des directives systèmes qui vont donc chambouler le reste de l’impression.</p>
<blockquote>
<p>Malheureusement bien trop de développeurs et développeuses ont découvert <code>@.</code> en même temps que <code>Format</code> et s’en servent sans restriction. Au risque de me répéter souvent : n’utilisez pas <code>@.</code> !</p>
</blockquote>
<h3>II.1. Le spécificateur <code>@</code></h3>
<p>On l’avait vu, une chaîne de formattage est composée de caractères et de spécificateurs de caractères commençant par <code>%</code> Les spécificateurs sont des caractères qui ne sont pas affichés et qui seront remplacés avant l’affichage final.</p>
<p><code>Format</code> ajoute son propre spécificateur de caractère : <code>@</code>.</p>
<h4>II.1.a. Le vidage (<em>flush</em>)</h4>
<p>La première spécification qu’on a vue est donc celle qu’il ne faut  presque jamais utiliser (ce qui pose la question de l’avoir mentionnée en premier lieu) : <code>@.</code>. Cette spécification indique seulement au moteur d’impression qu’à ce niveau là il faut sauter une ligne et vider l’imprimeur. Les deux autres spécifications semblables sont <code>@n</code> qui n’indique que le saut de ligne et <code>@?</code> qui n’indique que le vidage de l’imprimeur. L’inconvénient de ces trois spécificateurs est qu’ils sont trop puissants et chamboulent donc le bon fonctionnement du reste de l’impression. Je n’ai personnellement jamais utilisé <code>@n</code> (autant utiliser une boîte avec un spécificateur de coupure comme nous le verrons immédiatement après) et n’utilise <code>@. </code>que lorsque je sais qu’il ne reste rien à imprimer.</p>
<h4>II.1.b. Les indications de coupure ou d’espace</h4>
<p>Important :</p>
<ul>
<li>Une indication de coupure saute à la ligne s’il le faut sinon elle ne fait rien
</li>
<li>Une indication d’espace sécable saute à la ligne s’il le faut, sinon elle affiche une espace
</li>
</ul>
<p>Les deux sont donc des indications de saut de ligne si nécessaire, il
n’existe pas d’indication d’espace par défaut ou rien s’il n’y a pas
assez d’espace (utiliser <code> </code> affichera toujours une espace).</p>
<p>Les indications sont au nombre de trois (et leur fonctionnement sera bien plus clair lorsque vous verrez les boîtes) :</p>
<ul>
<li><code>@,</code> : indication de coupure (c’est-à-dire rien prioritairement ou un saut à la ligne s’il le faut)
</li>
<li><code>@⎵</code> : indique une espace sécable (c’est-à-dire une espace prioritairement
ou un saut à la ligne s’il le faut) (Il faut bien évidemment comprendre
le caractère <code>⎵</code> comme l’espace blanc habituel)
</li>
<li><code>@;&lt;n o&gt;</code> : indique <code>n</code> espaces sécables ou une coupure indentée de <code>o</code> (c’est-à-dire <code>n</code> espaces sécables prioritairement ou un saut à la ligne <strong>avec une indentation supplémentaire de <code>o</code></strong> s’il le faut)
</li>
</ul>
<p>D’après ce que je viens d’écrire il devrait être évident maintenant que le caractère est une espace insécable qui ne provoquera donc pas de saut à la ligne quand bien même on dépasserait les limites de celle-ci. Contrairement à nos espaces de traitement de texte habituel qui sont des espaces sécables (pouvant provoquer des sauts de ligne), il faut spécifier quels espaces sont sécables lorsqu’on utilise Format.</p>
<p>On écrira par exemple <code>fprintf fmt "let rec f =@ %a" pp_expr e</code> car on ne veut pas que <code>let rec f =</code> soit séparé en plusieurs lignes mais on met bien <code>@⎵</code> avant <code>%a</code> car l’expression sera soit sur la même ligne si suffisament petite soit à la ligne suivante si trop grande (on devrait même écrire <code>@;&lt;1 2&gt;</code> pour que l’expression soit indentée si on saute à la ligne suivante mais, on va le voir immédiatement, c’est là que les boîtes nous permettent d’automatiser ce genre de comportement)</p>
<h4>II.1.c. Les boîtes</h4>
<p>La deuxième spécification est celle permettant d’ouvrir et de fermer des boîtes.</p>
<p>Une boîte se commence par <code>@[</code> et se termine par <code>@]</code>. Entre ces deux bornes, on fait ce qu’on veut (<strong>sauf utiliser <code>@.</code>, <code>@?</code> ou <code>@\n</code> !</strong>). Tout ce qui se passe à l’intérieur de la boîte reste (et doit rester) à l’intérieur de celle-ci. Indentation, coupures, boîtes verticales, horizontales, les deux, l’une ou l’autre, toutes ces options sont accessibles une fois qu’une boîte a été ouverte. Voyons-les rapidement (pour rappel, la version détaillée est disponible dans le <a href="https://ocaml.org/learn/tutorials/format.fr.html">tutoriel</a>.</p>
<p>Une fois qu’une boîte a été ouverte on peut préciser entre deux chevrons le comportement qu’on veut qu’elle ait en cas d’indication de coupure, en voici un rapide aperçu :</p>
<ul>
<li><code>&lt;v&gt;</code> : Toute indication de coupure entraîne un saut à la ligne
</li>
<li><code>&lt;h&gt;</code> : Toute indication d’espace entraîne une espace, les indications de coupure n’ont aucun effet
</li>
<li><code>&lt;hv&gt;</code>: Si toute la boîte peut être imprimée sur la même ligne alors seules les indications d’espace sont prises en compte sinon seules les indications de coupure le sont et chaque élément est imprimé sur sa propre ligne
</li>
<li><code>&lt;hov&gt;</code> : Tant que des éléments peuvent être imprimés sur une ligne ils le sont avec leurs indications d’espace. Les indications de coupure sont utilisées lorsqu’il faut sauter une ligne.
</li>
</ul>
<p>Chacun de ces comportements peut se voir attribuer une valeur supplémentaire, sa valeur d’indentation, qui indique l’indentation par rapport au début de la boîte qui devra être ajoutée à chaque saut de ligne.</p>
<p>Soit le code suivant permettant d’afficher une liste d’items séparés soit par une indication de coupure <code>@,</code>, soit par une indication d’espace <code>@⎵</code> soit par une indication d’espace ou de coupure indentée <code>@;&lt;2 3&gt;</code> (2 espaces ou une coupure indentée de trois espaces) :</p>
<pre><code class="language-ocaml">open Format

let l = ["toto"; "tata"; "titi"]

let pp_item fmt s = fprintf fmt "%s" s

let pp_cut fmt () = fprintf fmt "@,"
let pp_spc fmt () = fprintf fmt "@ "
let pp_brk fmt () = fprintf fmt "@;&lt;2 3&gt;"


let pp_list pp_sep fmt l =
  pp_print_list pp_item ~pp_sep fmt l
</code></pre>
<p>Voici un récapitulatif des différents comportements de boîtes en fonction des indications de coupure/espace rencontrées :</p>
<pre><code class="language-ocaml">(* Boite verticale (tout est coupure) *)
printf "------------@.";
printf "v@.";
printf "------------@.";
printf "@[&lt;v 2&gt;[%a]@]@." (pp_list pp_cut) l;
printf "@[&lt;v 2&gt;[%a]@]@." (pp_list pp_spc) l;
printf "@[&lt;v 2&gt;[%a]@]@." (pp_list pp_brk) l;
(* Sortie attendue:
------------
v
------------
[toto
  tata
  titi]
[toto
  tata
  titi]
[toto
     tata
     titi]
*)


(* Boîte horizontale (pas de coupure) *)
printf "------------@.";
printf "h@.";
printf "------------@.";
printf "@[&lt;h 2&gt;[%a]@]@." (pp_list pp_cut) l;
printf "@[&lt;h 2&gt;[%a]@]@." (pp_list pp_spc) l;
printf "@[&lt;h 2&gt;[%a]@]@." (pp_list pp_brk) l;
(* Sortie attendue:
------------
h
------------
[tototatatiti]
[toto tata titi]
[toto  tata  titi]
*)


(* Boîte horizontale-verticale
  (Affiche tout sur une ligne si possible sinon boîte verticale) *)
printf "------------@.";
printf "hv@.";
printf "------------@.";
printf "@[&lt;hv 2&gt;[%a]@]@." (pp_list pp_cut) l;
printf "@[&lt;hv 2&gt;[%a]@]@." (pp_list pp_spc) l;
printf "@[&lt;hv 2&gt;[%a]@]@." (pp_list pp_brk) l;
(* Sortie attendue:
------------
hv
------------
[toto
  tata
  titi]
[toto
  tata
  titi]
[toto
     tata
     titi]
*)


(* Boîte horizontale ou verticale tassante
  (Affiche le maximum possible sur une ligne avant de sauter à la
   ligne suivante et recommencer) *)
printf "------------@.";
printf "hov@.";
printf "------------@.";
printf "@[&lt;hov 2&gt;[%a]@]@." (pp_list pp_cut) l;
printf "@[&lt;hov 2&gt;[%a]@]@." (pp_list pp_spc) l;
printf "@[&lt;hov 2&gt;[%a]@]@." (pp_list pp_brk) l;
(* Sortie attendue:
------------
hov
------------
[tototata
  titi]
[toto tata
  titi]
[toto
     tata
     titi]
*)

(*  Boîte horizontale ou verticale structurelle
   (Même fonctionnement que la boîte tassante sauf pour le dernier
    retour à la ligne qui tente de favoriser une indentation de
    niveau 0) *)
printf "------------@.";
printf "b@.";
printf "------------@.";
printf "@[&lt;b 2&gt;[%a]@]@." (pp_list pp_cut) l;
printf "@[&lt;b 2&gt;[%a]@]@." (pp_list pp_spc) l;
printf "@[&lt;b 2&gt;[%a]@]@." (pp_list pp_brk) l
(* Sortie attendue:
------------
b
------------
[tototata
  titi]
[toto tata
  titi]
[toto
     tata
     titi]
*)
</code></pre>
<p>Petite précision sur l’utilisation ici des <code>@.</code> alors qu’il est recommandé de ne jamais les utiliser. Il ne faut en réalité pas <strong>jamais</strong> les utiliser, il faut seulement les utiliser lorsqu’on est sûr de n’être dans aucune boîte. Ici, par exemple, on souhaite marquer distinctement les différentes impressions de boîtes, il est donc tout à fait correct d’utiliser <code>@.</code> étant donné qu’on est sûr d’être au dernier niveau d’impression (rien au-dessus) et de ne pas casser une passe d’impression élégante. Il serait donc bien plus précis de dire</p>
<blockquote>
<p>Il ne faut pas utiliser <code>@.</code>, <code>@n</code> et <code>@?</code> dans des impressions qui sont ou seront potentiellement imbriquées</p>
</blockquote>
<p>Mais il est bien plus simple pour commencer de ne jamais les utiliser quitte à les rajouter après.</p>
<p>Le comportement de la boîte <code>b</code> (boîte structurelle) semble être le même que celui de la boîte <code>hov</code> (boîte tassante) mais il se trouve des cas où les deux diffèrent (généralement lorsqu’un saut de ligne réduit l’indentation courante, la boîte structurelle saute à la ligne même s’il reste de la place sur la ligne courante). Je vous invite à consulter le <a href="https://ocaml.org/learn/tutorials/format.fr.html">tutoriel</a> pour plus de précisions (je dois aussi avouer que leur fonctionnement est très proche de ce qu’on pourrait appeler “opaque” étant donné qu’en fonction de la taille de marge le comportement attendu aura lieu ou non. L’auteur de ce tutoriel tient à préciser qu’il utilise plutôt des boîtes verticales avec une indentation nulle s’il lui arrive de vouloir obtenir le comportement des boîtes structurelles, un exemple est fourni lors de l’affichage en HTML à la fin de ce document).</p>
<h3>II.2. Récapitulatif</h3>
<ul>
<li>Il faut utiliser des boîtes
</li>
<li>Les indications de vidage fermant toutes les boîtes, il ne faut surtout pas les utiliser dans des fonctions d’affichage internes, il faut se limiter aux indications de
coupure et d’espace
</li>
<li>Il faut vraiment utiliser des boîtes
</li>
</ul>
<p>Vous voilà armé-e-s pour utiliser Format dans sa version la plus simple, avec des boîtes, de l’indentation, des indications de coupure et d’espace.</p>
<p>Reprenons notre affichage d’erreur :</p>
<pre><code class="language-ocaml">let pp_error fmt = function
  | Type_Error (s1, s2) -&gt; fprintf fmt "@[&lt;hov 2&gt;Type is %s@ instead of %s@]" s1 s2
  | Apply_non_function s -&gt; fprintf fmt "@[&lt;hov 2&gt;Type is %s,@ this is not a function@]" s

let pp_list fmt l =
  pp_print_list pp_error fmt l
</code></pre>
<p>On a encapsulé l’affichage des deux erreurs dans des boîtes <code>hov</code> avec une indication d’espace sécable au milieu et utilisé la fonction <code>pp_print_list</code> du module <a href="https://caml.inria.fr/pub/docs/manual-ocaml/libref/Format.html">Format</a></p>
<p>Si je tente maintenant d’afficher une liste d’erreurs dans deux environnements, un de 50 colonnes et l’autre de 25 colonnes de largeur avec le code suivant :</p>
<pre><code class="language-ocaml">let () =
  let e1 = Type_Error ("int", "bool") in
  let e2 = Apply_non_function ("int") in
  let e3 = Type_Error ("int", "float") in
  let e4 = Apply_non_function ("bool") in

  let el = [e1; e2; e3; e4] in
  pp_set_margin std_formatter 50;
  fprintf std_formatter "--------------------------------------------------@.";
  fprintf std_formatter "@[&lt;v 0&gt;%a@]@." pp_list el;
  pp_set_margin std_formatter 25;
  fprintf std_formatter "-------------------------@.";
  fprintf std_formatter "@[&lt;v 0&gt;%a@]@." pp_list el;
</code></pre>
<p>J’obtiens le résultat suivant :</p>
<pre><code class="language-ocaml">--------------------------------------------------
Type is int instead of bool
Type is int, this is not a function
Type is int instead of float
Type is bool, this is not a function
-------------------------
Type is int
  instead of bool
Type is int,
  this is not a function
Type is int
  instead of float
Type is bool,
  this is not a function
</code></pre>
<p>Ce qu’on rajoute en verbosité on le gagne en élégance. Et en parlant d’élégance, ça manque de couleurs.</p>
<h2>III. Les tags sémantiques</h2>
<p>Cette partie n’est pas présente dans le tutoriel mais dans un <a href="https://hal.archives-ouvertes.fr/hal-01503081/file/format-unraveled.pdf">article tutoriel</a> qui l’explique assez rapidement.</p>
<p>La troisième spécification, donc (après celles de coupure et de boîtes), est la spécification de tag sémantique : <code>@{</code> pour en ouvrir un et <code>@}</code> pour le fermer.</p>
<h3>III.1. Marquer son texte</h3>
<p>Mais avant de comprendre leur fonctionnement, cherchons à comprendre leur intérêt. Que vous souhaitiez afficher dans un terminal, dans une page html ou autre, il y a de fortes chances que cette sortie accepte les marques de texte comme l’italique, la coloration etc. Utilisateur d’emacs et d’un <a href="https://en.wikipedia.org/wiki/ANSI_escape_code">terminal ANSI</a> je peux modifier l’apparence de mon texte grâce aux codes ANSI :</p>
<p><img src="https://ocamlpro.com/blog/assets/img/exemple-ansiterm.png" alt="Exemple de marquage de texte dans un terminal ANSI"></p>
<p>Si je crée un programme OCaml qui affiche cette chaîne de charactère et que je l’exécute directement dans mon terminal je devrais obtenir le même résultat :</p>
<p><img src="https://ocamlpro.com/blog/assets/img/exemple-ansiocaml-bad.png" alt="Exemple de marquage de texte dans un terminal ANSI depuis un programme OCaml qui se passe mal"></p>
<p>Naturellement, ça ne fonctionne pas, si l’informatique était standardisée et si tout le monde savait communiquer ça se saurait. Il s’avère que le caractère <code>033</code> est interprété en octal par les terminaux ANSI mais en décimal par OCaml (ce qui semble être l’interprétation normale). OCaml permet de représenter un <a href="https://caml.inria.fr/pub/docs/manual-ocaml/lex.html#sss:character-literals">caractère</a> selon plusieurs séquences d’échappement différentes :</p>
<table><thead><tr><th>Séquence</th><th>Caractère résultant</th></tr></thead><tbody><tr><td><code>DDD</code></td><td>le caractère correspondant au code ASCII <code>DDD</code> en décimal</td></tr><tr><td><code>xHH</code></td><td>le caractère correspondant au code ASCII <code>HH</code> en hexadécimal</td></tr><tr><td><code>oOOO</code></td><td>le caractère correspondant au code ASCII <code>OOO</code> en octal</td></tr></tbody></table>
<p>On peut donc écrire au choix</p>
<pre><code class="language-ocaml">let () = Format.printf "\027[36mBlue Text \027[0;3;30;47mItalic WhiteBG Black Text"
let () = Format.printf "\x1B[36mBlue Text \x1B[0;3;30;47mItalic WhiteBG Black Text"
let () = Format.printf "\o033[36mBlue Text \o033[0;3;30;47mItalic WhiteBG Black Text"
</code></pre>
<p>Dans tous les cas, on obtient le résultat suivant :</p>
<p><img src="https://ocamlpro.com/blog/assets/img/exemple-ansiocaml-good.png" alt="Exemple de marquage de texte dans un terminal ANSI depuis un programme OCaml qui se passe bien"></p>
<p>Que se passe-t-il, par contre, si j’exécute une de ces lignes dans un terminal non ANSI ? En testant sur <a href="https://try.ocamlpro.com/">TryOCaml</a>:</p>
<p><img src="https://ocamlpro.com/blog/assets/img/tryocaml-ansi.png" alt="Exemple de marquage de texte dans un navigateur depuis TryOCaml"></p>
<p>On ne veut surtout pas que ce genre d’affichage puisse arriver. Il faudrait donc pouvoir s’assurer que le marquage du texte soit actif uniquement quand on le décide. L’idée de créer deux chaînes de formattage en fonction de notre capacité ou non à afficher du texte marqué n’est clairement pas une bonne pratique de programmation (changer
une formulation demande de changer deux chaînes de formattage, le code est difficilement maintenable). Il faudrait donc un outil qui puisse faire un pré-traitement de notre chaîne de formattage pour lui ajouter des décorations.</p>
<p>Cet outil est déjà fourni par Format, ce sont les tags sémantiques.</p>
<h3>III.2 Les tags sémantiques</h3>
<p>Introduits par <code>@{</code> et fermés par <code>@}</code>, comme les boîtes ils sont paramétrés par la construction <code>&lt;t&gt;</code> pour indiquer l’ouverture (et la fermeture) du tag <code>t</code>. Contrairement aux boîtes, les tags n’ont aucune signification pour l’imprimeur (on peut faire l’analogie avec les types de base d’OCaml que sont <code>int</code>, <code>bool</code>, <code>float</code> etc et les types définis par le programmeur ou la programmeuse (<code>type t = A | B</code>, par exemple. Les types de base ont déjà une quantité de fonctions qui leurs sont associés alors que les types définis ne signifient rien tant qu’on n’écrit pas les fonctions qui les manipuleront). L’avantage premier de ces tags est donc que, n’ayant aucune signification, ils sont tout simplement ignorés par l’imprimeur lors de l’affichage de notre chaîne de caractère finale:</p>
<p><img src="https://ocamlpro.com/blog/assets/img/exemple-stag-tryocaml.png" alt="Exemple de marquage avec un tag sémantique de texte dans un navigateur depuis TryOCaml"></p>
<p>Par défaut, l’imprimeur ne traite pas les tags sémantiques (ce qui
permet d’avoir un comportement d’affichage aussi simple que possible par
défaut). Le traitement des tags sémantiques peut être activé pour
chaque <code>formatter</code> indépendamment avec les fonctions <code>val pp_set_tags : formatter -&gt; bool -&gt; unit</code>, <code>val pp_set_print_tags : formatter -&gt; bool -&gt; unit</code> et <code>val pp_set_mark_tags : formatter -&gt; bool -&gt; unit</code> dont on verra les effets immédiatement. Voyons déjà ce qui se passe avec la fonction générale <code>pp_set_tags</code> qui combine les deux suivantes :</p>
<p><img src="https://ocamlpro.com/blog/assets/img/exemple-stag-actif-tryocaml.png" alt="Exemple de traitement du marquage avec un tag sémantique de texte dans un navigateur depuis TryOCaml"></p>
<p>Que s’est-il passé ?</p>
<p>Une fois que le traitement des tags sémantiques est activé, quatre
opérations vont être effectuées à chaque ouverture et fermeture de tag :</p>
<ul>
<li><code>print_open_stag</code> suivie de <code>mark_open_stag</code> pour chaque tag <code>t</code> ouvert avec <code>@{&lt;t&gt;</code>
</li>
<li><code>mark_close_stag</code> suivie de <code>print_close_stag</code> pour chaque tag <code>t</code> fermé avec <code>@}</code> correspondant à la dernière ouverture <code>@{&lt;t&gt;</code>
</li>
</ul>
<p>Regardons les signatures de ces quatre opérations :</p>
<pre><code class="language-ocaml">type formatter_stag_functions = {
    mark_open_stag : stag -&gt; string;
       mark_close_stag : stag -&gt; string;
       print_open_stag : stag -&gt; unit;
       print_close_stag : stag -&gt; unit;
}
</code></pre>
<p>Les fonctions <code>mark_*_stag</code> prennent un tag sémantique en paramètre et renvoie une chaîne de caractères quand les fonctions <code>print_*_stag</code> prennent le même paramètre mais ne renvoient rien. La raison derrière est en réalité toute simple :</p>
<ul>
<li>Les fonctions de marquage écrivent directement dans la cible d’affichage (le terminal, le fichier ou autre)
</li>
<li>Les fonctions d’affichage écrivent dans le <code>formatter</code> qui les traite comme des chaînes de caractères normales qui peuvent donc entraîner des sauts de ligne, des coupures, de nouvelles boîtes etc
</li>
</ul>
<p>Une indication de couleur pour un terminal ANSI n’apparaît pas à l’affichage, le texte se retrouve coloré, il semble donc naturel de ne pas vouloir que cette indication ait un effet sur l’impression élégante. En revanche, si on voulait avoir une sortie vers un fichier LaTeX ou HTML, cette indication de couleur apparaîtraît et devrait donc avoir une influence sur l’impression élégante.</p>
<p>Il est donc assez simple de savoir dans quel cas on veut utiliser <code>print_*_stag</code> ou <code>mark_*_stag</code> :</p>
<ul>
<li>Si le tag doit avoir un impact immédiat sur l’apparence du texte affiché (couleur, taille, décorations…) et non pas son contenu, il faut utiliser <code>mark_*_stag</code>
</li>
<li>Si le tag doit avoir un impact sur le contenu du texte affiché et non pas sur son apparence, il faut utiliser <code>print_*_stag</code>
</li>
<li>Si le tag doit avoir un impact à la fois sur le contenu et l’apparence du texte affiché alors il faut utiliser les deux en séparant bien entre contenu géré par <code>print_*_stag</code> et apparence gérée par <code>mark_*_stag</code>
</li>
</ul>
<p>Ces quatres fonctions ont chacune un comportement par défaut que voici :</p>
<pre><code class="language-ocaml">let mark_open_stag = function
  | String_tag s -&gt; "&lt;" ^ s ^ "&gt;"
  | _ -&gt; ""
let mark_close_stag = function
  | String_tag s -&gt; "&lt;/" ^ s ^ "&gt;"
  
let print_open_stag = ignore
let print_close_stag = ignore
</code></pre>
<p>Le type <code>stag</code> est un type somme extensible (introduits dans <a href="https://ocaml.org/releases/4.02.html">OCaml 4.02.0</a>) c’est-à-dire qu’il est défini de la sorte</p>
<pre><code class="language-ocaml">type stag = ..

type stag += String_tag of string
</code></pre>
<p>Par défaut seuls les <code>String_tag of string</code> sont donc reconnus comme des tags sémantiques (ce sont aussi les seuls qui peuvent être obtenus par la construction <code>@{&lt;t&gt; ... @}</code>, ici <code>t</code> sera traité comme <code>String_tag t</code>) ce qui est illustré par le comportement par défaut de <code>mark_open_tag</code> et <code>mark_close_tag</code>. Ce comportement par défaut nous permet aussi de comprendre ce qui est arrivé ici :</p>
<p><img src="https://ocamlpro.com/blog/assets/img/exemple-stag-actif-tryocaml_002.png" alt="Exemple de traitement du marquage avec un tag sémantique de texte dans un navigateur depuis TryOCaml"></p>
<p>N’ayant pas personnalisé les opérations de manipulation des tags, leur comportement par défaut a été exécuté, ce qui revient à afficher directement le tag entre chevrons sans passer par le <code>formatter</code>. Il faut donc définir les comportements voulus pour nos tags (attention, ne manipulant que des chaînes de caractère, toute erreur est conséquemment difficile à identifier et corriger, il vaut mieux donc éviter les célèbres <code>| _ -&gt; ()</code> — il faudrait en réalité les éviter tout le temps si possible mais c’est une autre histoire).</p>
<p>Commençons donc par définir nos tags et ce à quoi on veut qu’ils correspondent :</p>
<pre><code class="language-ocaml">open Format

type style =
  | Normal

  | Italic
  | Italic_off

  | FG_Black
  | FG_Blue
  | FG_Default

  | BG_White
  | BG_Default

let close_tag = function
  | Italic -&gt; Italic_off
  | FG_Black | FG_Blue | FG_Default -&gt; FG_Default

  | BG_White | BG_Default -&gt; BG_Default

  | _ -&gt; Normal

let style_of_tag = function
  | String_tag s -&gt; begin match s with
      | "n" -&gt; Normal
      | "italic" -&gt; Italic
      | "/italic" -&gt; Italic_off

      | "fg_black" -&gt; FG_Black
      | "fg_blue" -&gt; FG_Blue
      | "fg_default" -&gt; FG_Default

      | "bg_white" -&gt; BG_White
      | "bg_default" -&gt; BG_Default

      | _ -&gt; raise Not_found
    end
  | _ -&gt; raise Not_found
</code></pre>
<p>Maintenant que chaque tag possible est géré, il nous faut les associer à leur valeur (ANSI dans ce cas) et implémenter nos propres fonctions de marquages (et pas d’affichage car a priori ces tags n’ont aucun effet sur le contenu du texte affiché) :</p>
<pre><code class="language-ocaml">(* See https://en.wikipedia.org/wiki/ANSI_escape_code#SGR_parameters for some values *)
let to_ansi_value = function
  | Normal -&gt; "0"

  | Italic -&gt; "3"
  | Italic_off -&gt; "23"

  | FG_Black -&gt; "30"
  | FG_Blue -&gt; "34"
  | FG_Default -&gt; "39"

  | BG_White -&gt; "47"
  | BG_Default -&gt; "49"

let ansi_tag = Printf.sprintf "\x1B[%sm"

let start_mark_ansi_stag t = ansi_tag @@ to_ansi_value @@ style_of_tag t

let stop_mark_ansi_stag t = ansi_tag @@ to_ansi_value @@ close_tag @@ style_of_tag t
</code></pre>
<p>On se le rappelle, l’ouverture d’un tag ANSI se fait avec la séquence d’échappement <code>x1B</code> suivie de une ou plusieurs valeurs de tags séparées par <code>;</code> entre <code>[</code> et <code>m</code>. Dans notre cas chaque tag n’est associé qu’à une valeur mais il serait tout à fait possible d’avoir un <code>Error -&gt; "1;4;31"</code> qui imposerait un affichage gras, souligné et en rouge. Tant que la chaîne de caractère renvoyée au terminal correspond bien à une séquence de marquage ANSI tout est possible.</p>
<p>Il faut ensuite faire en sorte que ces fonctions soient celles utilisées par le <code>formatter</code> lors de leur traitement :</p>
<pre><code class="language-ocaml">let add_ansi_marking formatter =
  let open Format in
  pp_set_mark_tags formatter true;
  let old_fs = pp_get_formatter_stag_functions formatter () in
  pp_set_formatter_stag_functions formatter
    { old_fs with
      mark_open_stag = start_mark_ansi_stag;
      mark_close_stag = stop_mark_ansi_stag }
</code></pre>
<p>On utilise la fonction <code>pp_set_mark_tags</code> (au lieu de <code>pp_set_tags</code>) car on ne se sert pas de <code>print_*_stags</code> et on associe aux fonctions <code>mark_*_stag</code> les fonctions <code>*_ansi_stag</code>.</p>
<p>Il ne nous reste plus qu’à faire en sorte que les tags sémantiques soient traités et avec nos fonctions avant d’afficher notre chaîne de caractères :</p>
<pre><code class="language-ocaml">let () =
  add_ansi_marking std_formatter;
  Format.printf "@{&lt;fg_blue&gt;Blue Text @}@{&lt;italic&gt;@{&lt;bg_white&gt;@{&lt;fg_black&gt;Italic WhiteBG BlackFG Text@}@}@}"
</code></pre>
<p>Et l’affichage dans le terminal sera bien celui voulu :</p>
<p><img src="https://ocamlpro.com/blog/assets/img/ansi-color-term-stag.png" alt="Exemple de marquage avec la gestion des tags sémantiques par Format dans un terminal ANSI"></p>
<p>Si le programme doit être affiché dans un terminal non ANSI il suffit simplement d’enlever la ligne <code>add_ansi_marking std_formatter;</code> :</p>
<p><img src="https://ocamlpro.com/blog/assets/img/ansi-color-try-stag.png" alt="Exemple de marquage avec la gestion des tags sémantiques par Format dans un terminal ANSI"></p>
<p>On pourrait aussi faire en sorte que notre texte puisse être envoyé vers un document HTML.</p>
<p>Il faut déjà changer les valeurs associées aux tags (on voit ici l’utilisation de boîtes verticales à indentation nulle mentionnée lors du paragraphe sur les boîtes structurelles) :</p>
<pre><code class="language-ocaml">let to_html_value fmt =
  let fg_color c = Format.fprintf fmt {|@[&lt;v 0&gt;@[&lt;v 2&gt;&lt;span style="color:%s;"&gt;@,|} c in
  let bg_color c = Format.fprintf fmt {|@[&lt;v 0&gt;@[&lt;v 2&gt;&lt;span style="background-color:%s;"&gt;@,|} c in
  let close_span () = Format.fprintf fmt "@]@,&lt;/span&gt;@]" in
  let default = Format.fprintf fmt in
  fun t -&gt; match t with
    | Normal -&gt; ()

    | Italic -&gt; default "&lt;i&gt;"
    | Italic_off -&gt; default "&lt;/i&gt;"

    | FG_Black -&gt; fg_color "black"
    | FG_Blue -&gt; fg_color "blue"
    | FG_Default -&gt; close_span ()

    | BG_White -&gt; bg_color "white"
    | BG_Default -&gt; close_span ()
</code></pre>
<p>La construction <code>{| ... |}</code> permet d’avoir des chaînes de caractères sans les caractères spéciaux <code>"</code> et `` ce qui permet d’écrire <code>{|"This is a nice "|}</code> sans espacer ces caractères.</p>
<p>De même, la construction</p>
<pre><code class="language-ocaml">let fonction arg1 ... argn =
  let expr1 = ... in
  ...
  let exprn = ... in
fun argn1 ... argnm -&gt;
</code></pre>
<p>Permet de définir des expressions internes à une fonction qui
dépendent des arguments fournis avant et donc, dans le cas d’une
application partielle, de calculer cet environnement une seule fois.
Dans le cas de la fonction <code>to_html_value</code> je pourrai donc créer la nouvelle application partielle <code>let to_html_value_std = to_html_value std_formatter</code> qui contiendra donc directment les implémentations de <code>fg_color</code>, <code>bg_color</code>, <code>close_span</code> et <code>default</code> pour <code>std_formatter</code>.</p>
<p>Contrairement au cas du terminal ANSI, ce qui changera sera le
contenu et non pas l’apparence du texte, nous utiliserons donc les
fonctions <code>print_*_stag</code>. C’est pourquoi nos fonctions doivent directement écrire dans le <code>formatter</code> et non pas renvoyer une chaîne de caractères.</p>
<p>Les fonctions d’ouverture et de fermeture ne changent pas énormément :</p><p></p>
<pre><code class="language-ocaml">let start_print_html_stag fmt t =
  to_html_value fmt @@ style_of_tag t

let stop_print_html_stag fmt t =
  to_html_value fmt @@ close_tag @@ style_of_tag t
</code></pre>
<p>On associe ensuite ces fonctions aux fonctions <code>print_*_stag</code> :</p>
<pre><code class="language-ocaml">let add_html_printings formatter =
  let open Format in
  pp_set_mark_tags formatter false;
  pp_set_print_tags formatter true;
  let old_fs = pp_get_formatter_stag_functions formatter () in
  pp_set_formatter_stag_functions formatter
    { old_fs with
      print_open_stag = start_print_html_stag formatter;
      print_close_stag = stop_print_html_stag formatter}
</code></pre>
<p>On en profite pour désactiver le marquage sur le formatter passé en
paramètre. Cela évite d’avoir de mauvaises surprises au cas où il aurait
été activé précédemment (il aurait fallu faire de même lors du marquage
pour le terminal ANSI).</p>
<p>Finalement, l’appel à :</p>
<pre><code class="language-ocaml">let () =
  add_html_printings std_formatter;
  Format.printf "@[&lt;v 0&gt;@{&lt;fg_blue&gt;Blue Text @}@,@{&lt;italic&gt;@{&lt;bg_white&gt;@{&lt;fg_black&gt;Italic WhiteBG BlackFG Text@}@}@}@]@."
</code></pre>
<p>Nous donne le résultat attendu :</p>
<pre><code class="language-html">&lt;span style="color:blue;"&gt;
  Blue Text
&lt;/span&gt;
&lt;i&gt;
  &lt;span style="background-color:white;"&gt;
     &lt;span style="color:black;"&gt;
       Italic WhiteBG BlackFG Text
     &lt;/span&gt;
   &lt;/span&gt;
&lt;/i&gt;
</code></pre>
<h2>Conclusion</h2>
<p>Nous voici arrivés à la fin de ce tutoriel qui, je l’espère, vous
permettra d’appréhender le module Format avec bien plus de sérénité.</p>
<p>Dans les possibilités non présentées ici mais qu’il est intéressant d’avoir en mémoire :</p>
<ul>
<li>Possibilité de redéfinir intégralement toutes les fonctions d’affichage définies dans l’enregistrement :
</li>
</ul>
<pre><code class="language-html">&lt;span class="hljs-keyword"&gt;type&lt;/span&gt;
formatter_out_functions = {
    out_string :
    &lt;span class="hljs-built_in"&gt;string&lt;/span&gt; -&gt; &lt;span class="hljs-built_in"&gt;int&lt;/span&gt; -&gt; &lt;span class="hljs-built_in"&gt;int&lt;/span&gt; -&gt; &lt;span class="hljs-built_in"&gt;unit&lt;/span&gt;;
    out_flush :
    &lt;span class="hljs-built_in"&gt;unit&lt;/span&gt; -&gt; &lt;span class="hljs-built_in"&gt;unit&lt;/span&gt;;
    out_newline :
    &lt;span class="hljs-built_in"&gt;unit&lt;/span&gt; -&gt; &lt;span class="hljs-built_in"&gt;unit&lt;/span&gt;;
    out_spaces :
    &lt;span class="hljs-built_in"&gt;int&lt;/span&gt; -&gt; &lt;span class="hljs-built_in"&gt;unit&lt;/span&gt;;
    out_indent :
    &lt;span class="hljs-built_in"&gt;int&lt;/span&gt; -&gt; &lt;span class="hljs-built_in"&gt;unit&lt;/span&gt;;
}
</code></pre>
<ul>
<li>Possibilité de transformer n’importe quel sortie en un formatter pour écrire directement dedans sans avoir à passer par des chaînes de caractère intermédiaire (notamment la fonction <code>val formatter_of_buffer : Buffer.t -&gt; formatter</code> qui permet directement d’écrire dans un buffer
</li>
<li>L’impression élégante symbolique qui imprime de façon symbolique donc permet de voir directement quelles directives seront envoyées au <code>formatter</code> à l’impression. Très utile pour débuguer en cas d’impression cacophonique mais aussi extrêmement puissant pour effectuer une phase de post-traitement (par exemple si on veut ajouter un symbole à chaque début de ligne)
</li>
<li>Les fonctions utiles qu’il ne faut pas oublier d’utiliser (je sais que les devs OCaml aiment réinventer la roue mais il existe déjà des fonctions pour afficher des listes, des options et les résultats <code>Ok _ | Error _</code>) :
</li>
</ul>
<pre><code class="language-ocaml">val pp_print_list : ?pp_sep:(formatter -&gt; unit -&gt; unit) -&gt; (formatter -&gt; 'a -&gt; unit) -&gt; formatter -&gt; 'a list -&gt; unit

(* Affiche une liste dont chaque élément est séparé par le séparateur par défaut `@,` ou celui fourni *)

val pp_print_option : ?none:(formatter -&gt; unit -&gt; unit) -&gt; (formatter -&gt; ‘a -&gt; unit) -&gt; formatter -&gt; ‘a option -&gt; unit

(* Affiche le contenu d’une option en cas de Some contenu et rien par défaut si None ou l’affichage fourni *)&lt;/p&gt;

val pp_print result : ok:(formatter -&gt; ‘a -&gt; unit) -&gt; error:(formatter -&gt; ‘e -&gt; unit) -&gt; formatter -&gt; (‘a, ‘e) result -&gt; unit

(* Affiche le contenu d’un result. Les arguments ne sont ici pas optionnels et conditionnent l’affichage en cas de Ok &lt;/em&gt; et de Error _ *)
</code></pre>
<ul>
<li>Enfin, une pelletée de fonctions à la <code>printf</code> telles que, donc :
</li>
<li><code>fprintf</code> que nous avons déjà vue
</li>
<li><code>dprintf</code> qui permet de retarder l'évaluation de l'impression et donc de ne pas calculer des impressions qui ne seront jamais faites
</li>
<li><code>ifprintf</code> qui n'affiche rien (utile lorsqu'on veut avoir la même signature que <code>fprintf</code> mais en étant sûr que rien ne sera fait)
</li>
</ul>
<p>Sources :</p>
<ul>
<li>
<p>Tutoriel du site OCaml</p>
</li>
<li>
<p>Richard Bonichon, Pierre Weis. Format Unraveled. 28ièmes Journées Francophones des LangagesApplicatifs, Jan 2017, Gourette, France. hal-01503081</p>
</li>
</ul>
<p>Codes sources :</p>
<p>Code LaTeX correspondant à <code>printf</code></p>
<pre><code class="language-latex">\documentclass[tikz,border=10pt]{standalone}

\usepackage{tikz}
\usetikzlibrary{math}
\usetikzlibrary{tikzmark}

\usepackage{xcolor}

\pagecolor[rgb]{0,0,0}
\color[rgb]{1,1,1}

\colorlet{color1}{blue!50!white}
\colorlet{color2}{red!50!white}
\colorlet{color3}{green!50!black}

\begin{document}

\begin{tikzpicture}[remember picture]
\node [align=left,font=\ttfamily] at (0,0) {
    let s = "toto" in\[2em]
    printf "{color{color1}\tikzmarknode{scd}{\%d}}
        {color{color2}\tikzmarknode{scc}{\%c}}
        {color{color3}\tikzmarknode{scs}{\%s}}"
    {\color{color1}\tikzmarknode{d}{3}}
    {\color{color2}\tikzmarknode{c}{'c'}}
    {\color{color3}\tikzmarknode{s}{s}}\\[2em]
    &gt; "3 c toto"
};
\draw[&lt;-, color1] (scd.north) -- ++(0,0.5) -| (d);
\draw[&lt;-, color2] (scc.south) -- ++(0,-0.4) -| (c);
\draw[&lt;-, color3] (scs.north) -- ++(0,0.4) -| (s);
\end{tikzpicture}

end{document}
</code></pre>
<p>Code LaTeX correspondant à <code>fprintf</code>:</p>
<pre><code class="language-latex">\documentclass[tikz,border=10pt]{standalone}

\usepackage{tikz}
\usetikzlibrary{math}
\usetikzlibrary{decorations.pathreplacing,tikzmark}

\usepackage{xcolor}

\pagecolor[rgb]{0,0,0}
\color[rgb]{1,1,1}

\colorlet{color1}{blue!50!white}
\colorlet{color2}{red!50!white}
\colorlet{color3}{green!50!black}

\begin{document}

\begin{tikzpicture}[remember picture]
\node [align=left,font=\ttfamily] at (0,0) {
    let s = "toto" in\\[2em]
    fprintf \tikzmarknode{fmt}{fmt} \tikzmarknode{str}{"{\color{color1}\tikzmarknode{scd}{\%d}}
        {\color{color2}\tikzmarknode{scc}{\%c}}
        {\color{color3}\tikzmarknode{scs}{\%s}}"}
    {\color{color1}\tikzmarknode{d}{3}}
    {\color{color2}\tikzmarknode{c}{'c'}}
    {\color{color3}\tikzmarknode{s}{s}}\\[2em]
    &gt; \\
    (* fmt &lt;- "3 c toto" *)
};
\draw[&lt;-, color1] (scd.north) -- ++(0,0.5) -| (d);
\draw[&lt;-, color2] (scc.south) -- ++(0,-0.3) -| (c);
\draw[&lt;-, color3] (scs.north) -- ++(0,0.4) -| (s);
\draw[decorate,decoration={brace, amplitude=5pt, raise=10pt},yshift=-2cm] (str.south east) -- (str.south west) node[midway, yshift=-13pt](a){} ;

\draw[-&gt;, white] (a.south) -- ++(0,-0.1) -| (fmt);
\end{tikzpicture}

\end{document}
</code></pre>
<p>Code LaTeX correspondant à <code>fprintf</code> avec utilisation de <code>%a</code></p>
<pre><code class="language-latex">\documentclass[tikz,border=10pt]{standalone}

\usepackage{tikz}
\usetikzlibrary{math}
\usetikzlibrary{decorations.pathreplacing,tikzmark}

\usepackage{xcolor}

\pagecolor[rgb]{0,0,0}
\color[rgb]{1,1,1}

\colorlet{color1}{blue!50!white}
\colorlet{color2}{red!50!white}
\colorlet{color3}{green!50!black}

\begin{document}

\begin{tikzpicture}[remember picture]
\node [align=left,font=\ttfamily] at (0,0) {
    let s = "toto" in\\[2em]
    type expr = \{i: int; j: int\}\\
    let pp\_expr fmt {i; j} = fprintf fmt "&lt;\%d, \%d&gt; i j" in\\[2em]
    fprintf \tikzmarknode{fmt}{std\_formatter} \tikzmarknode{str}{"{\color{color1}\tikzmarknode{scd}{\%d}}
        {\color{color2}\tikzmarknode{sca}{\%a}}
        {\color{color3}\tikzmarknode{scs}{\%s}}"}
    {\color{color1}\tikzmarknode{d}{3}}
    {\color{color2}\tikzmarknode{ppe}{pp\_expr}}
    {\color{color2}\tikzmarknode{e}{\{i=1; j=2\}}}
    {\color{color3}\tikzmarknode{s}{s}}\\[2em]
    &gt; "3 &lt;1, 2&gt; toto"
};
\draw[&lt;-, color1] (scd.north) -- ++(0,0.5) -| (d);
\draw[&lt;-, color2] (sca.south) -- ++(0,-0.3) -| (ppe);
\draw[&lt;-, color2] (sca.65) -- ++(0,0.3) -| (e);
\draw[-&gt;, color2] (fmt.north) -- ++(0,0.2) -| (sca.115);
\draw[&lt;-, color3] (scs.south) -- ++(0,-0.4) -| (s);
\draw[decorate,decoration={brace, amplitude=5pt, raise=12pt},yshift=-2cm]  (str.south east) -- (str.south west) node[midway, yshift=-13pt](a){} ;

\draw[-&gt;, white] (a.south) -- ++(0,-0.1) -| (fmt);
\end{tikzpicture}

\end{document}
</code></pre>

