# FirawSelector — mock guiado

Simulação de um desktop Windows 11 que percorre, em oito passos, o caminho inteiro do [FirawSelector](https://github.com/firawynix/firaw-selector): instalar, virar o navegador padrão, receber um link, escolher na hora, lembrar por site, criar regra e mandar o link para o perfil certo do Chrome.

Espelha o mock do [FolderPin](https://folderpin.firawynix.com.br) — mesmo palco, mesmo painel-guia, mesma navegação.

## O que é

Arquivo único, `index.html`, sem dependência além das fontes do Google. Nada de build, nada de framework.

- **palco 1280×760** com o desktop simulado, escalado por JS para caber em qualquer largura
- no celular a caixa em volta recebe o tamanho **já escalado**: sem ela o palco continuaria ocupando 1280×760 no layout e sobraria uma tarja preta gigante em volta. O botão **Ampliar** dobra a escala e a área rola de lado
- **painel do guia** à direita: os oito passos; clicar num passo salta para ele
- **rodapé sobreposto** com os links de download, do repositório e do Apoiar
- avança pelo botão, pelas setas do teclado ou clicando no ponto destacado do próprio desktop

Duas paletas convivem de propósito:

- **ciano `#22d3ee`** é o produto — as janelas do FirawSelector no palco usam a barra de título própria, com o risco ciano embaixo, igual ao programa de verdade
- **âmbar** é o narrador. Fica fora da paleta do Windows **e** fora da do produto, para o leitor nunca confundir o guia com a interface simulada

## Como editar

Abra o `index.html` no navegador. Os passos vivem no array `steps` do script:

```js
{
  t: "título do passo",
  d: "texto do painel (aceita HTML)",
  call: "balão sobre o desktop",
  at: ["#seletor-do-alvo", folga],
  pos: "abaixo",   // opcional: força o balão para baixo do alvo
  al: "dir"        // opcional: encosta o balão na direita do alvo
}
```

O que cada passo mostra no desktop fica na função `render()`, comparando o índice do passo atual.

**Armadilha já paga:** esconder uma página do Studio com `el.hidden` não basta — a classe `.st-page` tem `display:grid`, que ganha da regra `[hidden]{display:none}` do navegador. Sem a linha `.st-page[hidden]{display:none}` as duas páginas empilham e a de baixo aparece por cima. Se criar página nova, ela precisa da mesma regra.

## Publicar

nginx com bind mount: atualizar é só copiar o arquivo — sem rebuild, sem restart.

```bash
scp index.html srv1:/tmp/index.html
ssh srv1 'sudo install -o root -g root -m 644 /tmp/index.html /home/ksdev/firawselector-mock/public/index.html'
```

Primeira subida:

```bash
ssh srv1 'mkdir -p /home/ksdev/firawselector-mock/public'
scp -r deploy/* srv1:/home/ksdev/firawselector-mock/
scp index.html srv1:/home/ksdev/firawselector-mock/public/
ssh srv1 'cd /home/ksdev/firawselector-mock && docker compose up -d'
```

**Antes de subir, confira se a porta está livre** (`ss -ltnp | grep 26003`) — o padrão é `26003` e o `.env`/`PORTA_SITE` troca.

O domínio entra pelo **Cloudflare Tunnel**, não pelo apache: acrescente em `/etc/cloudflared/config.yml`

```yaml
  - hostname: firawselector.firawynix.com.br
    service: http://localhost:26003
```

e valide **antes** de reiniciar: `cloudflared tunnel ingress validate`.
