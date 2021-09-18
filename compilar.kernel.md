# Compilar kernel
Esse tutorial trás o básico e elementar sobre compilar kernel no Debian.


A título de comparação vamos analisar algumas informações.

### Quantidade de memória ocupada

```
free
              total        used        free      shared  buff/cache   available
Mem:         988080      131796      600900       12780      255384      703208
Swap:       3145724           0     3145724
```

### Quantidade módulos carregados
```
lsmod |wc -l
70

ls /sys/module/

8250               blk_cgroup        dns_resolver         glue_helper          iTCO_wdt       lpc_ich        ppp_generic  rtc_cmos               spurious      usbhid              xen_blkfront
acpi               block             drm                  gpiolib_acpi         joydev         mac_hid        printk       scsi_mod               srcutree      vfio                xen_netfront
acpi_cpufreq       button            drm_kms_helper       haltpoll             kdb            md_mod         processor    serio_raw              sr_mod        vfio_iommu_type1    xhci_hcd
acpiphp            cec               dynamic_debug        hid                  kernel         module         psmouse      sg                     suspend       vfio_pci            xor
aesni_intel        configfs          edac_core            hid_generic          keyboard       mousedev       pstore       shpchp                 syscopyarea   vfio_virqfd         x_tables
ahci               cpufreq           edd                  i2c_i801             kgdb_nmi       multipath      qemu_fw_cfg  slab_common            sysfillrect   virtio_blk          xz_dec
apparmor           cpuidle           efivars              i8042                kgdboc         net_failover   raid0        snd                    sysimgblt     virtio_gpu          zswap
async_memcpy       crc32_pclmul      ehci_hcd             ima                  kvm            netpoll        raid1        snd_hda_codec          sysrq         virtio_mmio
async_pq           crc_t10dif        eisa_bus             input_leds           kvm_intel      nls_iso8859_1  raid10       snd_hda_codec_generic  tcp_cubic     virtio_net
async_raid6_recov  crct10dif_pclmul  failover             intel_idle           ledtrig_audio  page_alloc     raid456      snd_hda_core           thermal       virtio_pci
async_tx           cryptd            fb                   intel_pmc_core       libahci        pata_sis       raid6_pq     snd_hda_intel          tpm           virtio_rng
async_xor          cryptomgr         fb_sys_fops          intel_rapl_common    libata         pcc_cpufreq    random       snd_hwdep              tpm_crb       virtual_root
ata_generic        crypto_simd       firmware_class       intel_rapl_msr       libcrc32c      pcie_aspm      rcupdate     snd_intel_dspcfg       tpm_tis       vt
ata_piix           debug_core        fscrypto             ip_tables            libnvdimm      pciehp         rcutree      snd_pcm                tpm_tis_core  watchdog
autofs4            dm_crypt          fuse                 ipv6                 linear         pci_hotplug    rfkill       snd_timer              uhci_hcd      workqueue
battery            dm_mod            ghash_clmulni_intel  iTCO_vendor_support  loop           pcspkr         rng_core     soundcore              usbcore       xen_acpi_processor

lsmod

Module                  Size  Used by
intel_rapl_msr         20480  0
intel_rapl_common      28672  1 intel_rapl_msr
kvm_intel             286720  0
kvm                   679936  1 kvm_intel
crct10dif_pclmul       16384  1
crc32_pclmul           16384  0
ghash_clmulni_intel    16384  0
snd_hda_codec_generic    81920  1
ledtrig_audio          16384  1 snd_hda_codec_generic
snd_hda_intel          53248  1
snd_intel_dspcfg       24576  1 snd_hda_intel
input_leds             16384  0
snd_hda_codec         139264  2 snd_hda_codec_generic,snd_hda_intel
pcspkr                 16384  0
serio_raw              20480  0
virtio_gpu             53248  1
snd_hda_core           94208  3 snd_hda_codec_generic,snd_hda_intel,snd_hda_codec
snd_hwdep              20480  1 snd_hda_codec
joydev                 24576  0
drm_kms_helper        200704  3 virtio_gpu
snd_pcm               114688  3 snd_hda_intel,snd_hda_codec,snd_hda_core
snd_timer              40960  1 snd_pcm
nls_iso8859_1          16384  1
snd                    94208  8 snd_hda_codec_generic,snd_hwdep,snd_hda_intel,snd_hda_codec,snd_timer,snd_pcm
iTCO_wdt               16384  0
soundcore              16384  1 snd
iTCO_vendor_support    16384  1 iTCO_wdt
drm                   528384  4 drm_kms_helper,virtio_gpu
fb_sys_fops            16384  1 drm_kms_helper
syscopyarea            16384  1 drm_kms_helper
sysfillrect            16384  1 drm_kms_helper
sysimgblt              16384  1 drm_kms_helper
qemu_fw_cfg            20480  0
mac_hid                16384  0
aesni_intel           372736  2
crypto_simd            16384  1 aesni_intel
cryptd                 24576  3 crypto_simd,ghash_clmulni_intel
glue_helper            16384  1 aesni_intel
dm_crypt               40960  1
virtio_rng             16384  0
ip_tables              32768  0
x_tables               45056  1 ip_tables
autofs4                45056  2
raid10                 57344  0
raid456               159744  0
async_raid6_recov      24576  1 raid456
async_memcpy           20480  2 raid456,async_raid6_recov
async_pq               24576  2 raid456,async_raid6_recov
async_xor              20480  3 async_pq,raid456,async_raid6_recov
async_tx               20480  5 async_pq,async_memcpy,async_xor,raid456,async_raid6_recov
xor                    24576  1 async_xor
raid6_pq              114688  3 async_pq,raid456,async_raid6_recov
libcrc32c              16384  1 raid456
raid1                  45056  0
raid0                  24576  0
multipath              20480  0
linear                 20480  0
hid_generic            16384  0
usbhid                 57344  0
hid                   135168  2 usbhid,hid_generic
psmouse               155648  0
ahci                   40960  0
libahci                36864  1 ahci
i2c_i801               32768  0
lpc_ich                24576  0
virtio_net             53248  0
virtio_blk             20480  3
net_failover           20480  1 virtio_net
failover               16384  1 net_failover
```

### Pacotes necessários
Para poder compilar o kernel é necessário ter os seguintes pacotes instalados.
```
apt install build-essential bc kmod cpio flex liblz4-tool lz4 libncurses-dev libelf-dev libssl-dev rsync dwarves
```

### Baixando e desempacotando os Fontes
Agora vamos baixar os fontes do Linux e desempacotar.
```
apt install linux-source
cd /usr/src

tar -xvf linux-source-4.19.tar.xz
```
A versão do fonte poderá ser mais recente no momento que lê esse tutorial.\
Rode o ls para saber qual é.
```
ls
```

### Tornando o diretório dos fontes o diretório corrente
```
cd linux-source-4.19
```

### Preparando um arquivo de configuração
Para compilar o kernel um arquivo de configuração (.config) é usado para definir:

 - Quais módulos serão compilados embutidos no kernel.
 - Quais módulos serão compilados separadamente.
 - Quais não serão compilados.

Para partir do mesmo do kernel atual
```
cp /boot/config-4.19.0-8-amd64 .config
```

Para criar um **.config** baseado nos módulos atualemnte carregados
```
make localmodconfig
```

Para alterar essas condições iniciais
```
make menuconfig
ou
make nconfig
```

### Comparando a quantidade de módulos
Comparando os módulos compilados do kernel atual com o que iremos compilar.

####  compilados separados
```
grep m$ /boot/config-4.19.0-8-amd64 |wc -l
3385

grep m$ .config|wc -l
76
```

#### compilados embutidos (bultin)
```
grep y$ /boot/config-4.19.0-8-amd64|wc -l
2014

grep y$ .config|wc -l
1692
```

### Compilando e construindo o pacote deb
Vou usar *time* para contar o tempo de compilação.
use -jX para usar X processadores.

```
time make -j4 deb-pkg

dpkg-deb: a compilar o pacote 'linux-headers-4.19.98' em '../linux-headers-4.19.98_4.19.98-1_amd64.deb'.
dpkg-deb: a compilar o pacote 'linux-libc-dev' em '../linux-libc-dev_4.19.98-1_amd64.deb'.
dpkg-deb: a compilar o pacote 'linux-image-4.19.98' em '../linux-image-4.19.98_4.19.98-1_amd64.deb'.

real    25m25,795s
user    22m19,842s
sys     2m54,491s
```

### Para instalar o pacote do kernel compilado nessa máquina use:
```
dpkg -i ../linux-image-4.19.98_4.19.98-1_amd64.deb
```

### Comparando o resultado
Pacote original 4.19.0-8 compilado 4.19.98
```
ls -lh /boot/ |grep '4.19'

-rw-r--r-- 1 root root 202K jan 26 17:01 config-4.19.0-8-amd64
-rw-r--r-- 1 root root 138K mar 26 02:56 config-4.19.98

-rw-r--r-- 1 root root  29M mar 23 19:55 initrd.img-4.19.0-8-amd64
-rw-r--r-- 1 root root  11M mar 26 03:32 initrd.img-4.19.98

-rw-r--r-- 1 root root 3,3M jan 26 17:01 System.map-4.19.0-8-amd64
-rw-r--r-- 1 root root 4,2M mar 26 02:56 System.map-4.19.98

-rw-r--r-- 1 root root 5,1M jan 26 17:01 vmlinuz-4.19.0-8-amd64
-rw-r--r-- 1 root root  11M mar 26 02:56 vmlinuz-4.19.98
```

Espaço ocupado pelos móduloes
```
du -shc /lib/modules/*

260M	/lib/modules/4.19.0-8-amd64
6,4M	/lib/modules/4.19.98
```


