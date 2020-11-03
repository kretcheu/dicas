## Como remover o ícone "chatão" que aparece no Firefox quando está numa vídeo conferência.

0. Descubra qual o seu diretório de perfil.

1. No menu do Firefox vá em **Ajuda** -> **Informações para resolver problemas**
   O diretório vai aparecer na linha: **Diretório do perfil**

2. Crie um sub-diretório chamado **chrome**.

3. Nesse diretório que acaba de criar, crie um arquivo chamado **userChrome.css** com o seguinte conteúdo, inclusive o #: **o C é maiúsculo!**

```
#webrtcIndicator { display: none !important; }
```

4. Na linha de endereço do Firefox digite: "about:config" e aceite o alerta de cuidados.

5. No campo de pesquisa digite **userprof**.
   Troque a opção **toolkit.legacyUserProfileCustomizations.stylesheets** para **true**.
   Basta um duplo-clique ou clique no botão de **alternar**.

6. Reinicie o Firefox.

