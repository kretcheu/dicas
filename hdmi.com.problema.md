## Dica enviado por Adfeno
https://libreplanet.org/wiki/User:Adfeno#vCard


Esta mensagem não é uma dúvida, mas uma dica.

Descobri que as entradas HDMI, além de terem o problema do EDID corrompido/variável, também podem se negar a funcionar ou funcionar parcialmente (só áudio ou só vídeo) dependendo dos códecs da mídia usada/reproduzida.

Um dos benefícios de usar o GNU Bash é que ele é um dos vários que oferece uma funcionalidade de “substituição de processo” (“process substitution”), onde usas um sinal de menor que ou maior que seguido de um comando dentro de um par de parenteses. No caso de comando1 <(comando2), cria-se arquivo temporário com o resultado de comando2 que se permite apenas leitura por parte de comando1 e o arquivo é excluído assim que inutilizado.
Já para comando1 >(comando2), permite-se escrita por parte de comando1 a comando2, mas o arquivo também é excluído depois.

Se tiveres problema com reprodução de mídia pela entrada HDMI, podes usar algo como:

reprodutor --opcao-de-ler-audio-ou-video-de-outro-arquivo=<(ffmpeg -i video-com-problema.webm -c codec-aceito-pela-entrada-HDMI -f formato-leve-aceito-pela-entrada-HDMI pipe:1) video-com-problema.webm

Isso deve reproduzir video-com-problema.webm em sincronia com o audio/video extraido dele usando ffmpeg e recodificado em tempo real.

