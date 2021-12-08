# WORKSHOP DUST COLLECTOR

This small repo shows how to configure multiple smart sockets running
[Tasmota](https://tasmota.github.io/docs/) firmware for dust collection in
a workshop.

## Goal

The physical setup assumes the following devices:

* Dust collector with a dedicated smart socket
* One or more machines, each with its own dedicated smart socket
  (please observe that your machines do not exceed the maximum power rating of the smart socket!)

The goal is to turn on the dust collector when at least one of the machines consumes power.
And it shall be turned off again when none of the machines is powered on.

## Design

While there are various ways for creating this functionality
(e.g. [using MQTT and Node-Red](https://www.youtube.com/watch?v=xD1uUO06SXk))
I wanted to go for the solution which requires the *least additional infrastructure*.
So the solution was to directly send commands between the sockets using
[Tasmota rules](https://tasmota.github.io/docs/Rules/).

### Machine Sockets

The machine sockets only need to monitor their power readings and send out different triggers
to the dust collector's socket
depending on whether power has exceeded a certain limit `OnWatts` or dropped below
a lower limit `OffWatts`.

### Dust Collector Socket

This socket receives the notifications from the machines and keeps track of the number of
running machines using a simple counter in `Var1`. If this counter if positive, the dust
collector shall turn on, but if it reaches 0 again, this indicates that all machines are turned off
and the dust collector shall power off.

Note that in order to reset the counter if it went out of sync, the user can manually turn off
socket power which also resets the counter to 0.

## Usage

### Setup

**Prerequisits**

* Two or more smart sockets already flashed with Tasmota, e.g. rebranded Gosund SP111 work well.
* Working network connection from PC to all smart sockets
* Ansible (Ansible Core package is sufficient)
* [Tobias Richter's tasmota ansible role](https://galaxy.ansible.com/tobias_richter/tasmota)

**Customization**

`inventory.yml`
* Adjust the host names to match yours, but remember to put them into the right groups,
  i.e. the socket for the dust collector into `vacuum`, the machine sockets into `machines`.
* Optional: Add authentication information (user + password) to your individual hosts.
* Adjust the power limit variables `OnWatts` and `OffWatts`, if required.

`playbook.yml`
* Please check the other settings (below the rule implementation) if they match your needs,
  e.g. turning off MQTT, etc.
* Possibly add further settings.

**Invocation**

`ansible-playbook -i inventory.yml playbook.yml`

### Operation

**Prerequisits**

* Working network connection between the smart sockets.

**Usage**

* Plug in your devices.
* Have fun!

