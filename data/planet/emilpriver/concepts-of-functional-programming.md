---
title: Concepts of Functional Programming
description: This post explores the concepts of functional programming, including
  immutability, pure functions, higher-order functions, recursion, and more. It also
  delves into the history of functional programming and introduces Lambda Calculus.
  If you're new to functional programming or want to deepen your understanding, this
  post is for you.
url: https://priver.dev/blog/functional-programming/concepts-of-functional-programming/
date: 2024-01-16T20:01:38-00:00
preview_image: https://priver.dev/images/ocaml.jpeg
authors:
- "Emil Priv\xE9r"
source:
---

<p>Hello and welcome to this post about the concepts of functional programming. This article is written for developers who want to be introduced to functional programming. The idea is to describe some important concepts within functional programming and then continue discussing OCaml in future posts. The topics covered in this post are:</p>
<ul>
<li>Immutability</li>
<li>Pure functions (isolated functions)</li>
<li>Higher-order functions</li>
<li>First-class functions</li>
<li>Recursion</li>
<li>Referential transparency</li>
<li>Statements and Expressions</li>
<li>Currying and Partial Application</li>
</ul>
<h2>History</h2>
<p>Before we dive into these concepts, let’s explore the history of functional programming. Understanding the purpose of functional programming can make it easier for you to write code in a functional style.</p>
<p>Functional programming is based on a mathematical foundation called Lambda Calculus, which was introduced by Alonzo Church in the 1930s. Lambda Calculus provides a formal system for expressing computation through function abstraction and application, using variable binding. This concept goes beyond functional programming and is also used in mathematics, physics, and philosophy.</p>
<p>Lambda Calculus can be understood using the following principles:</p>
<ul>
<li>Variable: A variable holds a value and is often represented by equations in math. For example, <code>x = y * 6</code> means that <code>x</code> is equal to <code>y</code> multiplied by 6. In programming, this could be written as <code>let t = 5 * 6</code>, where <code>t</code> holds the value of <code>5 * 6</code>.</li>
<li>Lambda Abstraction: Lambda abstraction is a way to define a function in Lambda Calculus. A function is written as λx.M, where “λ” represents a function, “x” is the input or parameter, and “M” defines what the function does with that input.</li>
<li>Application: This is how you use a function. If you have a function M and you want to apply it to an input N, you write it as (M N), similar to calling a function with an argument in programming.</li>
</ul>
<p>There are also rules for simplifying or ‘reducing’ expressions in Lambda Calculus:</p>
<ul>
<li><strong>α-conversion</strong>: This involves renaming variables to avoid confusion. If two different functions use the same variable name, you rename them to prevent clashes.</li>
<li><strong>β-reduction</strong>: This is where the actual computation happens. β-reduction involves replacing the variable in the function with the actual input value and then simplifying the result.</li>
</ul>
<p>For example, if your function is λx.(x+2) and your input is 3, applying the function to the input would mean replacing x with 3, resulting in (3+2), which simplifies to 5.</p>
<p>Lastly, there’s a concept called De Bruijn indexing, which is an alternative way of writing things to avoid the need for α-conversion. By repeatedly applying these reduction rules, you can reach a point where you can no longer simplify further, known as the β-normal form.</p>
<p>In simpler terms, lambda calculus provides a minimalistic way to describe functions and how they are applied, with specific rules for manipulating these descriptions to obtain results.</p>
<p>However, not all functional programming languages strictly adhere to these rules. For example, LISP, created back in the 1950s, had a limited impact from Lambda Calculus on the language.</p>
<h2>Immutability</h2>
<p>The concept of immutability is straightforward: it refers to the inability to change the initial state of a variable. This means that once we create a variable, we cannot modify its initial state. Instead, we need to take the data, use it, and return a new state.</p>
<p>The following code is an example that could work in a non-functional language like Rust (although the code provided is not in Rust, but rather in OCaml, imagine it as an example of how it could look in Rust).</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre tabindex="0" class="chroma"><code><span class="lnt">1
</span><span class="lnt">2
</span></code></pre></td>
<td class="lntd">
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="k">let</span> <span class="n">x</span> <span class="o">=</span> <span class="n">5</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl"><span class="n">x</span> <span class="o">:=</span> <span class="n">10</span> <span class="c">(* This will result in a compile-time error *)</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><p>And the example below demonstrates how we can work with data when the language enforces immutability.</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre tabindex="0" class="chroma"><code><span class="lnt">1
</span><span class="lnt">2
</span></code></pre></td>
<td class="lntd">
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="k">let</span> <span class="n">x</span> <span class="o">=</span> <span class="n">5</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="n">y</span> <span class="o">=</span> <span class="n">x</span> <span class="o">+</span> <span class="n">10</span> <span class="k">in</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><h2>Pure functions (isolated functions)</h2>
<p>I refer to pure functions as “isolated” functions. By isolated, I mean that nothing outside of the function should be able to alter its output. If you use the same arguments for a function, it should always return the same output. The concept of a pure function is that its output should remain unchanged if the arguments provided to the function are the same. In other words, nothing external to the function should have the ability to modify the output.</p>
<p>An example of a non-pure function is the one below, written in JavaScript:</p>
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
<pre tabindex="0" class="chroma"><code class="language-jsx" data-lang="jsx"><span class="line"><span class="cl"><span class="kd">function</span> <span class="nx">calculate</span><span class="p">(</span><span class="nx">input</span><span class="p">)</span> <span class="p">{</span>
</span></span><span class="line"><span class="cl">  <span class="k">return</span> <span class="nx">input</span> <span class="o">*</span> <span class="nx">externalFactor</span><span class="p">;</span>
</span></span><span class="line"><span class="cl"><span class="p">}</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="kd">let</span> <span class="nx">externalFactor</span> <span class="o">=</span> <span class="mi">10</span><span class="p">;</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="nx">console</span><span class="p">.</span><span class="nx">log</span><span class="p">(</span><span class="nx">calculate</span><span class="p">(</span><span class="mi">5</span><span class="p">));</span> <span class="c1">// Output will be 50
</span></span></span><span class="line"><span class="cl"><span class="c1"></span>
</span></span><span class="line"><span class="cl"><span class="c1">// Changing the external variable
</span></span></span><span class="line"><span class="cl"><span class="c1"></span><span class="nx">externalFactor</span> <span class="o">=</span> <span class="mi">20</span><span class="p">;</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="nx">console</span><span class="p">.</span><span class="nx">log</span><span class="p">(</span><span class="nx">calculate</span><span class="p">(</span><span class="mi">5</span><span class="p">));</span> <span class="c1">// Output will be 100
</span></span></span></code></pre></td></tr></tbody></table>
</div>
</div><p>And this is an example of a pure function in OCaml:</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre tabindex="0" class="chroma"><code><span class="lnt">1
</span><span class="lnt">2
</span><span class="lnt">3
</span><span class="lnt">4
</span></code></pre></td>
<td class="lntd">
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="k">let</span> <span class="n">square</span> <span class="n">x</span> <span class="o">=</span> <span class="n">x</span> <span class="o">*</span> <span class="n">x</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="n">result</span> <span class="o">=</span> <span class="n">square</span> <span class="n">5</span>
</span></span><span class="line"><span class="cl"><span class="c">(* result will always be 25 whenever square 5 is called *)</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><p>Another characteristic of pure functions is that they don’t change any global or local variables, or input/output streams.</p>
<h2>Higher-order and First-class functions</h2>
<p>First-class functions are a feature of programming languages that allow functions to be treated as values. This means that functions can be passed as arguments to other functions, returned from functions, or assigned to variables. This concept is often referred to as “first-class citizens” or “first-class objects.”</p>
<p>Here is an example in OCaml:</p>
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
</span></code></pre></td>
<td class="lntd">
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="c">(* Define a simple function *)</span>
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="n">multiply</span> <span class="n">x</span> <span class="n">y</span> <span class="o">=</span> <span class="n">x</span> <span class="o">*</span> <span class="n">y</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="c">(* Assigning a function to a variable *)</span>
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="n">double</span> <span class="o">=</span> <span class="n">multiply</span> <span class="n">2</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="c">(* Using the first-class function *)</span>
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="n">result</span> <span class="o">=</span> <span class="n">double</span> <span class="n">5</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl"><span class="c">(* result will be 10, as double is a partial application of multiply with 2 *)</span>
</span></span><span class="line"><span class="cl"><span class="n">print_int</span> <span class="n">result</span><span class="o">;</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><p>On the other hand, higher-order functions rely on the existence of first-class functions. A higher-order function is a function that either takes functions as arguments and executes them or returns a new function.</p>
<p>Here is an example in OCaml:</p>
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
</span></code></pre></td>
<td class="lntd">
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="c">(* A high-order function that takes a function 'f' and applies it to the number 3 *)</span>
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="n">apply_to_three</span> <span class="n">f</span> <span class="o">=</span> <span class="n">f</span> <span class="n">3</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="c">(* A simple function to be used with apply_to_three *)</span>
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="n">square</span> <span class="n">x</span> <span class="o">=</span> <span class="n">x</span> <span class="o">*</span> <span class="n">x</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="c">(* Using the high-order function *)</span>
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="n">result</span> <span class="o">=</span> <span class="n">apply_to_three</span> <span class="n">square</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl"><span class="c">(* result will be 9, as square is applied to 3 *)</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="n">print_int</span> <span class="n">result</span><span class="o">;</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><h2>Recursion</h2>
<p>Recursion in a functional programming language refers to a function that calls itself within the function until it reaches a base case or condition.</p>
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
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="c">(* A recursive function to calculate the factorial of a number *)</span>
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="k">rec</span> <span class="n">factorial</span> <span class="n">n</span> <span class="o">=</span>
</span></span><span class="line"><span class="cl">  <span class="k">if</span> <span class="n">n</span> <span class="o">&lt;=</span> <span class="n">1</span> <span class="k">then</span> <span class="n">1</span>
</span></span><span class="line"><span class="cl">  <span class="k">else</span> <span class="n">n</span> <span class="o">*</span> <span class="n">factorial</span> <span class="o">(</span><span class="n">n</span> <span class="o">-</span> <span class="n">1</span><span class="o">)</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="c">(* Using the recursive function *)</span>
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="n">result</span> <span class="o">=</span> <span class="n">factorial</span> <span class="n">5</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl"><span class="c">(* result will be 120, as the factorial of 5 is 5 * 4 * 3 * 2 * 1 *)</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><p>Using recursion can be more computationally expensive than using iteration due to the overhead of function calls and control shifting from one function to another. However, there are recursive patterns, such as tail recursion, that the compiler can optimize. A tail recursive function is a function where the only recursive call is the last one in the function.</p>
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
</span></code></pre></td>
<td class="lntd">
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="c">(* A tail-recursive function to calculate the factorial of a number *)</span>
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="n">factorial</span> <span class="n">n</span> <span class="o">=</span>
</span></span><span class="line"><span class="cl">  <span class="k">let</span> <span class="k">rec</span> <span class="n">aux</span> <span class="n">n</span> <span class="n">acc</span> <span class="o">=</span>
</span></span><span class="line"><span class="cl">    <span class="k">if</span> <span class="n">n</span> <span class="o">&lt;=</span> <span class="n">1</span> <span class="k">then</span> <span class="n">acc</span>
</span></span><span class="line"><span class="cl">    <span class="k">else</span> <span class="n">aux</span> <span class="o">(</span><span class="n">n</span> <span class="o">-</span> <span class="n">1</span><span class="o">)</span> <span class="o">(</span><span class="n">n</span> <span class="o">*</span> <span class="n">acc</span><span class="o">)</span>
</span></span><span class="line"><span class="cl">  <span class="k">in</span>
</span></span><span class="line"><span class="cl">  <span class="n">aux</span> <span class="n">n</span> <span class="n">1</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="c">(* Using the tail-recursive function *)</span>
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="n">result</span> <span class="o">=</span> <span class="n">factorial</span> <span class="n">5</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl"><span class="c">(* result will be 120, as factorial of 5 is 5 * 4 * 3 * 2 * 1 *)</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><p>Recursive functions were created as an alternative to <code>while</code> and <code>for</code> loops in order to eliminate the need for them. However, it seems that most functional languages still support them, such as <a href="https://ocaml.org/docs/if-statements-and-loops#for-loops-and-while-loops">OCaml</a>.</p>
<h2>Referential transparency</h2>
<p>The concept of referential transparency in a functional language is that the initial value of a variable remains constant throughout the program. Instead of directly modifying a variable when we need to change its value, we create a new variable. The benefit of referential transparency is that it helps prevent any side effects that could occur.</p>
<h3><strong>Equational reasoning</strong></h3>
<p>Equational reasoning is a concept that is often discussed when working with referential transparency. The reason for this is that equational reasoning requires referential transparency in order to be valid. This means that expressions can be replaced with their corresponding values or equivalent expressions without changing the program’s behavior. Let’s illustrate this with a code example:</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre tabindex="0" class="chroma"><code><span class="lnt">1
</span><span class="lnt">2
</span></code></pre></td>
<td class="lntd">
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="k">let</span> <span class="n">square</span> <span class="n">x</span> <span class="o">=</span> <span class="n">x</span> <span class="o">*</span> <span class="n">x</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="n">sum_of_squares</span> <span class="n">a</span> <span class="n">b</span> <span class="o">=</span> <span class="n">square</span> <span class="n">a</span> <span class="o">+</span> <span class="n">square</span> <span class="n">b</span> <span class="k">in</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><p>In this simple code, we define a function called <code>square</code> that takes one argument and squares it. Then, we define another function called <code>sum_of_squares</code> that takes two arguments and squares both of them. If we later call <code>sum_of_squares</code> with the arguments 3 and 4, we get the result of 25.</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre tabindex="0" class="chroma"><code><span class="lnt">1
</span><span class="lnt">2
</span></code></pre></td>
<td class="lntd">
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="k">let</span> <span class="n">res</span> <span class="o">=</span> <span class="n">sum_of_squares</span> <span class="n">3</span> <span class="n">4</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl"><span class="n">print_int</span> <span class="n">res</span><span class="o">;</span> <span class="c">(* prints 25 *)</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><p>So what is the equivalent reasoning of this concept? Well, now that we know that <code>sum_of_squares</code> summarizes two <code>square</code> function invocations, could we replace <code>let res = sum_of_squares 3 4 in</code> with <code>let res = square 3 + square 4 in</code> and then continue to break down the function even more with <code>let res = (3 * 3) + (4 * 4) in</code> and so on. This only works if the function we work with is a pure function (no side effects) and immutability, as otherwise a variable somewhere else could modify the output every time we run the function with the same arguments, resulting in different output.</p>
<h2>Statements and Expressions</h2>
<p>Another topic that I’ve heard might be good to talk about in relation to functional programming is statements and expressions.</p>
<p>Expressions are elements in programming that are expected to yield a value. Examples include:</p>
<ul>
<li>A string with content: “Hello World”</li>
<li>Function invocations: <code>square 4 5</code></li>
</ul>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre tabindex="0" class="chroma"><code><span class="lnt">1
</span><span class="lnt">2
</span></code></pre></td>
<td class="lntd">
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="k">let</span> <span class="n">square</span> <span class="n">x</span> <span class="o">=</span> <span class="n">x</span> <span class="o">*</span> <span class="n">x</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="n">result</span> <span class="o">=</span> <span class="n">square</span> <span class="n">5</span> <span class="k">in</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><p>Statements are actions that a program takes, but they do not yield a value. A good example of this is if/else statements, which allow the program to operate within the program.</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre tabindex="0" class="chroma"><code><span class="lnt">1
</span><span class="lnt">2
</span><span class="lnt">3
</span></code></pre></td>
<td class="lntd">
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="k">let</span> <span class="n">x</span> <span class="o">=</span> <span class="n">5</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl"><span class="k">if</span> <span class="n">x</span> <span class="o">&gt;</span> <span class="n">10</span> <span class="k">then</span>
</span></span><span class="line"><span class="cl">	<span class="n">print_string</span> <span class="s2">"Hello World"</span><span class="o">;</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><h2>Currying and Partial Application</h2>
<p>Currying and partial application is also 2 terms that I’ve heard might be something good to add and they also seem to be easy to mix up. Currying means pretty much that you are able to transform multiple arguments into sequence of functions. Example:</p>
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
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="c">(* Define a function that takes two arguments *)</span>
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="n">add</span> <span class="n">x</span> <span class="n">y</span> <span class="o">=</span> <span class="n">x</span> <span class="o">+</span> <span class="n">y</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="c">(* Currying: Created a curried function that returns a new function which executes the add function *)</span>
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="n">add_curried</span> <span class="n">x</span> <span class="o">=</span> <span class="o">(</span><span class="k">fun</span> <span class="n">y</span> <span class="o">-&gt;</span> <span class="n">add</span> <span class="n">x</span> <span class="n">y</span><span class="o">)</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="c">(* Usage *)</span>
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="n">add_to_5</span> <span class="o">=</span> <span class="n">add_curried</span> <span class="n">5</span> <span class="k">in</span> 
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="c">(* This is now a function that adds 5 to its argument *)</span>
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="n">result</span> <span class="o">=</span> <span class="n">add_to_5</span> <span class="n">7</span> <span class="k">in</span> 
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="c">(* result is 12 *)</span>
</span></span><span class="line"><span class="cl"><span class="n">print_int</span> <span class="n">result</span><span class="o">;</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><p>Currying is a useful technique when we want to pre-load a function before running it step-by-step, especially when we don’t know all the arguments yet. Essentially, currying means “taking one argument at a time, no more, no less”.</p>
<p>One example of currying in OCaml is <code>Array.iter</code>, where the function passed to <code>Array.iter</code> is executed with each element of the array.</p>
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
</span><span class="lnt">18
</span></code></pre></td>
<td class="lntd">
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="k">let</span> <span class="n">ids</span> <span class="o">=</span> <span class="nn">Array</span><span class="p">.</span><span class="n">init</span> <span class="n">10</span> <span class="o">(</span><span class="k">fun</span> <span class="n">i</span> <span class="o">-&gt;</span> <span class="n">i</span> <span class="o">+</span> <span class="n">2</span><span class="o">)</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl"><span class="nn">Array</span><span class="p">.</span><span class="n">iter</span>
</span></span><span class="line"><span class="cl">  <span class="o">(</span><span class="k">fun</span> <span class="n">i</span> <span class="o">-&gt;</span>
</span></span><span class="line"><span class="cl">    <span class="nn">Printf</span><span class="p">.</span><span class="n">printf</span> <span class="s2">"%d"</span> <span class="n">i</span><span class="o">;</span>
</span></span><span class="line"><span class="cl">    <span class="n">print_newline</span> <span class="bp">()</span><span class="o">)</span>
</span></span><span class="line"><span class="cl">  <span class="n">ids</span><span class="o">;</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="c">(* This would print *)</span>
</span></span><span class="line"><span class="cl"><span class="n">2</span>
</span></span><span class="line"><span class="cl"><span class="n">3</span>
</span></span><span class="line"><span class="cl"><span class="n">4</span>
</span></span><span class="line"><span class="cl"><span class="n">5</span>
</span></span><span class="line"><span class="cl"><span class="n">6</span>
</span></span><span class="line"><span class="cl"><span class="n">7</span>
</span></span><span class="line"><span class="cl"><span class="n">8</span>
</span></span><span class="line"><span class="cl"><span class="n">9</span>
</span></span><span class="line"><span class="cl"><span class="n">10</span>
</span></span><span class="line"><span class="cl"><span class="n">11</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><p>In the example above, the variable <code>i</code> in <code>fun i -&gt;</code> changes with each iteration of the array <code>ids</code>.</p>
<p>Partial application, on the other hand, allows us to provide the available arguments instead of one argument at a time. This means that if we have a function that takes 3 arguments and we only have 2 of them, we can pre-load the function by passing in the 2 arguments we have and add the third argument later when we have it.</p>
<p>Code example</p>
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
</span></code></pre></td>
<td class="lntd">
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="c">(* A function that takes three arguments *)</span>
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="n">multiply</span> <span class="n">x</span> <span class="n">y</span> <span class="n">z</span> <span class="o">=</span> <span class="n">x</span> <span class="o">*</span> <span class="n">y</span> <span class="o">*</span> <span class="n">z</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="c">(* Partially applying the first argument *)</span>
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="n">multiply_by_2</span> <span class="o">=</span> <span class="n">multiply</span> <span class="n">2</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="c">(* Now 'multiply_by_2' is a function that takes two arguments. *)</span>
</span></span><span class="line"><span class="cl"><span class="c">(* We can apply the remaining arguments *)</span>
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="n">result</span> <span class="o">=</span> <span class="n">multiply_by_2</span> <span class="n">3</span> <span class="n">4</span> <span class="c">(* This will be 2 * 3 * 4 = 24 *)</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl"><span class="n">print_int</span> <span class="n">result</span><span class="o">;</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><h2>The end</h2>
<p>Thank you for reading this. The purpose of this post is to prepare for more OCaml content. I believe it is easier to grasp the concept of functional programming before actually writing functional code. Many things become clearer when you start coding, but reading about it also helps to make sense of it all.</p>
<p>I want to express my sincere gratitude to everyone in the Caravan Discord community. When I asked if there was anything else I should cover, people responded happily. If you’re interested in joining a Discord community full of functional programming enthusiasts, here is the link: <a href="https://discord.gg/cvSTjvxDfU">Caravan Discord Community</a>.</p>
<p>Sometimes I post about OCaml and Rust on my Twitter page, <a href="https://twitter.com/emil_priver">https://twitter.com/emil_priver</a>. If you find that interesting, feel free to check it out! And if you think there’s something missing in this post, don’t hesitate to reach out to me.</p>

