
An extended brain dump to supplement [this rOpenSci unconf proposal](https://github.com/ropensci/unconf18/issues/42).

# Strategy 1: Storing the Input Data

One of the big challenge of round-trip R markdown is that the compiled document does not neccessarily contain
all of the information in the original.  This is different than, say, Jupyter notebooks, where the input and output of all codeblocks is stored, whether or not they are displayed.  So one needs to store input information (chunks, input files, etc) somewhere in the output file and have a way to re-extract them.

If the output file is editable, like Google Docs/Word, you also need some way to deal with the fact that the outputs of code could potentially be modieifed, reconcile somehow if someone makes changes in a code output or input during editing, or protect anything that is an code from being edited at all.  This requires that one somehow tag and differentiate code output and pure text in the output.  This becomes challenging with things in-line text generated from Rmd (e.g., `r round(some_var)`), because it is easy to edit that text and lose its content as well as associated metadata.  

[This 2014 paper](https://www.stat.auckland.ac.nz/~paul/Reports/invert/invert.html) on "invertible reproducible documents" lays out one of the main structural approaches to this problem and proposes an _.Rhtml_ (not Rmd) format that would be editable as HTML in both input and output form. The extra information is stored in HTML comments, but collaborators would still have to edit raw HTML, and the workflow is fragile.

[R Notebooks](https://rmarkdown.rstudio.com/r_notebooks.html) are HTML R markdown outputs with some features to enable collaboration.  Notably they store chunk inputs and outputs, as well as storing a copy of the whole original `.Rmd` inside the HTML document (as a base64-encoded string).  This allows one to re-extract the original Rmd when passing around the HTML, but the HTML is not editable.

When we created [**gdoc**](https://github.com/ropenscilabs/gdoc) at the 2016 unconf, we poked around in the internal JSON structure of google docs to see if there were places where we could tuck uncompiled chunks and other metadata.  There are object attribute fields in there, but the Google Doc internal structure is not well documented and could change without warning.  There similar annotation options in MS Word, and the office XML structure is actually more open and stable.

# Strategy 2: Having collaborators provide input text only

[ArchieML](http://archieml.org/) was created by the NYT of having non-coder writers write text in Google Docs that would be fed into live web pages (like interactive features).   The idea is that one can write body text (as well as captions, etc.), in the google doc with very easy, flexible, and robust markup, and then import that information into a compiled document and mix it with code-generated outputs like graphs.  

I created [**rchie**](https://github.com/ropensci/rchie) to parse this format for the purpose of pulling text written in Google Docs into R and using it to build R markdown docs where the text came from Google Docs.  This has the drawback of collaborators not seeing the live final doc when they edit, but they can edit text.   The NYT pipeline is JavaScript based, so presumably they could see the changes to the compiled document lives.  I had briefly experimented with a Shiny app that would read text from the Google Doc and display the compiled Rmd.  It worked but of course was somewhat clunky.

# Strategy 3: Reduce pain points in the edit-revise-recompile cycle

Neither of the strategies above has worked out for me in my current position, and even Google Docs are usually a stretch too far when dealing with many of our collaborators across sectors.  So I've focused on reducing the challenges associated with writing in Rmd, compiling, sending documents for comment (usually in Word with tracked changes), and re-incorporating those changes.

I've found the biggest of pain points in this is in formatting - the standard Rmd MS Word outputs leave a lot to be desired in terms of what the final output looks like, so there is usually a lot of formatting that occurs in MS Word, which needs to be re-done on each iteration.  Further, many collaborators prefer to edit what looks like a final product.  

One way I tackle this is more. elaborate R Markdown MS Word output types.  To get what looks like a finished output, one usually needs to both create an MS Word template for R Markdown/Pandoc to use, as well as do some post-processing to get features that office users expect but aren't part of Pandoc's markup.  David Gohel's series of packages ([**ReporteRs**](https://github.com/davidgohel/ReporteRs), [**officer**](https://github.com/davidgohel/officer), and now [**worded**](https://github.com/davidgohel/worded)) are all very helpful for this, and allow you to compile something that looks like a final published document for collaborators.  (Some examples of the outputs of my organizations' custom templates [here](http://livescience.ecohealthalliance.org/)).  This is, of course, far from having a round-trip workflow, but it makes people more appreciative of the programmatic approach, and does reduce the pain of how much needs to be updated in each round of edits.
