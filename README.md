# OpenThread Border Router Ubuntu Core Image

This project is for building Ubuntu Core images preloaded with OpenThread Border Router Snap.

## Build the gadget
```shell
snapcraft -v
```

The snap doesn't install anything, it only packages files.
It is therefore safe to build it in destructive mode for quicker builds:
`snapcraft -v --destructive-mode`


## Convert and sign the model
```shell
yq eval model.yaml -o=json | snap sign -k otbr-testing > model.signed.yaml
```

## Build the image
```shell
ubuntu-image snap model.signed.yaml -v --validation=enforce \
    --snap otbr-gadget_test_amd64.snap
```

It will be significantly faster and resource-friendly if rebuild the image with local snaps.
We can do this for all the snaps listed in the model assertion, except for those
that have default values seeded to them from the Gadget.
For our case, we need to pull the openthread-border-router from the store,
and during development from a branch.

Download the snaps:
```shell
snap download pc-kernel --channel=22/stable
snap download snapd --channel=latest/stable
snap download core22 --channel=latest/stable
snap download avahi --channel=22/stable
snap download bluez --channel=22/stable
```

Then build by side-loading them:
```shell
ubuntu-image snap model.signed.yaml -v --validation=enforce \
    --snap otbr-gadget_test_amd64.snap \
    --snap pc-kernel_*.snap \
    --snap snapd_*.snap \
    --snap core22_*.snap \
    --snap bluez_*.snap \
    --snap avahi_*.snap
```


## Test with QEMU
```shell
sudo qemu-system-x86_64 \
 -enable-kvm \
 -smp 1 \
 -m 2048 \
 -machine q35 \
 -cpu host \
 -global ICH9-LPC.disable_s3=1 \
 -net nic,model=virtio \
 -net user,hostfwd=tcp::8022-:22,hostfwd=tcp::8090-:80  \
 -drive file=/usr/share/OVMF/OVMF_CODE.secboot.fd,if=pflash,format=raw,unit=0,readonly=on \
 -drive file=/usr/share/OVMF/OVMF_VARS.ms.fd,if=pflash,format=raw,unit=1 \
 -drive "file=pc.img",if=none,format=raw,id=disk1 \
 -device virtio-blk-pci,drive=disk1,bootindex=1 \
 -serial mon:stdio
```

## Test on Intel NUC
The image can be flashed on disk by following the instructions given [here](https://ubuntu.com/download/intel-nuc).

However, for quicker tests, we flash the image and install the OS on a USB flash drive:
1. Use Ubuntu's Startup Disk Creator to flash the `pc.img` on the USB drive
2. Configure Intel NUC to prioritize booting from USB
3. Plug Nordic Semiconductor nRF52840 Dongle inside Intel NUC (RPC used for Thread communication)
4. Plug the USB drive inside Intel NUC and wait for the installation to complete.
5. Follow the Console Conf instructions to configure the network and user account

Sanity check:
```
$ ssh <ubuntu-one-username>@<device-ip>
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-91-generic x86_64)

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

 * Ubuntu Core:     https://www.ubuntu.com/core
 * Community:       https://forum.snapcraft.io
 * Snaps:           https://snapcraft.io

This Ubuntu Core 22 machine is a tiny, transactional edition of Ubuntu,
designed for appliances, firmware and fixed-function VMs.

If all the software you care about is available as snaps, you are in
the right place. If not, you will be more comfortable with classic
deb-based Ubuntu Server or Desktop, where you can mix snaps with
traditional debs. It's a brave new world here in Ubuntu Core!

Please see 'snap --help' for app installation and updates.

farshidtz@ubuntu:~$ snap list
Name                      Version                         Rev    Tracking       Publisher           Notes
avahi                     0.8                             327    22/stable      ondra               -
bluez                     5.64-4                          356    22/stable      canonical✓          -
core22                    20231123                        1033   latest/stable  canonical✓          base
openthread-border-router  thread-reference-20230119+snap  37     latest/edge/…  canonical-iot-labs  -
otbr-gadget               test                            x1     -              -                   gadget
pc-kernel                 5.15.0-91.101.1                 1540   22/stable      canonical✓          kernel
snapd                     2.60.4                          20290  latest/stable  canonical✓          snapd

farshidtz@ubuntu:~$ snap services
Service                              Startup  Current   Notes
avahi.daemon                         enabled  active    -
bluez.bluez                          enabled  active    -
openthread-border-router.otbr-agent  enabled  active    -
openthread-border-router.otbr-setup  enabled  inactive  -
openthread-border-router.otbr-web    enabled  active    -

farshidtz@ubuntu:~$ snap connections openthread-border-router 
Interface          Plug                                        Slot                                 Notes
avahi-control      openthread-border-router:avahi-control      avahi:avahi-control                  gadget
bluetooth-control  openthread-border-router:bluetooth-control  :bluetooth-control                   gadget
bluez              openthread-border-router:bluez              bluez:service                        gadget
dbus               -                                           openthread-border-router:dbus-wpan0  -
firewall-control   openthread-border-router:firewall-control   :firewall-control                    gadget
network            openthread-border-router:network            :network                             -
network-bind       openthread-border-router:network-bind       :network-bind                        -
network-control    openthread-border-router:network-control    :network-control                     gadget
raw-usb            openthread-border-router:raw-usb            :raw-usb                             gadget

farshidtz@ubuntu:~$ snap get openthread-border-router 
Key        Value
autostart  true
infra-if   enp88s0
radio-url  spinel+hdlc+uart:///dev/ttyACM0
thread-if  wpan0
```

# References
- [Testing Ubuntu Core with QEMU](https://ubuntu.com/core/docs/testing-with-qemu)
- [Create a custom Ubuntu Core image](https://ubuntu.com/core/docs/custom-images)
- [EdgeX Ubuntu Core Testing](https://github.com/canonical/edgex-ubuntu-core-testing)
