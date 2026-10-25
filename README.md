# fila-propostas

Sua IA segue instruções que você escreveu (skills, CLAUDE.md, ganchos). Elas ficam velhas, e você só descobre quando algo quebra. Esta fila pega o erro na hora, guarda a prova e deixa **você** decidir se a instrução muda.

Feita para o Claude Code. Duas skills, um script e quatro ganchos.

---

## Como funciona

1. A IA segue um passo de uma skill e dá errado. Ou você corrige: "não, nunca faça assim".
2. `/propor-melhoria` anota o passo, a frase exata que mostrou o erro e o texto que resolveria. Vai para uma fila. **Nada é alterado.**
3. Depois de um commit, ou no fim da resposta, um gancho avisa que há propostas.
4. Você abre `/fila-propostas`. Ela mostra **uma proposta por vez**. Você decide: aprovar, editar ou rejeitar.
5. Só o aprovado é aplicado. Antes, a skill faz backup do arquivo. Depois, confere se o arquivo continua válido. Se não estiver, restaura o backup.

### Como é uma proposta

Esta é uma proposta real, de uma skill do [claude-dev-flow](https://github.com/ArthurFigg/claude-dev-flow):

```
P007 - A conferência de testes intactos só olhava os arquivos já fotografados;
       um arquivo novo em tests/ (ex.: conftest.py) passava sem ser visto
Arquivo: skills/implementar/SKILL.md
Por que mudar: sha256sum -c só confere as linhas do .sha; arquivo criado depois
               da foto não aparece
Por que a nova é melhor: o diff compara a lista de arquivos de antes com a de
               agora, então arquivo novo ou apagado também para a implementação
O que pode quebrar: depende de a foto ignorar a __pycache__
Antes:  1. Testes intactos: `sha256sum -c "$TMP/testes-{nome}.sha"`. Qualquer diferença:
Depois: 1. Testes intactos: `sha256sum -c "$TMP/testes-{nome}.sha"` e, para pegar
        arquivo novo, `diff <(cut -c67- ...) <(find tests -type f ... | sort)`. Qualquer diferença:
Aprovar / Editar / Rejeitar?
```

---

## O freio contra alarme falso

Uma proposta vista **uma única vez** fica em espera. Ela só chega até você quando se repete em outra sessão. Sem isso, qualquer tropeço vira proposta e a fila vira ruído.

Duas exceções, em que a proposta chega na hora:
- uma correção sua com marca de regra ("nunca", "sempre", "daqui pra frente");
- você dizer que algo "devia ter virado proposta".

Correção que vale só para o momento ("hoje", "dessa vez", "por enquanto") é descartada.

### Quem faz as contas

Status, soma de ocorrências, vencimento e métricas são feitos pelo script `fila.ps1`, não pela IA. Em testes, cada vez que essa conta ficou com a IA, ela errou de um jeito diferente. A IA só decide qual é o problema e qual texto muda.

---

## Instalação

Precisa de Windows com PowerShell e Claude Code.

1. Copie `skills/propor-melhoria/` e `skills/fila-propostas/` (com o `fila.ps1`) para `~/.claude/skills/`.
2. Copie a pasta `hooks/` para `~/.claude/hooks/`.
3. Acrescente os blocos de `settings.example.json` ao seu `~/.claude/settings.json`. Troque `<usuario>` pelo seu usuário do Windows.
4. Abra o Claude Code e rode `/fila-propostas`. Sem propostas, ela diz "fila vazia".

A fila fica em `~/fila-propostas/fila.json`, com backups e estado na mesma pasta. Para usar outro arquivo de fila (por exemplo, para testar sem misturar com a fila real), defina a variável de ambiente `FILA_PROPOSTAS_ARQUIVO`.

---

## O que tem aqui

```
skills/
  propor-melhoria/SKILL.md    # avalia o atrito e grava a proposta (nunca aplica)
  fila-propostas/SKILL.md     # mostra uma por vez e aplica só o aprovado
  fila-propostas/fila.ps1     # todas as contas da fila
hooks/
  fila-sinais-commit.ps1      # depois de um commit, procura sinais
  fila-notas-fim.ps1          # no fim da resposta, cobra os sinais não avaliados
  fila-sinais-comum.ps1       # parte comum dos dois
  lembrete-nota-fila.ps1      # lembra a IA de anotar quando um passo estava errado
settings.example.json         # como registrar os ganchos
```

## Limites

- Só PowerShell (Windows). Em outro sistema, o `fila.ps1` e os ganchos precisam ser portados.
- Só propõe mudança em arquivo do fluxo: qualquer coisa dentro de uma pasta `.claude` (skills, agentes, ganchos, memória) ou um arquivo chamado `CLAUDE.md`, `fluxo.md` ou `GEMINI.md`. Qualquer outro destino é recusado.
- Os textos das skills estão em português.

## Origem

Peça do [claude-dev-flow](https://github.com/ArthurFigg/claude-dev-flow), um fluxo de desenvolvimento com Claude Code. A fila funciona sozinha: não precisa do resto do fluxo.
