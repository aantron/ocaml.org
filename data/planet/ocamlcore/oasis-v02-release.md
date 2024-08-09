---
title: OASIS v0.2 release
description:
url: http://www.ocamlcore.com/wp/2010/10/oasis-v02-release/
date: 2010-10-21T22:56:20-00:00
preview_image:
authors:
- OCamlCore.com
source:
---

<p><img src="http://www.ocamlcore.com/wp/wp-content/uploads/logo.png" align="left" width="100" height="94" alt="OASIS logo"><a href="http://oasis.forge.ocamlcore.org">OASIS</a> is a tool to integrate a configure, build and install system in your OCaml  project. It helps to create standard entry points in your build system and  allows external tools to analyse your project easily. It is the building brick of OASIS-DB, a CPAN for OCaml.</p>
<p>This release takes almost 6 months to complete. This is a very long time, but the changelog matches the length. It closes 25 bugs and grants 31 feature requests.</p>
<p>A big part of the project was to match the need of the project <a href="http://oasis.forge.ocamlcore.org/oasis-db.html">OASIS-DB</a>. The most visible part of this refactoring is the fully documented<a href="http://oasis.forge.ocamlcore.org/api-oasis/"> API</a> and the new library oasis. This library is used to parse <em>_oasis</em> files and to manipulate its data. We use it in OASIS-DB but one can also use it to extract data and process it.</p>
<p>The other visible change for the user, is the command-line interface. We decided to use a lowercase name for the executable and a Subversion-like subcommand system. So the former command <strong>"OASIS -setup"</strong> is now <strong>"oasis setup"</strong>. We manage now existing subcommand as plugins. We implemented also a set of new subcommands.</p>
<p>Among these subcommands, one is of a particular interest: "query". It parses an <em>_oasis</em> file and can return the value of any fields. For example: <strong>"oasis query Version"</strong> will return the version contained in the <em>_oasis</em> file located in the current working directory. It can be very useful to build shell scripts using OASIS. To have a complete listing of available fields, run <strong>"oasis query ListFields" </strong>on a particular files.</p>
<p>Most of the feature requests and bugs closed by this release are the result of the OCaml Hacking day demonstration and further work by other people. I would like to thank everyone that has submitted a bug or a feature request. The best way to make the OASIS project a great tool is to try it and tell us what is wrong with it — or what should be improved.</p>
<p>This release, as the one before, has been tested on Windows and Linux. The installers are now built on Centos 5.5, to allow wider compatibility.</p>
<p style="text-align: center;"><a href="https://forge.ocamlcore.org/frs/?group_id=54&amp;release_id=343">Download OASIS v0.2.0</a></p>
<p>&nbsp;Full changelog:</p>
<ul>
<li>Split the project into 3 libraries and one executable:
<ul>
<li>oasis: the core library</li>
<li>oasis.base: the runtime library</li>
<li>oasis.builtin-plugins: various plugins (ocamlbuild, internal, none, custom)</li>
<li>the executable ‘oasis’ in lowercase which was ‘OASIS’ before</li>
</ul>
</li>
<li>Publish .mli and improve ocamldoc generated documentation</li>
<li>oasis library:
<ul>
<li>Ignore plugins even when parsing field</li>
<li>Allow to redirect messages through a function and use a context to avoid global variables. This is an OASIS-DB website requirement, but we fallback to a global variable in oasis.base</li>
<li>Use the same policy as Debian for version comparison (copied from dpkg)</li>
<li>Add MIT, CeCILL licenses and make unknown license less fatal</li>
<li>Allow https, ftp, mailto, svn, svn+ssh for URL</li>
<li>Replace Str by Pcre</li>
<li>Don’t modify package data structure through plugins, we just issue warnings and error when something is missing. This is compensated by a better ‘quickstart’ that can automatically complete required fields (e.g. it adds ‘ocamlbuild’ as a BuildDependency for ‘ocamlbuild’ plugin).</li>
<li>Set default for test type to ‘custom’ plugin</li>
<li>Use a more simple lexer for _oasis</li>
<li>Warn if the use of ‘\t’ to indent lines is inconsistent in an _oasis file (e.g. mix of ‘ ‘ and ‘\t’)</li>
<li>Allow to use ‘flag’ as environment variables in _oasis (e.g. flag test can be used as Command: echo $test</li>
</ul>
</li>
<li>oasis.base library:
<ul>
<li>Exit with an error code when tests fail</li>
<li>Don’t account skipped test</li>
<li>Delegate the "setup-dev" actions to the executable ‘oasis’ rather than embedding it into setup.ml</li>
<li>Add a ‘-version’ to setup.ml to know what version has generated the file</li>
<li>Add a ‘-all’ target that does "-configure", "-build", "-doc" and "-test" in one run</li>
<li>Add a ‘-reinstall’ target that ‘uninstall’ and ‘install’</li>
<li>Use the right command to delete file on Windows</li>
</ul>
</li>
<li>executable ‘oasis’:
<ul>
<li>Use a subcommand scheme, like subversion. For example, it replaces the former "OASIS -setup" by "oasis setup". Each subcommand can be a small plugin</li>
<li>Add a "query" subcommand to extract data of _oasis from command line</li>
<li>Add a "setup-clean" subcommand that removes generated files and helps cleaning OASIS_START/STOP section of their content</li>
<li>Add a "check" subcommend that checks _oasis files&nbsp;</li>
<li>&nbsp;Greatly improve the "quickstart" subcommand:
<ul>
<li>Take into account plugins in quickstart</li>
<li>Allow to have multiple choices for field Plugins in quickstart mode</li>
<li>Don’t display help text at each question</li>
<li>Allow to create a doc</li>
<li>Allow to run a pager, editor or "oasis setup-dev" at the end<strong><br>
            </strong></li>
<li>Don’t accept ‘?’ as an answer in quickstart</li>
<li>Add examples and all available licenses in the help of License field</li>
<li>Keep generated files when ‘oasis setup-dev’ is called</li>
</ul>
</li>
</ul>
</li>
<li>Plugin "ocamlbuild":
<ul>
<li>Handle "Path: ." in generated _tags correctly</li>
<li>Quick fix to handle .h files directly in CSources field</li>
<li>Include .mli in _tags</li>
<li>Pass -cclib and -dllpath options to ocamlmklib</li>
<li>Don’t pass -dlllib and -dllpath options to ocamlopt</li>
</ul>
</li>
<li>Plugin "internal":
<ul>
<li>Create parent directories when installing with InternalInstall</li>
<li>Don’t install data when section is not built</li>
</ul>
</li>
<li>Plugin "META":
<ul>
<li>Add an exists_if field to generated META file</li>
</ul>
</li>
</ul>
<p>&nbsp;</p>

