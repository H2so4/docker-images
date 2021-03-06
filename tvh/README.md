## Tvheadend

A docker image of Tvheadend Server.

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
 

- [Overview](#overview)
- [Image Tags](#image-tags)
- [Config](#config)
- [Usage](#usage)
- [GUI configuration](#gui-configuration)
- [Debugging Tvheadend](#debugging-tvheadend)
  - [To get core files, and other artifact files](#to-get-core-files-and-other-artifact-files)
  - [To get a stack trace when tvheadend hangs / freezes (Deadlock or Livelock)](#to-get-a-stack-trace-when-tvheadend-hangs--freezes-deadlock-or-livelock)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

### Overview

There are 3 kinds of tvheadend docker images:

[`dreamcat4/tvheadend`](https://registry.hub.docker.com/u/dreamcat4/tvheadend)

  * The main release image
  * Very simple, 2 volumes: `/config` and `/recordings`
  * The `/config` volume provided by this image should NOT be bind-mounted. It should be a regular docker volume. Since docker only pre-seeds volumes data for the regular type of volumes.
  * Ubuntu trusty base image.
  * Uses excellent automated build of all branches. Including stable and development release etc.
  * Updated every 24 hours automatically from latest pkgs.

[`dreamcat4/tvh.debug`](https://registry.hub.docker.com/u/dreamcat4/tvh.debug)

  * A version for Debugging
  * To do a debigging run - just swap it out for the main `tvheadend` image - shares the same `tvh.config` image.
  * You can swap between these 2 debugging and release docker images whenever you wish. They are interchangeable.
  * Must bind mount a `/crash` volume from a host folder. Core dump file is not written otherwise (with native docker images).
    * Example: `docker create --volume $HOME/tvh_crashes:/crash`
  * Includes all required debugging symbols and `gdb`
  * If tvheadend crashes, it automatically takes a core dump and runs it through `gdb`
  * Contains `stacktrace` script to take running snapshot with `gdb`
  * Trace is not switched on by default. Just add the `--trace` flags as arguments by specifying extra optional `CMD` params

[`dreamcat4/tvh.config`](https://registry.hub.docker.com/u/dreamcat4/tvh.config)

  * You should build your own version of the `tvh.config` image and use `volumes-from` to mount it to the main tvheadend image (or debug image).
  * The `/config` volume provided by this image should NOT be bind-mounted. It should be a regular docker volume. Since docker only pre-seeds volumes data for the regular type of volumes.
  * You should use Ubuntu trusty `ubuntu-debootstrap:14.04` as the base image.
  * Locale and timezone files are symlinked from this `tvh.config` image.
    * They are missing from the main image
  * See examlpe `Dockefile` in `dreamcat4/tvh.config` for more help / instructions.

### Image Tags

* `dreamcat4/tvheadend:latest` - Stable branch - official releases

* `dreamcat4/tvheadend:master` - Master branch - nightly builds

* `dreamcat4/tvheadend:unstable` - Master branch - 'unstable' releases

* `dreamcat4/tvheadend:testing` - Stable branch - nightly builds

* `dreamcat4/tvheadend:stable` - Stable branch - official releases

Same image tags are available for the Debugging version.

By default, the `:latest` image tag maps onto the latest official 'Stable' release of tvheadend. e.g. `v4.0.4` at this time of writing.

### Config

* The default configuration folder is `VOLUME /config`. A docker volume. It is not some `.hts` folder or anything else like that.
* The `VOLUME /config` is pre-seeded from the docker Build context `config/`.

### Usage

```sh
docker run -v /path/to/your/recordings:/recordings dreamcat4/tvheadend [extra tvheadend args...]
```

* The default arguments to start tvheadend are always `tvheadend -u hts -g video -c /config`. You can change this by overriding the `ENTRYPOINT` array.

* Do not make your `/config` volume a host mount `/host_folder:/mountpount`. If you want this image to copy the tvheadend pre-seed configration data into `/config`. Else you will need to configure everything yourself from scratch. And there will be no pre-seeded configration.

* You should create your own variant of the example `dreamcat4/tvh.config` image, with a localized configuration that gets pre-seeded into the tvheadend `/config` folder. See: `dreamcat4/tvh.config/dockerfile`.

* SAT>IP server(s) on your local network will not work with Docker's default NAT style of networking. Tvheadend will log: 'Scan no data failed' or similar messages indicating a failure to communicate with the SAT>IP server. To solve that you can either:

  * Specify the docker networking mode as `--net="host"`

  * Use `dreamcat4/pipework` to configure the networking instead.

* To access DVB Tuners attached to your host machine, you will need to specify the `--device` flag, multiple times. For example:

```sh
device:
  - /dev/dvb/adapter0/demux0
  - /dev/dvb/adapter0/dvr0
  - /dev/dvb/adapter0/frontend0
  - /dev/dvb/adapter0/net0
```

* The August DVB-T210v1 / Geniatech T220 tuner requires Linux Kernel 3.19 or higher. Other tuners will have different requirements. Run the `w_scan` or similar on you host system to check your DVB tuner is working properly... before trying to use it inside Tvheadend.

### GUI configuration

See official Tvheadend Documentation about that.

### Debugging Tvheadend

* Official tvheadend [debugging guidelines](https://tvheadend.org/projects/tvheadend/wiki/Debugging)

#### To get core files, and other artifact files

Then `/crash` must be bind-mounted as a host volume. It doesn't work otherwise.

```sh
docker create --volume |/tvh_debug:/crash --privileged=true --name tvh.debug dreamcat4/tvh.debug --abort
docker start tvh.debug
```


* Must be run in privileged mode, to allow modification of `/proc` to set the core file path.

#### To get a stack trace when tvheadend hangs / freezes (Deadlock or Livelock)

If the container name is `tvh.debug`, you can just run:

```sh
docker exec -it tvh.debug stacktrace
```

And it will invoke the `stacktrace` script, dumping all output to stdout.



