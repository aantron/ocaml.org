---
title: An alternative to camlp4 - Part 2
description:
url: https://lpw25.net/2013/02/05/camlp4-alternative-part-2
date: 2013-02-05T00:00:00-00:00
preview_image:
authors:
- Leo White
source:
---

<p>In my <a href="http://lpw25.net/2013/01/23/camlp4-alternative-part-1.html">previous blog post</a> I
discussed how we might use AST transformers, attributes and quotations as a
simpler alternative to camlp4. While AST transformers are much simpler to use
than camlp4 they still require knowledge of the OCaml syntax tree, and they are
still implemented outside of the language as preprocessors.</p>

<p>In this post I’ll explore how to implement extensions:</p>

<ul>
  <li>within the language itself without external preprocessors</li>
  <li>without the need for detailed knowledge of the OCaml syntax tree</li>
</ul>

<p>By including these extensions in the language itself the increasing number of
tools being built to support OCaml (e.g. typerex) can handle them directly. For
instance, IDEs could show the expansions of quotations by using the information
in “.cmt” files.</p>

<p>I will start with quotations, which can be implemented without any knowledge of
the OCaml syntax tree, and then expand my proposal to include other kinds of
extension.</p>

<p>Since my previous post, there has been a lot of discussion on
<a href="http://lists.ocaml.org/listinfo/wg-camlp4">wg-camlp4@lists.ocaml.org</a> about
possible syntaxes for quotations and attributes and other kinds of extension. In
keeping with those ongoing discussions, I will use <code class="language-plaintext highlighter-rouge">{:id { string }}</code> as the
syntax for quotations (which transform a string into an AST node) and <code class="language-plaintext highlighter-rouge">(:id
expr)</code> as the syntax for extensions that transform an OCaml expression into an
AST node.</p>

<p>Note that the proposals in this post are more long-term than the “ppx” solution
discussed in the previous post. Moving an extension from ppx to the mechanism
described in this post would require only minimal work. So in the short/medium
term extension authors should implement their extensions using ppx.</p>

<h4>Quotations</h4>

<p>A quotation is simply a function which takes a string and returns an AST
node. To provide built-in support we need to, for every quotation <code class="language-plaintext highlighter-rouge">{:foo { bar
}}</code>:</p>

<ol>
  <li>find a function that corresponds to <code class="language-plaintext highlighter-rouge">foo</code></li>
  <li>apply it to the string “ bar “</li>
  <li>copy the resulting AST node in place of the original quotation</li>
</ol>

<h5>Quotations in modules</h5>

<p>We might want to find the function corresponding to quotation <code class="language-plaintext highlighter-rouge">foo</code> by simply
looking in the current module, or one of the other modules in our environment,
for a function called <code class="language-plaintext highlighter-rouge">foo</code>. However there are a few problems with this simple
scheme:</p>

<ol>
  <li>The function we call must exist and be compiled before we can use it.</li>
  <li>There is no clear separation between what is being executed at compile-time
and what is being executed and run-time.</li>
</ol>

<p>The first problem basically means that we can only use functions defined in
other files. The other problem is more subtle.</p>

<p>OCaml modules do not really exist at compile time. They are created at run-time,
and their creation encompasses the entire execution of the program. For example,
consider this simple module:</p>

<figure class="highlight"><pre><code class="language-ocaml" data-lang="ocaml"><span class="c">(* main.ml *)</span>
<span class="k">let</span> <span class="n">x</span> <span class="o">=</span> <span class="nn">Printf</span><span class="p">.</span><span class="n">printf</span> <span class="s2">"Hello, world!</span><span class="se">\n</span><span class="s2">"</span></code></pre></figure>

<p>To create the <code class="language-plaintext highlighter-rouge">Main</code> module, we must create its member <code class="language-plaintext highlighter-rouge">x</code>. Once the creation of
<code class="language-plaintext highlighter-rouge">Main</code> is finished the <code class="language-plaintext highlighter-rouge">printf</code> has been executed and the whole program has
completed. Now if we add a quotation function <code class="language-plaintext highlighter-rouge">foo</code> to this module:</p>

<figure class="highlight"><pre><code class="language-ocaml" data-lang="ocaml"><span class="c">(* main.ml *)</span>
<span class="k">let</span> <span class="n">x</span> <span class="o">=</span> <span class="nn">Printf</span><span class="p">.</span><span class="n">printf</span> <span class="s2">"Hello, world!</span><span class="se">\n</span><span class="s2">"</span>

<span class="k">let</span> <span class="n">keywords</span> <span class="o">=</span> <span class="nn">Hashtbl</span><span class="p">.</span><span class="n">create</span> <span class="mi">13</span>
<span class="k">let</span> <span class="n">foo</span> <span class="n">str</span> <span class="o">=</span> <span class="c">(* Some expression using keywords *)</span></code></pre></figure>

<p>How do we distinguish between data such as <code class="language-plaintext highlighter-rouge">keywords</code> which are needed at
compile-time when the quotation is run, and data like <code class="language-plaintext highlighter-rouge">x</code> whose creation is
meant to drive the program at run-time? There is nothing explicit in the
definitions of <code class="language-plaintext highlighter-rouge">foo</code> or <code class="language-plaintext highlighter-rouge">keywords</code> that indicates that they are intended for
compile-time execution.</p>

<p>These problems are related to the fact that OCaml is an impure language. Any
expression (including module definitions) can have side-effects, and the
run-time behaviour of the program is simply the combination of all these
side-effects. This makes it difficult to separate the side-effects that are
related to a quotation from the side-effects that are part of the program’s
execution.</p>

<p>Despite appearing alongside other functions in the program, <code class="language-plaintext highlighter-rouge">foo</code> must be
executed in a completely separate environment. Any side-effects (e.g. mutable
state, I/O) that are produced while creating and executing <code class="language-plaintext highlighter-rouge">foo</code> will be
completely separate from the side-effects of the other functions in its module.</p>

<h5>Where can we put them?</h5>

<p>If we don’t want to put quotation functions in our modules, where should we put
them? The module system provides the only mechanism for referring to functions in
other files, how can we refer to functions which are not included in a module?</p>

<p>The answer to these questions comes from the idea of <em>namespaces</em>. Namespaces
are a way to give longer names to top-level modules without changing the
module’s filename. They also allow these top-level modules to be grouped
together.</p>

<p>The details of proposals for namespaces vary on their details, but they
basically allow you to take the module defined by a file “baz.ml” and refer to
it as “Bar.Baz”. Here “Bar” is not a module (it cannot be used as the argument
to a functor) but a namespace.</p>

<p>Namespaces seem likely to be included in OCaml in the near future,
and they provide a convenient way to refer to quotations without putting
quotations within modules.</p>

<p>The idea is to write quotations in a “bar.mlq” file (compiled to
“bar.cmq”). These quotations would then be placed in the namespace “Bar”.</p>

<p>Quotations would be defined with a syntax like:</p>

<figure class="highlight"><pre><code class="language-ocaml" data-lang="ocaml"><span class="c">(* bar.mlq *)</span>
<span class="n">quotation</span> <span class="n">foo</span> <span class="n">str</span> <span class="o">=</span> <span class="o">...</span></code></pre></figure>

<p>This could then be used with the syntax:</p>

<figure class="highlight"><pre><code class="language-ocaml" data-lang="ocaml"><span class="p">{</span><span class="o">:</span><span class="nn">Bar</span><span class="p">.</span><span class="n">foo</span><span class="p">{</span> <span class="n">some</span> <span class="n">text</span> <span class="p">}}</span></code></pre></figure>

<p>This will make it easy for quotations to be provided by libraries. So that the
following code would perfectly possible:</p>

<figure class="highlight"><pre><code class="language-ocaml" data-lang="ocaml"><span class="p">{</span><span class="o">:</span><span class="nn">Core</span><span class="p">.</span><span class="nn">Web</span><span class="p">.</span><span class="n">html</span><span class="p">{</span><span class="o">&lt;</span><span class="n">body</span><span class="o">&gt;</span> <span class="n">hello</span><span class="o">,</span> <span class="n">world</span><span class="o">!</span> <span class="o">&lt;/</span><span class="n">body</span><span class="o">&gt;</span><span class="p">}}</span></code></pre></figure>

<h5>Quotations in different contexts</h5>

<p>So far, we have ignored the question of what type is used to represent an AST
node. The standard library would need to provide such a type so that quotations
could be written without linking to compiler-libs. There would also need to be
different types for different kinds of AST nodes. We do not want a quotation
used as an expression returning an AST node that represents a pattern</p>

<p>However, we also might want to create quotations that can be used as both
expressions and patterns. This means that the quotation must return a different
type depending on where it is used.</p>

<p>The solution to this issue is to give quotations the type <code class="language-plaintext highlighter-rouge">'a ctx -&gt; string -&gt;
'a</code>.  The <code class="language-plaintext highlighter-rouge">ctx</code> type would be a GADT that described what context a quotation was
being used from. It could also contain other information about the context, such
as its location in the source file.</p>

<h5>Building Quotations</h5>

<p>Creating the quotation functions requires some facility for creating AST
nodes. For this purpose, the standard library would include special quotations,
for example: <code class="language-plaintext highlighter-rouge">{:Ast.expr{ x + 3 }}</code>. These quotations would be implemented
directly using the compiler’s lexer and parser.</p>

<p>It would also be useful (especially for handling anti-quotations) to allow
quotations to be built from other quotations. For this we could provide another
syntax: <code class="language-plaintext highlighter-rouge">{:foo}</code> that would refer directly to the quotation function
<code class="language-plaintext highlighter-rouge">foo</code>. Obviously, this syntax would only be allowed within “.mlq” files.</p>

<h4>Other extensions</h4>

<p>This system could easily be extended to other kinds of extension. Rather than
declaring “quotations” with type <code class="language-plaintext highlighter-rouge">'a ctx -&gt; string -&gt; 'a</code>, we could declare
<em>templates</em> with type <code class="language-plaintext highlighter-rouge">'a ctx -&gt; 'a</code>. The context would contain the arguments to
the template (a string for quotations, an AST node for other templates).</p>

<p>So a template declared as:</p>

<figure class="highlight"><pre><code class="language-ocaml" data-lang="ocaml"><span class="c">(* bar.mlq *)</span>
<span class="n">template</span> <span class="n">foo</span> <span class="n">str</span> <span class="o">=</span> <span class="o">...</span></code></pre></figure>

<p>could be used with the syntax:</p>

<figure class="highlight"><pre><code class="language-ocaml" data-lang="ocaml"><span class="p">(</span><span class="o">:</span><span class="nn">Bar</span><span class="p">.</span><span class="n">foo</span> <span class="n">expr</span><span class="p">)</span></code></pre></figure>

<p>Unlike quotations, more general templates must be able to interpret AST nodes
themselves. This means we must provide mechanisms for handling AST nodes. For
this purpose, the standard library would include a simpler version of the
<code class="language-plaintext highlighter-rouge">AstMapper</code> module that is in compiler-libs.</p>

<p>We could also allow the AST quotations (e.g. `Ast.expr) to be used as
patterns. This approach can be a bit fragile because syntactic sugar can cause a
pattern to match ASTs that it was not expected to match. However, for matching
simple AST nodes it is probably fairly robust.</p>

<h4>Summary</h4>

<ol>
  <li>Allow extensions to be written as OCaml functions within “.mlq” files.</li>
  <li>Refer to these functions by attaching them directly to namespaces.</li>
  <li>Require these functions to have type <code class="language-plaintext highlighter-rouge">'a ctx -&gt; 'a</code>, where <code class="language-plaintext highlighter-rouge">ctx</code> includes a
GADT describing the context that the extension has been used in.</li>
  <li>Provide AST quotations in the standard library (e.g. <code class="language-plaintext highlighter-rouge">{:Ast.expr{ x + 3 }}</code>)
which use the compiler’s own lexer and parser.</li>
</ol>

