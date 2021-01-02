### Fazendo DNAT para um serviço de uma VM ser acessado da rede local

- IP Host Real: 192.168.15.9
- Porta do Dnat --dport 80

- IP VM: 192.168.100.134
- Porta do serviço rodando na VM :80

Dessa forma ao acessar a porta 80 do host a partir da rede local acessará o serviço na VM escutando na porta 80

```
iptables -t nat -I PREROUTING -p tcp -d 192.168.15.9 --dport 80 -j DNAT --to-destination 192.168.100.134:80
iptables -I FORWARD -m state -d 192.168.100.0/24 --state NEW,RELATED,ESTABLISHED -j ACCEPT
```


