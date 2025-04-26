---
layout: article
title: "Excel Sheet Index to Revit Placeholder Sheets"
date: 2017-01-06 03:40:00
cover: /work/img/Excel-Sheet-Index-to-Revit-Cover.png
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
---

Convert an Excel Sheet Index into Revit Placeholder Sheets using Dynamo and Python.

<!--more-->

### The Challenge

Architects typically include a combined sheet index for all disciplines at the beginning of a drawing set. While Revit can easily create a list of sheets included in a model (and can also grab sheets from linked models), not all offices use Revit and not every transmission arrives with a model.

Rather than typing incoming sheet information line-by-line into a Revit schedule, Dynamo and Python can help convert Excel data into Revit Placeholder Sheets. Manual entry that could have taken hours (on a large project with hundreds of incoming drawing sheets) can now be completed in seconds.

### Dynamo Definition: Excel Sheet Index to Revit Placeholder Sheets

This definition opens an Excel document, grabs values by column, replaces null values with "" to avoid a list mismatch, and feeds that data into a Python node.

{% include image.html image="Excel-Sheet-Index-to-Revit-001.png" %}

### Python Code: Excel Sheet Index to Revit Placeholder Sheets

Dynamo does not currently provide an OOTB node for Revit Placeholder Sheets so we will need to use Python code and the Revit API.

{% highlight python %}
# Python Code for Dynamo
# Excel Sheet Index to Revit Placeholder Sheets
# Version 0.3
# Coded by Andrew King
# https://andrewk.ing
#
# 2017-01-03 Version 0.1
# Hello World
#
# 2013-01-03 Version 0.2
# Added ability to modify additional sheet parameters (sheetDiscipline).
#
# 2013-01-03 Version 0.3
# Replace Excel null values with "" to avoid a list mismatch. Refactored Python code.

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
sheetNumbers = IN[0]
sheetNames = IN[1]
sheetDiscipline = IN[2]

sheets = []

# Start the Transaction
TransactionManager.Instance.EnsureInTransaction(DocumentManager.Instance.CurrentDBDocument)

# Create Placeholder Sheets and Assign Parameter Values
for s0, s1, s2 in zip(sheetNumbers, sheetNames, sheetDiscipline):
  newSheet = ViewSheet.CreatePlaceholder(DocumentManager.Instance.CurrentDBDocument)
  newSheet.SheetNumber = s0
  newSheet.Name = s1
  newSheet.LookupParameter('Discipline').Set(s2)
  sheets.append('Added Placeholder ' + newSheet.SheetNumber.ToString())

# End the Transaction
TransactionManager.Instance.TransactionTaskDone()

#Assign your output to the OUT variable
OUT = sheets
{% endhighlight %}

### Recommended Workflow

If you receive an updated sheet index, purge placeholder sheets in the model and re-import the new sheet index.

### Dynamo Definition: Purge Existing Placeholder Sheets

This definition grabs all sheets in the model and feeds that data into a Python node.

{% include image.html image="Excel-Sheet-Index-to-Revit-002.png" %}

### Python Code: Purge Existing Placeholder Sheets

Similar to above, we will use Python and the Revit API to delete all Placeholder Sheets in the model. Non-placeholder sheets (visibile in the project browser) will remain.

{% highlight python %}
# Python Code for Dynamo
# Purge Existing Placeholder Sheets
# Version 0.1
# Coded by Andrew King
# https://andrewk.ing
#
# 2017-01-03 Version 0.1
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
sheets = UnwrapElement(IN[0])

deleted = []

TransactionManager.Instance.EnsureInTransaction(DocumentManager.Instance.CurrentDBDocument)

for s in sheets:
  if s.IsPlaceholder:
    deleted.append('Deleted Placeholder ' + s.SheetNumber.ToString())
    DocumentManager.Instance.CurrentDBDocument.Delete(s.Id)

TransactionManager.Instance.TransactionTaskDone()

OUT = deleted
{% endhighlight %}

### Result

The code above can be modified to include additional parameters based on your project and office standards. I plan to include phase information in my next iteration.

{% include image.html image="Excel-Sheet-Index-to-Revit-003.png" caption="Excel Spreadsheet" %}
{% include image.html image="Excel-Sheet-Index-to-Revit-004.png" caption="Revit Schedule" %}
