---
title: Building a Connnection Pool for DBCaml on top of riot
description: This article talks about how I wrote the connection pool for DBCaml
url: https://priver.dev/blog/dbcaml/building-a-connnection-pool/
date: 2024-02-19T12:00:23-00:00
preview_image: https://priver.dev/images/poolparty.jpeg
authors:
- "Emil Priv\xE9r"
source:
---

<p>While developing DBCaml, I needed to build a connection pool to manage all the connections to the database. This allows us to create a pool of connections that can be used by multiple processes to send queries to the database and then return the connection back to the pool. In this post, I will provide a more in-depth explanation of how the pool, named PoolParty, looks and functions. It’s important to note that DBCaml is currently in an unstable version (v0.0.1), so there may be changes in the future. I will make an effort to update this article with any significant changes we make.</p>
<p>Now, let’s dive into the background of how the connection pool in DBCaml, or PoolParty, works. When I started the DBcaml project, I initially tried to write everything in a non-functional way because I had more experience with Rust and Go. However, when implementing the connection pool, I needed to rethink my coding approach to fit a more functional style. The first version of the pool didn’t meet our expectations because I had written the functionality to resemble an HTTP backend rather than a connection pool. The main difference between the two is that an HTTP backend can terminate a process after handling a socket connection, whereas a connection pool needs to persist the connection. Otherwise, we would create a new connection for every request to the database, which is not what we want.</p>
<p>So when I realized that I needed to rewrite the functionality for the pool, I reached out to <a href="https://twitter.com/leostera">Leostera</a>, the author of Riot, which is the core actor-model DBCaml uses to operate. I asked for help, and together we decided to study Elixir’s <a href="https://github.com/elixir-ecto/ecto">ecto</a> and how it’s connection pool was implemented. This gave me a clearer understanding.</p>
<blockquote>
<p>Before I continue discussing the pool, I want to give a big shoutout to Leostera for all the help. It has been an amazing experience, and you are extremely talented! You can follow Leandro on <a href="https://twitter.com/leostera">Twitter</a> or on <a href="https://www.twitch.tv/leostera">Twitch</a>.</p>
</blockquote>
<h2>Terms that will be good to understand</h2>
<p>This section will include descriptions of some terms used in this article to make it easier to follow along when discussing the pool.</p>
<h3>Supervisor</h3>
<p>A supervisor is something that controls processes, if a process dies do the supervisor need to know how to start it up again and it’s done via providing a initial state to the supervisor for the child it’s holding</p>
<h3>Supervisor child</h3>
<p>It’s something that the supervisor controls and keeps an eye on. The child is responsible for taking the action, while the supervisor ensures that it doesn’t fail. You can imagine it looking something like this:</p>
<p>

  <picture>
    <source media="(max-width: 640px)" srcset="/images/poolparty/supervisor_hubfcbdef86f1af6f8aeb9d993d5758d3d_63153_1280x0_resize_q100_h2_box_3.webp" type="image/webp" loading="lazy">
    <source media="(min-width: 640px)" srcset="/images/poolparty/supervisor_hubfcbdef86f1af6f8aeb9d993d5758d3d_63153_1920x0_resize_q100_h2_box_3.webp" type="image/webp" loading="lazy">

    <img src="https://priver.dev/images/poolparty/supervisor_hubfcbdef86f1af6f8aeb9d993d5758d3d_63153_1280x0_resize_q100_box_3.png" srcset="/images/poolparty/supervisor_hubfcbdef86f1af6f8aeb9d993d5758d3d_63153_1280x0_resize_q100_box_3.png 640w, /images/poolparty/supervisor_hubfcbdef86f1af6f8aeb9d993d5758d3d_63153_1920x0_resize_q100_box_3.png 800w" sizes="(max-width: 640px) 640px, 800px" loading="lazy" width="1280" alt="Building a Connnection Pool for DBCaml on top of riot">
  </picture></p>
<h3>Riot</h3>
<p>An actor-model multi-core scheduler for OCaml 5. When discussing actor-models, Erlang and the Beam are often mentioned. The Beam is a component of Erlang that controls all the processes created. The main difference between the Beam and Riot is that Riot only operates on a single machine, whereas the Beam can manage multiple hosts. Another distinction is the programming language they are built on. Despite these differences, Riot aims to emulate the capabilities of the Beam, as I understand it.</p>
<h3>Process</h3>
<p>In computing, a process is the instance of a computer program that is being executed by one or many threads.</p>
<h3>Holder</h3>
<p>A holder in this context is something that hold something and give it away when needed. Imagine that you are a holder and you hold a box and you don’t know what is inside the box but you hold something and when someone asks for it do you give it away. This  is the idea of a holder in the pool</p>
<h3>Pool manager</h3>
<p>A pool manager is responsible for controlling and managing all the holders. It can be likened to a team lead in a development scenario. Just as a team lead takes care of background tasks such as meetings and communication with sales, the pool manager is responsible for responding to messages and knowing the current state of the holder in this context.</p>
<p>The pool manager in DBCaml is called PoolParty.</p>
<h2>How it works</h2>
<p>

  <picture>
    <source media="(max-width: 640px)" srcset="/images/poolparty/poolmanager_huea25e8f36a4ca35e85b25efaa3bacc85_94090_1280x0_resize_q100_h2_box_3.webp" type="image/webp" loading="lazy">
    <source media="(min-width: 640px)" srcset="/images/poolparty/poolmanager_huea25e8f36a4ca35e85b25efaa3bacc85_94090_1920x0_resize_q100_h2_box_3.webp" type="image/webp" loading="lazy">

    <img src="https://priver.dev/images/poolparty/poolmanager_huea25e8f36a4ca35e85b25efaa3bacc85_94090_1280x0_resize_q100_box_3.png" srcset="/images/poolparty/poolmanager_huea25e8f36a4ca35e85b25efaa3bacc85_94090_1280x0_resize_q100_box_3.png 640w, /images/poolparty/poolmanager_huea25e8f36a4ca35e85b25efaa3bacc85_94090_1920x0_resize_q100_box_3.png 800w" sizes="(max-width: 640px) 640px, 800px" loading="lazy" width="1280" alt="Building a Connnection Pool for DBCaml on top of riot">
  </picture></p>
<p>What you are currently looking at is an explanation of how the pool works in real-time. Please note that this information may change, and if any changes occur, I will update the DBCaml documentation page at <a href="https://dbca.ml/">https://dbca.ml</a>.</p>
<p>Here are some clarifications:</p>
<ul>
<li>In this context, “App” refers to your program, the code you write, and the code that calls DBCaml.</li>
<li>“PoolParty” is the name for the connection pool in DBCaml. The idea is to separate PoolParty into its own library, allowing developers to use it independently of DBCaml. For example, it could be used as a Redis pool.</li>
<li>“Holder” refers to an item in the connection pool. A holder stores some data, but it shouldn’t be aware of what it is holding. It is up to the app to know the details (more about this will be explained later).</li>
</ul>
<h3>Creating the pool</h3>
<p>In the schema above, the app creates a new pool and receives a pool manager PID (Process ID). Once it obtains the PID, the app starts registering all of its holder items, which in the context of DBCaml refers to all the connections. An example of what this code could look like is provided below:</p>
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
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="c">(*Create the pool manager*)</span>
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="n">pool_id</span> <span class="o">=</span> <span class="nn">Poolparty</span><span class="p">.</span><span class="n">start_link</span> <span class="o">~</span><span class="n">pool_size</span><span class="o">:</span><span class="n">connections</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="c">(* Add new holders to the pool *)</span>
</span></span><span class="line"><span class="cl">  <span class="k">let</span> <span class="n">pids</span> <span class="o">=</span>
</span></span><span class="line"><span class="cl">    <span class="nn">List</span><span class="p">.</span><span class="n">init</span> <span class="n">connections</span> <span class="o">(</span><span class="k">fun</span> <span class="o">_</span> <span class="o">-&gt;</span>
</span></span><span class="line"><span class="cl">        <span class="k">let</span> <span class="n">pid</span> <span class="o">=</span>
</span></span><span class="line"><span class="cl">          <span class="n">spawn</span> <span class="o">(</span><span class="k">fun</span> <span class="bp">()</span> <span class="o">-&gt;</span>
</span></span><span class="line"><span class="cl">              <span class="k">match</span> <span class="nn">Driver</span><span class="p">.</span><span class="n">connect</span> <span class="n">driver</span> <span class="k">with</span>
</span></span><span class="line"><span class="cl">              <span class="o">|</span> <span class="nc">Ok</span> <span class="n">c</span> <span class="o">-&gt;</span> <span class="nn">Poolparty</span><span class="p">.</span><span class="n">add_item</span> <span class="n">pool_id</span> <span class="n">c</span>
</span></span><span class="line"><span class="cl">              <span class="o">|</span> <span class="nc">Error</span> <span class="o">_</span> <span class="o">-&gt;</span> <span class="n">error</span> <span class="o">(</span><span class="k">fun</span> <span class="n">f</span> <span class="o">-&gt;</span> <span class="n">f</span> <span class="s2">"failed to start driver"</span><span class="o">))</span>
</span></span><span class="line"><span class="cl">        <span class="k">in</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl">        <span class="n">pid</span><span class="o">)</span>
</span></span><span class="line"><span class="cl">  <span class="k">in</span>
</span></span><span class="line"><span class="cl"><span class="c">(*wait before all items are added to the pool before we continue *)</span>
</span></span><span class="line"><span class="cl">  <span class="n">wait_pids</span> <span class="n">pids</span><span class="o">;</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><p>Under the hood, PoolParty spins up a supervisor that controls the pool manager. The pool manager waits for messages to be added to its mailbox and handles each message as it receives them.</p>
<h3>Register items</h3>
<p>When an app registers a new item to the pool, PoolParty stores the item in a local in-memory table, which is currently implemented using <code>Hashtbl</code>. After the item has been added to the <code>Hashtbl</code>, the holder item starts a Supervisor and a process. It then retrieves the process ID (PID) and sends it to the connection manager in a <code>CheckIn</code> message. The purpose of the <code>CheckIn</code> message is to either add the PID to the memory or change the state in the memory table to <code>Ready</code>.</p>
<p>Once the <code>CheckIn</code> message has been handled, the pool manager is aware of the process that holds the item, but it does not perform any further actions with it.</p>
<h3>Request a holder item</h3>
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
<pre tabindex="0" class="chroma"><code class="language-ocaml" data-lang="ocaml"><span class="line"><span class="cl"><span class="k">let</span> <span class="n">item</span> <span class="o">=</span> <span class="nn">Poolparty</span><span class="p">.</span><span class="n">get_holder_item</span> <span class="n">pool_id</span> <span class="o">|&gt;</span> <span class="nn">Result</span><span class="p">.</span><span class="n">get_ok</span> <span class="k">in</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="k">let</span> <span class="n">result</span> <span class="o">=</span>
</span></span><span class="line"><span class="cl">  <span class="k">match</span> <span class="nn">Connection</span><span class="p">.</span><span class="n">execute</span> <span class="n">item</span><span class="o">.</span><span class="n">item</span> <span class="n">p</span> <span class="n">query</span> <span class="k">with</span>
</span></span><span class="line"><span class="cl">  <span class="o">|</span> <span class="nc">Ok</span> <span class="n">rows</span> <span class="o">-&gt;</span>
</span></span><span class="line"><span class="cl">    <span class="o">(</span><span class="k">match</span> <span class="n">rows</span> <span class="k">with</span>
</span></span><span class="line"><span class="cl">    <span class="o">|</span> <span class="bp">[]</span> <span class="o">-&gt;</span> <span class="nc">Error</span> <span class="nn">Res</span><span class="p">.</span><span class="nc">NoRows</span>
</span></span><span class="line"><span class="cl">    <span class="o">|</span> <span class="n">r</span> <span class="o">-&gt;</span> <span class="nc">Ok</span> <span class="o">(</span><span class="nn">List</span><span class="p">.</span><span class="n">hd</span> <span class="n">r</span><span class="o">))</span>
</span></span><span class="line"><span class="cl">  <span class="o">|</span> <span class="nc">Error</span> <span class="n">e</span> <span class="o">-&gt;</span> <span class="nc">Error</span> <span class="n">e</span>
</span></span><span class="line"><span class="cl"><span class="k">in</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl"><span class="nn">Poolparty</span><span class="p">.</span><span class="n">release</span> <span class="n">pool_id</span> <span class="n">item</span><span class="o">.</span><span class="n">holder_pid</span><span class="o">;</span>
</span></span></code></pre></td></tr></tbody></table>
</div>
</div><p>I believe it would be easier if I show the code and then explain what happens at each step.</p>
<p>When the app needs a holder’s item (in this context, a connection to the database), it uses a function called <code>get_holder_item</code>. This function sends a <code>CheckOut</code> message to the pool manager. Here’s what the pool manager does when it receives a <code>CheckOut</code> message:</p>
<p>It checks with the in-memory table if there is any holder with a <code>Ready</code> state. Depending on the answer, it does one of two things:</p>
<ol>
<li>If there is a ready holder:
<ol>
<li>It changes the holder’s state to <code>Busy</code>.</li>
<li>It reads the PID (Process Identifier) for the holder and sends a <code>CheckOut</code> message to the holder. The holder then needs to send whatever it’s holding to a requester PID, which is the process asking for a connection.</li>
<li>The app uses the received connection from the holder and then sends a <code>CheckIn</code> message with the holder’s PID to the pool manager, which changes the state of the holder to <code>Ready</code>.</li>
<li>The next requester can then re-use the same holder, as it is ready to be used.</li>
</ol>
</li>
<li>If there are no available holders:
<ol>
<li>The pool manager sends the same <code>CheckOut</code> message back to its own mailbox. This step is important to prevent the loop from getting stuck. There was a bug where, if the pool didn’t have any ready holders, it would get stuck because the pool manager was using a recursive function that repeatedly asked the storage if there were any available holders. However, this recursive function always returned false because the storage was never updated.</li>
</ol>
</li>
</ol>
<h2>The future</h2>
<p>Currently, PoolParty only works with DBCaml for two reasons:</p>
<ol>
<li>I haven’t found a good way yet to allow the holders to store a generic item. Currently, it only works with a local type for DBCaml.</li>
<li>PoolParty is still far from stable and lacks some needed functionality. However, I want to release a basic early version of DBCaml to allow developers to experiment with it. I hope that any issues that I am not aware of will be reported, so that I can fix them.</li>
<li>PoolParty will break out from DBCaml to be able to work with more than just database connections.</li>
</ol>
<h2>The end</h2>
<p>Thank you for reading this article. I hope you liked it and learned something. DBCaml is still in its early stages, and my current focus is on creating a basic v0.0.1 version for developers to test. The purpose of this article is to serve as a reference for the article I will release when I launch v0.0.1 of DBCaml, as this part of the project is quite significant.</p>
<p>If you are interested and would like to follow along with the development, I can recommend some links for you:</p>
<ul>
<li>Riot discord: <a href="https://discord.gg/CaykCHrAXN">https://discord.gg/CaykCHrAXN</a> The core discussions about the project are conducted on the Riot discord.</li>
<li>The Caravan discord: <a href="https://discord.gg/XpAWHYbFB3">https://discord.gg/XpAWHYbFB3</a></li>
<li>DBCaml GitHub: <a href="https://github.com/dbcaml/dbcaml">https://github.com/dbcaml/dbcaml</a></li>
</ul>
<p>I also post status updates on my Twitter account, which you can find at <a href="https://twitter.com/emil_priver">https://twitter.com/emil_priver</a> if you want to follow along.</p>

