# Casa Segura — Orientações completas de desenvolvimento

> **Para quem é este documento:** o assistente de IA (Claude ou outro) que continuar o
> desenvolvimento deste jogo. Leia ANTES de alterar qualquer arquivo. Ele contém o estado
> do projeto, as decisões de design (e os porquês, que não podem ser quebrados), a
> arquitetura do código, as armadilhas desta máquina Windows e as receitas prontas de
> teste e publicação. Última atualização: 08/07/2026, pelo modelo Claude Fable 5.

---

## 1. O projeto em uma página

**Casa Segura — Movimento sem quedas** é um jogo educativo gratuito, sem apostas e sem
coleta de dados, cujo objetivo real é **disseminar a prevenção de quedas de idosos em
casa**. O jogo é o veículo; a mensagem é: tirar tapetes soltos, iluminar o caminho
noturno, instalar barras de apoio, guardar coisas na altura das mãos etc.

- **Jogo publicado:** https://sacwbr-ai.github.io/casa-segura/jogo.html
- **Enquete (fase anterior):** https://sacwbr-ai.github.io/casa-segura/
- **Repositório:** https://github.com/sacwbr-ai/casa-segura (conta `sacwbr-ai`)
- **Pasta local:** `C:\Users\SW\Claude\Projects\CasaSegura`

**Modelo do jogo (decidido pelo usuário, não mudar sem ele pedir):** Caça aos Riscos em
rodada de **7 dias**, um ambiente novo por dia numa casa térrea: sala → cozinha → quarto
→ banheiro → corredor → área de serviço → jardim. 35 perigos no total (5 por ambiente).

## 2. Mapa dos arquivos

| Arquivo | O que é |
|---|---|
| `jogo.html` | **O jogo completo.** Página única, sem dependências. Todo o desenvolvimento atual acontece aqui. |
| `index.html` | A enquete de validação com 3 mini-protótipos (fase anterior do projeto; está em campo com dois grupos — não mexer sem pedido). |
| `README.md` | Documentação pública do repo: regras do jogo, publicação, integração com planilha. Manter sincronizado com mudanças no jogo. |
| `Manual-do-Jogador-Casa-Segura.docx` | Manual do jogador (12 seções). Regenerável — ver §9. |
| `Detalhamento-do-Jogo-Casa-Segura.docx` | Documento para a comissão de apoio. **ESTÁ DESATUALIZADO** (descreve rodada de 2 semanas e jogo em 3 fases, anterior à decisão dos 7 dias). Atualizar quando o usuário pedir. |
| `ORIENTACOES-DESENVOLVIMENTO.md` | Este documento. Atualize-o quando tomar decisões novas. |

## 3. Estado atual e o que falta (backlog na ordem provável)

1. **Enquete em campo** — o usuário está consultando dois grupos (idosos amplos +
   gerentes/diretores). As respostas podem gerar ajustes no jogo.
2. **Ligar a planilha central** — configurar `PLANILHA_URL` no `CONFIG` de `jogo.html`
   com um Web App do Google Apps Script (código pronto no §8). Pré-requisito da banca.
3. **Implantar a banca da IA** — processo diário de avaliação das ideias (especificação
   completa e prompt pronto no §8).
4. **Preparar a rodada real** — no `CONFIG`: `MODO_TESTE: false` e `DATA_INICIO` com a
   data do dia 1; commit + push (o GitHub Pages publica sozinho em ~1 min).
5. **Atualizar o docx da comissão** para o desenho atual (7 dias, ronda única, fichas).
6. Futuro possível: placar online ao vivo (Firebase/Supabase), mais ambientes, personas.

## 4. Regras de design que NÃO podem ser quebradas (e os porquês)

Cada uma destas foi uma decisão explícita do usuário, tomada para resolver um problema
real. Não as remova nem "simplifique" sem ele pedir:

| Regra | Por quê |
|---|---|
| **Ronda única por ambiente** (sem replay) | Com replay, o jogador decora as posições e todos empatam no teto. Estilo Wordle: uma chance, resultado definitivo. |
| **90 segundos de busca** | Sem limite de tempo, dá para testar item por item friamente. O tempo **PAUSA durante lições e perguntas** — a pressão é só na busca, nunca na leitura (público idoso). |
| **8 fichas de inspeção** | Cada toque em objeto gasta 1. Com mais objetos que fichas (sempre ≥9 objetos), tocar em tudo é punido. É pressão de *precisão*, não de velocidade. |
| **Itens seguros (distratores)** | 4–5 por ambiente. Coisas que parecem perigosas mas estão certas (tapete COM borda de borracha). Sem eles, achar os riscos é trivial. |
| **Risco "invisível" (+20)** | 1 por ambiente. É uma AUSÊNCIA (box sem barra, interruptor longe da cama), não um objeto errado. Testa o olhar de prevenção de verdade. |
| **Sem `cursor:pointer` nos alvos** | No desktop, o cursor entregava onde estavam os objetos. CSS atual: `.hz,.sf{cursor:default}`. Nunca reverta. |
| **Anti-toque-duplo (350–400 ms)** | Idosos dão toque duplo com frequência; sem o guarda, pulavam telas e respondiam duas perguntas. Função `toqueValido()` — use-a em TODO botão novo. |
| **Fonte grande, linguagem simples, uma ação por tela** | Público 60+. Botões com min-height 56–64px. |
| **Sem cadastro, sem dados pessoais, sem dinheiro** | Compromisso público do projeto. Só apelido. |
| **Pontuação: 10 visível / 20 invisível / 5 solução / 5 V-F / 3 por ficha sobrando** | Máximo 84 por ambiente, 588 na rodada. Calibrado para gerar dispersão entre jogadores. |

## 5. Arquitetura do `jogo.html`

Tudo em um arquivo: CSS no `<head>`, HTML das telas, `<script>` no final. Sem
bibliotecas. Comentários e nomes em português.

### 5.1 CONFIG (topo do script)

```js
const CONFIG = {
  DATA_INICIO: "2026-07-13",  // dia 1 da rodada; ambiente N abre no dia N à meia-noite
  PLANILHA_URL: "",           // Web App do Apps Script; vazio = envio só por WhatsApp
  MODO_TESTE: true            // true: 7 ambientes abertos + rondas refazíveis. FALSE antes da rodada real!
};
```

### 5.2 Telas (sections com classe `.tela`, navegação por `mostrarTela(id)`)

`tela-inicio` (apelido) → `tela-mapa` (a casa, cards dos 7 ambientes) →
`tela-preronda` (aviso das regras) → `tela-jogo` (a ronda) → volta ao mapa.
Paralelas: `tela-ideia` (banca) e `tela-placar`.

### 5.3 Dados dos ambientes — `const COMODOS = [...]`

Cada ambiente: `{ id, nome, icone, dia, intro, riscos, seguros, bonus, svg }`.

- `riscos`: objeto com 5 entradas. Cada uma: `{ licao, pergunta? , invisivel? }`.
  Exatamente **1 com `invisivel:true`** (vale 20) e **2 com `pergunta`**
  (`{p, ops:[3 strings], certa:indice, explica}`).
- `seguros`: 4–5 entradas, string da explicação (começa com "✅").
- `bonus`: pergunta V/F do dia — `{p, certaVF:bool, explica}`.
- `svg`: string com a cena (viewBox `0 0 800 450`; parede y 0–300, chão y 300–450).

### 5.4 Convenção do SVG (crítica para adicionar conteúdo)

Cada objeto interativo é um grupo:

```svg
<g class="hz" data-r="idDoRisco">   <!-- risco: class hz  + data-r -->
  ... desenho ...
  <circle cx="X" cy="Y" r="R" fill="transparent"/>                     <!-- área de toque -->
  <circle class="marca" cx="X" cy="Y" r="R-6" fill="none" stroke="#2e7d32" stroke-width="6" opacity="0"/>
</g>
<g class="sf" data-s="idDoSeguro">  <!-- item seguro: class sf + data-s -->
```

- Raio da área de toque: 34–58 (dedos idosos precisam de alvo generoso).
- **REGRA DE OURO DA GEOMETRIA:** a distância entre os centros de duas áreas de toque
  deve ser ≥ (soma dos raios + 10px). Depois de QUALQUER mudança em cena, rode a
  checagem do §7.3 — foi assim que se acharam 4 sobreposições na primeira versão.
- A marca fica verde (achado); o código pinta de laranja tracejado (`#e67e22`) os
  perdidos, na revelação final.

### 5.5 Estado e fluxo

- Persistência: `localStorage`, chave **`casaSegura_jogo_v2`** —
  `{ apelido, comodos: {id: {pontos, achados[], acertosSolucao, bonusVF, fichasRestantes, motivo}}, ideias[] }`.
  Se mudar o formato do estado, **troque o sufixo da chave** (v3...) para não corromper
  jogos em andamento.
- Fluxo da ronda: `abrirComodo(id)` → `iniciarRonda()` → `tocarObjeto(g,tipo,oid)` →
  (`abrirPergunta` | `abrirPainel`) → `verificarFim()` → `encerrarRonda(motivo)` →
  `perguntaBonus()` → `finalizarResultado()`.
- O timer (`tic()`, 1 s) **não desconta** quando `ronda.pend` está setado (painel/pergunta
  abertos) — é assim que a leitura pausa o relógio.
- Ambiente já jogado → `abrirRevisao()` (toques só mostram lições; em `MODO_TESTE` há
  botão de refazer que apaga o resultado daquele ambiente).
- `enviarPlanilha(dados)`: POST `no-cors` para `PLANILHA_URL`; envia
  `tipo:"pontuacao"` (ao fechar ambiente) e `tipo:"ideia"` (ao sugerir).

## 6. Como adicionar conteúdo (receitas)

**Novo risco/item seguro num ambiente existente:** desenhe o grupo SVG na cena (formas
simples, paleta terrosa já usada), adicione a entrada em `riscos`/`seguros`, respeite a
regra de geometria (§5.4), rode a checagem (§7.3) e o teste E2E (§7.2). Se mudar o nº de
riscos por ambiente, revise os textos que dizem "5 perigos" e a intro.

**Novo ambiente:** copie a estrutura de um item de `COMODOS`, atribua `dia` e desenhe a
cena. Atenção: os textos de interface dizem "7 dias/7 ambientes" e `diaAtual()` clampa
em 7 — busque por `7` no arquivo. Também atualize README e manual.

**Nova pergunta de solução:** 3 opções plausíveis, a certa NÃO precisa ser sempre a
primeira do array... mas hoje todas usam `certa:0`. Melhoria bem-vinda: variar o índice
(a UI já suporta).

## 7. Como testar (receitas prontas)

### 7.1 Servidor de preview

`.claude/launch.json` do projeto `projecoes fluxo caixa familiar` (diretório de sessão
habitual do usuário) já tem a configuração `casa-segura`: `python -m http.server 8123
--directory C:/Users/SW/Claude/Projects/CasaSegura`. Use as ferramentas `preview_*`;
carregue `/jogo.html`.

⚠️ **`preview_screenshot` trava com frequência nesta máquina** (timeout de 30 s) mesmo
com a página saudável. Reiniciar o servidor às vezes resolve; se não, NÃO insista —
verifique por `preview_eval`/`preview_snapshot` e checagens programáticas.

### 7.2 Teste E2E programático (via `preview_eval`)

Padrão que funciona (o guarda anti-toque-duplo exige zerar `_ultimoToque` antes de cada
clique; elementos SVG não têm `.click()` — use `dispatchEvent`):

```js
const cliq = el => { _ultimoToque = 0; el.dispatchEvent(new MouseEvent('click', {bubbles:true})); };
const svgObj = sel => document.querySelector('#jg-palco ' + sel);
const painelBtn = () => document.querySelector('#jg-painel .btn-grande');
// ex.: cliq(svgObj(".hz[data-r='tapete']")); depois cliq(painelBtn()) para fechar a lição;
// perguntas: cliq(document.querySelectorAll('#jg-painel .btn-opcao')[i]) e depois painelBtn().
```

Cubra: ronda completa (84 pts se perfeita), fichas esgotadas, tempo esgotado
(`ronda.tempo = 1; tic();`), toque repetido (não gasta ficha), revisão, refazer em modo
teste, limite de 3 ideias/dia, placar. Termine com `localStorage.clear()` para o usuário
testar do zero.

### 7.3 Checagem de geometria (rodar após QUALQUER mudança de cena)

```js
COMODOS.forEach(c => { const div = document.createElement('div');
  div.innerHTML = '<svg xmlns="http://www.w3.org/2000/svg">' + c.svg + '</svg>';
  const z = [...div.querySelectorAll('.hz, .sf')].map(g => {
    const h = [...g.querySelectorAll('circle')].find(x => x.getAttribute('fill') === 'transparent');
    return {id: g.dataset.r || g.dataset.s, x: +h.getAttribute('cx'), y: +h.getAttribute('cy'), r: +h.getAttribute('r')}; });
  for(let i = 0; i < z.length; i++) for(let j = i + 1; j < z.length; j++){
    const folga = Math.hypot(z[i].x - z[j].x, z[i].y - z[j].y) - z[i].r - z[j].r;
    if(folga < 10) console.log(c.id, z[i].id, 'x', z[j].id, Math.round(folga)); }});
```

Saída vazia = OK. Também confira: 5 riscos (1 `invisivel`), 4+ seguros, total de objetos ≥ 9.

## 8. Planilha central e banca da IA (especificação pronta)

### 8.1 Apps Script (Google Planilhas → Extensões → Apps Script → Implantar como App da Web, acesso "Qualquer pessoa")

```javascript
function doPost(e) {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var d = JSON.parse(e.postData.contents);
  if (d.tipo === "ideia") {
    ss.getSheetByName("ideias").appendRow([new Date(), d.apelido, d.tipoIdeia, d.comodo, d.texto]);
  } else if (d.tipo === "pontuacao") {
    ss.getSheetByName("pontuacoes").appendRow([new Date(), d.apelido, d.comodo, d.pontosAmbiente, d.pontosTotal, d.achados]);
  }
  return ContentService.createTextOutput("ok");
}
```

Crie as abas `ideias` e `pontuacoes`. Cole a URL do App da Web em `CONFIG.PLANILHA_URL`.

### 8.2 A banca da IA (processo diário — decidido pelo usuário: em LOTE, nunca em tempo real)

Motivos da decisão: não expor chave de API no navegador, custo previsível, supervisão
humana, e o resultado matinal cria hábito de retorno diário. **A IA propõe, o usuário
(curador) decide.** Fluxo: o usuário traz as ideias do dia (da planilha) numa sessão de
chat → a IA avalia com a rubrica → devolve tabela → usuário revisa e publica no grupo
de WhatsApp da rodada.

**Rubrica (pública para os jogadores):** Validade 0–20 (previne quedas de verdade?),
Ineditismo 0–20 (não está no jogo nem foi sugerida antes NA RODADA — repetida vale menos,
a primeira leva mais), Clareza/aplicabilidade 0–10, bônus pioneirismo +10. Descartar sem
pontos: sem fundamento, perigosa ou ofensiva.

**Prompt pronto para a avaliação diária** (colar as ideias no lugar indicado):

> Você é a banca do jogo Casa Segura (prevenção de quedas de idosos em casa). Avalie as
> ideias abaixo com a rubrica: validade 0–20, ineditismo 0–20 (compare com o catálogo do
> jogo e com as ideias já aceitas na rodada, listadas a seguir), clareza 0–10, bônus
> pioneiro +10 para a primeira ocorrência de cada ideia no dia. Catálogo atual do jogo:
> [35 riscos: tapetes soltos, fios no caminho, chinelos, banquinho/cadeira como escada,
> sofá baixo, água/sabão no chão, pano na pia, gaveta aberta, lâmpada queimada, panelas
> no alto, tapete/colcha/sapatos/mala no quarto, interruptor longe, tapete de pano no
> banheiro, frascos no chão do box, toalha no chão, degrau sem sinalização, box sem
> barra, passadeira, extensão, vaso no chão, móvel no corredor, corredor sem apoio,
> balde, varal alto, produtos no alto, cesto no caminho, mangueira, limo, lajota solta,
> escada sem corrimão, luz externa queimada]. Ideias já aceitas nesta rodada: [colar].
> Ideias de hoje (apelido | tipo | ambiente | texto): [colar]. Responda em tabela:
> apelido, ideia resumida, validade, ineditismo, clareza, bônus, TOTAL, aceita (sim/não),
> justificativa de uma linha. Marque para descarte o que for sem fundamento, perigoso ou
> ofensivo.

## 9. Publicação e Git (receitas desta máquina)

- **⚠️ `gh auth login --with-token` FALHA com HTTP 401 nesta máquina** mesmo com token
  válido (bug local não resolvido). **Use sempre a variável `GH_TOKEN`**, que já está
  persistida no ambiente de usuário. Em PowerShell:
  `$env:GH_TOKEN = [Environment]::GetEnvironmentVariable("GH_TOKEN","User")` e
  `$env:Path = "$env:ProgramFiles\GitHub CLI;$env:Path"` antes de qualquer `gh`/`git push`.
- Publicar = `git add` → `git commit` (mensagem em português, terminar com
  `Co-Authored-By:` do modelo) → `git push`. O GitHub Pages resserve `main` na raiz em
  ~1 minuto. Verifique com `Invoke-WebRequest` nas duas URLs.
- Identidade git: `sacwbr-ai` / `sacw.contas@gmail.com` (global, já configurada).

## 10. Armadilhas do ambiente Windows (aprendidas a custo)

- **PowerShell é 5.1**: sem `&&`/`||` (use `;` e `if ($?)`), sem ternário; `Out-File`
  padrão é UTF-16 (use `-Encoding utf8`).
- **node/npm NÃO estão no PATH**: prefixe
  `$env:Path = "C:\Users\SW\AppData\Local\node\node-v22.17.0-win-x64;$env:Path"`.
- **python é o da Windows Store**: não consegue abrir arquivos em
  `AppData\Roaming\Claude\...` (sandbox MSIX) — copie scripts de skills para o scratchpad
  antes de rodar. Para validadores docx: `$env:PYTHONUTF8 = "1"` (senão erra com acentos).
- **Pacote npm `docx`**: instalado no scratchpad da sessão (efêmero) — em sessão nova,
  `npm install docx` de novo onde for rodar o gerador.
- **docx-js**: bordas de parágrafo só top+bottom (as 4 violam a ordem do esquema OOXML e
  a validação falha); tabelas precisam de `columnWidths` + `width` por célula (DXA);
  `ShadingType.CLEAR`; A4 = 11906×16838, conteúdo 9026.
- Os geradores dos documentos (`gerar-doc.js` do docx da comissão e `gerar-manual.js` do
  manual) viveram no scratchpad e se perderam ao fim das sessões. Para regenerar/editar
  um docx: recrie o gerador (estrutura descrita nos commits) ou edite o docx existente
  via unpack/pack da skill.

## 11. O usuário — como trabalhar com ele

- Comunica-se em **português**; responda sempre em português.
- **Decide o design em diálogo**: apresente opções com recomendação, ele escolhe e
  refina (foi dele: os 7 dias, o tempo máximo, a banca em lote, o 2º cômodo da enquete).
  Quando ele aponta um problema ("todos terminam com os mesmos pontos"), leve a sério —
  ele enxerga furos de mecânica com precisão.
- Gosta de **revisar em planilha/documento**, não em fluxos interativos de terminal.
- Materiais para terceiros (comissão, jogadores) devem ser **detalhados** — ele prefere
  completo a resumido.
- O propósito social do projeto importa mais que o jogo: em qualquer trade-off, ganhe o
  lado que **disseminar melhor a prevenção de quedas**.

## 12. Linha do tempo das decisões (para entender o porquê das coisas)

1. Ideia inicial: jogo de deixar a casa do idoso segura; objetivo real = disseminação.
2. 3 modelos propostos: Reforma (orçamento), Simulação (Seu José), Caça aos Riscos.
3. Enquete interativa única para dois públicos (em vez de Forms com 3 colunas — celular
   não comporta colunas; mostrar > descrever). Está em campo.
4. Usuário escolheu **Caça aos Riscos, 7 dias, um cômodo/dia + jardim**.
5. Usuário apontou os furos: todos empatam no teto + mouse entrega os alvos. Redesenho
   aprovado por ele item a item: itens seguros, fichas, tempo máximo (dele), ronda única
   (estilo Wordle), risco invisível, perguntas de solução, bônus V/F.
6. Banca da IA **em lote diário com curador humano** (decisão dele; nunca em tempo real).
7. MVP publicado no GitHub Pages em 07-08/07/2026; manual do jogador gerado.
