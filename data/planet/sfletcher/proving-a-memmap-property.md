---
title: Proving a mem/map property
description: Here are two well known "classic" functions over polymorphic       lists.                 map
  f l  computes a ne...
url: http://blog.shaynefletcher.org/2017/05/proving-mem-map-property.html
date: 2017-05-11T20:17:00-00:00
preview_image:
authors:
- Shayne Fletcher
source:
---

<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd"><html><head>

    <title></title>
  </head>
  <body>
    <p>
      Here are two well known "classic" functions over polymorphic
      lists.
    </p>
    <p>
      <code class="code">map f l</code> computes a new list from <code class="code">l</code>
      by applying <code class="code">f</code> to each of its elements.
</p><pre><code class="code"><span class="keyword">let</span> <span class="keyword">rec</span> map (f : <span class="keywordsign">'</span>a <span class="keywordsign">-&gt;</span> <span class="keywordsign">'</span>b) : <span class="keywordsign">'</span>a list <span class="keywordsign">-&gt;</span> <span class="keywordsign">'</span>b list = <span class="keyword">function</span>
        <span class="keywordsign">|</span> [] <span class="keywordsign">-&gt;</span> []
        <span class="keywordsign">|</span> h :: t <span class="keywordsign">-&gt;</span> f h :: map f t
        ;;
</code></pre>
    <p></p>

    <p><code class="code">mem x l</code> returns <code class="code">true</code> is <code class="code">x</code>
      is an element of <code class="code">l</code> and returns <code class="code">false</code> if it
      is not.
</p><pre><code class="code"><span class="keyword">let</span> <span class="keyword">rec</span> mem (a : <span class="keywordsign">'</span>a) : <span class="keywordsign">'</span>a list <span class="keywordsign">-&gt;</span> bool  = <span class="keyword">function</span>
        <span class="keywordsign">|</span> [] <span class="keywordsign">-&gt;</span> <span class="keyword">false</span>
        <span class="keywordsign">|</span> x :: l <span class="keywordsign">-&gt;</span> a = x <span class="keywordsign">||</span> mem a l
        ;;
</code></pre>
    <p></p>
    <p>
      If <code class="code">y</code> is an element of the list obtained by
      mapping <code class="code">f</code> over <code class="code">l</code> then there must be an
      element <code class="code">x</code> in <code class="code">l</code> such that 
      <code class="code">f x = y</code>. Conversely, if there exists an <code class="code">x</code>
      in <code class="code">l</code> such that <code class="code">y = f x</code>,
      then <code class="code">y</code> must be a member of the list obtained by
      mapping <code class="code">f</code> over <code class="code">l</code>.
   </p>
    <p>
      We attempt a proof of correctness of the given definitions with
      respect to this property.
    </p>

    <b>Lemma</b> <code>mem_map_iff</code>:
    <pre><code class="code">
    ∀ (f : α → β) (l : α list) (y : β),
        mem y (map f l) ⇔ ∃(x : α), f x = y ∧ mem x l.
    </code></pre>
    <b>Proof:</b><br>
    <ul>

      <li>We first treat the forward implication
        <pre>          <code class="code">
    ∀ (f : α → β) (l : α list) (y : β),
      mem y (map f l) ⇒ ∃(x : α), f x = y ∧ mem x l
        </code></pre>
        and proceed by induction on <code class="code">l</code>.
        <br>
        <br>

        <ul>
          <li><code class="code">l = []</code>:
            <ul>
              <li>Show <code class="code">mem y (map f []) ⇒ ∃(x : α), f x = y ∧ mem x []</code>.</li>
              <li><code class="code">mem y (map f []) ≡ False</code>.</li>
              <li>Proof follows <i>(ex falso quodlibet)</i>.</li>
            </ul>
            <br>
          </li> 

        <li><code class="code">l</code> has form <code class="code">x' :: l</code> (use <code class="code">l</code> now to refer to the tail):
            <ul>
              <li>Assume the induction hypothesis:
                <ul><li><code class="code">mem y (map f l) ⇒ ∃x, f x = y ∧ mem x l</code>.</li></ul></li>
              <li>We are required to show for an arbitrary <code class="code">(x' : α)</code>:
                <ul><li><code class="code">mem y (map f (x' :: l)) ⇒ ∃(x : α), f x = y ∧ mem x (x' :: l)</code>.</li></ul>
              </li>
              <li>By simplification, we can rewrite the above to:
                <ul><li><code class="code">f x' = y ∨ mem y (map f l) ⇒ ∃(x : α), f x = y ∧ (x' = x ∨ mem x l).</code></li></ul>
              </li>
              <li>We assume then an <code class="code">(x' : α)</code> and a <code class="code">(y : β)</code> such
                that:
                <ol>
                  <li><code class="code">f x' = y ∨ mem y (map f l)</code>.</li>
                  <li><code class="code">mem y (map f l) ⇒ ∃(x : α), f x = y ∧ mem x l</code>.</li>
                </ol>
              </li>
              <li>Show <code class="code">∃(x : α), f x = y ∧ (x' = x ∨ mem x l)</code>:
                <ul>
                  <li>First consider <code class="code">f x' = y</code> in (1).
                    <ul>
                      <li>Take <code class="code">x = x'</code> in the goal.</li>
                      <li>Then by (1) <code class="code">f x = y ∧ x = x'</code>.</li>
                      <li>So <code class="code">x'</code> is a witness.</li>
                    </ul>
                  </li>
                  <li>Now consider <code class="code">mem y (map f l)</code> in (1).
                    <ul>
                      <li><code class="code">∃(x<sup>*</sup> : α), f x<sup>*</sup> = y ∧ mem x<sup>*</sup> l</code> by (2).</li>
                      <li>Take <code class="code">x = x<sup>*</sup></code> in the goal.</li>
                      <li>By the above <code class="code">f x<sup>*</sup> = y ∧ mem x<sup>*</sup> l</code></li>
                      <li>So <code class="code">x<sup>*</sup></code> is a witness.</li>
                    </ul>
                  </li>
                </ul>
              </li>
            </ul>
          </li> 
        </ul>
        <br><br>
      </li>

      <li>
      We now work on the reverse implication. We want to show that
      <pre><code class="code">
    ∀ (f : α → β) (l : α list) (y : β),
       ∃(x : α), f x = y ∧ mem x l ⇒ mem y (map f l)
      </code></pre>
      and proceed by induction on <code class="code">l</code>.
      <br><br>
      <ul>
        <li><code class="code">l = []</code>:
        <ul>
          <li>Assume <code class="code">x</code>, <code class="code">y</code> with <code class="code">f x = y ∧ mem x []</code>.</li>
          <li>Show <code class="code">mem y (map f [])</code>:</li>
           <ul>
             <li><code class="code">mem x [] ≡ false</code>.</li>
             <li>Proof follows <i>(ex falso quodlibet)</i>.</li>
           </ul>
        </ul>
        </li>

        <li><code class="code">l</code> has form <code class="code">x' :: l</code> (use <code class="code">l</code> now to refer to the tail):
          <ul>
            <li>Assume the induction hypothesis:
            <ul><li><code class="code">∃(x : α), f x = y ∧ mem x l ⇒ mem y (map f l)</code>.</li></ul>
            </li>
            <li>We are required to show for an arbitrary <code class="code">(x' : α)</code>:
              <ul><li><code class="code">∃ (x : α), f x = y ∧ mem x (x' :: l) ⇒ mem y (map f (x' :: l))</code></li></ul>
            </li>
            <li>By simplification, we can rewrite the above to:
              <ul><li><code class="code">∃ (x : α), f x = y ∧ x = x' ∨ mem x l ⇒ f x' = y ∨ mem y (map f l)</code>.</li></ul>
            </li>
            <li>Assume the goal and induction hypotheses:
              <ul>
                <li>There is <code class="code">(x : α)</code> and <code class="code">(y : β)</code> such that:
                  <ol>
                    <li><code class="code">f x = y ∧ (x = x' ∨ mem x l)</code></li>
                    <li><code class="code">f x = y ∧ mem x l ⇒ mem y (map f l)</code></li>
                  </ol>
                </li>
              </ul>
            </li>
            <li>Show <code class="code">f x' = y ∨ mem y (map f l)</code>:
              <ul>
                <li>Assume <code class="code">x = x'</code> in (1) and show <code class="code">f x' = y</code>:
                  <ul>
                   <li>Since, <code class="code">f x = y</code> is given by (1.), <code class="code">f x' = y</code>.</li>
                  </ul>
                </li>
                <li>Assume <code class="code">mem x l</code> in (1) and show <code class="code">mem y (map f l)</code>:
                  <ul>
                    <li>Rewrite <code class="code">mem y (map f l)</code> via (2) to <code class="code">f x = y ∧ mem x l</code>.</li>
                    <li><code class="code">f x = y</code> by (1) so <code class="code">mem y (map f l)</code>.</li>
                  </ul>
                </li>
              </ul>
            </li>
          </ul>
        </li>
      </ul>
      </li>
    </ul>
∎
    <hr>
    <p>
    References:<br>
    <a href="https://www.cis.upenn.edu/~bcpierce/sf/current/index.html">"Sofware Foundations"</a> -- Pierce et. al.
    </p>
  

</body></html>
