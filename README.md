# iPhone XR Ramdisk Boot

This script is still rough. PRs for more payloads or device support are welcome. If this project helps you, a Star would be appreciated.

Boot a custom ramdisk on iPhone XR and open a shell through usbmux / SSH.

This project is based on [prdgmshift/usbliter8](https://github.com/prdgmshift/usbliter8). `tools/usbliter8ctl` boots `payload/iBSS.raw` from pwned DFU, then `exploit.sh` sends the firmware, DeviceTree, ramdisk, trustcache, and kernelcache through `irecovery`.

### Target

- Device: iPhone XR
- Board: `n841ap`
- Boot flow: pwned DFU -> iBSS -> Recovery -> ramdisk boot -> SSH

### Hardware

- PR2350-A development board
- iPhone XR
- USB cables
- macOS or Linux host

Before running the script, use PR2350-A with the usbliter8 flow to put the device into pwned DFU.

### Requirements

- `python3`
- Python package: `pyusb`
- `irecovery`
- `iproxy`
- `idevice_id`
- `sshpass`
- `ssh`

macOS + Homebrew:

```bash
brew install libirecovery libimobiledevice usbmuxd sshpass
python3 -m pip install pyusb
```

If global Python package installation is blocked, use a virtual environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
python3 -m pip install pyusb
```

### Files

```text
exploit.sh                 Main boot script
ssh_connect.sh             SSH reconnect script
tools/usbliter8ctl         Local helper based on the usbliter8 USB flow
payload/iBSS.raw           Raw iBSS loaded from pwned DFU
payload/*.img4             Firmware, DeviceTree, ramdisk, trustcache, kernelcache
assets/ssh-ramdisk.png     SSH success screenshot
```

### Usage

1. Install the requirements.

2. Connect the PR2350-A board and iPhone XR.

3. Use the PR2350-A / usbliter8 flow to enter pwned DFU.

4. Run from the project directory:

```bash
chmod +x exploit.sh ssh_connect.sh tools/usbliter8ctl
./exploit.sh
```

The script will:

- create `logs/`
- check required payload files
- boot `payload/iBSS.raw`
- wait for Recovery
- send firmware images
- send DeviceTree, ramdisk, trustcache, and kernelcache
- set boot-args
- run `bootx`
- start local `iproxy 2222 22`
- try to SSH into the ramdisk as `root`

Default SSH password:

```text
alpine
```

Reconnect after boot:

```bash
./ssh_connect.sh
```

Manual connection:

```bash
iproxy 2222 22
ssh root@localhost -p 2222
```

### Environment Overrides

If a tool is not in your PATH:

```bash
IRECOVERY=/path/to/irecovery \
PYTHON=/path/to/python3 \
USBLITER8CTL=tools/usbliter8ctl \
IPROXY=/path/to/iproxy \
SSHPASS=/path/to/sshpass \
./exploit.sh
```

### Troubleshooting

`usbliter8ctl` says the device is not pwned: repeat the PR2350-A pwned DFU step.

Recovery timeout: reconnect USB and run the pwned DFU step again.

SSH does not connect: wait a few seconds, then run:

```bash
./ssh_connect.sh
```

Local port `2222` is already in use:

```bash
pkill -f 'iproxy .*2222.*22'
```

### Credits

Based on [prdgmshift/usbliter8](https://github.com/prdgmshift/usbliter8).

Thanks to the usbliter8 author for publishing the A12/A13 SecureROM exploit research.

### Disclaimer

This repository is for security research, device recovery research, and devices you own or have permission to test. Use it responsibly and follow local laws.
