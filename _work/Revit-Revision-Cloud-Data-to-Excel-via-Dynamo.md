---
layout: article
title: "Revit Revision Cloud Data to Excel via Dynamo"
date: 2016-02-06 22:12:00
cover: /work/img/Revit-Revision-Cloud-Data-to-Excel-via-Dynamo-Cover.png
collection: work
tags:
  - all
  - visualprogramming
  - code
  - revit
  - dynamo
  - python
---

Scheduling revision clouds in Revit is limited. Revit can output the following built-in type parameters, but only these parameters and only within a title block family.

<!--more-->

* Revision Sequence
* Revision Number
* Revision Date
* Revision Description
* Issued to
* Issued by

I needed a way to manage all construction administration phase revisions within Revit revision clouds and then regularly export an itemized list to Excel for distribution to the contrator. At a minimum, the Excel document would need to list the following.

* Sheet Number
* Sheet Name
* Revision Number
* Revision Description
* Revision Date
* Comments

The solution: Dynamo.

Using the following graph and custom Python script, I was able to extract the necesary data from Revit and push it to Excel.

### First Half of Dynamo Graph

The first half of the graph pulls in all of the revision clouds within the active document and directly extracts the Revision Number, Revision Description, and Revision Date. Comments are pulled in as instance parameters so that every revision cloud can have a different note.

![Revit Revision Cloud Data to Excel via Dynamo](/{{ page.collection }}/img/Revit-Revision-Cloud-Data-to-Excel-via-Dynamo-001a.png)

### Custom Python Node

Revit does not automatically report the Sheet Number or Sheet Name to a revision cloud parameter, so a custom Python node was required to match up the elements and views.

{% highlight python %}
# Python Node for Dynamo
# Input: Revit Elements, Output: Matching Views
# Version 0.1
# Coded by Andrew King
# http://andrewkingme.com
# 
# 2016-02-06 Version 0.1
# Hello World

import clr
clr.AddReference('ProtoGeometry')
from Autodesk.DesignScript.Geometry import *

# Import DocumentManager and TransactionManager
clr.AddReference('RevitServices')
import RevitServices
from RevitServices.Persistence import DocumentManager
from RevitServices.Transactions import TransactionManager

# Import RevitAPI
clr.AddReference('RevitAPI')
import Autodesk
from Autodesk.Revit.DB import *

doc = DocumentManager.Instance.CurrentDBDocument

#The inputs to this node will be stored as a list in the IN variables.
dataEnteringNode = IN

elements = UnwrapElement(IN[0])

viewcollector = FilteredElementCollector(doc).OfClass(View)
viewtemplates = []
views = []

viewelements = []
matchingview = []

for view in viewcollector:
	if view.IsTemplate == True:
		viewtemplates.append(view)
	else:
		views.append(view)

for element in elements:
	for view in views:
		viewelements = FilteredElementCollector(doc, view.Id)
		for viewelement in viewelements:
			if viewelement.Id == element.Id:
				matchingview.append(view)

#Assign your output to the OUT variable.
OUT = matchingview
{% endhighlight %}

### Second Half of Dynamo Graph

The second half of the graph extracts the relevant parameter values, builds an itemized list, and sends it to Excel.

![Revit Revision Cloud Data to Excel via Dynamo](/{{ page.collection }}/img/Revit-Revision-Cloud-Data-to-Excel-via-Dynamo-001b.png)

### Complete Dynamo Graph

Here it is all together. Simple, efficient output of relevant revision cloud data.

![Revit Revision Cloud Data to Excel via Dynamo](/{{ page.collection }}/img/Revit-Revision-Cloud-Data-to-Excel-via-Dynamo-001.png)
