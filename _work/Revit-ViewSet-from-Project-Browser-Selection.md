---
layout: article
title: "Revit Sheet Set (ViewSet) from Project Browser Selection"
date: 2018-12-14 02:53:00
cover: /work/img/Revit-ViewSet-from-Project-Browser-Selection-Cover.png
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
  - /work/ViewSet-from-Revit-Project-Browser-Selection/
---

Use Dynamo and Python to create a Revit Sheet Set (ViewSet) from a Project Browser selection.

<!--more-->

### Avoiding the ViewSet Dialog

Wouldn't it be nice to select a series of views or sheets and generate a new ViewSet (for publishing or printing) from that selection? With Dynamo we can select views from the Revit Project Browser and create a ViewSet- skipping the ViewSet dialog altogether.

### Dynamo Definition: ViewSet from Revit Project Browser Selection

Most of the work happens within Python Script nodes.

{% include image.html image="Revit-ViewSet-from-Project-Browser-Selection-001.png" %}

### Python Code: Output Selected Elements

The first Python node outputs Revit elements from your current selection.

{% highlight python %}
# Python Code for Dynamo
# Output Selected Elements
# Version 0.1
# Coded by Andrew King
# https://andrewk.ing
#
# 2018-11-12 Version 0.1
# Hello World

import clr
clr.AddReference('ProtoGeometry')
from Autodesk.DesignScript.Geometry import *

#Import RevitNodes
clr.AddReference("RevitNodes")
import Revit
clr.ImportExtensions(Revit.Elements)

# Import RevitAPI
clr.AddReference('RevitAPI')
import Autodesk
from Autodesk.Revit.DB import *

# Import DocumentManager and TransactionManager
clr.AddReference('RevitServices')
import RevitServices
from RevitServices.Persistence import DocumentManager
from RevitServices.Transactions import TransactionManager

output = []

# Get ElementIds of Revit selection.
selection = DocumentManager.Instance.CurrentUIApplication.ActiveUIDocument.Selection.GetElementIds()

# Convert ElementIds to Revit elements.
for s in selection:
  output.append(DocumentManager.Instance.CurrentDBDocument.GetElement(s).ToDSType(True))

#Assign your output to the OUT variable.
OUT = output
{% endhighlight %}

### Python Code: Create ViewSet from Input

The second Python node creates a ViewSet from views or sheets passed to the input.

{% highlight python %}
# Python Code for Dynamo
# Create ViewSet from Input
# Version 0.1
# Coded by Andrew King
# https://andrewk.ing
#
# 2018-11-12 Version 0.1
# Hello World

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
viewSetName = IN[0]
views = UnwrapElement(IN[1])

# Create empty ViewSet.
viewSet = ViewSet()

# Add incoming views/sheets to the ViewSet.
for v in views:
  viewSet.Insert(v)

# Activate the PrintManager.
printManager = DocumentManager.Instance.CurrentDBDocument.PrintManager
printManager.PrintRange = PrintRange.Select

# Save the ViewSet.
viewSheetSetting = printManager.ViewSheetSetting
viewSheetSetting.CurrentViewSheetSet.Views = viewSet
viewSheetSetting.SaveAs(viewSetName)
output = "ViewSet '" + viewSetName + "' created."

#Assign your output to the OUT variable.
OUT = output
{% endhighlight %}

### Result

Here is a sample ViewSet created using the code above.

{% include image.html image="Revit-ViewSet-from-Project-Browser-Selection-002.png" caption="Sample ViewSet" %}
