---
layout: post
title: Building a Tessel-compatible Driver for the MPU-6050 Accelerometer and Gyroscope
---

I've recently been working on a project using the
[Tessel](https://tessel.io){:.link.dim.blue} development
board. For those who aren't familiar, the Tessel is a relatively cheap
microcontroller board similar to an Arduino that runs Javascript - at
time of writing this post, this is achieved through a custom runtime
that ships with the board.

The Tessel has a lot of useful, pre-built "plug and play" modules which are a cinch to setup and get running.
Unfortunately, I discovered there was no pre-built module that included
a gyroscope, only a standalone accelerometer module. I found myself
needing both an accelerometer and a gyroscope for a project so I decided
to look at building a custom module.

Fortunately, there are other hardware options available that the Tessel
can work with. We picked up a couple of [InvenSense MPU-6050 IMU](http://www.invensense.com/mems/gyro/mpu6050.html){:.link.dim.blue} chips
mounted on the GY-521 breakout board for around $5/board. The MPU-6050
combines a 3-axis accelerometer and 3-axis gyroscope so it was perfect
for our needs.

The MPU-6050 can communicate with a Tessel or any other development
board over I2C, a nice and simple protocol that only requires four pins
be connected for communication. The [Hardware API](https://tessel.io/docs/hardwareAPI){:.link.dim.blue} section of
the official Tessel documentation explains the I2C protocol pretty well.

The wiring diagram below shows how the
GY-521 can be connected to the Tessel. The four pins that need to be
connected are GND for ground, 3.3V for power, SDA for I2C serial data,
and SCL for the I2C serial clock. The dashed line in the diagram shows
how the AD0 pin can optionally be pulled high, this changes the I2C
address of the chip to 69 (usually 68), which is handy if you want
to connect two of the boards to the same I2C bus.

![GY-521 Wiring Diagram](/images/2015/04/18/gy521-wiring-diagram.png)

Communicating with the MPU-6050 over I2C is basically a case of reading
and writing bytes to and from a set of registers on the device. Here is an example of reading
the `WHO_AM_I` register from the MPU-6050 using Tessel's I2C library.
The response from reading this register will hopefully be a byte with
the value 68 or 69 depending on whether or not you have pulled the AD0
pin high.

index.js:

{% highlight js %}
var tessel = require('tessel');
// Using 0x68 as the I2C address
var i2c = tessel.port['A'].I2C(0x68)
i2c.transfer(new Buffer([0x75]), 1, function(err, rx) {
  console.log('Address is', rx);
});
{% endhighlight %}

Saving that and running on your Tessel should give you what you are
looking for:

{% highlight sh %}
tessel run index.js

...
INFO Running script...
Address is <Buffer 68>
{% endhighlight %}

Congratulations, your Tessel and the MPU-6050 are talking over I2C.

I've written a driver/library to make it easy for others to quickly get up and
running with a Tessel and the GY-521 board. It's designed to be very
easy to read raw accelerometer and gyroscope measurements and/or
calculate pitch and roll angles directly. As a quick example, here's how
you would use the driver to have your Tessel log out the pitch and roll
of the chip every 100ms:

{% highlight js %}
var tessel = require('tessel');
var mpu6050 = require('tessel-mpu6050').use(tessel.port['A']);

mpu6050.on('ready', function() {
  setInterval(function() {
    mpu6050.getPitchAndRoll(100, function(pitch, roll) {
      console.log('pitch:', pitch, 'roll:', roll);
    });
  }, 100);
});
{% endhighlight %}

The driver library I've implemented was inspired by the excellent work of the core
Tessel team on other drivers for the official Tessel modules. I would
recommend having a read over the driver for the accelerometer chip, see [accel-mma84](https://github.com/tessel/accel-mma84){:.link.dim.blue} on Github.

I've made my MPU-6050 driver open source - you can take a look at it on [Github](https://github.com/jamesmccann/tessel-mpu6050){:.link.dim.blue}
or [npm](https://www.npmjs.com/package/tessel-mpu6050){:.link.dim.blue}.

Any thoughts, ideas, annoyances, or contributions are always
appreciated and if you've used the library or found it helpful, I'd love
to hear about it - hit me up on [Twitter](https://twitter.com/jmccnz){:.link.dim.blue}!

---

Credit to [Rob Hidalgo](https://twitter.com/unrob){:.link.dim.blue} for his [Tessel SVG](https://forums.tessel.io/t/8-bit-ish-music-player/453){:.link.dim.blue}
and charlesrct for the [GY-521 SVG](http://fritzing.org/projects/mpu-6050-board-gy-521-acelerometro-y-giroscopio){:.link.dim.blue}, both of which I used to make the wiring diagram for this article.
