# Máquinas com dual boot e RST

O recurso RST ainda incompatível com kernel Linux.
Para desabilitar numa máquina já com windows instalada para preparar um dual boot.

## No Windows:

- Clique no botão iniciar e digite `cmd`.
- Clique com botão direito e selecione `Run as administrator` ou `Rodar como administrador`
- Digite o seguinte comando e tecle ENTER:
  `bcdedit /set safeboot minimal`
  ou
  `bcdedit /set safeboot {current} minimal`

- Reinicie e acesse o setup da BIOS.
- Troque "SATA Operation mode" para AHCI tanto para IDE ou RAID
- Salve as alterações e saia do setup da BIOS.

- O windows irá dar boot no modo "safe", se é que pode ser seguro usar o windows!!

- Clique com botão direito no menu iniciar e escolha "Prompt de comandos como (ADMIN)"
- Digite o seguinte comando e tecle ENTER:
  `bcdedit /deletevalue safeboot`
  ou
  `bcdedit /deletevalue {current} safeboot`

- Feito, o windows já será capaz de funcionar com AHCI e vc poderá fazer o dual boot.

Dica que foi necessária para ajudar Saulo, aluno do curso que gostaria de se libertar usando Debian! Finalmente pode fazer isso!


