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


# References
- [Testing Ubuntu Core with QEMU](https://ubuntu.com/core/docs/testing-with-qemu)
- [Create a custom Ubuntu Core image](https://ubuntu.com/core/docs/custom-images)
- [EdgeX Ubuntu Core Testing](https://github.com/canonical/edgex-ubuntu-core-testing)
