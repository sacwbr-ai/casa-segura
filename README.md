# Casa Segura

Dois artefatos neste projeto:

- **`jogo.html`** — o jogo completo: *Caça aos Riscos*, rodada de 7 dias (ver abaixo)
- **`index.html`** — a enquete de validação com 3 mini-jogos (histórico do projeto)

## O jogo: Caça aos Riscos (rodada de 7 dias)

Uma casa térrea com **35 perigos de queda** distribuídos em 7 ambientes — sala,
cozinha, quarto, banheiro, corredor, área de serviço e jardim. A cada dia da
rodada abre um ambiente novo (`DATA_INICIO` na configuração do arquivo).

**A ronda é única por ambiente** (estilo Wordle: sem replay, sem decorar posição):

| Regra | Valor |
|---|---|
| Tentativas por ambiente | 1 (rondas atrasadas podem ser feitas depois, uma vez) |
| Tempo de busca | 90 s (pausa durante leituras e perguntas) |
| Fichas de inspeção | 8 por ambiente — cada toque em objeto gasta 1 |
| Perigo visível | +10 (4 por ambiente) |
| Risco "invisível" (ausência/situação) | +20 (1 por ambiente) |
| Itens seguros (distratores) | 4–5 por ambiente: gastam ficha e ensinam por quê |
| Pergunta de solução (2 por ambiente) | +5 por acerto |
| Pergunta bônus V/F do dia | +5 |
| Fichas economizadas (ambiente completo) | +3 cada |
| Máximo por ambiente | 84 pontos |

Ao final da ronda, os riscos não achados aparecem em **laranja** e podem ser
tocados para aprender (sem pontos). Ambientes já jogados ficam em modo revisão.

**Ideias para a banca da IA**: botão "Tenho uma ideia" — até 3 sugestões por dia
(novo risco, nova solução ou situação do dia a dia), gravadas localmente e
enviadas à planilha central quando `PLANILHA_URL` está configurada. A banca
avalia à noite e publica os pontos extras no placar do dia seguinte.

**Configuração** (bloco `CONFIG` no topo do script de `jogo.html`):
`DATA_INICIO` (dia 1 da rodada), `PLANILHA_URL` (Apps Script) e `MODO_TESTE`
(true = todos os ambientes abertos e rondas refazíveis, para revisão; **colocar
false antes de publicar**).

No Apps Script, os envios chegam com `tipo: "pontuacao"` (apelido, comodo,
pontosAmbiente, pontosTotal, achados) ou `tipo: "ideia"` (apelido, tipoIdeia,
comodo, texto, data, hora) — grave cada tipo numa aba própria.

---

# Enquete dos 3 mini-jogos (histórico)

Página única (`index.html`) que apresenta 3 micro-protótipos jogáveis do futuro jogo
**Casa Segura — Movimento sem quedas** e coleta a opinião do participante em um
questionário de 9 perguntas (uma por tela, fonte grande, linguagem simples).

## Os 3 mini-jogos

| Mini-jogo | Mecânica testada |
|---|---|
| 🔍 Caça aos Riscos | tocar nos perigos escondidos em 2 cômodos (sala e cozinha, 4 perigos cada) |
| 🛠️ Reforma da Casa | escolher melhorias com orçamento limitado (R$ 500) |
| 🌙 Um Dia do Seu José | acompanhar uma história de quase-queda, passo a passo |

## Como usar

**Grupo de profissionais (respondem sozinhos):** envie o link da página publicada.

**Grupo amplo de idosos (sessões acompanhadas):** abra a página num tablet ou celular,
deixe a pessoa jogar e responder. Ao final, toque em **"Nova resposta"** para a próxima
pessoa. Todas as respostas ficam guardadas no aparelho — na tela final, o link
*"respostas guardadas neste aparelho (N)"* abre a área do facilitador, com a tabela em
CSV pronta para copiar no Excel/Google Planilhas.

## Como as respostas chegam até você

1. **WhatsApp** — o participante toca em "Enviar por WhatsApp" e escolhe o seu contato.
2. **Copiar resposta** — copia o texto para colar em qualquer lugar.
3. **Guarda local** — toda resposta concluída fica salva no aparelho (área do facilitador).
4. **Automático para planilha Google (opcional)** — veja abaixo.

## Publicação

- **Teste local:** basta abrir `index.html` no navegador (dois cliques no arquivo).
- **GitHub Pages (recomendado):** crie um repositório, envie `index.html` e ative o
  Pages nas configurações. O link fica no formato `https://SEUUSUARIO.github.io/REPO/`.
- Qualquer outra hospedagem estática (Netlify Drop, Vercel) também funciona — é um
  arquivo único, sem dependências.

## Envio automático para planilha Google (opcional)

1. Crie uma planilha nova no Google Planilhas.
2. Menu **Extensões → Apps Script** e cole:

```javascript
function doPost(e) {
  var s = SpreadsheetApp.getActiveSpreadsheet().getSheets()[0];
  var d = JSON.parse(e.postData.contents);
  s.appendRow([new Date(), d.facil, d.denovo, d.mudar, d.escolha, d.novorisco,
               d.idade, d.mora, d.tec, d.perfil, d.comentario]);
  return ContentService.createTextOutput("ok");
}
```

3. **Implantar → Nova implantação → App da Web**, com acesso
   "Qualquer pessoa". Copie a URL gerada.
4. No `index.html`, cole a URL na constante `PLANILHA_URL` (início do `<script>`).

Com isso, cada resposta concluída cai numa linha da planilha automaticamente
(os botões de WhatsApp continuam funcionando como reforço).

## Campos coletados

`data` · `mais_facil` · `jogaria_de_novo` · `mudaria_casa` · `escolha_final` ·
`novo_risco_lembrado` (pergunta aberta: perigo/solução que o jogo não mostrou —
material bruto para o futuro modo "contribua com o jogo" avaliado por IA) ·
`idade` (faixas) · `mora_sozinho` · `tecnologia` (conforto digital) ·
`perfil` (idoso / familiar-cuidador / profissional — separa os dois grupos da consulta) ·
`comentario`

Nenhum dado pessoal identificável é coletado.
