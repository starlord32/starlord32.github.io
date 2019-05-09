---
layout: post
title: Webscraping with wget
subtitle: Step by step description of a practical example
gh-repo: starlord32/starlord32.github.io
gh-badge: [star, fork, follow]
tags: [historian, git, codecademy, science, wget, webscraping]
comments: true
---

# Webscraping with wget

The following wget commands allow to download content:

wget link
wget -i file_with_links.txt
wget -i links.txt -P ./folderYouWantToSaveTo/ -nc

Practice Webscraping by downloading the following database collection:

[http://www.perseus.tufts.edu/hopper/collection?collection=Perseus:collection:RichTimes](http://www.perseus.tufts.edu/hopper/collection?collection=Perseus:collection:RichTimes)

Steps:
* The first step is to allocate the original link were the data is stored. In this case every article links to the source on the bottom of the page. The source is a .xml file.

* Identify the pattern of the link. In this case the main link looked like this:

[http://www.perseus.tufts.edu/hopper/dltext?doc=Perseus%3Atext%3A2006.05.0001](http://www.perseus.tufts.edu/hopper/dltext?doc=Perseus%3Atext%3A2006.05.0001) while the links to the various articles differ and look like this

[http://www.perseus.tufts.edu/hopper/text?doc=Perseus%3Atext%3A2006.05.0001%3Aarticle%3D17](http://www.perseus.tufts.edu/hopper/text?doc=Perseus%3Atext%3A2006.05.0001%3Aarticle%3D17). As you can see the main link follows the pattner of ...hopper/dltext... while the other link goes like ...hopper/text...

* Download the sourcecode of the wegpage with all the links to the various articles.

* Clean the document with regular expression to extracts the links

* Add or change the link containing ...dltext... instead of ...text... only.

* Save all links in a .txt file

* Open a Command Line Programm like Git Bash or other and enter the wget command as instructed above to download the files
