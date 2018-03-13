# Running Clojure on the Raspberry Pi

## Rationale

Dynamic languages have gradually become prominent for building more or less
smart devices. The Raspberry Pi is a perfect example of cheap but powerful
hardware used for building all sorts of devices. While Python and NodeJS are
popular dynamic languages, this documentation explains the advantages of using
Clojure and provides various how-to's. It aims to be accessible to people
familiar with Clojure and the Raspberry Pi ecosystem while remaining
beginner friendly.

### Asynchronous programming

The smarter the device and the more asynchronous its behavior will be. Clojure
provides strong concurrency primitives as well as the excellent
[core.async](https://clojure.org/reference/compilation) library, an
implementation of [communicating sequential
processes](https://en.wikipedia.org/wiki/Communicating_sequential_processes)
akin to the go programming language.

Frameworks such as the [Robot Operating
System](https://en.wikipedia.org/wiki/Robot_Operating_System) describe the
importance of loosely coupled modules talking via
[pub/sub](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern).
Clojure can successfuly implement such a scheme within a single process via
core.async. Several libraries/micro-frameworks exist for writing and managing
modules. The strong support for information provided by the language makes it
perfectly suitable for writing complex but extensible systems.

### Unified development

Clojure targets Java as well as Javascript and code can be shared between
platforms. This result in a unique experience of trully [writting once and
running anywhere](https://en.wikipedia.org/wiki/Write_once,_run_anywhere). Other
languages providing such multi-plaform support are not as mature yet.

The process of sharing code between devices, back-end and front-end greatly
simplifies data modeling. Utilities written for describing and validating data
do not need to be written again in another language resulting in perfect
consistency. This alone eliminates an important class of bugs besides being a
valuable time saver.

Clojure now ships [Spec](https://clojure.org/about/spec), a core library for
describing, validading and generating data amongst other things. It provides a
standard framework for data modeling. Typicaly, the robustness of field devices
and the system they belong to is assessed by feeding test data for [fuzzy
testing](https://en.wikipedia.org/wiki/Fuzzing). This is a laborious, error
prone and unsatisfying process. Using Spec, data can be generated automatically,
which means that not much is then needed for testing the field devices or
anything in the system by feeding valid and unvalid data. Much bugs usually
discovered in production can be uncover using this method.

## Running Java

As of today, the three main fully-fledged Java 8 environments supported by the
Raspberry Pi are :

- Oracle
- OpenJDK
- Zulu Embedded

As far as we know, OpenJDK is still not optimized and runs a lot slower than
Oracle, making it unsuitable. The latter is optimized but requires a license
when used for anything else than education and personal projects. Hence, our
recommended choice is [Zulu
Embedded](https://www.azul.com/solutions/embedded-and-the-iot/) by [Azul
Systems](https://www.azul.com/). It is fully certified and offers performance
parity with Oracle. More importantly, it is open source and freely downloadable.
Following the open source model, the company finances the project by providing
paid support.

Zulu Embedded releases are available
[here](https://www.azul.com/downloads/zulu-embedded/) for manual installation.

The easiest way is to [add the Azul Systems debian repository and use
apt](http://zulu.org/zuludocs-folder/#ZuluUserGuide/PrepareZuluPlatform/AttachAPTRepositoryUbuntuOrDebianSys.htm)
:
```
$ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 0x219BD9C9
$ echo 'deb http://repos.azulsystems.com/debian stable main' | sudo tee /etc/apt/sources.list.d/zulu.list > /dev/null
$ sudo apt update
$ sudo apt install zulu-embedded-8
$ java -version
```

Java 9 is net yet supported by Azul Systems but it is probably a matter of time.

## Configuring IO

There are no special requirements for using GPIO, I2C or SPI. Nonetheless,
regardless of Clojure, the serial port behaves oddly on the Raspberry Pi 3. Many
articles and posts have been written on the subject. In short, the Raspberry Pi
3 is equipped with two built-in
[UART](https://en.wikipedia.org/wiki/Universal_asynchronous_receiver-transmitter)'s
for serial communication :

- __/dev/ttyAMA0__ is a high performance hardware UART used by bluetooth.
- __/dev/S0__ is a mini UART, much less capable, used by the serial port.

Running the serial port on the mini UART will be unsuitable for many
applications. The solution is to choose an alternative [device
tree](https://en.wikipedia.org/wiki/Device_tree) overlay in
__/boot/config.txt__.

This line device tree will swap bluetooth and the serial port, meaning
bluetooth will run on mini UART, which might be enough :
```
dtoverlay=pi3-miniuart-bt
```

This line will simply switch bluetooth off and is recommended if bluetooth is
not needed :
```
dtoverlay=pi3-disable-bt
```

After a reboot, the serial port will be accessible at __/dev/ttyAMA0__ just like
on prior Raspberry Pi models. Read more about this matter
[here](https://www.raspberrypi.org/documentation/configuration/uart.md).

## Leiningen

[Leiningen](https://leiningen.org/) is installed as usual. Starting a REPL is
usually slow, much slower than on a desktop and the process might timeout. This
is preventable by adding this option to the project file :

```clj
{:repl-options {:timeout 180000}}  ;; 3 minutes for example
```

Once the REPL is launched, it behaves as usual. Projects can be compiled on a
development machine rather than on the Raspberry Pi itself.

## Development via SSH

While it is possible to write code on the Raspberry Pi, it is probably no
replacement to your desktop environment. The easiest way is to keep everything
on your machine and use
[SSHFS](https://www.digitalocean.com/community/tutorials/how-to-use-sshfs-to-mount-remote-file-systems-over-ssh)
to mount the folder where the project resides on the Raspberry Pi in order to
launch a REPL. Otherwise, it is possible to use [rsync](https://rsync.samba.org)
and synchronize files everytime before starting the REPL.

The REPL can be made available to the local network by typing the following :

```
$ lein repl :start :host 0.0.0.0 :port 4000
```

The rest is just traditional Clojure development.

## Optimizing uberjars

For better performance and faster boot time, it is best for the user to be
acquainted with the [compilation](https://clojure.org/reference/compilation)
processs and its various options. This is true for any environment but become
more relevant for the Raspberry Pi as it is much more limited than a desktop
computer or a server.

## Recommended libraries and practises

### GPIO

The main library for using the GPIO pins from Java is [PI4J](http://pi4j.com/).
It is mainly built upon [wiringPi](http://wiringpi.com/), a renowned C library.

[pi4clj](https://github.com/dvlopt/pi4clj) is a thin Clojure wrapper for
wiringPi using the stable JNI bindings from PI4J.

Our attemps to provide [JNA](https://en.wikipedia.org/wiki/Java_Native_Access)
bindings for PIGPIO resulted in an utter failure. Indeed, it led to unexpected
segfaults and early terminations for unknown reasons. This problem occured in
other Java projects and also happens with
[JNI](https://en.wikipedia.org/wiki/Java_Native_Interface) bindings.

### HTTP

Raspberry Pies are often used for running small HTTP servers.

While there are quite few libraries for HTTP servers and clients, we recommend
[Aleph](https://github.com/ztellman/aleph). It is a popular choice and provides
asynchronous utilities for spinning up a server or performing requests. It
offers great support for [websockets](https://en.wikipedia.org/wiki/WebSocket).

### I2C

[dvlopt.i2c](https://github.com/dvlopt/i2c) is the only way to talk to I2C slave
devices from Clojure on the JVM. Here are current sub-libraries targeting
specific sensors and devices :

- [bme280](https://github.com/dvlopt/i2c.bme280), a popular environment sensor
  built by Bosh.
- [horter-i2hae](https://github.com/dvlopt/i2c.horter-i2hae), a simple A/D
  converter.
- [mcp342x](https://github.com/dvlopt/i2c.mcp342x), a family of A/D converters.

### Logging

There are a few logging libraries for Clojure but the most widely used and
recommended one is [Timbre](https://github.com/ptaoussanis/timbre). It is fairly
easy to write log appenders and be up and running in minutes.

### Modules

Complex programs are often modular and it is easier to extend modular programs.
We recommend [Integrant](https://github.com/weavejester/integrant), a great
micro-framework for writing modules, connecting them and managing them.

### Meter-Bus

[JMbus](https://www.openmuc.org/m-bus/) is the only stable and actively
maintained Java library for talking to Meter-Bus slaves, typically meters. It
requires Meter-bus converters such as those provided by
[Solvimus](http://www.solvimus.de/en/).

[dvlopt.mbus](https://github.com/dvlopt/mbus) is a Clojure wrapper around JMbus.
As of today, it supports Meter-Bus via the serial port and TCP/IP.

### MQTT

[MQTT](https://en.wikipedia.org/wiki/MQTT) has become popular for internet of
things projects for providing bidirectional communication organized around
topics. It provides useful functionalities over something like barebone
[websockets](https://en.wikipedia.org/wiki/WebSocket).

The [Paho MQTT Java client](https://github.com/eclipse/paho.mqtt.java) is
probably the most active and well-maintained MQTT client library in the
ecosystem.

[dvlopt.mqtt](https://github.com/dvlopt/mqtt) is a Clojure wrapper around the
Paho library.

### Network over 3G/4G

The [Huawei e3772](https://consumer.huawei.com/en/mobile-broadband/e3372/) USB
4G dongle is recommended as it is supported by Raspbian and does not require any
installation nor configuration. The only shortcoming is that it auto-disconnect
after a few minutes. There is no way to disable this behaviour but it can be
configured to raise the interval to two hours. A simple solution for keeping the
dongle active is to setup a CRON job for pinging google or anything else within
the disconnect interval.

The dongle runs a webserver accessible like a typical router. Run this in the
console and find out which address it is :

```
$ ip route
```

### Remote management

[Ansible](https://www.ansible.com/) is an administration tool used for managing
servers using declarative configuration files. It leverages SSH and hosts do not
require any setup. A machine can actually manage itself and this can be used as
a remote update mechanism. A Raspberry Pi can indeed be prepared for
periodically fetching configuration files via FTP or by any other mean and then
use Ansible on itself. An alternative similar solution would be to take
advantage of
[ansible-pull](http://docs.ansible.com/ansible/latest/ansible-pull.html).

### Serial port

Talking via the serial port from the JVM has always been a bit burdensome.
[Purejavacomm](https://github.com/nyholku/purejavacomm) aims to provide a
multi-platform compatibility without any prior installation of native libraries.

To our knowledge, [Clj-serial](https://github.com/peterschwarz/clj-serial) is
the only Clojure wrapper for Purejavacomm.

### SPI

We haven't managed to find any Java library for talking to SPI slaves. The only
existing Clojure library is [dvlopt.spi](https://github.com/dvlopt/spi) which is
rather an experiment for the time being.
