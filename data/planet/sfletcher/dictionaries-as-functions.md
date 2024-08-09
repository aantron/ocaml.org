---
title: Dictionaries as functions
description: This is an "oldie but a goodie". It's super easy.   A dictionary is a
  data structure that represents a map from keys to values. The questi...
url: http://blog.shaynefletcher.org/2016/04/dictionaries-as-functions.html
date: 2016-04-13T16:58:00-00:00
preview_image:
authors:
- Shayne Fletcher
source:
---

<h2></h2>
<p>
This is an "oldie but a goodie". It's super easy.
</p>
<p>
A dictionary is a data structure that represents a map from keys to values. The question is, can this data structure and its characteristic operations be encoded using only functions?
</p>
<p>
The answer of course is yes and indeed, here's one such an encoding in OCaml.
</p><p>
</p><pre class="prettyprint ml">(*The type of a dictionary with keys of type [α] and values of type
  [β]*)
type (α, β) dict = α -&gt; β option

(*The empty dictionary maps every key to [None]*)
let empty (k : α) : β option = None

(*[add d k v] is the dictionary [d] together with a binding of [k] to
  [v]*)
let add (d : (α, β) dict) (k : α) (v : β) : (α, β) dict = 
  fun l -&gt; 
    if l = k then Some v else d l

(*[find d k] retrieves the value bound to [k]*)
let find (d : (α, β) dict) (k : α) : β option = d k
</pre>
Test it like this.
<pre class="prettyprint ml">(*e.g.

  Name                            | Age
  ================================+====
  "Felonius Gru"                  |  53
  "Dave the Minion"               | 4.54e9
  "Dr. Joseph Albert Nefario"     |  80

*)
let despicable = 
  add 
    (add 
       (add 
          empty "Felonius Gru" 53
       ) 
       "Dave the Minion" (int_of_float 4.54e9)
    )
    "Dr. Nefario" 80 

let _ = 
  find despicable "Dave the Minion" |&gt; 
      function | Some x -&gt; x | _ -&gt; failwith "Not found"
</pre>
<p></p>
<p>Moving on, can we implement this in C++? Sure. Here's one way.
</p><pre class="prettyprint ml">#include &lt;pgs/pgs.hpp&gt;

#include &lt;functional&gt;
#include &lt;iostream&gt;
#include &lt;cstdint&gt;

using namespace pgs;

// -- A rough and ready `'a option` (given the absence of
// `std::experimental::optional`

struct None {};

template &lt;class A&gt;
struct Some { 
  A val;
  template &lt;class Arg&gt;
  explicit Some (Arg&amp;&amp; s) : val { std::forward&lt;Arg&gt; (s) }
  {}
};

template &lt;class B&gt;
using option = sum_type&lt;None, Some&lt;B&gt;&gt;;

template &lt;class B&gt;
std::ostream&amp; operator &lt;&lt; (std::ostream&amp; os, option&lt;B&gt; const&amp; o) {
  return o.match&lt;std::ostream&amp;&gt;(
    [&amp;](Some&lt;B&gt; const&amp; a) -&gt; std::ostream&amp; { return os &lt;&lt; a.val; },
    [&amp;](None) -&gt; std::ostream&amp; { return os &lt;&lt; "&lt;empty&gt;"; }
  );
}

//-- Encoding of dictionaries as functions

template &lt;class K, class V&gt;
using dict_type = std::function&lt;option&lt;V&gt;(K)&gt;;

//`empty` is a dictionary constant (a function that maps any key to
//`None`)
template &lt;class A, class B&gt;
dict_type&lt;A, B&gt; empty = 
  [](A const&amp;) { 
    return option&lt;B&gt;{ constructor&lt;None&gt;{} }; 
};

//`add (d, k, v)` extends `d` with a binding of `k` to `v`
template &lt;class A, class B&gt;
dict_type&lt;A, B&gt; add (dict_type&lt;A, B&gt; const&amp; d, A const&amp; k, B const&amp; v) {
  return [=](A const&amp; l) {
    return (k == l) ? option&lt;B&gt;{ constructor&lt;Some&lt;B&gt;&gt;{}, v} : d (l);
  };
}

//`find (d, k)` searches for a binding in `d` for `k`
template &lt;class A, class B&gt;
option&lt;B&gt; find (dict_type&lt;A, B&gt; const&amp; d, A const&amp; k) {
  return d (k);
}

//-- Test driver

int main () {

  using dict_t = dict_type&lt;std::string, std::int64_t&gt;;

  auto nil = empty&lt;std::string, std::int64_t&gt;;
  dict_t(*insert)(dict_t const&amp;, std::string const&amp;, std::int64_t const&amp;) = &amp;add;


  dict_t despicable = 
    insert (
      insert (
        insert (nil
           , std::string {"Felonius Gru"}, std::int64_t{53})
           , std::string {"Dave the Minion"}, std::int64_t{4530000000})
          , std::string {"Dr. Joseph Albert Nefario"}, std::int64_t{80})
     ;

  std::cout &lt;&lt; 
    find (despicable, std::string {"Dave the Minion"}) &lt;&lt; std::endl;

  return 0;
}
</pre>
<p></p>
