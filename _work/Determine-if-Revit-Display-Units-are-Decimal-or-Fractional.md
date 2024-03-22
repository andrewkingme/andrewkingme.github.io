---
layout: article
title: "Determine if Revit Display Units are Decimal or Fractional"
date: 2024-03-22 14:41:00
cover: /work/img/Determine-if-Revit-Display-Units-are-Decimal-or-Fractional-Cover.png
collection: work
funding: false
tags:
  - all
  - visualprogramming
  - code
  - revit
  - dynamo
  - python
---

Python code that can be used in conjunction with `Symbol.StringifyDecimal` and `Symbol.StringifyFraction` to create Dynamo graphs that work with various unit display types in Revit.

<!--more-->

### Python Code

Note: This code is designed to work with Revit 2021 or later. It utilizes Forge schema API calls.

{% highlight python %}
import clr

clr.AddReference('RevitAPI')
import Autodesk
from Autodesk.Revit.DB import *

clr.AddReference('RevitServices')
import RevitServices
from RevitServices.Persistence import DocumentManager

doc = DocumentManager.Instance.CurrentDBDocument
docUnitType = doc.GetUnits().GetFormatOptions(SpecTypeId.Length).GetUnitTypeId()

if docUnitType.TypeId == UnitTypeId.FeetFractionalInches.TypeId:
    result = "Fractional"
elif docUnitType.TypeId == UnitTypeId.FractionalInches.TypeId:
    result = "Fractional"
else:
    result = "Decimal"

OUT = result
{% endhighlight %}

### Result

{% include image.html image="Determine-if-Revit-Display-Units-are-Decimal-or-Fractional-001.png" %}
