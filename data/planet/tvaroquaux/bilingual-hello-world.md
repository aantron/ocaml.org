---
title: Bilingual "hello world"
description: 'Here is a fun (and slightly useless) hack: #cd "." (* echo "Hello world"
  <<"OCAMLCODE_END" *) let  ()  =  print_endline "Bonjour le monde" (...'
url: https://till-varoquaux.blogspot.com/2007/11/bilingual-hello-world.html
date: 2007-11-28T04:53:00-00:00
preview_image:
authors:
- Till
source:
---

<p>Here is a fun (and slightly useless) hack:</p><div style="background:#e6e6e6;border:1px solid #a0a0a0;"><tt>#cd <span style="color: #FF0000">"."</span><span style="font-style: italic"><span style="color: #9A1900">(*</span></span><br><span style="font-style: italic"><span style="color: #9A1900">echo "Hello world"</span></span><br><span style="font-style: italic"><span style="color: #9A1900">&lt;&lt;"OCAMLCODE_END"</span></span><br><span style="font-style: italic"><span style="color: #9A1900">*)</span></span><br><span style="font-weight: bold"><span style="color: #0000FF">let</span></span> <span style="color: #990000">()</span> <span style="color: #990000">=</span> print_endline <span style="color: #FF0000">"Bonjour le monde"</span><br><span style="font-style: italic"><span style="color: #9A1900">(*</span></span><br><span style="font-style: italic"><span style="color: #9A1900">OCAMLCODE_END</span></span><br><span style="font-style: italic"><span style="color: #9A1900">#*)</span></span></tt></div><p>This program is both a shell program and an ocaml program. If you run it using <em>sh</em> it will print "Hello world" but if you run it in <em>ocaml</em> it will output "Bonjour le monde" (Ocaml is a french programming language after all).</p><p>There is actually a small interest in this hack: suppose you want to run an ocaml script but need to make a couple of checks before running it (for instance checking whether <a href="http://www.ocaml-programming.de/programming/findlib.html" class="externalLink">findlib</a> is installed or the interpreter is recent enough) you can now bundle it as a shell executable that calls itself again after having done the checks as an ocaml program. </p>
