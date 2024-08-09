---
title: XML for Resumes
description: "I hate writing resumes because there are no good tools for the job.
  I would prefer to use an open source word processor, such as Open Office, but companies
  often ask for MS Word format, which OO do\u2026"
url: https://mcclurmc.wordpress.com/2010/02/09/xml-for-resumes/
date: 2010-02-10T00:01:53-00:00
preview_image: https://s0.wp.com/i/blank.jpg
authors:
- Mike McClurg
source:
---

<p>I hate writing resumes because there are no good tools for the job. I would prefer to use an open source word processor, such as Open Office, but companies often ask for MS Word format, which OO doesn’t do particularly well. I’ve thought about LaTeX, but unless you’re doing a lot of math that’s kind of overkill. Besides, I want to be able to keep the content of my resume separate from the formatting of the resume. I also want good revision control, and I want to be able to select content for my resume based on the particular job I’m applying for (such as leaving off my DoD clearance from non-DoD job applications).</p>
<p>All of this sounds like a job for XML – though I’m sure some would find this more overkill than LaTeX. But with a simple XML language, or perhaps even XHTML with class and span tags, I think I can make this happen. There are already two open “standards” for the job, <a href="http://xmlresume.sourceforge.net/">XML Resume</a> and <a href="http://microformats.org/wiki/hresume">hResume</a> microformats, but I use the word “standards” very loosely. The XML Resume project on SourceForge hasn’t been updated since 2005, and I’m not convinced that the microformats solution completely solves the problem of keeping content and formatting orthogonal – though it gives us the great benefit of having a fully indexable online resume.</p>
<p>My idea is to use either XML Resume or hResume (or some bastard of the two) to write the content of the resume, but then use an OO plugin to import the resume language into OO, where we can use it’s great formatting tools. If I can get this rolling, we could also write a corresponding plugin for MS Word, so that I can send off resumes to Microsoft shops.</p>
<p>Now that I think about it, this is basically the model-view-controller pattern. The model is the XML representation of the resume, and all the different data points that you may want to include at some point in a real resume. The word processor is the view, where you can make your resume look as pretty as you want. The controller is the plugin that allows you to have your pretty resume template backed by data in the XML file – and, if the plugin does what I want it to do, display certain bullet points while hiding others, change the order of items, etc.</p>
<p>I’d really like to get this project started, but every time I think about it it’s because I need to work on my resume. And so I find myself in the programmer’s endless dilemma: do I build a tool to solve the problem for me, or do I just solve the problem? My resume is looking pretty good, so I hope I’ll find some time to work on this plugin.</p>

