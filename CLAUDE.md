# Casa Segura — instruções para o assistente

**ANTES de alterar qualquer arquivo, leia `ORIENTACOES-DESENVOLVIMENTO.md`** (neste
diretório). Ele contém a arquitetura, as decisões de design com seus porquês, as
armadilhas desta máquina e as receitas de teste e publicação. Este CLAUDE.md é só o
resumo de sobrevivência.

## O que é

Jogo educativo de prevenção de quedas de idosos. `jogo.html` = o jogo (Caça aos Riscos,
7 dias, ronda única por ambiente). Publicado em
https://sacwbr-ai.github.io/casa-segura/jogo.html (repo `sacwbr-ai/casa-segura`).

## Regras invioláveis (decisões do usuário — não "simplificar")

- Ronda ÚNICA por ambiente; 90s que PAUSAM durante leituras; 8 fichas por ambiente.
- Itens seguros (distratores) misturados aos riscos; 1 risco invisível (+20) por ambiente.
- NUNCA `cursor:pointer` em objetos da cena (entregaria os alvos no desktop).
- Todo botão novo passa por `toqueValido()` (anti-toque-duplo de idosos).
- Sem cadastro, sem dados pessoais, sem dinheiro. Fontes grandes, linguagem simples.
- Banca da IA avalia ideias em LOTE diário com curador humano — nunca em tempo real.

## Sobrevivência nesta máquina (detalhes no ORIENTACOES §9–10)

- Responder e escrever código/commits em **português**.
- `gh auth login --with-token` dá 401 falso aqui: use `GH_TOKEN`
  (`[Environment]::GetEnvironmentVariable("GH_TOKEN","User")`) + gh em
  `C:\Program Files\GitHub CLI`.
- node em `C:\Users\SW\AppData\Local\node\node-v22.17.0-win-x64` (fora do PATH);
  PowerShell é 5.1 (sem `&&`); python da Store não lê `AppData\Roaming` (copie scripts
  para o scratchpad) e precisa de `PYTHONUTF8=1`.
- `preview_screenshot` trava com frequência: verifique via `preview_eval` (receitas de
  teste E2E e checagem de geometria no ORIENTACOES §7).
- Depois de QUALQUER mudança em cena SVG: rodar a checagem de sobreposição (§7.3).
- Antes da rodada real: `MODO_TESTE: false` + `DATA_INICIO` no CONFIG + push.

## Fluxo de publicação

`git add` → commit em português (com rodapé Co-Authored-By do modelo) → `git push` →
Pages atualiza sozinho em ~1 min → conferir as URLs com `Invoke-WebRequest`.
