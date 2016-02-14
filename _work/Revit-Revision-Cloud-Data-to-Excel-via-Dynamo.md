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
* RFI/ASI/PR/CCD Number
* Comments

The solution: Dynamo.

Using the following graph and custom Python code, I was able to extract the necesary data from Revit and push it to Excel.

### Dynamo Graph: Part 1

Part 1 of the graph pulls in all of the revision clouds within the active document and runs the elements through a custom Python node. RFI/ASI/PR/CCD Numbers and Comments are pulled in as instance parameters so that every revision cloud can have a unique identifier and a unique comment.

![Revit Revision Cloud Data to Excel via Dynamo](/{{ page.collection }}/img/Revit-Revision-Cloud-Data-to-Excel-via-Dynamo-001a.png)

### Custom Python Node

Revit does not automatically report the Sheet Number or Sheet Name to a revision cloud parameter, so a custom Python node was required to match up the elements and views.

{% highlight python %}
# Python Node for Dynamo
# Input: Revision Cloud
# Output: Matching View, Matching Revision Cloud, Referencing View
# Version 0.4
# Coded by Andrew King
# http://andrewkingme.com
# 
# 2016-02-06 Version 0.1
# Hello World
#
# 2016-02-08 Version 0.2
# Output matchingrevisionclouds from Python node to avoid hidden element mismatch.
#
# 2016-02-14 Version 0.4
# Expanded Python code to output every instance of a revision cloud in every instance of a legend.

import clr
clr.AddReference('ProtoGeometry')
from Autodesk.DesignScript.Geometry import *

clr.AddReference('RevitServices')
import RevitServices
from RevitServices.Persistence import DocumentManager

clr.AddReference('RevitAPI')
import Autodesk
from Autodesk.Revit.DB import *

revisioncloudinput = UnwrapElement(IN[0])

viewtemplates = []
views = []

matchingviews = []
matchingrevisionclouds = []
referencingviews = []

for view in FilteredElementCollector(DocumentManager.Instance.CurrentDBDocument).OfClass(View):
	if view.IsTemplate == True:
		viewtemplates.append(view)
	else:
		views.append(view)

for revisioncloud in revisioncloudinput:
	for view in views:
		viewelements = FilteredElementCollector(DocumentManager.Instance.CurrentDBDocument, view.Id)
		for viewelement in viewelements:
			if viewelement.Id == revisioncloud.Id:
				if view.ViewType == ViewType.Legend:
					
					for sheet in FilteredElementCollector(DocumentManager.Instance.CurrentDBDocument).OfClass(ViewSheet):
						viewports = sheet.GetAllPlacedViews()
						for viewport in viewports:
							legendview = DocumentManager.Instance.CurrentDBDocument.GetElement(viewport)
							if view.Id == legendview.Id:
								matchingviews.append(sheet)
								matchingrevisionclouds.append(revisioncloud)
								referencingviews.append(view)

				else:
					matchingviews.append(view)
					matchingrevisionclouds.append(revisioncloud)
					referencingviews.append(view)

#Assign your output to the OUT variable.
OUT = matchingviews, matchingrevisionclouds, referencingviews
{% endhighlight %}

### Dynamo Graph: Part 2

Part 2 of the graph filters the output list to remove duplicate dependent view items. For example, if a revision cloud was placed in a dependent view of Floor Plan Level 1, Revit would find the revision in both the primary and dependent view. This portion of the graph will make sure that only revision clouds that show on a sheet will export to Excel.

![Revit Revision Cloud Data to Excel via Dynamo](/{{ page.collection }}/img/Revit-Revision-Cloud-Data-to-Excel-via-Dynamo-001b.png)

### Dynamo Graph: Part 3

Part 3 of the graph extracts the relevant parameter values, builds an itemized list, and sends it to Excel.

![Revit Revision Cloud Data to Excel via Dynamo](/{{ page.collection }}/img/Revit-Revision-Cloud-Data-to-Excel-via-Dynamo-001c.png)

### Complete Dynamo Graph

Here it is all together. Simple, efficient output of relevant revision cloud data.

![Revit Revision Cloud Data to Excel via Dynamo](/{{ page.collection }}/img/Revit-Revision-Cloud-Data-to-Excel-via-Dynamo-001.png)

*Post updated to reflect Version 0.4 on 14 Feb 2016.*