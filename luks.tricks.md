# Lucks Tricks

## Como está
os blocos:
```
lsblk

NAME      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
vda       254:0    0    7G  0 disk
├─vda1    254:1    0   50M  0 part  /boot/grub
└─vda2    254:2    0    7G  0 part
  └─croot 253:0    0    7G  0 crypt /
```
Os pontos de monatgem:
```
mount | grep mapper
/dev/mapper/croot on / type ext4 (rw,relatime)
```

Os Slots:
```
cryptsetup luksDump /dev/vda2
LUKS header information for /dev/vda2

Version:       	1
Cipher name:   	aes
Cipher mode:   	xts-plain64
Hash spec:     	sha256
Payload offset:	4096
MK bits:       	512
MK digest:     	9e f8 5d 17 2e cf 4e 18 91 db e7 e1 79 20 b6 9c 37 84 c2 0e
MK salt:       	86 a1 dd 8e 94 10 2f af 84 8a 48 ce e7 22 95 31
               	41 60 e0 a8 96 56 a2 0c ca 9f 45 96 49 6e c8 e0
MK iterations: 	73306
UUID:          	a50a9263-bdd5-4f8d-941d-0b6c883e1bcf

Key Slot 0: ENABLED
	Iterations:         	1168980
	Salt:               	e0 c0 9c 8a f0 28 18 57 7a 52 63 c1 ec bd 66 a6
	                      	35 67 7c 59 13 dd 90 fa 44 e5 18 67 58 b2 1f 7e
	Key material offset:	8
	AF stripes:            	4000
Key Slot 1: ENABLED
	Iterations:         	1021008
	Salt:               	6e 7d 90 b2 fa db c0 2f 65 f2 74 ae 94 4d a9 62
	                      	76 a4 ae b1 42 05 f7 09 90 08 0f 09 57 8d f5 3f
	Key material offset:	512
	AF stripes:            	4000
Key Slot 2: DISABLED
Key Slot 3: DISABLED
Key Slot 4: DISABLED
Key Slot 5: DISABLED
Key Slot 6: DISABLED
Key Slot 7: DISABLED
```
Os MAPs
```
dmsetup ls --target crypt
croot	(253, 0)
```

## Qual dispositivo tem luks
```
blkid -t TYPE=crypto_LUKS -o device
```

## Extraindo a master-key
```
dmsetup table --showkey /dev/mapper/croot
0 14571520 crypt aes-xts-plain64 06259064d6c546edec23543a1ce57d4fa3ade111b3f59dae45f85a07eb0a8f7e06c0e3d97c6335be571a54b5466d14412b7baecdab8d1511c535d7376081cbd9 0 254:2 4096 1 allow_discards
```

## Usando a master-key para colocar outra chave

cryptsetup luksAddKey /dev/vda2 --master-key-file <(dmsetup table --showkey /dev/mapper/croot | awk '{print$5}' | xxd -r -p)

## Como ficou
```
cryptsetup luksDump /dev/vda2
LUKS header information for /dev/vda2

Version:       	1
Cipher name:   	aes
Cipher mode:   	xts-plain64
Hash spec:     	sha256
Payload offset:	4096
MK bits:       	512
MK digest:     	9e f8 5d 17 2e cf 4e 18 91 db e7 e1 79 20 b6 9c 37 84 c2 0e
MK salt:       	86 a1 dd 8e 94 10 2f af 84 8a 48 ce e7 22 95 31
               	41 60 e0 a8 96 56 a2 0c ca 9f 45 96 49 6e c8 e0
MK iterations: 	73306
UUID:          	a50a9263-bdd5-4f8d-941d-0b6c883e1bcf

Key Slot 0: ENABLED
	Iterations:         	1168980
	Salt:               	e0 c0 9c 8a f0 28 18 57 7a 52 63 c1 ec bd 66 a6
	                      	35 67 7c 59 13 dd 90 fa 44 e5 18 67 58 b2 1f 7e
	Key material offset:	8
	AF stripes:            	4000
Key Slot 1: ENABLED
	Iterations:         	1021008
	Salt:               	6e 7d 90 b2 fa db c0 2f 65 f2 74 ae 94 4d a9 62
	                      	76 a4 ae b1 42 05 f7 09 90 08 0f 09 57 8d f5 3f
	Key material offset:	512
	AF stripes:            	4000
Key Slot 2: ENABLED
	Iterations:         	1162500
	Salt:               	78 40 28 3d 7d 1c 5b f8 ec 83 0c e1 88 17 4a 4d
	                      	3b 38 6c 0c 68 be 20 59 74 7e d5 a8 f5 6e ae bb
	Key material offset:	1016
	AF stripes:            	4000
Key Slot 3: DISABLED
Key Slot 4: DISABLED
Key Slot 5: DISABLED
Key Slot 6: DISABLED
Key Slot 7: DISABLED
```

## Para ver os slots
```
cryptsetup luksDump /dev/vda2 | grep Key.Slot

Key Slot 0: ENABLED
Key Slot 1: ENABLED
Key Slot 2: ENABLED
Key Slot 3: DISABLED
Key Slot 4: DISABLED
Key Slot 5: DISABLED
Key Slot 6: DISABLED
Key Slot 7: DISABLED
```

## Para adicionar uma nova chave digtando passphrase
```
cryptsetup luksAddKey /dev/vda2

Enter any existing passphrase: Existing passphrase which can be used to open DEV
Enter new passphrase for key slot: New passphrase to add to DEV
Adding a key file to an existing LUKS volume:
```

## Para adicionar uma nova chave de um arquivo
```
dd if=/dev/random bs=32 count=1 of=crypto-keyfile.bin
cryptsetup luksAddKey /dev/vda2 cripto-keyfile.bin
```

## Salvando a chave criptografando com gpg
```
dmsetup table --showkey croot | awk '{print $5}'
06259064d6c546edec23543a1ce57d4fa3ade111b3f59dae45f85a07eb0a8f7e06c0e3d97c6335be571a54b5466d14412b7baecdab8d1511c535d7376081cbd9
```

## Criptografando para um arquivo:
```
dmsetup table --showkey croot | awk '{print $5}' | gpg -aco masterkey.gpg --force-mdc --cipher-algo aes256 --pinentry-mode loopback
```

## Decriptografando para usar:
```
gpg -d masterkey.gpg

cryptsetup luksAddKey /dev/vda2 --master-key-file <(gpg -d masterkey.gpg | xxd -r -p)
```

