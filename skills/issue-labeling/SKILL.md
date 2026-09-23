---
name: issue-labeling
description: Vocabulario canonico do fluxo de issues - tipos (MAP, SPEC, SUB-TASK e os pontuais BUG/FEAT/REFACTOR/DOCS/INFRA/TASK), labels de estado (need-*, ready-for-*, wontfix), label de trabalho (work-in-progress) e regras de transicao. Use ao aplicar, trocar ou interpretar um label ou prefixo de tipo em um issue.
---

# Labeling

> **Skills ainda nao criadas:** `issue-discovery`, `issue-refinement` e
> `issue-break-down` sao citadas neste arquivo como proximo passo, mas ainda
> nao existem neste plugin. Ao indicar uma delas, avise que nao e possivel
> usar a skill referenciada porque ela ainda nao foi criada.

Referencia canonica de tipos e labels do fluxo de issues. Toda skill que mexe
em labels segue este vocabulario.

## Tipos

O tipo e um **prefixo no titulo** (`[MAP] ...`) e diz o que a issue **e**.
O tipo e **imutavel**: um `[MAP]` nunca vira `[SPEC]` — ele gera filhos.

| Prefixo | Significado |
|---|---|
| `[MAP]` | Ideia nebulosa, sem arestas definidas. Nao se sabe o que sera desenvolvido |
| `[SPEC]` | Recorte de produto ja conhecido, requisitos por lapidar. Bloco de entrega deployavel |
| `[BUG]` | Pontual: comportamento errado a corrigir. O brief pede reproducao e causa |
| `[FEAT]` | Pontual: funcionalidade nova e bem delimitada (ampla seria `[SPEC]`) |
| `[REFACTOR]` | Pontual: mudanca interna sem alterar comportamento visivel |
| `[DOCS]` | Pontual: somente documentacao |
| `[INFRA]` | Pontual: CI, workflows, build, tooling, configuracao de repo |
| `[TASK]` | Pontual generica — fallback quando nenhum pontual acima encaixa |
| `[SUB-TASK]` | Unidade executavel de trabalho, que vira um PR |

`[BUG]`, `[FEAT]`, `[REFACTOR]`, `[DOCS]`, `[INFRA]` e `[TASK]` sao os
**tipos pontuais**: no fluxo se comportam de forma identica (mesmos labels de
estado, transicoes e formato de brief), e onde outra skill diz "tipo pontual"
qualquer um deles vale. Escolha sempre o mais especifico — `[TASK]` e o
ultimo recurso, nunca a primeira opcao.

## Labels de estado

O estado e um **label** e diz o **proximo passo** da issue.

| Label | Significado | Proximo passo | Cor |
|---|---|---|---|
| `need-triage` | Issue crua aguardando triagem. Aplicado por humano (ou issue template) | `/issue-triage` | `FBCA04` |
| `need-discovery` | Falta descobrir os limites do que se quer construir | `/issue-discovery` | `FBCA04` |
| `need-refinement` | Falta lapidar requisitos / esclarecer a solicitacao | `/issue-refinement` | `FBCA04` |
| `need-break-down` | Escopo entendido, falta quebrar em sub-tasks | `/issue-break-down` | `FBCA04` |
| `ready-for-agent` | Unidade executavel que o agente faz sozinho | `/issue-remote-implement` | `0E8A16` |
| `ready-for-human` | Unidade executavel exclusivamente humana; o agente so orienta | Execucao humana | `1D76DB` |
| `wontfix` | Nao sera feito. Terminal: a issue e fechada com esse label | — | `FFFFFF` |

## Label de trabalho

`work-in-progress` nao e estado: nao diz o proximo passo da issue, e sim que
um worker esta atuando nela **agora**. Por isso convive com o label de estado
vigente — e o unico label que acompanha outro.

| Label | Significado | Quem aplica e remove | Cor |
|---|---|---|---|
| `work-in-progress` | Worker em execucao nesta issue | O workflow do worker, no inicio e no fim da execucao — nenhuma skill o troca | `D93F0B` |

Issue com `work-in-progress` nao dispara outro worker: os gatilhos dos
workflows chamadores filtram por ele, e o workflow reutilizavel confere de
novo antes de comecar (guarda de concorrencia).

## Regras

1. **Um label de estado por vez.** Ao mover a issue de estagio, remova o label
   de estado atual e aplique o novo na mesma operacao.
2. **O fluxo so anda para frente:**
   `need-triage → need-discovery → need-refinement → need-break-down →
   ready-for-*` e, por fim, issue fechada (merge do PR ou fechamento manual). Correcao de
   classificacao ou problema pos-merge vira **issue nova** vinculada — o label
   de uma issue existente permanece no estagio em que esta.
3. **Quem troca o label e a skill que conclui o estagio.** Cada skill entrega a
   issue ja no estado que a proxima skill espera.
4. **`wontfix` fecha.** Aplique o label e feche a issue no mesmo passo, com um
   comentario explicando o motivo e vinculando precedentes.
5. **Label inexistente no repositorio: crie antes de aplicar.** Use a cor da
   tabela e o significado como descricao:

   ```bash
   gh label create "need-discovery" --color FBCA04 \
     --description "Falta descobrir os limites do que se quer construir"
   ```

   `gh issue edit --add-label` falha se o label nao existe — criar primeiro
   evita perder a rodada.
6. **`work-in-progress` fica fora destas regras.** E label de trabalho, nao
   de estado — ver a secao "Label de trabalho" acima. Nenhuma skill o aplica,
   remove ou conta como label de estado.
