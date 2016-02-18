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

Part 1 of the graph pulls in all of the revision clouds and sheets within the active document and runs the elements through a custom Python node. RFI/ASI/PR/CCD Numbers and Comments are pulled in as instance parameters so that every revision cloud can have a unique identifier and a unique comment.

![Revit Revision Cloud Data to Excel via Dynamo](/{{ page.collection }}/img/Revit-Revision-Cloud-Data-to-Excel-via-Dynamo-001a.png)

### Custom Python Node

Revit does not automatically report the Sheet Number or Sheet Name to a revision cloud parameter, so a custom Python node was required to match up the elements and views.

{% highlight python %}
# Python Node for Dynamo
# Input: Revision Clouds, Sheets
# Output: Matching Sheets, Matching Revision Clouds, Referencing Views
# Version 0.5
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
#
# 2016-02-16 Version 0.5
# Restructured code to reduce number of cycles.
# Eliminated the need for a separate legend/dependency path.
# Boolean selector for Revisions on Sheets/Revisions in Views on Sheets.
# Revit 2014 compatibility.

import clr
clr.AddReference('ProtoGeometry')
from Autodesk.DesignScript.Geometry import *

# Import RevitAPI
clr.AddReference('RevitAPI')
import Autodesk
from Autodesk.Revit.DB import *

# Import DocumentManager and TransactionManager
clr.AddReference('RevitServices')
import RevitServices
from RevitServices.Persistence import DocumentManager
from RevitServices.Transactions import TransactionManager

#Assign input to the IN variables.
revisioncloudinput = UnwrapElement(IN[0])
sheetinput = UnwrapElement(IN[1])
revisionsonsheets = IN[2]
revisionsinviewsonsheets = IN[3]

matchingsheets = []
matchingrevisionclouds = []
referencingviews = []

# Look for revision clouds on sheets.
if revisionsonsheets == True:
	for sheet in sheetinput:
		for sheetelement in FilteredElementCollector(DocumentManager.Instance.CurrentDBDocument, sheet.Id):
			for revisioncloud in revisioncloudinput:
				if sheetelement.Id == revisioncloud.Id:
					matchingsheets.append(sheet)
					matchingrevisionclouds.append(revisioncloud)
					referencingviews.append(sheet)

# Look for revision clouds in views on sheets.
if revisionsinviewsonsheets == True:
	for sheet in sheetinput:
		if DocumentManager.Instance.CurrentUIApplication.Application.VersionName == "Autodesk Revit 2014":
			for viewport in sheet.GetAllViewports():
				for view in [DocumentManager.Instance.CurrentDBDocument.GetElement(viewport)]:
					for viewid in [DocumentManager.Instance.CurrentDBDocument.GetElement(view.ViewId)]:
						for viewelement in FilteredElementCollector(DocumentManager.Instance.CurrentDBDocument, viewid.Id):
							for revisioncloud in revisioncloudinput:
								if viewelement.Id == revisioncloud.Id:
									matchingsheets.append(sheet)
									matchingrevisionclouds.append(revisioncloud)
									referencingviews.append(viewid)
		else:
			for viewport in sheet.GetAllPlacedViews():
				for view in [DocumentManager.Instance.CurrentDBDocument.GetElement(viewport)]:
					for viewelement in FilteredElementCollector(DocumentManager.Instance.CurrentDBDocument, view.Id):
						for revisioncloud in revisioncloudinput:
							if viewelement.Id == revisioncloud.Id:
								matchingsheets.append(sheet)
								matchingrevisionclouds.append(revisioncloud)
								referencingviews.append(view)

# Assign output to the OUT variable.
OUT = matchingsheets, matchingrevisionclouds, referencingviews
{% endhighlight %}

### Dynamo Graph: Part 2

Part 2 of the graph extracts the relevant parameter values, builds an itemized list, and sends it to Excel.

![Revit Revision Cloud Data to Excel via Dynamo](/{{ page.collection }}/img/Revit-Revision-Cloud-Data-to-Excel-via-Dynamo-001b.png)

### Complete Dynamo Graph

Here it is all together. Simple, efficient output of relevant revision cloud data.

![Revit Revision Cloud Data to Excel via Dynamo](/{{ page.collection }}/img/Revit-Revision-Cloud-Data-to-Excel-via-Dynamo-001.png)

*Post updated to reflect Version 0.5 on 16 Feb 2016.*