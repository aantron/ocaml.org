---
title: Generic mappings over pairs
description: Browsing around on Oleg Kiselyov's  excellent site, I came across a very
  interesting paper about " Advanced Polymorphism in Simpler-Typed L...
url: http://blog.shaynefletcher.org/2016/06/generic-mappings-over-pairs.html
date: 2016-06-17T19:39:00-00:00
preview_image:
authors:
- Shayne Fletcher
source:
---

<h2></h2>
<p>Browsing around on <a href="http://okmij.org/ftp/">Oleg Kiselyov's</a> excellent site, I came across a very interesting paper about "<a href="http://okmij.org/ftp/Computation/extra-polymorphism.html">Advanced Polymorphism in Simpler-Typed Languages</a>". One of the neat examples I'm about to present is concerned with expressing mappings over pairs that are generic not only in the datatypes involved but also over the number of arguments. The idea is to produce a family of functions $pair\_map_{i}$ such that
</p><pre>pair_map_1 f g (x, y) (x', y') → (f x, g y) 
pair_map_2 f g (x, y) (x', y') → (f x x', g y y') 
pair_map_3 f g (x, y) (x', y') (x'', y'', z'') → (f x x' x'', g y y' y'')
       .
       .
       .
</pre>
The technique used to achieve this brings a whole bunch of functional programming ideas together : higher order functions, combinators and continuation passing style (and also leads into topics like the "value restriction" typing rules in the Hindley-Milner system).
<pre class="prettyprint ml">let ( ** ) app k = fun x y -&gt; k (app x y)
let pc k a b = k (a, b)
let papp (f1, f2) (x1, x2) = (f1 x1, f2 x2)
let pu x = x
</pre>
With the above definitions, $pair\_map_{i}$ is generated like so.
<pre class="prettyprint ml">(*The argument [f] in the below is for the sake of value restriction*)
let pair_map_1 f = pc (papp ** pu) (f : α -&gt; β)
let pair_map_2 f = pc (papp ** papp ** pu) (f : α -&gt; β -&gt; γ)
let pair_map_3 f = pc (papp ** papp ** papp ** pu) (f : α -&gt; β -&gt; γ -&gt; δ)
</pre>
For example,
<pre># pair_map_2 ( + ) ( - ) (1, 2) (3, 4) ;;
- : int * int = (4, -2)
</pre>
<p></p>
<p>Reverse engineering how this works requires a bit of algebra.
</p>
<p>Let's tackle $pair\_map_{1}$. First
</p><pre>pc (papp ** pu) = (λk f g. k (f, g)) (papp ** pu) = λf g. (papp ** pu) (f, g)
</pre>
and,
<pre>papp ** pu = λx y. pu (papp x y) = λx y. papp x y
</pre>
so,
<pre>λf g. (papp ** pu) (f, g) =
    λf g. (λ(a, b) (x, y). (a x, b y)) (f, g) =
    λf g (x, y). (f x, g y)
</pre>
that is,
<code>pair_map_1 = pc (papp ** pu) = λf g (x, y). (f x, g y)</code> and, we can read the type off from that last equation as <code>(α → β) → (γ → δ) → α * γ → β * δ</code>.
<p></p>
<p>Now for $pair\_map_{2}$. We have
</p><pre>pc (papp ** papp ** pu) =
    (λk f g. k (f, g)) (papp ** papp ** pu) =
    λf g. (papp ** papp ** pu) (f, g)
</pre>
where,
<pre>papp ** papp ** pu = papp ** (papp ** pu) =
    papp ** (λa' b'. pu (papp a' b')) =
    papp ** (λa' b'. papp a' b') = 
    λa b. (λa' b'. papp a' b') (papp a b)
</pre>
which means,
<pre>pc (papp ** papp ** pu) = 
    λf g. (papp ** papp ** pu) (f, g) =
    λf g. (λa b.(λa' b'. papp a' b') (papp a b)) (f, g) =
    λf g. (λb. (λa' b'. papp a' b') (papp (f, g) b)) =
    λf g. λ(x, y). λa' b'. (papp a' b') (papp (f, g) (x, y)) =
    λf g. λ(x, y). λa' b'. (papp a' b') (f x, g y) =
    λf g. λ(x, y). λb'. papp (f x, g y) b' =
    λf g. λ(x, y). λ(x', y'). papp (f x, g y) (x', y') =
    λf g (x, y) (x', y'). (f x x', g y y')
</pre>
that is, a function in two binary functions and two pairs as we expect. Phew! The type in this instance is <code>(α → β → γ) → (δ → ε → ζ) → α * δ → β * ε → γ * ζ</code>.
<p></p>
<p>
To finish off, here's the program transliterated into C++(14).
</p><pre class="prettyprint c++">#include &lt;utility&gt;
#include &lt;iostream&gt;

//let pu x = x
auto pu = [](auto x) { return x; };

//let ( ** ) app k  = fun x y -&gt; k (app x y)
template &lt;class F, class K&gt;
auto operator ^ (F app, K k) {
  return [=](auto x) {
    return [=] (auto y) {
      return k ((app (x)) (y));
    };
  };
}

//let pc k a b = k (a, b)
auto pc = [](auto k) {
  return [=](auto a) {
    return [=](auto b) { 
      return k (std::make_pair (a, b)); };
  };
};

//let papp (f, g) (x, y) = (f x, g y)
auto papp = [](auto f) { 
  return [=](auto x) { 
    return std::make_pair (f.first (x.first), f.second (x.second)); };
};

int main () {

  auto pair = &amp;std::make_pair&lt;int, int&gt;;

  {
  auto succ= [](int x){ return x + 1; };
  auto pred= [](int x){ return x - 1; };
  auto p  = (pc (papp ^ pu)) (succ) (pred) (pair (1, 2));
  std::cout &lt;&lt; p.first &lt;&lt; ", " &lt;&lt; p.second &lt;&lt; std::endl;
  }

  {
  auto add = [](int x) { return [=](int y) { return x + y; }; };
  auto sub = [](int x) { return [=](int y) { return x - y; }; };
  auto p = pc (papp ^ papp ^ pu) (add) (sub) (pair(1, 2)) (pair (3, 4));
  std::cout &lt;&lt; p.first &lt;&lt; ", " &lt;&lt; p.second &lt;&lt; std::endl;
  }

  return 0;
}
</pre>
<p></p>

