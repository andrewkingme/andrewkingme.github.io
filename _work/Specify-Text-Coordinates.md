---
layout: article
title: "Specify Coordinates for a Text Element in Revit"
date: 2024-07-10 13:23:00
cover: /work/img/Specify-Text-Coordinates-Cover.png
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

While working on company standards like title blocks and tags, I wanted more precise control over text alignment and positioning. The following Dynamo definition can be used to set a specific XYZ and width for a Text Element in Revit.

<!--more-->

### Custom Dialog

{% include image.html image="Specify-Text-Coordinates-003.png" %}

Built-in Dynamo nodes like `Utilities.ParseExpressionByUnit` and `Utilities.ConvertByUnits` allow this script to function natively in both Metric and Imperial units.

### Dynamo Definition

{% include image.html image="Specify-Text-Coordinates-001.png" %}

### Python Script: Obtain Coordinates of Selected Text Element

Operating on the selection as a Text Element instead of Text Note allows us to modify Labels as well.

{% highlight python %}
import clr

clr.AddReference("RevitNodes")
import Revit
clr.ImportExtensions(Revit.GeometryConversion)

# The inputs to this node will be stored as a list in the IN variables.
textElement = UnwrapElement(IN[0])

# Place your code below this line
textElementCoordinates = textElement.Coord.ToPoint()

# Assign your output to the OUT variable.
OUT = textElementCoordinates
{% endhighlight %}

### Python Script: Obtain Width and Height of Selected Text Element

Height is a read-only parameter that automatically adjusts with the Text Size and number of lines. It will be presented in the Input Dialog as an unmodifiable value.

{% highlight python %}
import clr
clr.AddReference("RevitNodes")
import Revit.GeometryConversion

# The inputs to this node will be stored as a list in the IN variables.
textElement = UnwrapElement(IN[0])

# Place your code below this line
textElementW = textElement.Width
textElementH = textElement.Height

# Assign your output to the OUT variable.
OUT = textElementW, textElementH
{% endhighlight %}

### Python Script: Obtain Vertical Alignment and Horizontal Alignment of Selected Text Element

The XYZ placement of a Text Element sits at the intersection of its vertical and horizontal alignment. For example, Top-Left. While alignment information is readily available within the Revit Properties Panel, I wanted to also present it in the Input Dialog for convenience.

{% highlight python %}
import clr

clr.AddReference('RevitAPI')
import Autodesk
from Autodesk.Revit.DB import *

# The inputs to this node will be stored as a list in the IN variables.
textElement = UnwrapElement(IN[0])

# Place your code below this line

if textElement.VerticalAlignment == VerticalTextAlignment.Top:
    verticalAlignment = "Top"
elif textElement.VerticalAlignment == VerticalTextAlignment.Middle:
    verticalAlignment = "Middle"
elif textElement.VerticalAlignment == VerticalTextAlignment.Bottom:
    verticalAlignment = "Bottom"
else:
    verticalAlignment = "Null"

# Assign your output to the OUT variable.
OUT = verticalAlignment
{% endhighlight %}

{% highlight python %}
import clr

clr.AddReference('RevitAPI')
import Autodesk
from Autodesk.Revit.DB import *

# The inputs to this node will be stored as a list in the IN variables.
textElement = UnwrapElement(IN[0])

# Place your code below this line

if textElement.HorizontalAlignment == HorizontalTextAlignment.Left:
    horizontalAlignment = "Left"
elif textElement.HorizontalAlignment == HorizontalTextAlignment.Center:
    horizontalAlignment = "Center"
elif textElement.HorizontalAlignment == HorizontalTextAlignment.Right:
    horizontalAlignment = "Right"
else:
    horizontalAlignment = "Null"

# Assign your output to the OUT variable.
OUT = horizontalAlignment
{% endhighlight %}

### Python Script: Input Dialog

Using Windows Forms, we can present the existing Text Element positioning and ask for modifications mid-run.

{% highlight python %}
import clr 
import System.Windows

textElementX = IN[0]
textElementY = IN[1]
textElementZ = IN[2]
textElementW = IN[3]
textElementH = IN[4]
textElementVerticalAlignment = IN[5]
textElementHorizontalAlignment = IN[6]

# Initialize Windows Form Elements

form = System.Windows.Forms.Form()
groupBoxLocationPoint = System.Windows.Forms.GroupBox()
labelLocationPoint = System.Windows.Forms.Label()
groupBoxCoordinates = System.Windows.Forms.GroupBox()
labelX = System.Windows.Forms.Label()
inputX = System.Windows.Forms.TextBox()
labelY = System.Windows.Forms.Label()
inputY = System.Windows.Forms.TextBox()
labelZ = System.Windows.Forms.Label()
inputZ = System.Windows.Forms.TextBox()
groupBoxDimensions = System.Windows.Forms.GroupBox()
labelW = System.Windows.Forms.Label()
inputW = System.Windows.Forms.TextBox()
labelH = System.Windows.Forms.Label()
inputH = System.Windows.Forms.TextBox()
acceptButton = System.Windows.Forms.Button()
cancelButton = System.Windows.Forms.Button()

toolTipLocationPoint = System.Windows.Forms.ToolTip()
toolTipInputH = System.Windows.Forms.ToolTip()

# Location Point Group

groupBoxLocationPoint.Controls.Add(labelLocationPoint)
groupBoxLocationPoint.Location = System.Drawing.Point(10, 10)
groupBoxLocationPoint.Name = "groupBoxLocationPoint"
groupBoxLocationPoint.Size = System.Drawing.Size(280, 50)
groupBoxLocationPoint.TabIndex = 0
groupBoxLocationPoint.TabStop = False
groupBoxLocationPoint.Text = "Location Point"
groupBoxLocationPoint.UseCompatibleTextRendering = True

labelLocationPoint.Location = System.Drawing.Point(10, 20)
labelLocationPoint.Name = "labelLocationPoint"
labelLocationPoint.Size = System.Drawing.Size(240, 20)
labelLocationPoint.TabIndex = 1
labelLocationPoint.Text = textElementVerticalAlignment + "-" + textElementHorizontalAlignment
labelLocationPoint.TextAlign = System.Drawing.ContentAlignment.MiddleLeft
labelLocationPoint.UseCompatibleTextRendering = True

toolTipLocationPoint.ToolTipTitle = "Location Point"
toolTipLocationPoint.SetToolTip(labelLocationPoint, "The selected text element has a Vertical Alignment parameter\nset to [" + textElementVerticalAlignment + "] and a Horizontal Alignment parameter set to [" + textElementHorizontalAlignment + "].\nThe intersection of these alignments determine the location\npoint used to position the text element.\n\nTo change the location point, cancel this dialog and adjust the\ntext element alignment parameters.")
toolTipLocationPoint.AutoPopDelay = 30000

# Coordinates Group

groupBoxCoordinates.Controls.Add(labelX)
groupBoxCoordinates.Controls.Add(inputX)
groupBoxCoordinates.Controls.Add(labelY)
groupBoxCoordinates.Controls.Add(inputY)
groupBoxCoordinates.Controls.Add(labelZ)
groupBoxCoordinates.Controls.Add(inputZ)
groupBoxCoordinates.Location = System.Drawing.Point(10, 70)
groupBoxCoordinates.Name = "groupBoxCoordinates"
groupBoxCoordinates.Size = System.Drawing.Size(280, 120)
groupBoxCoordinates.TabIndex = 2
groupBoxCoordinates.TabStop = False
groupBoxCoordinates.Text = "Coordinates"
groupBoxCoordinates.UseCompatibleTextRendering = True

labelX.Location = System.Drawing.Point(5, 25)
labelX.Name = "labelX"
labelX.Size = System.Drawing.Size(25, 20)
labelX.TabIndex = 3
labelX.Text = "X:"
labelX.TextAlign = System.Drawing.ContentAlignment.MiddleCenter
labelX.UseCompatibleTextRendering = True

inputX.BorderStyle = System.Windows.Forms.BorderStyle.FixedSingle
inputX.Location = System.Drawing.Point(35, 25)
inputX.Name = "textBoxX"
inputX.Text = textElementX
inputX.Size = System.Drawing.Size(235, 20)
inputX.TabIndex = 4

labelY.Location = System.Drawing.Point(5, 55)
labelY.Name = "labelY"
labelY.Size = System.Drawing.Size(25, 20)
labelY.TabIndex = 5
labelY.Text = "Y:"
labelY.TextAlign = System.Drawing.ContentAlignment.MiddleCenter
labelY.UseCompatibleTextRendering = True

inputY.BorderStyle = System.Windows.Forms.BorderStyle.FixedSingle
inputY.Location = System.Drawing.Point(35, 55)
inputY.Name = "textBoxY"
inputY.Text = textElementY
inputY.Size = System.Drawing.Size(235, 20)
inputY.TabIndex = 6

labelZ.Location = System.Drawing.Point(5, 85)
labelZ.Name = "labelZ"
labelZ.Size = System.Drawing.Size(25, 20)
labelZ.TabIndex = 7
labelZ.Text = "Z:"
labelZ.TextAlign = System.Drawing.ContentAlignment.MiddleCenter
labelZ.UseCompatibleTextRendering = True

inputZ.BorderStyle = System.Windows.Forms.BorderStyle.FixedSingle
inputZ.Location = System.Drawing.Point(35, 85)
inputZ.Name = "textBoxZ"
inputZ.Text = textElementZ
inputZ.Size = System.Drawing.Size(235, 20)
inputZ.TabIndex = 8

# Dimensions Group

groupBoxDimensions.Controls.Add(labelW)
groupBoxDimensions.Controls.Add(inputW)
groupBoxDimensions.Controls.Add(labelH)
groupBoxDimensions.Controls.Add(inputH)
groupBoxDimensions.Location = System.Drawing.Point(10, 200)
groupBoxDimensions.Name = "groupBoxDimensions"
groupBoxDimensions.Size = System.Drawing.Size(280, 90)
groupBoxDimensions.TabIndex = 9
groupBoxDimensions.TabStop = False
groupBoxDimensions.Text = "Dimensions"
groupBoxDimensions.UseCompatibleTextRendering = True

labelW.Location = System.Drawing.Point(5, 25)
labelW.Name = "labelW"
labelW.Size = System.Drawing.Size(25, 20)
labelW.TabIndex = 10
labelW.Text = "W:"
labelW.TextAlign = System.Drawing.ContentAlignment.MiddleCenter
labelW.UseCompatibleTextRendering = True

inputW.BorderStyle = System.Windows.Forms.BorderStyle.FixedSingle
inputW.Location = System.Drawing.Point(35, 25)
inputW.Name = "textBoxW"
inputW.Text = textElementW
inputW.Size = System.Drawing.Size(235, 20)
inputW.TabIndex = 11

labelH.Location = System.Drawing.Point(5, 55)
labelH.Name = "labelH"
labelH.Size = System.Drawing.Size(25, 20)
labelH.TabIndex = 12
labelH.Text = "H:"
labelH.TextAlign = System.Drawing.ContentAlignment.MiddleCenter
labelH.UseCompatibleTextRendering = True

inputH.BorderStyle = System.Windows.Forms.BorderStyle.FixedSingle
inputH.Location = System.Drawing.Point(35, 55)
inputH.Name = "textBoxH"
inputH.Text = textElementH
inputH.Size = System.Drawing.Size(235, 20)
inputH.ReadOnly = True
inputH.TabIndex = 13

toolTipInputH.ToolTipTitle = "Height"
toolTipInputH.SetToolTip(inputH, "This value is unmodifiable. It automatically adjusts to accommodate\nthe associated Text Size parameter and number of lines.")
toolTipInputH.AutoPopDelay = 30000

# Buttons

acceptButton.DialogResult = System.Windows.Forms.DialogResult.OK
acceptButton.Location = System.Drawing.Point(10, 300)
acceptButton.Name = "OK"
acceptButton.Size = System.Drawing.Size(135, 30)
acceptButton.TabIndex = 7
acceptButton.Text = "OK"

cancelButton.DialogResult = System.Windows.Forms.DialogResult.Cancel
cancelButton.Location = System.Drawing.Point(155, 300)
cancelButton.Name = "Cancel"
cancelButton.Size = System.Drawing.Size(135, 30)
cancelButton.TabIndex = 7
cancelButton.Text = "Cancel"

# Form

form.ClientSize = System.Drawing.Size(300, 340)
form.Name = "Specify Text Coordinates"
form.Text = "Specify Text Coordinates"
form.FormBorderStyle = System.Windows.Forms.FormBorderStyle.FixedSingle;
form.MaximizeBox = False
form.MinimizeBox = False
form.ShowIcon = False
form.TopMost = True
form.Controls.Add(groupBoxLocationPoint)
form.Controls.Add(groupBoxCoordinates)
form.Controls.Add(groupBoxDimensions)
form.Controls.Add(acceptButton)
form.Controls.Add(cancelButton)
form.AcceptButton = acceptButton

form.ShowDialog()

output = [False, textElementX, textElementY, textElementZ, textElementW]

if form.DialogResult == System.Windows.Forms.DialogResult.OK:
    output = [True, inputX.Text, inputY.Text, inputZ.Text, inputW.Text]

OUT = output
{% endhighlight %}

### Python Script: Modify Text Element
Finally, this script modifies the selected Text Element using input and actions from the Input Dialog.

{% highlight python %}
import clr

clr.AddReference("RevitNodes")
import Revit
clr.ImportExtensions(Revit.GeometryConversion)

clr.AddReference("RevitServices")
import RevitServices
from RevitServices.Persistence import DocumentManager
from RevitServices.Transactions import TransactionManager

# The inputs to this node will be stored as a list in the IN variables.
dialogResultOK = IN[0]
textElement = UnwrapElement(IN [1])
point = IN[2]
width = IN[3]

# Get the document
doc = DocumentManager.Instance.CurrentDBDocument

# "Start" the transaction
TransactionManager.Instance.EnsureInTransaction(doc)

# Place your code below this line
if dialogResultOK == True:
    textElement.Coord = point.ToXyz()
    if width > 0:
        textElement.Width = width
    
# "End" the transaction
TransactionManager.Instance.TransactionTaskDone()

# Assign your output to the OUT variable.
OUT = point, width
{% endhighlight %}

### In Action: Dynamo Player and the Custom Input Dialog

{% include image.html image="Specify-Text-Coordinates-002.png" %}
{% include image.html image="Specify-Text-Coordinates-003.png" %}
