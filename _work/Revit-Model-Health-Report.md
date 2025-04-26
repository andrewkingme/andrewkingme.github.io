---
layout: article
title: "Revit Model Health Report with .GetWarnings()"
date: 2019-03-06 03:09:00
cover: /work/img/Revit-Model-Health-Report-Cover.png
collection: work
funding: false
tags:
  - all
  - visualprogramming
  - code
  - revit
  - dynamo
  - python
redirect_from:
  - /work/Model-Health-Report/
---

With the introduction of `.GetWarnings()` in the Revit 2018 API, I retooled my model health scripts from scratch. For those out there with a similar mindset, here is an approachable starting point that can be easily expanded to include your desired metrics.

<!--more-->

### Dynamo Definition

The following definition is an expandable framework currently outputting three data-points.

{% include image.html image="Revit-Model-Health-Report-001.png" %}

### Python Code: Get Warnings

Please note that the following Python code utilizing `.GetWarnings()` requires Revit 2018 or later.

{% highlight python %}
# Python Node for Dynamo
# Get Warnings
# Version 0.1
# Coded by Andrew King
# http://andrewk.ing
#
# 2019-03-05 Version 0.1
# Hello World

import clr

# Import RevitAPI
clr.AddReference('RevitAPI')
import Autodesk

# Import DocumentManager
clr.AddReference('RevitServices')
import RevitServices
from RevitServices.Persistence import DocumentManager

warnings = DocumentManager.Instance.CurrentDBDocument.GetWarnings()
descriptions = []
elements = []

for warning in warnings:
  descriptions.append(Autodesk.Revit.DB.FailureMessage.GetDescriptionText(warning))
  elements.append(Autodesk.Revit.DB.FailureMessage.GetFailingElements(warning))

#Assign your output to the OUT variable
OUT = warnings, descriptions, elements
{% endhighlight %}

### Dynamo Definition: Collect Warnings and Imports

The first half of the graph collects warnings and imports.

{% include image.html image="Revit-Model-Health-Report-001a.png" %}

### Dynamo Definition: Write Summary to Revit Starting View

The second half writes a summary of the report to a text note on the Revit Starting View.

{% include image.html image="Revit-Model-Health-Report-001b.png" %}

### Result

Expanding on this early framework- I'm thinking the end result will include a few dozen data-points, presented on the Revit Starting View, with updating available via Dynamo Player.

{% include image.html image="Revit-Model-Health-Report-002.png" %}
