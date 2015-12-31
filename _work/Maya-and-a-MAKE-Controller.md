---
layout: article
title: "Maya and a MAKE Controller: Form Creation Using Programmable Sensors"
date: 2014-10-21 01:31:00
cover: /img/work/Maya-and-a-MAKE-Controller-Cover.png
collection: work
tags:
  - all
  - code
  - maya
  - sciarc
---

Flow of Information: [Input Sensors] to [Breadboard] to [MAKE Controller] to [USB Interface] to [mchelper] to [Flash Connector] to [MEL Script in Maya]

<!--more-->

![Maya and a MAKE Controller](/img/work/Maya-and-a-MAKE-Controller-001.png)
*Input Sensors, Breadboard, and Make Controller*

<p>
<div class="videoWrapper">
<video autoplay loop muted poster="/img/work/Maya-and-a-MAKE-Controller-002.png">
  <source src="/img/work/Maya-and-a-MAKE-Controller-002.mp4" type="video/mp4">
  <source src="/img/work/Maya-and-a-MAKE-Controller-002.webm" type="video/webm">
  <source src="/img/work/Maya-and-a-MAKE-Controller-002.ogg" type="video/ogg">
  <img src="/img/work/Maya-and-a-MAKE-Controller-002.png" alt="Maya and a MAKE Controller" />
</video>
</div>
<em>mchelper, Flash Connector, and MEL Script in Maya</em>
</p>

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

<p>
<div class="videoWrapper">
<video autoplay loop muted poster="/img/work/Maya-and-a-MAKE-Controller-003.png">
  <source src="/img/work/Maya-and-a-MAKE-Controller-003.mp4" type="video/mp4">
  <source src="/img/work/Maya-and-a-MAKE-Controller-003.webm" type="video/webm">
  <source src="/img/work/Maya-and-a-MAKE-Controller-003.ogg" type="video/ogg">
  <img src="/img/work/Maya-and-a-MAKE-Controller-003.png" alt="Maya and a MAKE Controller" />
</video>
</div>
<em>Screen Capture of Curation in Maya</em>
</p>
