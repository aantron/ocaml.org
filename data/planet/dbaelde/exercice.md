---
title: Exercice
description:
url: https://misterpingouin.blogspot.com/2008/10/exercice.html
date: 2008-10-13T13:15:00-00:00
preview_image:
authors:
- mrpingouin
source:
---

Il me reste treize jours pour finir de rédiger ma thèse. Pendant ce temps là, pour que je ne sois pas le seul à bosser, voici un petit exercice de prog qui m'est passé par la tête. On peut représenter du texte par le type <code>string</code> en Caml. On veut ensuite représenter du texte annoté, quand les annotations forment un arbre: soit deux annotations sont disjointes, soit l'une est incluse dans l'autre. Pensez par exemple à la grammaire au collège, "sujet" , "verbe", "complément" sont regroupés pour former une "phrase":<br><br><div class="code">type annot =<br>  (* Par exemple.. *)<br>  | Sujet | Verbe | Complément | Phrase<br>type t =<br>  | Text of string<br>  | Annot of string * t<br>  | Concat of t list<br></div><br>Il est aisé de travailler avec ce type. Et il représente exactement ce qu'on veut, il y a bijection. Maintenant, je veux une solution aussi satisfaisante quand les annotations ne forment plus un arbre mais peuvent se recouvrir partiellement. Pensez par exemple à du HTML mal formé: <code>&lt;red&gt;b&lt;b&gt;a&lt;/red&gt;b&lt;/b&gt;i</code>. Sauf que moi j'ai une utilisation moins crade en tête, mais c'est une autre histoire. Vos idées sont bienvenues, sinon je posterai peut être une solution plus tard...
