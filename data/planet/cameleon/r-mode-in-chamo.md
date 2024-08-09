---
title: R mode in Chamo
description: A R mode is now available for Chamo, as a separate snippet . By now,
  this mode has only one key binding, to call the "r_eval" command. This ...
url: https://ocameleon.blogspot.com/2008/05/r-mode-in-chamo.html
date: 2008-05-06T14:37:00-00:00
preview_image: https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiMugrmc4JExnAtJS1THLf1YkRKDPq8cDDsiENFmUoVjQAnr1dA8vIzm5ze4daZC65Vc9vDp-PHhHSG9b4nx7G3zGc7vyHY8GFN1yCrgWDfM8Rfovw2p4QTa1AA0OX47d-JKxJTUF37mSE/w1200-h630-p-k-no-nu/r_mode.png
authors:
- Zoggy
source:
---

A R mode is now available for Chamo, as a <a href="http://home.gna.org/cameleon/snippets.en.html">separate snippet</a>. By now, this mode has only one key binding, to call the "r_eval" command. This command sends the contents of the active view to a background R process. The output of this process is displayed in the "outputs" window. Other commands can easily be defined the same way to interact with other programs (see <a href="http://home.gna.org/cameleon/snippets/code_R_mode.html">the details</a>).<br><br>If some text is selected in the view, then only this text is sent to the R process. Closing the R output tab in the "outputs" window closes the R process. When the "r_eval" command is executed again, a new process is created. If the command is executed and the R process is still running, then it is used and no other process is launched.<br><br><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiMugrmc4JExnAtJS1THLf1YkRKDPq8cDDsiENFmUoVjQAnr1dA8vIzm5ze4daZC65Vc9vDp-PHhHSG9b4nx7G3zGc7vyHY8GFN1yCrgWDfM8Rfovw2p4QTa1AA0OX47d-JKxJTUF37mSE/s1600-h/r_mode.png" onblur="try {parent.deselectBloggerImageGracefully();} catch(e) {}"><img src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiMugrmc4JExnAtJS1THLf1YkRKDPq8cDDsiENFmUoVjQAnr1dA8vIzm5ze4daZC65Vc9vDp-PHhHSG9b4nx7G3zGc7vyHY8GFN1yCrgWDfM8Rfovw2p4QTa1AA0OX47d-JKxJTUF37mSE/s320/r_mode.png" style="margin: 0px auto 10px; display: block; text-align: center; cursor: pointer;" alt="" border="0"></a>
