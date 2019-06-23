---
layout: post
title: Text to Map HW 11
subtitle: How to retrieve text and values and arrange them with coordinates
gh-repo: starlord32/starlord32.github.io
gh-badge: [star, fork, follow]
tags: [historian, python, codecademy, science, coordinates, qgis]
comments: true
---

**Example with BeautifulSoup**

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
            # finding the placename
            place = p.get_text()
            # finding the key but only if it is a digit
            keyTmp = [d for d in p['key'] if d.isdigit()]
            # joining keys
            key = "".join(keyTmp)
            # creating a tab seperated list
            placeList = "\t".join([place,key])
            # appending the tab sepreated list to one list
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

resultsCSV = sorted(resultsCSV, reverse=True)
print(len(resultsCSV)) # will print out the number of items in the list
resultsToSave = "\n".join(resultsCSV)

# creates a new file in a target folder with name + article counter + name + issue_date + txt file
newfile2 = newPathToFolder  + "results.csv"
# opens the newfile and writes each article into a sperate file
with open(newfile2, "w", encoding="utf8") as f8:
    f8.write("".join(resultsToSave))
```

**Result with BeautifulSoup**

![results1](/img/results_hw10.png)

**Example with Regex**

```
import re, os

source = "source files"
target = "target folder to save files"


lof = os.listdir(source)
counter = 0 # general counter to keep track of the progress

def generate(filter):

    topCountDictionary = {}

    print(filter)
    counter = 0
    for f in lof:
        if f.startswith("dltext"): # fileName test
            with open(source + f, "r", encoding="utf8") as f1:
                text = f1.read()

                text = text.replace("&amp;", "&")

                # try to find the date
                date = re.search(r'<date value="([\d-]+)"', text).group(1)
                #print(date)

                if date.startswith(filter):
                    for tg in re.findall(r"(tgn,\d+)", text):
                        tgn = tg.split(",")[1]

                        if tgn in topCountDictionary:
                            topCountDictionary[tgn] += 1
                        else:
                            topCountDictionary[tgn]  = 1

                        #input(topCountDictionary)

    top_TSV = []

    for k,v in topCountDictionary.items():
        val = "%09d\t%s" % (v, k)
        #input(val)
        top_TSV.append(val)

    # saving
    header = "freq\ttgn\n"
    with open("dispatch_toponyms_%s.tsv" % filter, "w", encoding="utf8") as f9:
        f9.write(header+"\n".join(top_TSV))
    #print(counter)

generate("186")
generate("1861")
generate("1862")
generate("1863")
generate("1864")
generate("1865")
```

**Results for Regex**

![Results2](/img/results_hw10.2.png)

**Regex Example Continued**

```
import re, os

source = "tgn downloaded XML files"
target = "target folder for saving"

def generateTGNdata(source):

    lof = os.listdir(source)

    tgnList = []
    tgnListNA = []
    count = 0

    for f in lof:
        if f.startswith("TGN"): # fileName test
            print(f)
            with open(source+f, "r", encoding="utf8") as f1:
                data = f1.read()

                data = re.split("</Subject>", data)

                for d in data:
                    d = re.sub("\n +", "", d)
                    #print(d)

                    if "Subject_ID" in d:
                        # SUBJECT ID
                        placeID = re.search(r"Subject_ID=\"(\d+)\"", d).group(1)
                        #print(placeID)

                        # NAME OF THE PLACE
                        placeName = re.search(r"<Term_Text>([^<]+)</Term_Text>", d).group(1)
                        #print(placeName)

                        # COORDINATES
                        if "<Coordinates>" in d:
                            latGr = re.search(r"<Latitude>(.*)</Latitude>", d).group(1)
                            lat = re.search(r"<Decimal>(.*)</Decimal>", latGr).group(1)

                            lonGr = re.search(r"<Longitude>(.*)</Longitude>", d).group(1)
                            lon = re.search(r"<Decimal>(.*)</Decimal>", lonGr).group(1)
                            #print(lat)
                            #print(lon)
                        else:
                            lat = "NA"
                            lon = "NA"

                        tgnList.append("\t".join([placeID, placeName, lat, lon]))
                        #input(tgnList)

                        if lat == "NA":
                            print("\t"+ "; ".join([placeID, placeName, lat, lon]))
                            tgnListNA.append("\t".join([placeID, placeName, lat, lon]))

    # saving
    header = "tgnID\tplacename\tlat\tlon\n"

    with open("tgn_data_light.tsv", "w", encoding="utf8") as f9a:
         f9a.write(header+"\n".join(tgnList))

    with open("tgn_data_light_NA.tsv", "w", encoding="utf8") as f9b:
         f9b.write(header+"\n".join(tgnListNA))

    print("TGN has %d items" % len(tgnList))

generateTGNdata(source)
```

**Results to Regex Continued**

![Results3](/img/results_hw10.3.png)

**Generating a dictionary for the TGN data and matching them with the dispatch files**

```
import re, os

def loadTGN(tgnTSV):
    with open(tgnTSV, "r", encoding="utf8") as f1:
        data = f1.read().split("\n")

        dic = {}

        for d in data:
            d = d.split("\t")

            dic[d[0]] = d

    return(dic)

def match(freqFile, dicToMatch):
    with open(freqFile, "r", encoding="utf8") as f1:
        data = f1.read().split("\n")

        dataNew = []
        dataNewNA = []
        count = 0

        for d in data[1:]:
            tgnID = d.split("\t")[1]
            freq  = d.split("\t")[0]

            if tgnID in dicToMatch:
                val = "\t".join(dicToMatch[tgnID])
                val  = val + "\t" + freq

                if "\tNA\t" in val:
                    dataNewNA.append(val)
                else:
                    dataNew.append(val)
            else:
                print("%s (%d) not in TGN!" % (tgnID, int(freq)))
                count += 1

    header = "tgnID\tplacename\tlat\tlon\tfreq\n"

    with open("coord_"+freqFile, "w", encoding="utf8") as f9a:
        f9a.write(header + "\n".join(dataNew))

    with open("coord_NA_"+freqFile, "w", encoding="utf8") as f9b:
        f9b.write(header + "\n".join(dataNewNA))

    print("%d item have not been matched..." % count)

dictionary = loadTGN("tgn_data_light.tsv")

match("dispatch_toponyms_186.tsv", dictionary)
```

**Matching results**

![Results4](/img/results_hw10.4.png)
