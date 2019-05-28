---
layout: post
title: Text to Map
subtitle: How to retrieve text and values and arrange them for coordinates
gh-repo: starlord32/starlord32.github.io
gh-badge: [star, fork, follow]
tags: [historian, python, codecademy, science, coordinates, qgis]
comments: true
---

```
from bs4 import BeautifulSoup
import re
import os
newPathToFolder = "---path to target folder---"
pathToFolder = "---path to source folder---"
listOfFiles = os.listdir(pathToFolder)
dicFreq = {}
placeNames = []
resultsCSV = []

# for loop that opens all files of a defined folder and stores it with BeautifulSoup in variable soup
for f in listOfFiles:
    soup = BeautifulSoup (open(pathToFolder+f, "r", encoding="utf8"), features="html.parser")
    # searches for list items of "date" and return maximum two elemens
    issue_date = soup.find_all("date", limit=2)[1] # only returns the second match
    issue_date = issue_date.get("value")
    # searches for all tags with "div3" and stores it in variable "articles"
    articles = soup.find_all("div3", type = True)

    # Searching for "placename" with a "key" attribute
    for a in articles:
        places = a.find_all("placename", key = True)

        # Pulling out the text of "placename" as well as the value of the attribute "key"
        for p in places:
            place = p.get_text()
            key = p["key"]

            placeList = "\t".join([place,key])

            placeNames.append(placeList)

# Creating a frequency counter
for i in placeNames:
    if i in dicFreq:
        dicFreq[i] += 1
    else:
        dicFreq[i]  = 1

# formatting the frequency counter
for key, value in dicFreq.items():
    if value > 1: # this will exclude items with frequency 1
        newVal = "%09d\t%s" % (value, key)
        # newVal will looks like: `000005486 TAB Richmond`
        resultsCSV.append(newVal)

        #for r in resultsCSV:
        #    cleanResults = re.sub("\t", "", r)

resultsCSV = sorted(resultsCSV, reverse=True)
print(len(resultsCSV)) # will print out the number of items in the list
resultsToSave = "\n".join(resultsCSV)

# creates a new file in a target folder with name + article counter + name + issue_date + txt file
newfile = newPathToFolder  + "places.csv"
newfile2 = newPathToFolder  + "results.csv"
# opens the newfile and writes each article into a sperate file
with open(newfile, "w", encoding="utf8") as f9:
    f9.write("\n".join(placeNames))
with open(newfile2, "w", encoding="utf8") as f8:
    f8.write("".join(resultsToSave))
```
