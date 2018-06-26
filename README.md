# Running Clojure on the Raspberry Pi

[![Create Commons
License](https://i.creativecommons.org/l/by-sa/4.0/88x31.png)](http://creativecommons.org/licenses/by-sa/4.0/)

This work is licensed under a [Creative Commons Attribution-ShareAlike 4.0
International License](http://creativecommons.org/licenses/by-sa/4.0/).

## 1. Rationale

Dynamic languages have gradually become prominent for building more or less
smart devices. The Raspberry Pi is a perfect example of cheap but powerful
hardware used for building such of devices. While Python and NodeJS are popular
dynamic languages, this documentation explains the advantages of using Clojure
and provides various how-to's. It aims to be accessible to people familiar with
Clojure and the Raspberry Pi ecosystem while remaining beginner friendly.

### 1.1. Asynchronous programming

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
Using Clojure, arbitrarly complex programs can be written by composing modules
exchanging data asynchronously.

### 1.2. Unified development

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
standard framework for data modeling. Typically, the robustness of field devices
and the system they belong to is assessed by feeding test data for [fuzzy
testing](https://en.wikipedia.org/wiki/Fuzzing). This is a laborious, error
prone and unsatisfying process. Using Spec, data can be generated automatically,
which means that not much is then needed for testing the field devices, or
anything in the system, by feeding valid and unvalid data. Much bugs usually
discovered in production can be uncover using this method.

## 2. Choosing and installing a JVM

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

## 3. Configuring IO

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

This line will swap bluetooth and the serial port, meaning bluetooth will run on
mini UART, which might be enough or not at all :
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

## 4. Leiningen

[Leiningen](https://leiningen.org/) is installed as usual. Starting a REPL is
usually slow, much slower than on a desktop and the process might timeout. This
is preventable by adding this option to the project file :

```clj
{:repl-options {:timeout 180000}}  ;; 3 minutes for example
```

Once the REPL is launched and everything is compiled, it behaves as usual. For
production, projects can be compiled on a development machine rather than on the
Raspberry Pi itself.

## 5. Development via SSH

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

The rest is just traditional Clojure development of connecting your text
editor/IDE to the REPL and doing wonders.

## 6. Optimizing uberjars

For better performance and faster boot time, it is best for the user to be
acquainted with the [compilation](https://clojure.org/reference/compilation)
processs and its various options. This is true for any environment but become
more relevant for the Raspberry Pi as it is much more limited than a desktop
computer or a server.

## 7. Recommended libraries

### 7.1. Miscellaneous

#### 7.1.1. Logging

There are a few logging libraries for Clojure but the most widely used and
recommended one is [Timbre](https://github.com/ptaoussanis/timbre). It is fairly
easy to write log appenders and be up and running in minutes.

#### 7.1.2. Modules

Complex programs are often modular and it is easier to extend modular programs.
We recommend [Integrant](https://github.com/weavejester/integrant), a great
micro-framework for writing modules, connecting them and managing them.

### 7.2. IO

#### 7.2.1. GPIO

The Raspberry Pi community often relies on either
[wiringPi](http://wiringpi.com) or [PIGPIO](https://github.com/joan2937/pigpio),
two common C libraries for handling GPIO lines among other things. Higher level
languages often provide wrappers, specially for wiringPi. The main library for
accessing wiringPi from Java is [PI4J](http://pi4j.com), although it is not
actively maintained anymore. Any attempts (including our own) to provide
bindings to PIGPIO resulted in an utter failure. For unknown reasons, sooner or
later the JVM presents unexpected segfaults or early terminations, both when
using [JNA](https://en.wikipedia.org/wiki/Java_Native_Access) and
[JNI](https://en.wikipedia.org/wiki/Java_Native_Interface).

In any case, those C underlying libraries directly write to /dev/mem or
/dev/gpiomem. While this is fast, a number of problems arise. For instance,
there is no automatic clean-up which means that if an application crashes for
any reason, the state of the lines remains as is.

A best approach would be to rely on a proper Linux driver. For years, driving
lines has been done via the SysFS approach by exporting lines as files and then
performing simple IO. Unfortunately, this was slow and presented the same kind
of problems as writing directly to /dev/mem. The whole process was messy.

Linux now offers a new API since version 4.18. Each GPIO chip is accessible as a
char device. This new scheme is surprisingly fast, offers better support for
interrupts as well as a number of other desirable features. For instance, lines
need to be properly requested. A single handle can drive several lines at once
and when it is released, whether on purpose or when the process terminates,
lines go back to their initial state.

The only java library built for this new API is
[dvlopt/linux-gpio.java](https://github.com/dvlopt/linux-gpio.java). It serves
as the basis for
[dvlopt/linux.gpio.clj](https://github.com/dvlopt/linux.gpio.clj), a
higher-level clojure wrapper. Furthermore, relying on a Linux utility means
those libraries can run on any machine as they are not specific to the Raspberry
Pi.

#### 7.2.2. I2C

[dvlopt/linux.i2c.clj](https://github.com/dvlopt/linux.i2c.clj) is the only
Clojure library for talking to I2C slave devices. Here are current sub-libraries
targeting specific sensors and devices :

- [bme280](https://github.com/dvlopt/linux.i2c.bme280), a popular environment
  sensor built by Bosh.
- [horter-i2hae](https://github.com/dvlopt/linux.i2c.horter-i2hae), a simple A/D
  converter.
- [mcp342x](https://github.com/dvlopt/linux.i2c.mcp342x), a family of A/D
  converters.

#### 7.2.3. Meter-Bus

[JMbus](https://www.openmuc.org/m-bus/) is the only stable and actively
maintained Java library for talking to Meter-Bus slaves, typically meters. It
requires Meter-bus converters such as those provided by
[Solvimus](http://www.solvimus.de/en/).

[dvlopt/mbus](https://github.com/dvlopt/mbus) is a Clojure wrapper around JMbus.
As of today, it supports Meter-Bus via the serial port and TCP/IP.

#### 7.2.4. Serial port

Talking via the serial port from the JVM has always been a bit burdensome.
[Purejavacomm](https://github.com/nyholku/purejavacomm) aims to provide a
multi-platform compatibility without any prior installation of native libraries.
To our knowledge, [Clj-serial](https://github.com/peterschwarz/clj-serial) is
the only Clojure wrapper for Purejavacomm.

Historically, RXTX has been widely used for serial IO. The project is now
discontinued. However, [openmuc/jrxtx](https://github.com/openmuc/jrxtx) aims to
provide a replacement as well as a few improvements. The slight downside is that
it requires the installation of native libraries. Because a lot of legacy
projects (and a few present ones) relies on RXTX, this library might be
preferable over purejavacomm. [dvlopt/rxtx](https://github.com/dvlopt/rxtx) is a
simple to use clojure wrapper.

#### 7.2.5. SPI

Basic SPI utilities are proposed by the aforementioned PI4J library. To our
knowledge, there is not any specific Java library.
[dvlopt/spi](https://github.com/dvlopt/spi) is an attempt to provide bindings to
the Linux API but remains a poorly documented experiment for the time being.

### 7.3. Networking

#### 7.3.1. HTTP and websockets

Raspberry Pies are often used for running small HTTP servers.

While there are quite few libraries for HTTP servers and clients, we recommend
[Aleph](https://github.com/ztellman/aleph). It is a popular choice and provides
asynchronous utilities for spinning up a server or performing requests as well
as for executing requests. It offers great support for
[websockets](https://en.wikipedia.org/wiki/WebSocket).

#### 7.3.2. MQTT

[MQTT](https://en.wikipedia.org/wiki/MQTT) has become popular for internet of
things projects for providing bidirectional communication organized around
topics. It provides useful functionalities over something like barebone
[websockets](https://en.wikipedia.org/wiki/WebSocket).

The [Paho MQTT Java client](https://github.com/eclipse/paho.mqtt.java) is
probably the most active and well-maintained MQTT client library in the
ecosystem.

[dvlopt/mqtt](https://github.com/dvlopt/mqtt) is a Clojure wrapper around the
Paho library.

## 8. Recommended tools and practises

### 8.1. Writing modular programs

Complex programs benefit from being modular in order to remain extensible.
Networking and handling various IO's will result in plenty of asynchronicity.
Core.async can provide a pub/sub event bus and modules can subscribes to the
needed topics and start the necessary goroutines. The event bus would in itself
be a top-level module requested by all other participating modules.

However, when an integrant system is started, modules are initialized
synchronous after resolving dependencies between them. If a module starts
pushing values asynchronously on the event bus before other dependent modules
are ready, the outcome will be data loss and inconsistency. Given this fact,
modules producing values should wait a signal. In practise, besides the event
bus should be a promise channel serving the purpose of signaling that the system
is ready to work. This channel should deliver the signal once integrant returns,
meaning that every module is ready and had the change to subscribe to the needed
topics.

### 8.2. Network over 3G/4G

The [Huawei e3772](https://consumer.huawei.com/en/mobile-broadband/e3372/) USB
4G dongle is recommended as it is supported by Raspbian and does not require any
installation nor configuration. The only shortcoming is that it auto-disconnect
after a few minutes. There is no way to disable this behaviour but it can be
configured to raise the interval to two hours. A simple solution for keeping the
dongle active is to setup a CRON job for pinging google or anything else within
the disconnect interval.

The dongle runs a webserver accessible like a typical router. Run this in the
terminal and find out which address it is :

```
$ ip route
```

### 8.3. Remote management

[Ansible](https://www.ansible.com/) is an administration tool used for managing
servers using declarative configuration files. It leverages SSH and hosts do not
require any setup. A machine can actually manage itself and this can be used as
a remote update mechanism. A Raspberry Pi can indeed be prepared for
periodically fetching configuration files via FTP or by any other mean and then
use Ansible on itself. An alternative similar solution would be to take
advantage of
[ansible-pull](http://docs.ansible.com/ansible/latest/ansible-pull.html).
