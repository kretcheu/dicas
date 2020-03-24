## Java no Debian 10

### Preparativos

```
su ⁻

echo 'deb http://ppa.launchpad.net/linuxuprising/java/ubuntu bionic main' >> /etc/apt/sources.list

gpg --keyserver keyserver.ubuntu.com --recv-keys EA8CACC073C3DB2A
gpg --export EA8CACC073C3DB2A | apt-key add -

apt update
```

### Agora o Java
```
apt install oracle-java13-installer

java -version
java version "13.0.2" 2020-01-14
Java(TM) SE Runtime Environment (build 13.0.2+8)
Java HotSpot(TM) 64-Bit Server VM (build 13.0.2+8, mixed mode, sharing)

```

