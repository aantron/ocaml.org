---
title: Introducing DBCaml, Database toolkit for OCaml
description: Introducing DBCaml, Database toolkit for OCaml
url: https://priver.dev/blog/dbcaml/dbcaml/
date: 2024-02-20T00:01:38-00:00
preview_image: https://priver.dev/images/ocaml/dbcaml.jpeg
authors:
- "Emil Priv\xE9r"
source:
---

<p>It’s time for me to discuss what I’ve been working on recently: Dbcaml. Dbcaml is a database toolkit built for OCaml, based on Riot. Riot is an actor-model multi-core scheduler for OCaml 5. Riot works by creating lightweight processes that execute code and communicate with the rest of the system using message-passing.</p>
<p>You can find Riot on <a href="https://github.com/riot-ml/riot">GitHub</a>.</p>
<p>The core idea of DBCaml is to provide a toolkit that assists with the “boring” tasks you don’t want to deal with, allowing you to focus on your queries. Some examples of these tasks include:</p>
<ul>
<li><strong>Database pooling</strong>. Built with using Riots lightweight process to spin up a connection pool.</li>
<li><strong>Database Agnostic</strong>. Support for Postgres, and more to come(MySQL, MariaDB, SQLite)</li>
<li><strong>Built in security</strong>. With built in security allows users to focus on writing queries and don’t be afraid of security breaches.</li>
<li><strong>Cross Platform</strong>. DBCaml compiles anywhere</li>
<li><strong>Not an ORM</strong>: DBCaml is not an ORM. It simply handles the boring stuff that you don’t want to deal with and allows you to have full insight into what’s going on. However, it should be possible to build an ORM/query builder on top of DBCaml (more information about this later).</li>
<li>Mapping data to types.</li>
</ul>
<p>This is an example of how you can use DBCaml:</p>
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
</span><span class="lnt">19
</span></code></pre></td>
<td class="lntd">
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="k">let</span> <span class="n">driver</span> <span class="o">=</span>
</span></span><span class="line"><span class="cl">  <span class="nn">Dbcaml_driver_postgres</span><span class="p">.</span><span class="n">connection</span>
</span></span><span class="line"><span class="cl">    <span class="s2">"postgresql://postgres:mysecretpassword@localhost:6432/development"</span>
</span></span><span class="line"><span class="cl"><span class="k">in</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="n">pool_id</span> <span class="o">=</span> <span class="nn">Dbcaml</span><span class="p">.</span><span class="n">start_link</span> <span class="o">~</span><span class="n">connections</span><span class="o">:</span><span class="n">10</span> <span class="n">driver</span> <span class="o">|&gt;</span> <span class="nn">Result</span><span class="p">.</span><span class="n">get_ok</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="c">(* Fetch 1 row from the database *)</span>
</span></span><span class="line"><span class="cl"><span class="o">(</span><span class="k">match</span>
</span></span><span class="line"><span class="cl">   <span class="nn">Dbcaml</span><span class="p">.</span><span class="n">fetch_one</span>
</span></span><span class="line"><span class="cl">     <span class="n">pool_id</span>
</span></span><span class="line"><span class="cl">     <span class="o">~</span><span class="n">params</span><span class="o">:[</span><span class="nn">Dbcaml</span><span class="p">.</span><span class="nn">Param</span><span class="p">.</span><span class="nc">String</span> <span class="s2">"1"</span><span class="o">]</span>
</span></span><span class="line"><span class="cl">     <span class="s2">"select * from users where id = $1"</span>
</span></span><span class="line"><span class="cl"> <span class="k">with</span>
</span></span><span class="line"><span class="cl"><span class="o">|</span> <span class="nc">Ok</span> <span class="n">x</span> <span class="o">-&gt;</span>
</span></span><span class="line"><span class="cl">  <span class="k">let</span> <span class="n">rows</span> <span class="o">=</span> <span class="nn">Dbcaml</span><span class="p">.</span><span class="nn">Row</span><span class="p">.</span><span class="n">row_to_type</span> <span class="n">x</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl">  <span class="c">(* Iterate over each column and print its values *)</span>
</span></span><span class="line"><span class="cl">  <span class="nn">List</span><span class="p">.</span><span class="n">iter</span> <span class="o">(</span><span class="k">fun</span> <span class="n">x</span> <span class="o">-&gt;</span> <span class="n">print_endline</span> <span class="n">x</span><span class="o">)</span> <span class="n">rows</span>
</span></span><span class="line"><span class="cl"><span class="o">|</span> <span class="nc">Error</span> <span class="n">x</span> <span class="o">-&gt;</span> <span class="n">print_endline</span> <span class="o">(</span><span class="nn">Dbcaml</span><span class="p">.</span><span class="nn">Res</span><span class="p">.</span><span class="n">execution_error_to_string</span> <span class="n">x</span><span class="o">));</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><h2>Installation</h2>
<p>During the initial v0.0.1 release, DBCaml can be installed using the following command:</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre tabindex="0" class="chroma"><code><span class="lnt">1
</span></code></pre></td>
<td class="lntd">
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="n">opam</span> <span class="n">pin</span> <span class="n">dbcaml</span><span class="o">.</span><span class="n">0</span><span class="o">.</span><span class="n">0</span><span class="o">.</span><span class="n">1</span> <span class="n">git</span><span class="o">+</span><span class="n">https</span><span class="o">://</span><span class="n">github</span><span class="o">.</span><span class="n">com</span><span class="o">/</span><span class="n">dbcaml</span><span class="o">/</span><span class="n">dbcaml</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><h2>Background</h2>
<p>I wanted to learn a new language and decided to explore functional programming. I came across OCaml online and found it interesting. When Advent of Code 2023 started, I chose OCaml as the language for my solutions. However, I didn’t build them using a functional approach. Instead, I wrote them in a non-functional way, using a lot of references. My solutions turned out to be so bad that a colleague had to rewrite my code.</p>
<p>However, this experience further sparked my interest. One day, I came across <a href="https://twitter.com/leostera">Leostera</a>, a developer working on Riot, an actor-model multi-core scheduler for OCaml 5. Riot is similar to Erlang’s beam, which intrigued me. It dawned on me that if I wanted to explore OCaml further, I needed a project to work on. That’s when I made the decision to build a database library for OCaml. I believed that it would be a useful addition to the Rio ecosystem.</p>
<h2>How it works</h2>
<p>DBCaml can be categorized into three layers: Driver, Connection pool, and the interface that the developer works with. I have already explained how the connection pool works in a previous post, which you can find here: <a href="https://priver.dev/blog/dbcaml/building-a-connnection-pool/">Building a Connection Pool</a>.</p>
<p>However, I would like to provide further explanation on drivers and the interface.</p>
<h3>Driver</h3>
<p>The driver is responsible for communicating with the database. It acts as a bridge between DBCaml and the database. The main idea behind having separate drivers as a library is to avoid the need for installing unnecessary libraries. For example, if you are working with a Postgres database, there is no need to install any MySQL drivers. By keeping the drivers separate, unnecessary dependencies can be avoided.</p>
<p>Additionally, the driver takes care of all the security measures within the library. DBCaml simply provides the necessary data to the drivers, which handle the security aspects.</p>
<h3>The interface</h3>
<p>I will describe the current functionality of everything and explain my vision for how I believe this library will evolve in future releases.</p>
<p>Currently, DBCaml provides four methods: start_link, fetch_one, fetch_many, and exec. These methods serve as the highest level of functionality in the package and are the primary interface used by developers for testing purposes in v0.0.1. These methods handle most of the tasks that developers don’t need to worry about, such as requesting a connection from the pool.</p>
<h2>My vision</h2>
<p>I have a broad vision for DBCaml, which encompasses three categories: testing, development, and runtime. The specifics of what will be included in the testing and development areas will become clearer as we start working on it. However, currently, the most important aspect is to have a v0.0.1 release for the connection pool. This is the critical component of the system, and we need feedback on its functionality and to identify any potential bugs or issues.</p>
<h3>Testing</h3>
<p>Writing effective tests can be challenging, particularly when it is not possible to mock queries. However, one solution to this problem is to utilize DBCaml. DBCaml can help you in writing tests by providing reusable code snippets. This includes the ability to define rows, close a database, and more, giving you control over how you test your application.</p>
<h3>Developing</h3>
<p>I believe SQLx by Rust (<a href="https://github.com/launchbadge/sqlx">https://github.com/launchbadge/sqlx</a>) has done an excellent job of providing a great developer experience (DX). It allows users to receive feedback on the queries they write without the need to test them during runtime. In other words, SQLx enables the use of macros to execute code against the database during the compilation process. This way, any issues with the queries can be identified early on. It is, of course, optional for users to opt in to this feature.</p>
<p>The advantage of this feedback during development is that users can work quickly without having to manually send additional HTTP requests in tools like Postman to trigger the queries they want to test. This saves users valuable time.</p>
<p>By allowing users to test queries during compilation, they can skip writing tests for queries. This provides feedback on whether the query works or not during development.</p>
<h3>Runtime</h3>
<p>During runtime, it is important to have a system that can handle pooling for your application. This ensures that if a connection dies, it is recreated and booted again.</p>
<h2>Roadmap and future ideas</h2>
<p>Currently, we are in version v0.0.1, which is a small release with limited functionality. However, I have big plans for the future of this package. The purpose of creating v0.0.1, despite knowing that there will be upcoming changes, is to test the connection pool and ensure its functionality.</p>
<p>The v0.0.1 release includes the ability to fetch data from the database and use it, along with a connection pool and a PostgreSQL driver. However, I will soon be branching out DBCaml into three new packages:</p>
<ol>
<li>PoolParty (name subject to change): This package will provide a connection pool that anyone can use(not only for database connections).</li>
<li>DBCaml: This library will utilize PoolParty to run queries against the database and return streaming data or bytes. The reason for this change is to allow developers to use tools like <a href="http://serde.ml/">Serde.ml</a> to map data to types. By returning streaming data or bytes instead of fixed static types (such as strings), developers will be able to build libraries on top of DBCaml. For example, it would be possible to build a query builder on top of DBCaml and use it.</li>
<li>Library X: This high-level package will use DBCaml to make queries and map responses to types using serde.ml. I expect this to be the package that most developers will use. It will include functions like <code>fetch_one</code>, <code>fetch_many</code>, <code>start_link</code>, and <code>exec</code>, which already exist in DBCaml.</li>
</ol>
<p>This significant change will be implemented in the v0.0.2 milestone.</p>
<h2>The end</h2>
<p>I want to give a special thank you to Leostera, who has helped me a lot during the development. I wouldn’t argue that this is something I’ve just worked on. This is a joint effort between me, Leostera, and other members of the Riot Discord to make this happen.</p>
<p>If you are interested and would like to follow along with the development, I can recommend some links for you:</p>
<ul>
<li>Riot discord:&nbsp;https://discord.gg/CaykCHrAXN&nbsp;The core discussions about the project are conducted on the Riot discord.</li>
<li>The Caravan discord:&nbsp;https://discord.gg/XpAWHYbFB3</li>
<li>DBCaml GitHub:&nbsp;https://github.com/dbcaml/dbcaml</li>
</ul>

