---
layout: post
title: Data Structuring
subtitle: How to create a single JSON file
gh-repo: starlord32/starlord32.github.io
gh-badge: [star, fork, follow]
tags: [historian, python, codecademy, science, json]
comments: true
---

***The following script structures data into a single JSON file***
**This Script is related to the webscraping and Text Markup blog post**

Starting from line 72 the final variable "var" is created which structures the data in a single .json file.
On line 25 a variable called "allFiles" as an empty list is created to store all the ouputs from the loop via the "var" variable.

```
import re, os, json

source = "~/Documents/Dh_Tools/lesson7/5th/"
target = "~/Documents/Dh_Tools/lesson9/combined_all/"

lof = os.listdir(source)
counter = 0 # general counter to keep track of the progress
allFiles = []

for f in lof:
    if f.startswith("dltext"): # fileName test
        with open(source + f, "r", encoding="utf8") as f1:
            text = f1.read()

            # try to find the date
            date = re.search(r'<date value="([\d-]+)"', text).group(1)

            # splitting the issue into articles/items
            split = re.split("<div3 ", text)

            c = 0 # item counter
            for s in split[1:]:
                c += 1
                s = "<div3 " + s # a step to restore the integrity of items
                #input(s)

                # try to find a unitType
                try:
                    unitType = re.search(r'type="([^\"]+)"', s).group(1)
                except:
                    unitType = "noType"
                    print(s)

                # try to find a header
                try:
                    header = re.search(r'<head>(.*)</head>', s).group(1)
                    header = re.sub("<[^<]+>", "", header)
                except:
                    header = "NO HEADER"
                    print("\nNo header found!\n")

                text = re.sub("<[^<]+>", "", s)
                #text = re.sub(" +\n|\n +", "\n", text)
                #text = re.sub("\n+", ";;; ", text)

                # generating necessary bits
                fName = date+"_"+unitType+"_"+str(c)

                itemID = date+"_"+unitType+"_"+str(c)
                dateVar   = date
                unitType = unitType
                header = header
                text = text

                # creating a json variable
                var = {
                "ID" : itemID,
                "date" : dateVar,
                "type" : unitType,
                "header" : header,
                "text" : text
                }
                #input(var)
                allFiles.append(var)

        # count processed issues and print progress counter at every 100
        counter += 1
        if counter % 100 == 0:
            print(counter)

#print(allFiles)
with open(target+"summary9.json", "w", encoding="utf8") as f9:
    f9.write(str(allFiles))
```
