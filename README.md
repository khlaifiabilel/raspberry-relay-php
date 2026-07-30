# Raspberry Pi Four-Relay Control with PHP

A small PHP page for switching four active-low relay inputs through Raspberry
Pi GPIO. The repository also includes standalone Python GPIO examples and a
Fritzing circuit source.

> **Status:** this is a hardware-specific demonstration from 2020, not a
> hardened remote-control service. It assumes the legacy WiringPi `gpio`
> executable exists at `/usr/local/bin/gpio`; WiringPi is no longer bundled
> with current Raspberry Pi OS releases. No dependency versions, authentication,
> authorization, CSRF protection, or deployment automation are provided.

## Repository contents

| File | Role |
| --- | --- |
| `gpio.php` | Four-relay web UI invoking WiringPi with `shell_exec()` |
| `4relay.py` | Standalone `RPi.GPIO` sequence that turns relays on one by one |
| `1relay_on.py` | Standalone active-low “on” example for BCM GPIO 3 |
| `1relay_off.py` | Standalone active-low “off” example for BCM GPIO 3 |
| `circuit.jpg` | Raspberry Pi 2 and 5 V four-relay wiring diagram |
| `module 4 relay 5v.fzz` | Editable Fritzing project used for the circuit |

The PHP page does **not** call the Python scripts. They are separate ways to
exercise GPIO, and running them concurrently can cause conflicting output
states. The Python examples import `RPi.GPIO`; the web page instead requires
WiringPi's command-line program.

## GPIO mapping

Both implementations use Broadcom (BCM) GPIO numbering. In `gpio.php`, a low
output is labelled **on** and a high output is labelled **off**, matching the
active-low relay module represented by the diagram.

| Relay | BCM GPIO | Raspberry Pi 40-pin header |
| --- | --- | --- |
| 1 | GPIO 2 | Physical pin 3 |
| 2 | GPIO 3 | Physical pin 5 |
| 3 | GPIO 4 | Physical pin 7 |
| 4 | GPIO 27 | Physical pin 13 |

The diagram connects relay-module VCC to the Pi's 5 V rail and GND to Pi ground.
It is specifically labelled for a Raspberry Pi 2 and a 5 V four-relay module.
Confirm the input polarity, logic-level compatibility, current requirement,
pinout, and power arrangement of the exact relay board and Pi revision before
connecting it; relay boards with similar appearance are not necessarily wired
or triggered the same way.

## Software and setup

The checked-in web path requires:

- Raspberry Pi OS or another Pi-compatible Linux installation
- A web server configured to execute PHP
- PHP with `shell_exec()` enabled
- A WiringPi-compatible `gpio` executable at `/usr/local/bin/gpio`
- Web-server OS permissions to control the listed GPIO lines

Place the project where the configured web server can serve `gpio.php`, then
open that page from a trusted host on the same network. The code itself contains
no install command and no alternative path for current GPIO tools, so this
repository does not provide a verified setup for modern Raspberry Pi OS.

The Python examples require Python and `RPi.GPIO`. They are direct scripts, not
web-page helpers. `4relay.py` first drives all four outputs high, then drives
GPIO 2, 3, 4, and 27 low at two-second intervals before calling
`GPIO.cleanup()`. The one-relay examples affect GPIO 3 only and call cleanup
only when interrupted.

## Electrical safety

- Disconnect all power while wiring or changing connections.
- Do not connect a load to the relay contacts until the low-voltage GPIO side
  has been verified independently.
- Keep relay contact wiring electrically isolated from Pi GPIO wiring as
  required by the relay module and load.
- Mains voltage can cause fire, injury, or death. The repository provides no
  enclosure, fusing, earthing, conductor sizing, clearance, or mains-wiring
  design. Use a qualified electrician for mains-connected loads.
- Do not assume the relay contact ratings printed in the diagram make a load or
  installation safe; switching rating depends on load type and conditions.

## Network safety

`gpio.php` changes physical outputs in response to unauthenticated GET
parameters. Do not expose it directly to the public Internet or configure
router port forwarding to it. Restrict access at the web server and network
layers, use authentication and HTTPS appropriate to the deployment, and treat
any host that can reach the page as able to operate the connected relays.

## Known limitations

- The page reports the requested command, not measured relay or load state.
- Relay state is not persisted or synchronized between PHP and Python.
- The web request sets all four GPIO directions on every page load.
- The unused temperature fragment in `gpio.php` references an Adafruit DHT
  example and GPIO 26, but there is no matching form control or complete,
  working temperature feature. No temperature sensor is required for relay
  control.
- No license file is present in this repository.
