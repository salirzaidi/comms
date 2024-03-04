---
layout: default
title: Communication
nav_order: 4
has_toc: false # on by default
has_children: true
comments: true
usetocbot: true
---
# {{ page.title }}
{: .no_toc }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}
---

In this lesson series, we will learn about computer-to-Arduino interaction. More specifically, serial communication, web serial, and using [p5js](https://p5js.org/) to communicate with Arduino.

## [L1: Intro to Serial](serial-intro.md)

In [this lesson](serial-intro.md), we’ll dive into asynchronous serial communication and how we can use it for bidrectional `Computer ↔ Arduino` communication. We'll show example serial communication clients using terminal programs and Python.

## [L2: Web Serial](web-serial.md)

In [this lesson](web-serial.md), you'll learn about the new [Web Serial API](https://wicg.github.io/serial/) and how to build simple web apps that communicate with Arduino.

## [L3: p5.js Serial](p5js-serial.md)

In [this lesson](p5js-serial.md), you'll learn about [Processing](https://processing.org/), [p5.js](https://p5js.org/), and how to use p5.js with Web Serial (with a focus on processing incoming serial input data).

## [L4: p5.js Serial I/O](p5js-serial-io.md)

In [this lesson](p5js-serial-io.md), we look at creating more complicated p5.js + Arduino applications that bidirectionally communicate together both Arduino to p5.js (`Arduino → Computer`) as well as p5.js to Arduino (`Computer → Arduino`).

## [L5: Paint I/O Example](p5js-paint-io.md)

In [this lesson](p5js-paint-io.md), we will bring everything together thus far to build a fully interactive p5.js + Arduino painting application, called PaintIO, with bidirectional serial communication and a custom paintbrush controller using an accelerometer to control brush location, a [force-sensitive resistor](../arduino/force-sensitive-resistors.md) to control brush size, and bimanual interaction using both Arduino + the mouse simultaneously for drawing.

## [L6: ml5.js Serial](ml5js-serial.md)

In [this lesson](ml5js-serial.md), we'll introduce machine learning frameworks like [Runway ML](https://runwayml.com/) and [ml5js](https://ml5js.org/) and show how to use them with Arduino.
