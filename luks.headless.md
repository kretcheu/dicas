## Luks com cabeçalho separado "headless" e criptografia "on the fly"

Memorial de linhas de comando para usar criptografia Luks com cabeçalho separado (headless).

Obs. O programa cryptsetup vai ser usado, está disponível no pacote do mesmo nome, caso não o tenha instalado, rode:
```
apt install cryptsetup
```

### Para criptografar uma partição e gerar o cabeçalho num arquivo em separado.

```
cryptsetup reencrypt --new --header arquivo_cabecalho /dev/vda2
```

### Obtendo as informaçãos do luks
```
cryptsetup luksDump /dev/vda2
```
Repare que não há nada, pois as informações da criptografia foram para o arquivo arquivo_cabecalho.
```
cryptsetup luksDump arquivo_cabecalho
```

### Abrindo o luks e associando ao dispositivo apelidado de cripto.
```
cryptsetup open /dev/vda2 cripto --header arquivo_cabecalho
```

### Montando na estrutura de diretórios
```
mount /dev/mapper/cripto /mnt
```

### Desmontando e fechando
```
umount /mnt
cryptsetup close cripto
```

### Desencriptando a partição
```
cryptsetup reencrypt --decrypt --header arquivo_cabecalho /dev/vda2
```

