---
layout: post
title: Text to Map Two HW 11
subtitle: Using Python 3 to create edges from a dispatch to create a social network graph with Gephi
gh-repo: starlord32/starlord32.github.io
gh-badge: [star, fork, follow]
tags: [historian, python, codecademy, science, QGIS]
comments: true
---

# QGis Mapping solution

Since the Laditude and Longitude is a Decimal only I could not really map them correctly. I found a script that might convert Decimal to usable coordinates but I haven't been able to run it yet.

**Script:**

```
# Pre-logic
def latDD(x):
  D = int(x[1:3])
  M = int(x[3:5])
  S = float(x[5:])
  DD = D + float(M)/60 + float(S)/3600
  return DD

# Expression
latDD(!Latitude!)
```

**Map created with QGis for dispatch year 1860**

The complete script is described in blog post "Text to Map One HW 10" [Link](https://starlord32.github.io/2019-05-28-text-to-map-one/).

![QGis_Map_Dispatch:1860](/img/dispatchMap.png)
