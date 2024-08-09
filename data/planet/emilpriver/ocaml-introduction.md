---
title: 'OCaml: Introduction'
description: Introduction for OCaml, a blog post for developers that want to dig into
  OCaml
url: https://priver.dev/blog/ocaml/ocaml-introduction/
date: 2024-02-12T00:01:38-00:00
preview_image: https://priver.dev/images/ocaml.jpeg
authors:
- "Emil Priv\xE9r"
source:
---

<p>Time to delve into OCaml, a functional programming language that was first released in 1996 and has gained popularity in the academic world. This article is for those who are interested in OCaml and want to learn more about the language. It covers parts that I felt I needed to learn when I started with OCaml, and it’s a continuation of the “<a href="https://priver.dev/blog/functional-programming/concepts-of-functional-programming/">concepts of functional programming</a>” article I wrote a while ago.</p>
<p>Coming from a non-functional background myself, where I’ve written a lot of Rust and Go, I decided to give a functional programming language a try when Advent of Code started in 2023. OCaml seemed like the perfect choice, especially because it shares some features with Rust, such as matching, options, and pattern matching. However, in my opinion, unlike Haskell, OCaml is not a strict pure functional language.</p>
<p>This post will cover some important aspects when working with OCaml, such as how to define a function, modules, and the REPL. It will also highlight some features that I really like about it.</p>
<p>OCaml, previously known as Objective Caml (Categorical Abstract Machine Language), is a programming language that evolved from the ML programming language. ML is a functional programming language famous for its Polymorphic Hindley-Milner type system, which is derived from the lambda calculus and supports parametric polymorphism. Parametric polymorphism allows code to be written using generic types represented by variables, which can later be instantiated with specific types as needed.</p>
<p>OCaml extends ML with object-oriented features, which are recognized as classes and objects. Classes in OCaml are denoted by the usage of <code>#</code>. For example:</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre tabindex="0" class="chroma"><code><span class="lnt"> 1
</span><span class="lnt"> 2
</span><span class="lnt"> 3
</span><span class="lnt"> 4
</span><span class="lnt"> 5
</span><span class="lnt"> 6
</span><span class="lnt"> 7
</span><span class="lnt"> 8
</span><span class="lnt"> 9
</span><span class="lnt">10
</span><span class="lnt">11
</span><span class="lnt">12
</span></code></pre></td>
<td class="lntd">
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="k">class</span> <span class="n">point</span> <span class="o">=</span>
</span></span><span class="line"><span class="cl">    <span class="k">object</span>
</span></span><span class="line"><span class="cl">      <span class="k">val</span> <span class="k">mutable</span> <span class="n">x</span> <span class="o">=</span> <span class="n">0</span>
</span></span><span class="line"><span class="cl">      <span class="k">method</span> <span class="n">get_x</span> <span class="o">=</span> <span class="n">x</span>
</span></span><span class="line"><span class="cl">      <span class="k">method</span> <span class="n">move</span> <span class="n">d</span> <span class="o">=</span> <span class="n">x</span> <span class="o">&lt;-</span> <span class="n">x</span> <span class="o">+</span> <span class="n">d</span>
</span></span><span class="line"><span class="cl">    <span class="k">end</span><span class="o">;;</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="bp">()</span> <span class="o">=</span>
</span></span><span class="line"><span class="cl">	<span class="k">let</span> <span class="n">p</span> <span class="o">=</span> <span class="k">new</span> <span class="n">point</span><span class="o">;;</span>
</span></span><span class="line"><span class="cl">	<span class="k">let</span> <span class="kt">int</span> <span class="o">=</span> <span class="n">p</span><span class="o">#</span><span class="n">get_x</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl">	<span class="bp">()</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><p>OCaml is a statically typed language with inferred types, depending on how the value is used. For example:</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre tabindex="0" class="chroma"><code><span class="lnt">1
</span><span class="lnt">2
</span><span class="lnt">3
</span><span class="lnt">4
</span><span class="lnt">5
</span></code></pre></td>
<td class="lntd">
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="k">let</span> <span class="n">print_message</span> <span class="n">message</span> <span class="o">=</span>
</span></span><span class="line"><span class="cl">  <span class="n">print_endline</span> <span class="n">message</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="bp">()</span> <span class="o">=</span>
</span></span><span class="line"><span class="cl">  <span class="n">print_message</span> <span class="s2">"Hello, OCaml world!"</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><p>In this example, the compiler knows that <code>print_endline</code> requires a string type, which is inferred to <code>print_message</code>. This means that <code>print_message</code> needs a string and will not compile if provided with a different type.</p>
<p>In some languages you might need to define the types before you use them which could make your code look like the Haskell
code below:</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre tabindex="0" class="chroma"><code><span class="lnt">1
</span><span class="lnt">2
</span><span class="lnt">3
</span><span class="lnt">4
</span><span class="lnt">5
</span></code></pre></td>
<td class="lntd">
<pre tabindex="0" class="chroma"><code class="language-haskell" data-lang="haskell"><span class="line"><span class="cl"><span class="nf">printMessage</span> <span class="ow">::</span> <span class="kt">String</span> <span class="ow">-&gt;</span> <span class="kt">IO</span> <span class="nb">()</span>
</span></span><span class="line"><span class="cl"><span class="nf">printMessage</span> <span class="n">message</span> <span class="ow">=</span> <span class="n">putStrLn</span> <span class="n">message</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="nf">main</span> <span class="ow">::</span> <span class="kt">IO</span> <span class="nb">()</span>
</span></span><span class="line"><span class="cl"><span class="nf">main</span> <span class="ow">=</span> <span class="n">printMessage</span> <span class="s">"Hello, Haskell world!"</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><blockquote>
<p>If you want to learn more about functional programming in general, I have written a blog post called “<a href="https://priver.dev/blog/functional-programming/concepts-of-functional-programming/">Concepts of functional programming</a>” where I discuss topics such as immutability and pure functions.</p>
</blockquote>
<p>However, sometimes I still define the return type of a function as it can make it easier to define the function. By explicitly stating the return type, the compiler knows what to expect from the function.</p>
<h2>Basic</h2>
<p>Let’s start by creating a function and a variable and using them.</p>
<p>To create a variable in Ocaml, you can use the following syntax:</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre tabindex="0" class="chroma"><code><span class="lnt">1
</span></code></pre></td>
<td class="lntd">
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="k">let</span> <span class="n">variable</span> <span class="o">=</span> <span class="s2">"Hello :D"</span> <span class="k">in</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><p>Here, we create a variable named <code>variable</code> and assign the value “Hello :D” to it. The <code>in</code> keyword is used to indicate the end of the variable declaration.</p>
<p>The <code>in</code> keyword also allows us to pipe functions and store the result in a variable. For example:</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre tabindex="0" class="chroma"><code><span class="lnt">1
</span><span class="lnt">2
</span><span class="lnt">3
</span></code></pre></td>
<td class="lntd">
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="k">let</span> <span class="n">uppercase_string</span> <span class="o">=</span> <span class="s2">"Hello World :D"</span> <span class="o">|&gt;</span> <span class="nn">String</span><span class="p">.</span><span class="n">uppercase_ascii</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="n">print_endline</span> <span class="n">uppercase_string</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><p>In this example, we demonstrate how to perform multiple functions on the string “Hello World :D” and store only the necessary information in the <code>uppercase_string</code> variable. This allows us to call an HTTP API, parse the output, retrieve a value, convert it to uppercase, and store it.</p>
<p>Creating a function can be done in a similar way using <code>let</code>. We define the logic inside the function, as shown in the following code:</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre tabindex="0" class="chroma"><code><span class="lnt">1
</span><span class="lnt">2
</span><span class="lnt">3
</span><span class="lnt">4
</span><span class="lnt">5
</span><span class="lnt">6
</span><span class="lnt">7
</span><span class="lnt">8
</span></code></pre></td>
<td class="lntd">
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="k">let</span> <span class="n">hello</span> <span class="n">what_to_say_hello_to</span> <span class="o">=</span>
</span></span><span class="line"><span class="cl">  <span class="n">print_endline</span> <span class="n">what_to_say_hello_to</span><span class="o">;</span>
</span></span><span class="line"><span class="cl">  <span class="bp">()</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="bp">()</span> <span class="o">=</span>
</span></span><span class="line"><span class="cl">  <span class="n">hello</span> <span class="s2">"World"</span><span class="o">;</span>
</span></span><span class="line"><span class="cl">	<span class="c">(* Write more logic here *)</span>
</span></span><span class="line"><span class="cl">  <span class="bp">()</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><p><a href="https://ocaml.org/play#code=CmxldCBoZWxsbyB3aGF0X3RvX3NheV9oZWxsb190byA9CiAgcHJpbnRfZW5kbGluZSB3aGF0X3RvX3NheV9oZWxsb190bzsKICAoKQoKbGV0ICgpID0KICBoZWxsbyAiV29ybGQiOwoKICAoKQ=="><em>Playground</em></a></p>
<p>In this code, we create a function called <code>hello</code> that takes one argument. We use the argument to print something with <code>print_endline</code> and then return a unit. A unit represents “nothing”, so when we write a function that returns nothing, we actually return a unit. A bigger example of how to write function exists <a href="https://ocaml.org/play#code=KCoKICBXZWxjb21lIHRvIHRoZSBvZmZpY2lhbCBPQ2FtbCBQbGF5Z3JvdW5kIQoKICBZb3UgZG9uJ3QgbmVlZCB0byBpbnN0YWxsIGFueXRoaW5nIC0ganVzdCB3cml0ZSB5b3VyIGNvZGUKICBhbmQgc2VlIHRoZSByZXN1bHRzIGFwcGVhciBpbiB0aGUgT3V0cHV0IHBhbmVsLgoKICBUaGlzIHBsYXlncm91bmQgaXMgcG93ZXJlZCBieSBPQ2FtbCA1IHdoaWNoIGNvbWVzIHdpdGgKICBzdXBwb3J0IGZvciBzaGFyZWQtbWVtb3J5IHBhcmFsbGVsaXNtIHRocm91Z2ggZG9tYWlucyBhbmQgZWZmZWN0cy4KICBCZWxvdyBpcyBzb21lIG5haXZlIGV4YW1wbGUgY29kZSB0aGF0IGNhbGN1bGF0ZXMKICB0aGUgRmlib25hY2NpIHNlcXVlbmNlIGluIHBhcmFsbGVsLgogIAogIEhhcHB5IGhhY2tpbmchCiopCgpsZXQgbnVtX2RvbWFpbnMgPSAyCmxldCBuID0gMjAKCmxldCByZWMgZmliIG4gPQogIGlmIG4gPCAyIHRoZW4gMQogIGVsc2UgZmliIChuLTEpICsgZmliIChuLTIpCgpsZXQgcmVjIGZpYl9wYXIgbiBkID0KICBpZiBkIDw9IDEgdGhlbiBmaWIgbgogIGVsc2UKICAgIGxldCBhID0gZmliX3BhciAobi0xKSAoZC0xKSBpbgogICAgbGV0IGIgPSBEb21haW4uc3Bhd24gKGZ1biBfIC0%2BIGZpYl9wYXIgKG4tMikgKGQtMSkpIGluCiAgICBhICsgRG9tYWluLmpvaW4gYgoKbGV0ICgpID0KICBsZXQgcmVzID0gZmliX3BhciBuIG51bV9kb21haW5zIGluCiAgUHJpbnRmLnByaW50ZiAiZmliKCVkKSA9ICVkXG4iIG4gcmVzCgooKgogIEJ5IHRoZSB3YXksIGEgbXVjaCBiZXR0ZXIsIHNpbmdsZS10aHJlYWRlZCBpbXBsZW1lbnRhdGlvbiB0aGF0IGNhbGN1bGF0ZXMKICB0aGUgRmlib25hY2NpIHNlcXVlbmNlIGlzIHRoaXM6CgogIGxldCByZWMgZmliIG0gbiBpID0KICAgIGlmIGkgPCAxIHRoZW4gbQogICAgZWxzZSBmaWIgbiAobiArIG0pIChpIC0gMSkKCiAgbGV0IGZpYiA9IGZpYiAwIDEKCiAgRm9yIGEgbW9yZSBpbi1kZXB0aCwgcmVhbGlzdGljIGV4YW1wbGUgb2YgaG93IHRvIHVzZQogIHBhcmFsbGVsIGNvbXB1dGF0aW9uLCB0YWtlIGEgbG9vayBhdAogIGh0dHBzOi8vdjIub2NhbWwub3JnL3JlbGVhc2VzLzUuMC9tYW51YWwvcGFyYWxsZWxpc20uaHRtbCNzOnBhcl9pdGVyYXRvcnMKKikK">here</a>.</p>
<h2>REPL</h2>
<p>REPL, or read-eval-print loop, is an interactive interface that developers can use to debug, test, or understand code within their OCaml projects. When writing code in the REPL, the compiler runs checks on your types, compiles, evaluates, and prints the inferred type and result value. The REPL tool for OCaml is called utop and comes with opam and dune.</p>
<p>

  <picture>
    <source media="(max-width: 640px)" srcset="/images/ocaml/repl_hub52a1ccd647197c2dedd0ec304e6282b_126408_1280x0_resize_q100_h2_box_3.webp" type="image/webp" loading="lazy">
    <source media="(min-width: 640px)" srcset="/images/ocaml/repl_hub52a1ccd647197c2dedd0ec304e6282b_126408_1920x0_resize_q100_h2_box_3.webp" type="image/webp" loading="lazy">

    <img src="https://priver.dev/images/ocaml/repl_hub52a1ccd647197c2dedd0ec304e6282b_126408_1280x0_resize_q100_box_3.png" srcset="/images/ocaml/repl_hub52a1ccd647197c2dedd0ec304e6282b_126408_1280x0_resize_q100_box_3.png 640w, /images/ocaml/repl_hub52a1ccd647197c2dedd0ec304e6282b_126408_1920x0_resize_q100_box_3.png 800w" sizes="(max-width: 640px) 640px, 800px" loading="lazy" width="1280" alt="OCaml: Introduction">
  </picture></p>
<p>In the image above, I create a function called <code>hello</code> and execute it. In the output, I can see that the function <code>hello</code> takes 1 argument of type string and returns a unit (no return). I also create another function called <code>another_hello</code> where I call the <code>hello</code> function and print some additional values.</p>
<p>Even if I am not a big user of REPL is it a tool worth mention if you just want to play around with the language and explore your code.</p>
<h2>Modules</h2>
<p>This section discusses modules, which are a concept within OCaml that allow us to call functions from another file or create submodules (modules within a file).</p>
<p>In OCaml, files and folders are referred to as modules. We can use the names of the files to call functions within these files. For example, suppose we have two files in the same folder: <code>one.ml</code> and <code>two.ml</code>. The <code>two.ml</code> file contains a function called <code>greetings</code> that we want to call from <code>one.ml</code> to greet the user. To do this, we would write <code>let () = Two.greetings ()</code> in our <code>one.ml</code> file. Alternatively, we can use the <code>open Two</code> statement to access the <code>greetings</code> function without specifying <code>Two</code> before calling it:</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre tabindex="0" class="chroma"><code><span class="lnt">1
</span><span class="lnt">2
</span><span class="lnt">3
</span></code></pre></td>
<td class="lntd">
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="k">open</span> <span class="nc">Two</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="bp">()</span> <span class="o">=</span> <span class="n">greetings</span> <span class="bp">()</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><p>In addition to modules, we can also create submodules within a file. Submodules work similarly, but instead of the entire file being a module, we define a part of the file as a module by creating a new <code>Module</code> within the file. Here’s an example:</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre tabindex="0" class="chroma"><code><span class="lnt">1
</span><span class="lnt">2
</span><span class="lnt">3
</span><span class="lnt">4
</span><span class="lnt">5
</span></code></pre></td>
<td class="lntd">
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="k">module</span> <span class="nc">Hello</span> <span class="o">=</span> <span class="k">struct</span>
</span></span><span class="line"><span class="cl">   <span class="k">let</span> <span class="n">greetings</span> <span class="bp">()</span> <span class="o">=</span> <span class="n">print_endline</span> <span class="s2">"hello"</span>
</span></span><span class="line"><span class="cl"><span class="k">end</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="bp">()</span> <span class="o">=</span> <span class="nn">Hello</span><span class="p">.</span><span class="n">greetings</span> <span class="bp">()</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><p>Modules can also refer to packages installed using opam. For example, if we install Riot using <code>opam install riot</code> and later want to use Riot within our code, we can call Riot directly or use <code>open Riot</code>. In this particular case, we consider Riot to be a module, even if it was imported from outside the project. Example on how we can use Riot in our code:</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre tabindex="0" class="chroma"><code><span class="lnt"> 1
</span><span class="lnt"> 2
</span><span class="lnt"> 3
</span><span class="lnt"> 4
</span><span class="lnt"> 5
</span><span class="lnt"> 6
</span><span class="lnt"> 7
</span><span class="lnt"> 8
</span><span class="lnt"> 9
</span><span class="lnt">10
</span><span class="lnt">11
</span><span class="lnt">12
</span><span class="lnt">13
</span><span class="lnt">14
</span></code></pre></td>
<td class="lntd">
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="k">open</span> <span class="nc">Riot</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="k">type</span> <span class="nn">Message</span><span class="p">.</span><span class="n">t</span> <span class="o">+=</span> <span class="nc">Hello_world</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="bp">()</span> <span class="o">=</span>
</span></span><span class="line"><span class="cl">  <span class="nn">Riot</span><span class="p">.</span><span class="n">run</span> <span class="o">@@</span> <span class="k">fun</span> <span class="bp">()</span> <span class="o">-&gt;</span>
</span></span><span class="line"><span class="cl">  <span class="k">let</span> <span class="n">pid</span> <span class="o">=</span>
</span></span><span class="line"><span class="cl">    <span class="n">spawn</span> <span class="o">(</span><span class="k">fun</span> <span class="bp">()</span> <span class="o">-&gt;</span>
</span></span><span class="line"><span class="cl">        <span class="k">match</span> <span class="n">receive</span> <span class="bp">()</span> <span class="k">with</span>
</span></span><span class="line"><span class="cl">        <span class="o">|</span> <span class="nc">Hello_world</span> <span class="o">-&gt;</span>
</span></span><span class="line"><span class="cl">            <span class="nn">Logger</span><span class="p">.</span><span class="n">info</span> <span class="o">(</span><span class="k">fun</span> <span class="n">f</span> <span class="o">-&gt;</span> <span class="n">f</span> <span class="s2">"hello world from %a!"</span> <span class="nn">Pid</span><span class="p">.</span><span class="n">pp</span> <span class="o">(</span><span class="n">self</span> <span class="bp">()</span><span class="o">));</span>
</span></span><span class="line"><span class="cl">            <span class="n">shutdown</span> <span class="bp">()</span><span class="o">)</span>
</span></span><span class="line"><span class="cl">  <span class="k">in</span>
</span></span><span class="line"><span class="cl">  <span class="n">send</span> <span class="n">pid</span> <span class="nc">Hello_world</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><h2>Functors</h2>
<p>While discussing modules, it is also important to mention functors. A functor is a construct that takes a module as a parameter and returns a new module.</p>
<blockquote>
<p>The examples in this part are taken from the functors documentation</p>
</blockquote>
<p>To explain it further, imagine you need a specific functionality, such as handling sets. However, the required functionality does not exist for <code>Sets</code>. In this case, you can create a module that implements the same functions as a <code>Set</code> by using a functor.</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre tabindex="0" class="chroma"><code><span class="lnt">1
</span><span class="lnt">2
</span><span class="lnt">3
</span><span class="lnt">4
</span><span class="lnt">5
</span><span class="lnt">6
</span></code></pre></td>
<td class="lntd">
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="k">module</span> <span class="nc">StringCompare</span> <span class="o">=</span> <span class="k">struct</span>
</span></span><span class="line"><span class="cl">  <span class="k">type</span> <span class="n">t</span> <span class="o">=</span> <span class="kt">string</span>
</span></span><span class="line"><span class="cl">  <span class="k">let</span> <span class="n">compare</span> <span class="o">=</span> <span class="nn">String</span><span class="p">.</span><span class="n">compare</span>
</span></span><span class="line"><span class="cl"><span class="k">end</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="k">module</span> <span class="nc">StringSet</span> <span class="o">=</span> <span class="nn">Set</span><span class="p">.</span><span class="nc">Make</span><span class="o">(</span><span class="nc">StringCompare</span><span class="o">)</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><p>In the above example, we create a new <code>Set</code> called <code>StringCompare</code>, which has a type of <code>string</code> and a method called <code>compare</code> that calls <code>String.compare</code> when needed. With this code, we can now use <code>StringSet</code> and its functions.</p>
<p><em>Disclaimer: Some functions in this code intentionally do not exist in the “StringCompare” class. The purpose is to demonstrate how it works, rather than providing a perfect example. Including all the necessary code for a perfect example would result in excessive code.</em></p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre tabindex="0" class="chroma"><code><span class="lnt"> 1
</span><span class="lnt"> 2
</span><span class="lnt"> 3
</span><span class="lnt"> 4
</span><span class="lnt"> 5
</span><span class="lnt"> 6
</span><span class="lnt"> 7
</span><span class="lnt"> 8
</span><span class="lnt"> 9
</span><span class="lnt">10
</span><span class="lnt">11
</span><span class="lnt">12
</span></code></pre></td>
<td class="lntd">
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="k">module</span> <span class="nc">StringCompare</span> <span class="o">=</span> <span class="k">struct</span>
</span></span><span class="line"><span class="cl">  <span class="k">type</span> <span class="n">t</span> <span class="o">=</span> <span class="kt">string</span>
</span></span><span class="line"><span class="cl">  <span class="k">let</span> <span class="n">compare</span> <span class="o">=</span> <span class="nn">String</span><span class="p">.</span><span class="n">compare</span>
</span></span><span class="line"><span class="cl"><span class="k">end</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="k">module</span> <span class="nc">StringSet</span> <span class="o">=</span> <span class="nn">Set</span><span class="p">.</span><span class="nc">Make</span><span class="o">(</span><span class="nc">StringCompare</span><span class="o">)</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="o">_</span> <span class="o">=</span>
</span></span><span class="line"><span class="cl">  <span class="nn">In_channel</span><span class="p">.</span><span class="n">input_lines</span> <span class="n">stdin</span>
</span></span><span class="line"><span class="cl">  <span class="o">|&gt;</span> <span class="nn">List</span><span class="p">.</span><span class="n">concat_map</span> <span class="nn">Str</span><span class="p">.</span><span class="o">(</span><span class="n">split</span> <span class="o">(</span><span class="n">regexp</span> <span class="s2">"[ </span><span class="se">\\</span><span class="s2">t.,;:()]+"</span><span class="o">))</span>
</span></span><span class="line"><span class="cl">  <span class="o">|&gt;</span> <span class="nn">StringSet</span><span class="p">.</span><span class="n">of_list</span>
</span></span><span class="line"><span class="cl">  <span class="o">|&gt;</span> <span class="nn">StringSet</span><span class="p">.</span><span class="n">iter</span> <span class="n">print_endline</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><h2>Operators</h2>
<p>In OCaml, operators allow you to combine one or more values (operands) to create a new value. These operations can involve mathematical calculations, logical evaluations, or manipulation of data structures, among other actions.</p>
<h3><strong>Arithmetic Operators</strong></h3>
<p>Developers often use arithmetic operations in OCaml to perform mathematical calculations. Let’s explore the two types of arithmetic operations:</p>
<ul>
<li><strong>Integer Arithmetic</strong>: This type of arithmetic involves using the operators <strong><code>+</code></strong>, <strong><code>-</code></strong>, <strong><code>*</code></strong>, <strong><code>/</code></strong> for addition, multiplication, subtraction, and division, respectively.</li>
<li><strong>Floating-point Arithmetic</strong>: On the other hand, floating-point arithmetic uses the operators <strong><code>+.</code></strong> , <strong><code>-.</code></strong>, <strong><code>*.</code></strong>  and <strong><code>/.</code></strong> for addition, multiplication, subtraction, and division, respectively.</li>
</ul>
<p>And example usage of these are</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre tabindex="0" class="chroma"><code><span class="lnt">1
</span><span class="lnt">2
</span></code></pre></td>
<td class="lntd">
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="k">let</span> <span class="n">sum</span> <span class="o">=</span> <span class="n">1</span> <span class="o">+</span> <span class="n">2</span><span class="o">;;</span>          <span class="c">(* Integer addition *)</span>
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="n">difference</span> <span class="o">=</span> <span class="n">5</span><span class="o">.</span><span class="n">0</span> <span class="o">-.</span> <span class="n">3</span><span class="o">.</span><span class="n">0</span><span class="o">;;</span>  <span class="c">(* Floating-point subtraction *)</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><h3><strong>Comparison Operators</strong></h3>
<p>Comparison operations return either <code>true</code> or <code>false</code>. The most commonly used operands for comparison are:</p>
<ul>
<li><strong><code>=</code></strong> (equals)</li>
<li><strong><code>&lt;&gt;</code></strong> (not equals)</li>
<li><strong><code>&lt;</code></strong> (less than)</li>
<li><strong><code>&gt;</code></strong> (greater than)</li>
<li><strong><code>&lt;=</code></strong> (less than or equal to)</li>
<li><strong><code>&gt;=</code></strong> (greater than or equal to)</li>
</ul>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre tabindex="0" class="chroma"><code><span class="lnt">1
</span><span class="lnt">2
</span></code></pre></td>
<td class="lntd">
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="k">let</span> <span class="n">isEqual</span> <span class="o">=</span> <span class="n">3</span> <span class="o">=</span> <span class="n">3</span><span class="o">;;</span>          <span class="c">(* true *)</span>
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="n">isGreater</span> <span class="o">=</span> <span class="n">5</span> <span class="o">&gt;</span> <span class="n">3</span><span class="o">;;</span>        <span class="c">(* true *)</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><h3>Binary Operators</h3>
<p>Binary operators in OCaml are regular functions, but they are used in a slightly different way. A binary operator allows us to simplify development by assigning logic to an operand, which can then be used instead of calling a function. The operand is defined using parentheses and can use the surrounding arguments. In the example below, the arguments are “hi” and “friend”.</p>
<p>It is important to note that if you define the operand name with only one character, the operand function will expect only one argument. Therefore, if you want to achieve a result similar to the example below, you need to use two characters when creating the operand.</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre tabindex="0" class="chroma"><code><span class="lnt">1
</span><span class="lnt">2
</span><span class="lnt">3
</span></code></pre></td>
<td class="lntd">
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="k">let</span> <span class="n">cat</span> <span class="n">s1</span> <span class="n">s2</span> <span class="o">=</span> <span class="n">s1</span> <span class="o">^</span> <span class="s2">" "</span> <span class="o">^</span> <span class="n">s2</span><span class="o">;;</span>
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="o">(</span> <span class="o">^?</span> <span class="o">)</span> <span class="o">=</span> <span class="n">cat</span><span class="o">;;</span>
</span></span><span class="line"><span class="cl"><span class="n">print_endline</span> <span class="o">(</span><span class="s2">"hi"</span> <span class="o">^?</span> <span class="s2">"friend"</span><span class="o">);;</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><p>In this example, we create a function called <code>cat</code> that takes two strings and returns a string. Then, we create an operand and assign the <code>cat</code> function to it. Later, we use this operand to add a space between “hi” and “friend”.</p>
<h3>Binding Operators</h3>
<p>Binding operators are quite handy when writing OCaml code as they provide a way to simplify the code. These operators allow us to create custom <code>let</code> bindings by assigning them a value. They can be useful in cases where we only want to handle successful values and don’t need to handle negative values that may occur in applications. For example, when dealing with an unsuccessful HTTP call, we may only want to handle an OK response, and a binding operator can be a useful tool.</p>
<p>Let’s use an example from the operator documentation on <a href="http://ocaml.org/">ocaml.org</a>:</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre tabindex="0" class="chroma"><code><span class="lnt"> 1
</span><span class="lnt"> 2
</span><span class="lnt"> 3
</span><span class="lnt"> 4
</span><span class="lnt"> 5
</span><span class="lnt"> 6
</span><span class="lnt"> 7
</span><span class="lnt"> 8
</span><span class="lnt"> 9
</span><span class="lnt">10
</span><span class="lnt">11
</span><span class="lnt">12
</span><span class="lnt">13
</span><span class="lnt">14
</span><span class="lnt">15
</span><span class="lnt">16
</span><span class="lnt">17
</span></code></pre></td>
<td class="lntd">
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="k">let</span> <span class="o">(</span> <span class="k">let</span><span class="o">*</span> <span class="o">)</span> <span class="o">=</span> <span class="nn">Option</span><span class="p">.</span><span class="n">bind</span><span class="o">;;</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="n">doi_parts</span> <span class="n">s</span> <span class="o">=</span>
</span></span><span class="line"><span class="cl">  <span class="k">let</span> <span class="k">open</span> <span class="nc">String</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl">  <span class="k">let</span><span class="o">*</span> <span class="n">slash</span> <span class="o">=</span> <span class="n">rindex_opt</span> <span class="n">s</span> <span class="sc">'/'</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl">  <span class="k">let</span><span class="o">*</span> <span class="n">dot</span> <span class="o">=</span> <span class="n">rindex_from_opt</span> <span class="n">s</span> <span class="n">slash</span> <span class="sc">'.'</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl">  <span class="k">let</span> <span class="n">prefix</span> <span class="o">=</span> <span class="n">sub</span> <span class="n">s</span> <span class="n">0</span> <span class="n">dot</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl">  <span class="k">let</span> <span class="n">len</span> <span class="o">=</span> <span class="n">slash</span> <span class="o">-</span> <span class="n">dot</span> <span class="o">-</span> <span class="n">1</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl">  <span class="k">if</span> <span class="n">len</span> <span class="o">&gt;=</span> <span class="n">4</span> <span class="o">&amp;&amp;</span> <span class="n">ends_with</span> <span class="o">~</span><span class="n">suffix</span><span class="o">:</span><span class="s2">"10"</span> <span class="n">prefix</span> <span class="k">then</span>
</span></span><span class="line"><span class="cl">    <span class="k">let</span> <span class="n">registrant</span> <span class="o">=</span> <span class="n">sub</span> <span class="n">s</span> <span class="o">(</span><span class="n">dot</span> <span class="o">+</span> <span class="n">1</span><span class="o">)</span> <span class="n">len</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl">    <span class="k">let</span> <span class="n">identifier</span> <span class="o">=</span> <span class="n">sub</span> <span class="n">s</span> <span class="o">(</span><span class="n">slash</span> <span class="o">+</span> <span class="n">1</span><span class="o">)</span> <span class="o">(</span><span class="n">length</span> <span class="n">s</span> <span class="o">-</span> <span class="n">slash</span> <span class="o">-</span> <span class="n">1</span><span class="o">)</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl">    <span class="nc">Some</span> <span class="o">(</span><span class="n">registrant</span><span class="o">,</span> <span class="n">identifier</span><span class="o">)</span>
</span></span><span class="line"><span class="cl">  <span class="k">else</span>
</span></span><span class="line"><span class="cl">    <span class="nc">None</span><span class="o">;;</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="n">doi_parts</span> <span class="s2">"doi:10.1000/182"</span><span class="o">;;</span> <span class="c">(* Some ("1000", "182") *)</span>
</span></span><span class="line"><span class="cl"><span class="n">doi_parts</span> <span class="s2">"&lt;https://doi.org/10.1000/182&gt;"</span><span class="o">;;</span> <span class="c">(* Some ("1000", "182") *)</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><p>In this code, we create a binding operator called <code>let*</code> and assign it to <code>Option.bind</code>. This allows us to return early if we don’t find <code>/</code> in the provided string when running <code>let* slash = rindex_opt s '/'</code> within the code. Thanks to the <code>let*</code>, we no longer need to match the value on <code>rindex_from_opt</code>, as we will already return <code>None</code> if there is no <code>/</code> in the string.</p>
<h2>End</h2>
<p>I hope you enjoyed this article. It covers the topics I found necessary to learn when transitioning to OCaml from a background primarily in Go and Rust. The most challenging aspect of starting with OCaml was not the language itself, but rather breaking free from a non-functional programming mindset. An interesting example of this occurred when I shared my Advent of Code challenge with a colleague. He couldn’t resist rewriting the entire code because I had used <a href="https://cs3110.github.io/textbook/chapters/mut/refs.html">refs</a> to solve it.</p>
<p>I want to express my sincere gratitude to everyone in the Caravan Discord community. When I asked if there was anything else I should cover, people responded happily. If you’re interested in joining a Discord community full of functional programming enthusiasts, here is the link:&nbsp;<a href="https://discord.gg/cvSTjvxDfU">Caravan Discord Community</a>.</p>

