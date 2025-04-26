---
layout: article
title: "Revit Revision Cloud Data to Excel"
date: 2016-02-06 22:12:00
cover: /work/img/Revit-Revision-Cloud-Data-to-Excel-Cover.png
collection: work
funding: false
tags:
  - all
  - visualprogramming
  - code
  - excel
  - revit
  - dynamo
  - python
redirect_from:
  - /work/Revit-Revision-Cloud-Data-to-Excel-via-Dynamo/
---

Extract revision cloud data from Revit using Dynamo and Python.

<!--more-->

### OOTB Revit Revision Schedule Limitations

Revit can schedule the following revision parameters in a title block family. However, instance parameters (Mark and Comments), sheet numbers, and sheet names are *not supported out-of-the-box*.

* Revision Sequence
* Revision Number
* Revision Date
* Revision Description
* Issued to
* Issued by

### Expanding on OOTB Functionality

Dynamo and Python can expand on OOTB functionality to generate an itemized list of revisions including previously unavailable instance parameters, sheet numbers, and sheet names. The definition and code in this post will extract the following.

* Sheet Number
* Sheet Name
* Revision Number
* Revision Date
* Revision Description
* Mark (For example, RFI 001) [Instance Parameter]
* Comments [Instance Parameter]

### Dynamo Definition: Part 1

Part 1 of the definition collects all revision clouds and sheets within the active document and sends that information to a Python node.

{% include image.html image="Revit-Revision-Cloud-Data-to-Excel-001a.png" %}

### Python Code

The Python code filters through sheets and views-on-sheets, looking for revision clouds. It collects the revision cloud element, associated sheet, and referencing view.

{% highlight python %}
# Python Code for Dynamo
# Input: Revision Clouds, Sheets
# Output: Matching Sheets, Matching Revision Clouds, Referencing Views
# Version 0.6
# Coded by Andrew King
# https://andrewk.ing
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

1. Element collection was optimized to run as fast as possible using `.OfCategory()`. On a large benchmark project (1400+ revision clouds), the process currently takes around 4 minutes to complete.
2. *Revisions on Sheets* and *Revisions in Views on Sheets* can be enabled separately on `IN[2]` and `IN[3]`. Consider having your team place all revisions on sheets and turn off *Revisions in Views on Sheets* to increase speed.
3. The code above was designed to collect only visible revisions. If the revision cloud is not visible on the sheet (printed set), it will not be collected.
4. The *Sheet Issues/Revisions* dialog in Revit can pre-filter your output by revision sequence (delta). Sequences set to *Cloud and Tag* or *Tag* will export. Sequences set to *None* are not visible and will not export.

### Dynamo Definition: Part 2

Part 2 of the definition extracts the relevant parameter values, builds an itemized list, sorts the list, and writes it to Excel.

{% include image.html image="Revit-Revision-Cloud-Data-to-Excel-001b.png" %}

### Complete Graph

Simple, efficient output of Revit revision cloud data.

{% include image.html image="Revit-Revision-Cloud-Data-to-Excel-001.png" %}

### Result

The Dynamo graph writes directly to the *Revision Data* tab in Excel. Using a Pivot Table, the raw data is converted to a user friendly format.

{% include image.html image="Revit-Revision-Cloud-Data-to-Excel-002.png" caption="Excel Output" %}

*Post updated to reflect Version 0.6 on 25 Feb 2016.*

*Update: 05 Mar 2020*

I'm often asked if it would be possible to utilize `.OwnerViewId` to pull sheet information directly from revision clouds. While this appears to work at the surface level, we will run into issues with *Legends* and *Dependent Views*. If your project utilizes revision clouds in views, the longer element collection route described above still makes the most sense. `.OwnerViewId` is, however, a viable solution if you are certain your project only has revisions on sheets.

On large projects with heavy geometry, it is possible to run into long sheet generation times. In those instances, temporarily unloading *User-Created* worksets will significantly reduce run time. (Revision clouds are automatically placed on a view workset separate from user-created geometry.)
