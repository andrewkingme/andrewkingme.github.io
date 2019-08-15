---
layout: article
title: "Maya and a MAKE Controller: Form Creation Using Programmable Sensors"
date: 2014-10-21 01:31:00
cover: /work/img/Maya-and-a-MAKE-Controller-Cover.png
collection: work
tags:
  - all
  - code
  - maya
  - sciarc
---

Flow of Information: [Input Sensors] to [Breadboard] to [MAKE Controller] to [USB Interface] to [mchelper] to [Flash Connector] to [MEL Script in Maya]

<!--more-->

{% include image.html image="Maya-and-a-MAKE-Controller-001.png" caption="Input Sensors, Breadboard, and Make Controller" %}

{% include video.html image="Maya-and-a-MAKE-Controller-002.png" mp4="Maya-and-a-MAKE-Controller-002.mp4" webm="Maya-and-a-MAKE-Controller-002.webm" ogg="Maya-and-a-MAKE-Controller-002.ogg" caption="mchelper, Flash Connector, and MEL Script in Maya" aspectratio="16-9" %}

The following MEL Script captures three input devices for form creation and a fourth input device for curation:

{% highlight Perl %}
wave1Handle.translateY = sensorInput.sensor1 *400;
twist1Handle.translateZ = sensorInput.sensor2 *400;
flare1Handle.translateZ = sensorInput.sensor3 *400;

if(sensorInput.sensor4<0.3)
{
string $newSurf[] = `duplicate -rr polySurface89`;
move -r 50 0 0 $newSurf[0];
}

wave1Handle.translateY = sensorInput.sensor1 *400;
twist1Handle.translateZ = sensorInput.sensor2 *400;
flare1Handle.translateZ = sensorInput.sensor3 *400;

float $count1;

if((sensorInput.sensor4 > 0.65) && (sensorInput.sensor4 < 0.7))
{

float $dist = 20 + ($count1*10);
string $newSurf[] = `duplicate -rr polySurface89`;
move -r $dist 0 0 $newSurf[0];

$count1++;
}
{% endhighlight %}

{% include video.html image="Maya-and-a-MAKE-Controller-003.png" mp4="Maya-and-a-MAKE-Controller-003.mp4" webm="Maya-and-a-MAKE-Controller-003.webm" ogg="Maya-and-a-MAKE-Controller-003.ogg" caption="Screen Capture of Curation in Maya" aspectratio="16-9" %}
