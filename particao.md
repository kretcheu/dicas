## Tutorial para ver a simplicidade das partições.

### Criando um disco virtual de 48MB.
```
dd if=/dev/zero of=disco-virtual bs=16M count=4
```
Assim é criado um arquivo de 48Mb para ser o disco.

### Para ver o resultado:

```
hexdump -C disco-virtual
```
O resultado será:

```
hexdump -C disco-virtual
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
04000000
```

Ou seja todas as linhas aparecem 00 e o * representa que uma linha igual se repete até o último byte do disco (arquivo).

### Agora vamos criar duas partições e um particionamento DOS/MBR

Vamos usar o sfdisk que receberá esse texto indicando o que deve fazer.

```
echo 'label: dos
label-id: 0x5b5cd99e
device: disco-virtual
unit: sectors
sector-size: 512
disco-virtual1 : start=        2048, size=       40960, type=83
disco-virtual2 : start=       43008, size=       88064, type=83
' | sfdisk disco-virtual

```

### Agora veja o resultado com fdisk

```
fdisk -l disco-virtual
```

Será:

```
Disk disco-virtual: 64 MiB, 67108864 bytes, 131072 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x5b5cd99e

Device         Boot Start    End Sectors Size Id Type
disco-virtual1       2048  43007   40960  20M 83 Linux
disco-virtual2      43008 131071   88064  43M 83 Linux
```

### Agora veja o que foi alterado

```
hexdump -C disco-virtual
```

O resultado será:

```
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
000001b0  00 00 00 00 00 00 00 00  9e d9 5c 5b 00 00 00 20  |..........\[... |
000001c0  21 00 83 ac 2a 02 00 08  00 00 00 a0 00 00 00 ac  |!...*...........|
000001d0  2b 02 83 28 20 08 00 a8  00 00 00 58 01 00 00 00  |+..( ......X....|
000001e0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000001f0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 55 aa  |..............U.|
00000200  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
```

Veja que apenas poucos bytes tiveram seu valor alterado.

### Desafio

Agora vamos criar um outro disco e o particionamento GPT e não DOS/MBR

- Criando o disco

```
dd if=/dev/zero of=disco-virtual-gpt bs=16M count=4
```

- Já sabe ver o resultado né!?

```
hexdump -C disco-virtual-gpt
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
04000000

```

Tudo 00 como esperado.

- Agora vamos criar o particionamento GPT e as partições.

```
echo 'label: gpt
label-id: A7D32928-917E-A34F-845F-14C7A3D9325D
device: disco-virtual-gpt
unit: sectors
first-lba: 2048
last-lba: 131038
sector-size: 512

disco-virtual-gpt1 : start=        2048, size=       40960, type=0FC63DAF-8483-4772-8E79-3D69D8477DE4, uuid=350146E8-C0A5-0A40-9710-71901157007C
disco-virtual-gpt2 : start=       43008, size=       86016, type=0FC63DAF-8483-4772-8E79-3D69D8477DE4, uuid=B03AEC34-9740-DB44-A0D2-2EBE8685272C
' | sfdisk disco-virtual-gpt

```

- Vendo o resultado com fdisk.

```
fdisk -l disco-virtual-gpt
```

O resultado é:
```
fdisk -l disco-virtual-gpt
Disk disco-virtual-gpt: 64 MiB, 67108864 bytes, 131072 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: A7D32928-917E-A34F-845F-14C7A3D9325D

Device             Start    End Sectors Size Type
disco-virtual-gpt1  2048  43007   40960  20M Linux filesystem
disco-virtual-gpt2 43008 129023   86016  42M Linux filesystem

```

- Agora com o hexdump

```
hexdump -C disco-virtual-gpt
```

o resultado será o seguinte, repare que o início e o final do disco recebem uma cópia da tabela de partições, no início é a tabela primária e no final a de backup ou secundária.

```
hexdump -C disco-virtual-gpt
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
000001c0  02 00 ee ff ff ff 01 00  00 00 ff ff 01 00 00 00  |................|
000001d0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
000001f0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 55 aa  |..............U.|
00000200  45 46 49 20 50 41 52 54  00 00 01 00 5c 00 00 00  |EFI PART....\...|
00000210  63 a6 b2 a2 00 00 00 00  01 00 00 00 00 00 00 00  |c...............|
00000220  ff ff 01 00 00 00 00 00  00 08 00 00 00 00 00 00  |................|
00000230  de ff 01 00 00 00 00 00  28 29 d3 a7 7e 91 4f a3  |........()..~.O.|
00000240  84 5f 14 c7 a3 d9 32 5d  02 00 00 00 00 00 00 00  |._....2]........|
00000250  80 00 00 00 80 00 00 00  cb 42 9d 2c 00 00 00 00  |.........B.,....|
00000260  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
00000400  af 3d c6 0f 83 84 72 47  8e 79 3d 69 d8 47 7d e4  |.=....rG.y=i.G}.|
00000410  e8 46 01 35 a5 c0 40 0a  97 10 71 90 11 57 00 7c  |.F.5..@...q..W.||
00000420  00 08 00 00 00 00 00 00  ff a7 00 00 00 00 00 00  |................|
00000430  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
00000480  af 3d c6 0f 83 84 72 47  8e 79 3d 69 d8 47 7d e4  |.=....rG.y=i.G}.|
00000490  34 ec 3a b0 40 97 44 db  a0 d2 2e be 86 85 27 2c  |4.:.@.D.......',|
000004a0  00 a8 00 00 00 00 00 00  ff f7 01 00 00 00 00 00  |................|
000004b0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
03ffbe00  af 3d c6 0f 83 84 72 47  8e 79 3d 69 d8 47 7d e4  |.=....rG.y=i.G}.|
03ffbe10  e8 46 01 35 a5 c0 40 0a  97 10 71 90 11 57 00 7c  |.F.5..@...q..W.||
03ffbe20  00 08 00 00 00 00 00 00  ff a7 00 00 00 00 00 00  |................|
03ffbe30  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
03ffbe80  af 3d c6 0f 83 84 72 47  8e 79 3d 69 d8 47 7d e4  |.=....rG.y=i.G}.|
03ffbe90  34 ec 3a b0 40 97 44 db  a0 d2 2e be 86 85 27 2c  |4.:.@.D.......',|
03ffbea0  00 a8 00 00 00 00 00 00  ff f7 01 00 00 00 00 00  |................|
03ffbeb0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
03fffe00  45 46 49 20 50 41 52 54  00 00 01 00 5c 00 00 00  |EFI PART....\...|
03fffe10  a6 6a be 2c 00 00 00 00  ff ff 01 00 00 00 00 00  |.j.,............|
03fffe20  01 00 00 00 00 00 00 00  00 08 00 00 00 00 00 00  |................|
03fffe30  de ff 01 00 00 00 00 00  28 29 d3 a7 7e 91 4f a3  |........()..~.O.|
03fffe40  84 5f 14 c7 a3 d9 32 5d  df ff 01 00 00 00 00 00  |._....2]........|
03fffe50  80 00 00 00 80 00 00 00  cb 42 9d 2c 00 00 00 00  |.........B.,....|
03fffe60  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
04000000

```

Nota importantes:

Você pode fazer o mesmo usando um disco real ou um pendrive, basta trocar o **disco-virtual** por **/dev/sdb** ou **/dev/sdc** dependendo do caso, **CUIDADO**, não vá escrever um tabela no seu disco principal!!

Ou se fizer isso não reclame comigo!

Pode fazer um backup da tabela do seu disco:
```
sfdisk -d /dev/sda > tabela.sda
```

Veja o que foi escrito nesse backup:

```
cat tabela.sda
```

Já dá até para adivinhar como seria restaurar:

```
sfdisk /dev/sda < tabela.sda
```

## Desafio 2

Dá para fazer vários outros experimentos, eu sugiro você verificar com hexdump quando criar um sistema de arquivos na partição.

1. criar o disco virtual.
```
dd if=/dev/zero of=disco-virtual bs=16M count=4
```

2. criar o particionamento e as partições.
```
echo 'label: dos
label-id: 0x5b5cd99e
device: disco-virtual
unit: sectors
sector-size: 512
disco-virtual1 : start=        2048, size=       40960, type=83
disco-virtual2 : start=       43008, size=       88064, type=83
' | sfdisk disco-virtual
```

3. ligar as partições a dispositivos loopback.

```
loseup -P /dev/loop1 disco-virtual
```

Veja com o fdisk:

```
fdisk -l /dev/loop1
```

4. criar o sistema de arquivos.
```
mkfs.ext4 /dev/loop1p1
```

5. ver o resultado.

```
hexdump -C /dev/loop1p1
```

6. desligar as partições do dispositivo loopback.
```
losetup -d /dev/loop1
```

## Conclusão

Agora use a sua criatividade e faça experimentos para entender ainda mais o que ocorre no misterioso mundo das partições!! hehehe.


