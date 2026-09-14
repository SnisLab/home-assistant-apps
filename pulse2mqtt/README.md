# Pulse2MQTT Home Assistant App

Pulse2MQTT connects a Tibber Pulse to Home Assistant. It reads the meter data
locally and publishes energy, power and diagnostic values through MQTT. Home
Assistant MQTT Discovery creates the device and its entities automatically.

## Requirements

- Home Assistant with the MQTT integration configured
- A Tibber Pulse reachable from Home Assistant on the local network
- The Home Assistant Mosquitto Broker app, or an external MQTT broker

The app uses the Home Assistant MQTT service automatically when it is
available. An external broker can be configured in the app options.

## Installation

Install the app from the Pulse2MQTT Home Assistant app repository and open its
configuration. Enter the Pulse host, username and password, then start the
app. The discovered Tibber Pulse device appears in Home Assistant after the
first successful MQTT connection.

The app supports `amd64` and `aarch64` systems.

## Features

- Energy consumption and feed-in energy
- Current power
- Battery level estimation and voltage
- Temperature and signal strength
- Pulse diagnostics
- MQTT Discovery for automatic Home Assistant entities
- Battery profiles for alkaline, LFB AA, NiMH and regulated 1.5 V cells

## Documentation

See [`DOCS.md`](DOCS.md) for all configuration options, MQTT topics, battery
profiles, Energy Dashboard setup and troubleshooting notes.

## Project

The application source and binary releases are maintained in
[SnisLab/pulse2mqtt](https://github.com/SnisLab/pulse2mqtt). This repository
only packages the released binaries as a Home Assistant app image.
