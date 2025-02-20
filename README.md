# Pi NUT

Raspberry Pi NUT ([Network UPS Tools](https://networkupstools.org)) configuration for UPS monitoring and safe server shutdowns.

## Hardware Setup

<p align="center"><img alt="Pi NUT in Rack" src="/resources/nut-pi-rack.jpg" height="auto" width="600"></p>

Because UPSes generally have a single USB or serial port, and the Ethernet port (if your UPS has one) either requires a subscription to activate, or runs wildly outdated software (so it would be a risk to connect it to your network), NUT is useful in allowing a single computer (in my case, a Raspberry Pi) to share the UPS status information with other servers powered by it.

This playbook assumes you're running a Raspberry Pi like I am, connected directly to your NAS via USB, but you could run any computer with Debian or Ubuntu, including a little mini PC or some other SBC.

It's useful to run NUT on a low-power computer though, as it will be the last system to power down, and if it's a server burning 500W at idle, your UPS will likely be on its last legs as your NUT server shuts down!

Anyway, here's my hardware setup:

  - Raspberry Pi CM4 Lite with 1GB of RAM (used because I had one laying on my desk...)
  - [BigTreeTech Pi4B carrier board](https://amzn.to/4gIYGvX)
  - [3D printed Raspberry Pi Rackmount - right hand mount](https://www.printables.com/model/843677-raspberry-pi-5-rack-mount-right-sided) (I printed in ASA)
  - USB-C power supply, USB-A to USB-B cable for UPS, and Ethernet connected to my switch

## Software Setup

This project uses Ansible (`pip install ansible`).

The Ansible playbook depends on the `geerlingguy.nut_client` role, which can be installed with:

```
ansible-galaxy install geerlingguy.nut_client
```

All the servers to be managed by this playbook are listed inside `hosts.yml`. There should be one `primary` server, and then multiple `clients` which will subscribe to the primary NUT server for UPS status.

To set up NUT on all configured hosts, run:

```
ansible-playbook main.yml
```

> **NOTE**: This playbook uses an Ansible Vault-encrypted secret inside `config-secrets.yml` for the `nut_client_password` variable. The `ansible.cfg` file defines the path to my `vault_password_file`—but if you have that on _your_ machine... I'm a little worried! You can either modify the playbook to remove the use of an encrypted secrets file, or you can use `ansible-vault encrypt` to encrypt your own secrets file with the client password inside and use _that_.

## Debugging

After NUT is installed, you can run `upsc` to verify the UPS is being seen correctly:

```
$ upsc server-room-rack
Init SSL without certificate database
battery.charge: 24
battery.energysave: no
battery.packs: 1
battery.protection: yes
battery.runtime: 0
battery.voltage: 50.60
battery.voltage.nominal: 48.0
device.model: LILVX2K0
device.type: ups
driver.name: nutdrv_qx
...
```

You can monitor NUT's logs with:

```
# On server
journalctl -f -u nut-server

# On client nodes
journalctl -f -u nut-monitor
```

You can also manage the attached UPS directly with `upscmd`:

```
# List commands supported on this UPS
upscmd -l server-room-rack

# Run a quick battery test (requires password)
upscmd -u admin server-room-rack test.battery.start.quick
```

To perform a full test of NUT's shutdown without unplugging your UPS and draining the battery (NOTE: This will shut down all servers monitoring your UPS!), run:

```
upsmon -c fsd
```

## License

GPLv3

## Author Information

This project was created in 2025 by [Jeff Geerling](https://www.jeffgeerling.com/).
