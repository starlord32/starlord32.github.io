---
layout: post
title: Text to Map HW 10
subtitle: How to retrieve text and values of two databases. Finding and merging coordinates.
gh-repo: starlord32/starlord32.github.io
gh-badge: [star, fork, follow]
tags: [historian, python, codecademy, science, coordinates, qgis]
comments: true
---

*Python Functions for retrieving data from two databases and matching values to create a Map*

**Code example one to retrieve Placenames and TGN number with BeautifulSoup**

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

**Code example two to retrieve Placenames and TGN number with Regex**

```
import re, os

source = "dispatach source files folder path"
target = "target path to save the new created files"


lof = os.listdir(source)                                                        # listing the source directory
counter = 0                                                                     # general counter to keep track of the progress

def generate(filter):                                                           # defining a function with one argument

    topCountDictionary = {}                                                     # empty dictionary

    print(filter)
    counter = 0
    for f in lof:                                                               # for loop to search in all files of a selected folder
        if f.startswith("dltext"): # fileName test                              # if condition to search in files starting with the string "dltext" only
            with open(source + f, "r", encoding="utf8") as f1:                  # if the above condition is true it opens the files and reads them
                text = f1.read()                                                # the loaded file is stored in the variable "text"

                text = text.replace("&amp;", "&")                               # search and replace function that removes "amp;"

                # try to find the date
                date = re.search(r'<date value="([\d-]+)"', text).group(1)      # search with regular expression to find the date of each article
                #print(date)

                if date.startswith(filter):                                     # if condition that allows to search for specific dates in the articles
                    for tg in re.findall(r"(tgn,\d+)", text):                   # for loop to find all tgn numbers in the articles of the previous specified date
                        tgn = tg.split(",")[1]                                  # creates a new variable and split the previous found string by comma and uses the second item which is the tgn number

                        if tgn in topCountDictionary:                           # if condition to create a counter if a new tgn number was found
                            topCountDictionary[tgn] += 1
                        else:                                                   # else the counter remains 1
                            topCountDictionary[tgn]  = 1

                        #input(topCountDictionary)

    top_TSV = []

    for k,v in topCountDictionary.items():                                      # for loop to search for item of the topCountDictionary dictionary
        val = "%09d\t%s" % (v, k)                                               # defines the format of the frequency counter
        #input(val)
        top_TSV.append(val)                                                     # appends the frequency to a list

    # saving
    header = "freq\ttgn\n"                                                      # creates a header for the final tsv table
    with open("dispatch_toponyms_%s.tsv" % filter, "w", encoding="utf8") as f9: # writing the data into a tsv file
        f9.write(header+"\n".join(top_TSV))
    #print(counter)

#generate("186")

generate("1861")                                                                # example to use defined function with a specifc date
generate("1862")
generate("1863")
generate("1864")
generate("1865")
```

**Results for Regex**

![Results2](/img/results_hw10.2.png)

**Code example three to simplify the TGN database with regular expression**

```
import re, os

source = "tgn xml files source path"
target = "target folder to save the new file"

def generateTGNdata(source):                                                    # creating a function with one argument

    lof = os.listdir(source)                                                    # loading the source directory

    tgnList = []                                                                # creating a new list
    tgnListNA = []                                                              # creating a new list
    count = 0                                                                   # creating a counter

    for f in lof:                                                               # for loop to search a files of the source directory
        if f.startswith("TGN"):                                                 # fileName test
            print(f)
            with open(source+f, "r", encoding="utf8") as f1:                    # opens all files that are true in the previous fileName test
                data = f1.read()                                                # stores the read data in a new variable

                data = re.split("</Subject>", data)                             # splits the data by closing tag

                for d in data:                                                  # for loop to search each subject previously splitted
                    d = re.sub("\n +", "", d)                                   # subetting with a regular expression to remove/replace all new lines
                    #print(d)

                    if "Subject_ID" in d:                                       # if condition to search if a Subject_ID tag is available
                        # SUBJECT ID
                        placeID = re.search(r"Subject_ID=\"(\d+)\"", d).group(1) # regular expression search for the Subject_ID value defined as group one
                        #print(placeID)

                        # NAME OF THE PLACE
                        placeName = re.search(r"<Term_Text>([^<]+)</Term_Text>", d).group(1) # regular expression search for place names defined in the xml as Term_Text tag
                        #print(placeName)

                        # COORDINATES
                        if "<Coordinates>" in d:                                # if condition to search if a Coordinates tag is available
                            latGr = re.search(r"<Latitude>(.*)</Latitude>", d).group(1) # regular expression to search for the value of the Latitude tag as group one
                            lat = re.search(r"<Decimal>(.*)</Decimal>", latGr).group(1) # regular expression to search for a the value of the Decimal tag but only with the previous Latitude result

                            lonGr = re.search(r"<Longitude>(.*)</Longitude>", d).group(1) # regular expression to search for the value of the Longitude tag as group one
                            lon = re.search(r"<Decimal>(.*)</Decimal>", lonGr).group(1) # regular expression to search for a the value of the Decimal tag but only with the previous Longitude result
                            #print(lat)
                            #print(lon)
                        else:                                                   # else lat and long is NA
                            lat = "NA"
                            lon = "NA"

                        tgnList.append("\t".join([placeID, placeName, lat, lon])) # appending the final strings to a list joined as tab seperated
                        #input(tgnList)

                        if lat == "NA":                                         # all NA values are stored in a seperate list
                            print("\t"+ "; ".join([placeID, placeName, lat, lon]))
                            tgnListNA.append("\t".join([placeID, placeName, lat, lon]))

    # saving
    header = "tgnID\tplacename\tlat\tlon\n"                                     # creating a header for the tsv tables

    with open("tgn_data_light.tsv", "w", encoding="utf8") as f9a:               # saving the list as tsv original
         f9a.write(header+"\n".join(tgnList))

    with open("tgn_data_light_NA.tsv", "w", encoding="utf8") as f9b:            # saving the NA list as tsv
         f9b.write(header+"\n".join(tgnListNA))

    print("TGN has %d items" % len(tgnList))                                    # print function to see how many items are found

generateTGNdata(source)                                                         # example of how to use the new function
```

**Results to Regex Continued**

![Results3](/img/results_hw10.3.png)

**Code example four to generate a dictionary of TGN numbers and coordinates and match it with TGN numbers and placenames**

```
import re, os

def loadTGN(tgnTSV):                                                            # creates a new function with one argument
    with open(tgnTSV, "r", encoding="utf8") as f1:                              # opens a file defined in the function argument
        data = f1.read().split("\n")                                            # reads the file and stores the data splitted by new lines

        dic = {}                                                                # creates an empty dictionary

        for d in data:                                                          # for loop to search items of the loaded data
            d = d.split("\t")                                                   # splits the data by tab

            dic[d[0]] = d                                                       # saves the data in the dictionary starting at the first entry

    return(dic)                                                                 # to return the dictionary

def match(freqFile, dicToMatch):                                                # new function with two arguments
    with open(freqFile, "r", encoding="utf8") as f1:                            # opens the first file later defined as the first argument
        data = f1.read().split("\n")                                            # loads the data as variable and splits it by new lines

        dataNew = []                                                            # creating a new empty list
        dataNewNA = []                                                          # creating a new empty list
        count = 0                                                               # creating a counter

        for d in data[1:]:                                                      # for loop to search each item of the loaded data
            tgnID = d.split("\t")[1]                                            # splits each item by tab for the second colum of the loaded data
            freq  = d.split("\t")[0]                                            # splits each item by tab for the first column of the loaded data

            if tgnID in dicToMatch:                                             # if condition to match the tgnID of the loaded data (first argument) with the tgnID of a new file (second argument)
                val = "\t".join(dicToMatch[tgnID])                              # found matches are stored in a new variable that joins them tab seperated
                val  = val + "\t" + freq                                        # reformatting the variable to add the frequency to the tgnID

                if "\tNA\t" in val:                                             # if condition to searchfor NA values in the new variable
                    dataNewNA.append(val)                                       # if found it appends it to the NA list
                else:
                    dataNew.append(val)                                         # else it appends it to the regular list
            else:
                print("%s (%d) not in TGN!" % (tgnID, int(freq)))               # prints information in the console about not found data in the tgn file
                count += 1

    header = "tgnID\tplacename\tlat\tlon\tfreq\n"                               # creates a header for the tsv file

    with open("coord_"+freqFile, "w", encoding="utf8") as f9a:                  # saves the tsv file with found results
        f9a.write(header + "\n".join(dataNew))

    with open("coord_NA_"+freqFile, "w", encoding="utf8") as f9b:               # saves the tsv file of the not found NA data
        f9b.write(header + "\n".join(dataNewNA))

    print("%d item have not been matched..." % count)                           # prints to the console the amount of files not found

dictionary = loadTGN("tgn_data_light.tsv")                                      # loads the function for the tgn data file and stores it as dictionary in a new variable

match("dispatch_toponyms_1861.tsv", dictionary)                                 # example of how to use the match function
match("dispatch_toponyms_1862.tsv", dictionary)                                 # the first argument is the filename (must be stored in same folder path)
match("dispatch_toponyms_1863.tsv", dictionary)                                 # the second argument uses the previous loaded dictionary to match items with
match("dispatch_toponyms_1864.tsv", dictionary)
match("dispatch_toponyms_1865.tsv", dictionary)
```

**Matching results**

![Results4](/img/results_hw10.4.png)
