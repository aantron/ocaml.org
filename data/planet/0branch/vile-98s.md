---
title: vile 9.8s
description:
url: http://blog.0branch.com/posts/2016-12-18-vile-9.8s.html
date: 2016-12-17T18:05:00-00:00
preview_image:
authors:
- Marc Simpson
source:
---

<div>
  <div class="span-22">
    <div class="span-12"><h1>vile 9.8s</h1></div>
    <div style="text-align: right" class="span-10 last">
      <a href="https://blog.0branch.com/index.html">#</a> December 17, 2016
    </div>
  </div>
  <hr>
  <div>
    <p>Tom Dickey has just released <a href="http://invisible-island.net/vile">vile</a> 9.8s; this release includes the following changes:</p>
<pre><code>20161217 (s)
  &gt; Brendan O'Dea:
  + add command-line parsing for "--" token, assumed by visudo in the
    1.8.12 - 1.8.16 changes (report by Wayne Cuddy).
  &gt; Tom Dickey:
  + recompute majormode order when "after", "before" or "qualifiers" is
    modified for a majormode.
  + add yamlmode (discussion with Steve Lembark)
  + modify DSTRING definition in lex-filter to handle continuation lines.
  + modify cfgmode to reduce false-matches with random ".cfg" files.
  + improve ps syntax filter
    + interpret %%BeginData / %%EndData keywords
    + interpret %%BeginPreview / %%EndPreview keywords
  + add ".mcrl2" as suffix for mcrlmode.
  + fixes from test-script: conf, hs, nr, rc, rcs, txt, xq, xml
  + improved regression test-script to check for places where the syntax
    filter might have mixed buffered- and unbuffered-calls in the same
    state, causing tokens in the markup to "move".
  + remove a statement from flt_putc in the standalone filters that
    converted a bare ^A to ^A?.
  + remove escaping from digraphs.rc, since change in 9.7zg made that
    both unnecessary and incorrect (reports by Marc Simpson, Brendan
    O'Dea).
  + improve tcl syntax filter
    + color backslash-escapes in double-quotes.
    + add rules to handle regexp and regsub regular expressions.  This
      does not yet handle -regexp switch cases.
    + add call to flt_bfr_error to flag unbalanced quotes here and in
      a few other filters.
    + modify newline patterns to allow for cr/lf endings in continuations
    + add special case for literals like "{\1}" and "{\\1}".
    + add special case for html entities such as "{&amp;#123;}" and "{&amp;foo;}"
  + improve sh syntax filter
    + allow quoted strings within '${' parameter, a detail that can
      happen with ksh brace groups (report by j.  van den hoff).
    + handle ksh's "ANSI C quotes", i.e., "$'xxx'" using single quotes
      after a dollar sign.
    + use the ksh ("-K") option for bashmode and zshmode syntax.
    + interpret "$name" within '${' parameter
    + don't warn for inline-here documents
    + handle special case where matching tag for a here-document is on
      the same line as a closing ")" in $(xxx) command.
    + highlight ksh's "[[", "((", "$((" bracketing like "{".
    + handle ksh's "((" and "$((" arithmetic expressions.
    + handle ksh's base#value numbers
  + improve perl syntax highlighter:
    + fix state used to guess where a pattern might occur, e.g., after
      an "if" keyword with no preceding operator to account for line
      breaks.
    + correct a check for illegal numbers, which flagged hexadecimal
      numbers containing "e".
    + distinguish special case of "format =" vs "format =&gt;".
    + allow pod to begin without a preceding blank line, but warn.
    + allow for case where pod mode is turned on/off with only one blank
      line between the directives.
    + check for simple patterns that may follow operators such as "map".
    + allow '$', '+' or '&amp;' as a quote or substitution delimiter
    + allow angle brackets for quotes after 'q', etc.
    + fix highlighting when square-brackets are used as delimiters in a
      perl substitution, e.g., s[foo[bar]xxx][yyy]
  + quiet some unnecessary compiler warnings with glibc &gt; 2.20 by adding
    _DEFAULT_SOURCE as needed.
  + improve version-comparison for "new" flex to allow for 2.6.0, and
    accept that for built-in filters.  Also modify filters/mk-2nd.awk
    to work with "new" flex ifdef's to ignore yywrap (Debian #832973).
  + correct long-name for filename-ic mode (report Marc Simpson).</code></pre>
<p>See <a href="http://invisible-island.net/vile/CHANGES.html#index-v9_8s">here</a> for further information.</p>
  </div>
</div>

<hr>

<div></div>

<noscript>Please enable JavaScript to view the <a href="http://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
<a href="http://disqus.com" class="dsq-brlink">comments powered by <span class="logo-disqus">Disqus</span></a>

