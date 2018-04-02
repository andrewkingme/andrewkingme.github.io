---
layout: article
title: "Synchronize Revit Key Schedules Using an Excel Master List"
date: 2018-02-14 07:07:00
cover: /work/img/Synchronize-Revit-Key-Schedules-Cover.png
collection: work
tags:
  - all
  - visualprogramming
  - code
  - excel
  - revit
  - dynamo
  - python
---

Use Dynamo, Python, and Excel to synchronize Revit key schedules across multiple Revit files.

<!--more-->

On a campus-style project with 25 Revit files and a regularly changing area program, our team needed a workflow to generate an overall area schedule efficiently. Updating a single key schedule with the latest revisions could take up to one hour. Multiply that by 25 files and our team was looking at 25 hours of tedious key schedule revisions- in addition to potential issues with inconsistency.

### Key Schedules and Multiple Revit Files

While key schedules are simple to setup and commonly used to assign multiple parameters in a single Revit file, maintaining synchronized key schedules across multiple Revit files is not fully supported OOTB.

* **Option 1: Use 'Insert Views From File' to transfer a key schedule.**

  This one-time, one-way transfer is acceptable if no changes to the key schedule are expected. Modifications to the key schedule require either [A] a full delete and re-import (This is especially destructive if key parameters were already assigned in the model, as those assignments are reset.) or [B] manual modification. (Time consuming and prone to inconsistency.)

* **Option 2: Use Dynamo and Python to synchronize changes from an Excel master.**

  This post will demonstrate a method to synchronize a regularly changing key schedule across multiple Revit files using Dynamo, Python, and an Excel master.

### Part 1: Add Matching Parameters to Excel and Revit

Before synchronizing, it is important to setup Excel and each Revit file with matching parameters. To allow for maximum flexibility, consider using generic naming by parameter type.

1. **Key Name** [Key]
2. **Department** [Text]
3. **Key_UniqueID** [Text]
4. **Key_Data_Text_01** [Text]
5. **Key_Data_Text_02** [Text]
6. **Key_Data_Text_03** [Text]
7. **Key_Data_Text_04** [Text]
8. **Key_Data_Text_05** [Text]
9. **Key_Data_Number_01** [Number]
10. **Key_Data_Number_02** [Number]
11. **Key_Data_Number_03** [Number]
12. **Key_Data_Number_04** [Number]
13. **Key_Data_Number_05** [Number]
14. **Key_Data_Area_01** [Area]
15. **Key_Data_Area_02** [Area]
16. **Key_Data_Area_03** [Area]
17. **Key_Data_Area_04** [Area]
18. **Key_Data_Area_05** [Area]

Notes on Parameter Setup:
1. Key schedules do not support shared parameters. Use project parameters.
2. 'Department' is a built-in Revit parameter that is compatible with room tags.
3. Importing a key schedule using 'Import Views From File' will also import associated project parameters, if they do not already exist. This is useful for initializing multiple files.

{% include image.html image="Synchronize-Revit-Key-Schedules-001.png" caption="Excel Master" %}
{% include image.html image="Synchronize-Revit-Key-Schedules-002.png" caption="Revit Key Schedule" %}

### Part 2: Generate Key_UniqueID

Synchronization requires a constant for comparison. The following Excel Macro (modified from [this post](https://www.thespreadsheetguru.com/the-code-vault/generate-string-random-characters-vba-codet)) will generate a random string of 16 alphanumeric characters, which will serve as a UID constant.

{% highlight vb %}
Sub RandomString()

Dim CharacterBank As Variant
Dim i As Long
Dim str As String

CharacterBank = Array("a", "b", "c", "d", "e", "f", "g", "h", "i", "j", "k", "l", "m", "n", "o", "p", "q", "r", "s", "t", "u", "v", "w", "x", "y", "z", "0", "1", "2", "3", "4", "5", "6", "7", "8", "9", "A", "B", "C", "D", "E", "F", "G", "H", "I", "J", "K", "L", "M", "N", "O", "P", "Q", "R", "S", "T", "U", "V", "W", "X", "Y", "Z")

For Each cell In selection

    For i = 1 To 16
        Randomize
        str = str & CharacterBank(Int((UBound(CharacterBank) - LBound(CharacterBank) + 1) * Rnd + LBound(CharacterBank)))
    Next i

    cell.Value = str
    str = vbNullString

Next cell

End Sub
{% endhighlight %}

Check for collisions using conditional formatting. Regenerate individual cells, if necessary.

{% include image.html image="Synchronize-Revit-Key-Schedules-003.png" %}

### Part 3: Synchronize Using Dynamo and Python

The first half of the Dynamo definition generates an ExcelList and a RevitList and collects their UID parameters for comparison.

{% include image.html image="Synchronize-Revit-Key-Schedules-004a.png" %}

Dynamo does not currently provide an OOTB node for Revit key schedules. The following Python code will return all individual keys in a key schedule. (Credit to Konrad Sobon for [this post](https://forums.autodesk.com/t5/revit-api-forum/key-schedule-revit-api/td-p/5449134), which explains how the Revit API treats keys in a Revit key schedule.)

{% highlight python %}
# Python Node for Dynamo
# Input: Key Schedule
# Output: Keys
# Version 0.1
# Coded by Andrew King
# http://andrewkingme.com
#
# 2017-06-20 Version 0.1
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
keySchedule = UnwrapElement(IN[0])

# Collect keys in key schedule.
keys = FilteredElementCollector(DocumentManager.Instance.CurrentDBDocument, keySchedule.Id).ToElements()

# Assign your output to the OUT variable.
OUT = keys
{% endhighlight %}

The second half of the Dynamo definition compares UID parameters and synchronizes changes.

{% include image.html image="Synchronize-Revit-Key-Schedules-004b.png" %}

The following Python code creates new keys and outputs a list to Dynamo for parameter assignment.

{% highlight python %}
# Python Node for Dynamo
# Input: List of Keys to Add
# Output: Keys Added
# Version 0.1
# Coded by Andrew King
# http://andrewkingme.com
#
# 2017-06-20 Version 0.1
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
keySchedule = UnwrapElement(IN[0])
keyCount = IN[1]
newKeys = []

# Start the Transaction
TransactionManager.Instance.EnsureInTransaction(DocumentManager.Instance.CurrentDBDocument)

for i in xrange(keyCount):

	# Create before list of keys in key schedule.
	scheduleBefore = FilteredElementCollector(DocumentManager.Instance.CurrentDBDocument, keySchedule.Id).ToElementIds()

	# Add new key to key schedule.
	schedule = keySchedule.GetTableData().GetSectionData(SectionType.Body)
	schedule.InsertRow(schedule.FirstRowNumber + 1)

	# Create after list of keys in key schedule.
	scheduleAfter = FilteredElementCollector(DocumentManager.Instance.CurrentDBDocument, keySchedule.Id).ToElementIds()

	# Identify new key by comparing before and after lists.
	newRow = list(set(scheduleAfter) - set(scheduleBefore))[0]
	newKeys.append(DocumentManager.Instance.CurrentDBDocument.GetElement(newRow))

# End the Transaction
TransactionManager.Instance.TransactionTaskDone()

# Assign your output to the OUT variable
OUT = newKeys
{% endhighlight %}

18 parameters are assigned using the `SynchronizeKeySchedule.SetParameters` custom node.

{% include image.html image="Synchronize-Revit-Key-Schedules-005.png" %}

### Complete Graph

Once setup, modifications are easy. Simply open the Dynamo definition and run it on each Revit file.

{% include image.html image="Synchronize-Revit-Key-Schedules-006.png" %}

### Result

Key schedule modifications that would have taken hours (25 hours in the case study above) now complete in just a few seconds per file, with consistently accurate results.

{% include image.html image="Synchronize-Revit-Key-Schedules-007.png" caption="Key Schedule in Revit" %}

{% include image.html image="Synchronize-Revit-Key-Schedules-008.png" %}

{% include image.html image="Synchronize-Revit-Key-Schedules-009.png" caption="Key Selection" %}
