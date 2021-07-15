## Dando boot em ISO do Debian via console do GRUB

Imagem ISO: debian-testing-amd4-netinst.iso
Está no primeiro disco, primeira partição

```
set isofile=/debian-testing-amd4-netinst.iso
loopback loop (hd0,msdos1)/debian-testing-amd4-netinst.iso
linux (loop)/install.amd/vmlinuz boot=live findiso=$isofile
initrd (loop)/install.amd/initrd.gz
boot
```

Imagem ISO: debian-live-10.10.0-amd64-standard.iso
Está no primeiro disco, primeira partição.

```
set isofile=/debian-live-10.10.0-amd64-standard.iso
loopback loop (hd0,msdos1)/debian-live-10.10.0-amd64-standard.iso
linux (loop)/live/vmlinuz-4.19.0-17-amd64 boot=live findiso=$isofile
initrd (loop)/live/initrd.img-4.19.0-17-amd64
boot
```



