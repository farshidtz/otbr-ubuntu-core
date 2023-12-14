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
ubuntu-image snap model.signed.yaml -v --validation=enforce --snap otbr-gadget_test_amd64.snap
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
