---
layout: article
title: "Streaming Minecraft Blocks to Dynamo Sandbox"
date: 2019-08-15 02:59:00
cover: /work/img/Streaming-Minecraft-Blocks-to-Dynamo-Cover.png
collection: work
tags:
  - all
  - code
  - visualprogramming
  - dynamo
  - python
  - minecraft
---

Originally developed for the Raspberry Pi Edition of Minecraft, the [Minecraft Python API](https://www.stuffaboutcode.com/p/minecraft-api-reference.html) allows for manipulation of Minecraft via external Python code. This post utlizes the [Raspberry Jam Mod](https://github.com/arpruss/raspberryjammod) for [Minecraft Forge](https://files.minecraftforge.net) with the [Java Edition of Minecraft](https://www.minecraft.net/en-us/download/) to access a ported version of the API.

<!--more-->

{% include video.html image="Streaming-Minecraft-Blocks-to-Dynamo-001.png" mp4="Streaming-Minecraft-Blocks-to-Dynamo-001.mp4" webm="Streaming-Minecraft-Blocks-to-Dynamo-001.webm" ogg="Streaming-Minecraft-Blocks-to-Dynamo-001.ogg" caption="Pulling blocks from -5 to 5 in x, y, and z directions relative to the player position using the Minecraft Python API." aspectratio="16-10" %}

### Dynamo Definition Part 1

The definition below obtains the current current player position, generates a cross product of coordinates from `-5..5` in `x,y,x` around the player, and obtains the Minecraft blocks at those postions. `player.getPos()` and `getBlock()` are custom Python nodes. `DateTime.Now` allows the graph to run periodically.

{% include image.html image="Streaming-Minecraft-Blocks-to-Dynamo-002a.png" %}

### Custom Python Node: player.getPos()

`mc.player.getPos()` uses the Minecraft Python API to get the current player position.

{% highlight python %}
# Python Node for Dynamo
# Minecraft player.getPos()
# Version 0.1
# Coded by Andrew King
# https://andrewkingme.com
#
# 2019-08-13 Version 0.1
# Hello World

import sys
sys.path.append('C:\Python27\mcpipy')
from mcpi.minecraft import Minecraft

# The inputs to this node will be stored as a list in the IN variables.
dataEnteringNode = IN

mc = Minecraft.create()

playerPos = mc.player.getPos()
playerPosx = int(playerPos.x)
playerPosy = int(playerPos.y)
playerPosz = int(playerPos.z)

# Assign your output to the OUT variable.
OUT = playerPosx, playerPosy, playerPosz
{% endhighlight %}


### Custom Python Node: getBlock()

`mc.getBlock()` uses the Minecraft Python API to get the block type at the requested location.

{% highlight python %}
# Python Node for Dynamo
# Minecraft getBlock()
# Version 0.1
# Coded by Andrew King
# https://andrewkingme.com
#
# 2019-08-13 Version 0.1
# Hello World
import sys
sys.path.append('C:\Python27\mcpipy')
from mcpi.minecraft import Minecraft

# The inputs to this node will be stored as a list in the IN variables.
inputPosx = IN[0]
inputPosy = IN[1]
inputPosz = IN[2]

blockPosx = []
blockPosy = []
blockPosz = []
blockType = []

mc = Minecraft.create()

for x in inputPosx:
  for y in inputPosy:
    for z in inputPosz:
      blockPosx.append(x)
      blockPosy.append(y)
      blockPosz.append(z)
      blockType.append(mc.getBlock(int(x),int(y),int(z)))
      mc.postToChat(str(x) + ', ' + str(y) + ', ' + str(z))

# Assign your output to the OUT variable.
OUT = blockPosx, blockPosy, blockPosz, blockType
{% endhighlight %}

### Dynamo Definition Part 2

The definition continues by filtering out `AIR Block(0)` air blocks and generating a cuboid at each `x,y,z` coordinate.

{% include image.html image="Streaming-Minecraft-Blocks-to-Dynamo-002b.png" %}

### Result

Block locations are posted to Mincraft Chat using `mc.postToChat()` and then output to Dynamo.

{% include image.html image="Streaming-Minecraft-Blocks-to-Dynamo-003.png" %}

{% include image.html image="Streaming-Minecraft-Blocks-to-Dynamo-004.png" %}
