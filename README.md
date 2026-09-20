<h1 align="center">Homebridge LG webOS TV — Matter fork</h1>

<p align="center">
    <a href="https://www.npmjs.com/package/homebridge">
        <img src="https://img.shields.io/badge/powered%20by-homebridge-blue" alt="powered by homebridge">
    </a>
    <a href="#how-it-talks-to-the-tv">
        <img src="https://img.shields.io/badge/powered%20by-LG%20SSAP-blue" alt="powered by the LG SSAP protocol">
    </a>
    <a href="#the-tv-over-matter">
        <img src="https://img.shields.io/badge/matter-power-brightgreen" alt="Matter: power">
    </a>
    <a href="LICENSE">
        <img src="https://img.shields.io/badge/license-MIT-lightgrey" alt="license MIT">
    </a>
</p>

---

This is a fork of [`grzegorz914/homebridge-lgwebos-tv`](https://github.com/grzegorz914/homebridge-lgwebos-tv),
which does all the work of talking to an LG webOS television over its own SSAP protocol on the local
network. Everything that plugin does, this one does.

What the fork adds is **Matter**, so the same televisions reach Alexa, SmartThings and Aqara as well as
Apple Home — and a single path for power, so the two ecosystems never disagree about whether a TV is on.

## The TV over Matter

Matter does define a television. No controller renders it: Apple Home and the Aqara app both ignore the
media device types, and an accessory a controller cannot draw is one that does not appear at all. So what
is published is what every controller does draw:

| Accessory | What it is | When |
| --- | --- | --- |
| `<TV name>` | Power, as an outlet — or a light, if you prefer | Always |
| `<TV name> <input>` | One switch per configured input, on while the TV is showing it | Unless `matter.inputs` is false |
| `<TV name> Backlight` | A dimmer | When backlight control is enabled under Picture |

Turning a TV on and off has one method that both HomeKit and Matter go through, and the TV announces every
change — from HomeKit, from a controller, or from the remote on the sofa — so both sides hear it.

Matter is on wherever the Homebridge bridge running this plugin has Matter enabled; that is the real
opt-in. `matter.enable: false` on a device leaves that one out, and `matter.switchStyle` chooses between an
outlet and a light.

Two things keep the two ecosystems still, and both were learned the hard way:

- A command asking for what the TV is already doing is ignored.
- So is one arriving on the heels of this plugin's own report. Reporting a value makes a controller work
  out the others and send them back, and those arrive looking exactly like somebody pressing something.

## Installation

The package name is unchanged, so this installs over the upstream plugin and your existing configuration
and HomeKit accessories carry on as they were:

```
npm --prefix /var/lib/homebridge install github:lirik44/homebridge-lgtv-webos
```

Then enable Matter for the child bridge this plugin runs in, and add that bridge to your Matter app with
the pairing code Homebridge logs at startup.

## Configuration

Every option from the upstream plugin works as documented there. This fork adds one block per device:

```json
{
  "matter": {
    "enable": true,
    "switchStyle": "outlet",
    "inputs": true,
    "backlight": true,
    "stateSyncSeconds": 60
  }
}
```

## Development

```
npm install
npm test
```

The tests cover the parts that can be checked without a television: what gets published, the scales, and
the identities a bridged accessory carries. Aqara refuses a bridged device whose name runs past 32
characters or whose serial number does, which is a quiet failure worth a test.

## Credits

- [grzegorz914/homebridge-lgwebos-tv](https://github.com/grzegorz914/homebridge-lgwebos-tv) — the plugin
  this fork is based on, and everything that talks to the TV
- [homebridge/homebridge](https://github.com/homebridge/homebridge) — Homebridge, and its Matter support

## License

MIT, same as the upstream project.
