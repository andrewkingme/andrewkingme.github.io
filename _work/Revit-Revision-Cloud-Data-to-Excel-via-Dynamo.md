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

Scheduling revision clouds in Revit is limited to the following built-in type parameters.

<!--more-->

* Revision Sequence
* Revision Number
* Revision Date
* Revision Description
* Issued to
* Issued by

At a minimum, our team needed an itemized list of revision clouds with the following information.

* Sheet Number
* Sheet Name
* Revision Number
* Revision Date
* Revision Description
* Mark (For example, RFI 001) [Instance Parameter]
* Comments [Instance Parameter]

Revit can't do this out of the box, but Dynamo can!

The following Dynamo definition and custom Python code extracts the relevant revision data from Revit and pushes it to Excel.

### Dynamo Definition: Part 1

Part 1 of the definition pulls in all of the revision clouds and sheets within the active document and runs the elements through a custom Python node.

{% include image.html image="Revit-Revision-Cloud-Data-to-Excel-via-Dynamo-001a.png" %}

### Custom Python Node

The custom Python code below filters through sheets and views-on-sheets looking for revision clouds. For each match the code collects the revision cloud element, associated sheet, and referencing view.

{% highlight python %}
# Python Node for Dynamo
# Input: Revision Clouds, Sheets
# Output: Matching Sheets, Matching Revision Clouds, Referencing Views
# Version 0.6
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
#
# 2016-02-25 Version 0.6
# Added category filter to improve FilteredElementCollector performace.

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

# Assign input to the IN variables.
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
        for sheetelement in FilteredElementCollector(DocumentManager.Instance.CurrentDBDocument, sheet.Id).OfCategory(BuiltInCategory.OST_RevisionClouds):
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
                        for viewelement in FilteredElementCollector(DocumentManager.Instance.CurrentDBDocument, viewid.Id).OfCategory(BuiltInCategory.OST_RevisionClouds):
                            for revisioncloud in revisioncloudinput:
                                if viewelement.Id == revisioncloud.Id:
                                    matchingsheets.append(sheet)
                                    matchingrevisionclouds.append(revisioncloud)
                                    referencingviews.append(viewid)
        else:
            for viewport in sheet.GetAllPlacedViews():
                for view in [DocumentManager.Instance.CurrentDBDocument.GetElement(viewport)]:
                    for viewelement in FilteredElementCollector(DocumentManager.Instance.CurrentDBDocument, view.Id).OfCategory(BuiltInCategory.OST_RevisionClouds):
                        for revisioncloud in revisioncloudinput:
                            if viewelement.Id == revisioncloud.Id:
                                matchingsheets.append(sheet)
                                matchingrevisionclouds.append(revisioncloud)
                                referencingviews.append(view)

# Assign output to the OUT variable.
OUT = matchingsheets, matchingrevisionclouds, referencingviews
{% endhighlight %}

Notes on the Python code:

* The code above was optimized to run as fast as possible using FilteredElementCollector().OfCategory(). However, looking through all sheets and views in a project is still computationally intensive. On a large benchmark project (1400+ revision clouds), the process currently takes around 4 minutes to complete.
* "Revisions on Sheets" and "Revisions in Views on Sheets" can be enabled separately on IN[2] and IN[3]. Consider having your team place all revisions on sheets and turn off "Revisions in Views on Sheets" to increase speed.
* The code above was designed to collect only visible revisions. If the revision cloud is not visible on the sheet (printed set), it will not be collected.
* The "Sheet Issues/Revisions" dialog in Revit is a great way to pre-filter your output by revision sequence (delta). Sequences set to "Cloud and Tag" or "Tag" will export. Sequences set to "None" are not visible and will not export.

### Dynamo Definition: Part 2

Part 2 of the definition extracts the relevant parameter values, builds an itemized list, sorts the list, and sends it to Excel.

{% include image.html image="Revit-Revision-Cloud-Data-to-Excel-via-Dynamo-001b.png" %}

### Complete Dynamo Definition

Here it is all together. Simple, efficient output of Revit revision cloud data.

{% include image.html image="Revit-Revision-Cloud-Data-to-Excel-via-Dynamo-001.png" %}

*Post updated to reflect Version 0.6 on 25 Feb 2016.*
