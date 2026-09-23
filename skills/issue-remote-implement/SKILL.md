---
name: issue-remote-implement
description: Implementa um issue ready-for-agent de ponta a ponta - cria a branch conforme o git-flow, desenvolve test-first nos criterios de aceite, roda as verificacoes de qualidade e abre o PR com Closes. Termina na abertura do PR; verificacao vermelha nao vira PR.
argument-hint: "[numero-do-issue]"
arguments: issue
disable-model-invocation: true
allowed-tools: Bash(gh issue view:*) Bash(gh issue comment:*) Bash(gh pr list:*) Bash(gh pr create:*) Bash(gh pr view:*) Bash(git status:*) Bash(git branch:*) Bash(git checkout:*) Bash(git switch:*) Bash(git fetch:*) Bash(git pull:*) Bash(git add:*) Bash(git commit:*) Bash(git push:*) Bash(git diff:*) Bash(git log:*) Bash(git config:*) Read Grep Glob Edit Write
---

# Implement

Executa uma unidade de trabalho de ponta a ponta e termina em **um PR
aberto**. O merge e a revisao nao sao daqui — o fluxo desta skill acaba na
abertura do PR.

Siga a skill `issue-git-flow` para nome de branch, base e formato de PR.

Decisao a montante, execucao a jusante: o brief do issue ja diz **o que**
(criterios de aceite) e **onde** (a fronteira em que cada criterio se
observa). Voce nao redesenha o plano — transforma o brief em codigo
verificado.

Travou em algo que so o humano resolve (decisao de produto que o corpo nao
cobre, acesso faltando, verificacao que nao fica verde dentro do escopo)?
Siga as skills `grilling` e `issue-async-first`: handoff completo no issue e
encerre a rodada sem abrir PR. Termine a mensagem com: "responda e comente
`/issue-remote-implement` para retomar".

## Argumento

`$issue` — o numero do issue a implementar.

Se `$issue` estiver vazio, **pare e informe** que a skill precisa do numero do
issue.

## Verificacao de qualidade

Os comandos de verificacao (typecheck, lint, testes) chegam por uma destas
vias, nesta ordem de preferencia:

1. **Informados pelo workflow.** O system prompt traz "Verificacao de
   qualidade: ... comandos: ..." com o que o chamador liberou. Rode
   exatamente esses, na ordem dada.
2. **Descobertos no repositorio.** Sem lista informada, procure em
   `package.json` (scripts), `Makefile`, CLAUDE.md ou na config de CI. Rode
   so o que o ambiente permitir.
3. **Nenhum.** Nao achou, ou o ambiente nao deixa rodar: siga sem
   verificacao e **declare isso** no comentario do PR. Nunca afirme que
   verificou o que nao rodou.

## Passos

1. **Leia o issue e verifique se ele esta apto:**

   ```bash
   gh issue view $issue --json number,title,body,labels,state,url,comments
   ```

   Apto exige as tres condicoes:

   - titulo com prefixo de tipo pontual (ver `issue-labeling`), `[SPEC]` ou
     `[SUB-TASK]`, e label
     `ready-for-agent`;
   - **sem PR aberto vinculado** — verifique com
     `gh pr list --state open --search "Closes #$issue"`; se ja existe,
     comente apontando o PR e pare (iterar em PR existente e trabalho de
     outra versao desta skill);
   - **sem bloqueio pendente** — a secao `Dependencias` do corpo lista
     "Bloqueada por: #n"; confira cada um com `gh issue view <n> --json state`
     e, se algum estiver aberto, comente informando o bloqueio e pare.

   Issue inapto em qualquer condicao: comente o estado atual e pare.

2. **Leia as rodadas anteriores.** Comentarios carregam handoffs e respostas
   humanas — incorpore antes de comecar. O corpo do issue e lido **agora**:
   se o humano editou o brief antes de comentar `/issue-remote-implement`, vale a versao
   editada.

3. **Crie a branch** conforme a `issue-git-flow`: nome `<tipo>-<numero>`, base
   `main` — ou, para `SUB-TASK`, a branch de integracao do parent (criando e
   publicando a do parent primeiro, se nao existir). Garanta identidade de
   commit antes do primeiro commit:

   ```bash
   git config user.name "claude"
   git config user.email "noreply@anthropic.com"
   ```

4. **Implemente, test-first, um criterio de aceite por vez.** Os criterios do
   corpo sao o contrato, e cada um nomeia a fronteira (seam) onde o
   comportamento se observa — funcao publica, endpoint, comando, tela. Para
   cada criterio:

   1. escreva o teste **na fronteira que o criterio nomeia** e veja falhar;
   2. implemente o minimo que o faz passar;
   3. rode o typecheck e o arquivo de teste que mexeu (nao a suite inteira).

   Criterio sem fronteira nomeada: use a fronteira mais externa que o codigo
   oferece para aquele comportamento e anote a escolha para o comentario do
   PR. Repositorio sem infra de teste: implemente cobrindo os criterios, rode
   o que existir (typecheck, lint) e anote isso tambem. Em nenhum dos dois
   casos voce para para perguntar — a fronteira e fato do codigo, nao
   decisao.

   Commits pequenos com mensagem clara. Estilo: o codigo novo le como o
   codigo ao redor.

5. **Verifique.** Antes de abrir o PR, rode a bateria completa dos comandos
   da secao "Verificacao de qualidade" — uma vez, ao final. Resultado:

   - **Verde:** siga para o PR.
   - **Vermelho causado pela sua mudanca:** corrija e rode de novo.
   - **Vermelho fora do escopo do issue** (ja quebrado antes de voce chegar,
     ou exige decisao que o brief nao cobre): nao abra PR. Faca push da
     branch para nao perder o trabalho, handoff `issue-async-first` no issue com o
     erro exato colado e o que falta, e encerre a rodada.

   PR com verificacao vermelha nao existe nesta skill.

6. **Abra o PR** para a base definida pela `issue-git-flow`, com o template dela
   (`Closes #$issue` + "O que muda" + "Como verificar"). Depois comente no
   issue:

   ```markdown
   ## Implement

   **PR:** <url>
   **Criterios cobertos:** <lista curta mapeando criterio → teste e onde foi coberto>
   **Verificacao:** <comandos rodados e resultado — ou "nenhuma: <motivo>">
   **Atencao:** o CI nao roda sozinho em PR aberto pelo agente — de o play
   manualmente (feche e reabra o PR, ou dispare o workflow de CI).
   ```

O implement esta completo quando existe um PR aberto da branch da issue para
a base certa, cujo corpo comeca com `Closes #$issue`, cujo diff cobre todos os
criterios de aceite do corpo do issue com teste na fronteira de cada um, cuja
verificacao esta verde (ou declarada ausente, com motivo), e o issue tem o
comentario com o link. Nao ha troca de label: o merge do PR fecha o issue, e
"PR pronto" e derivado do vinculo.

## Fora de escopo

- Fazer merge, aprovar ou iterar em PR ja existente.
- Commit ou push direto na `main` ou na branch de integracao.
- Alterar labels, titulo ou corpo do issue.
- Substituir o CI do PR: a verificacao local e o primeiro filtro; o CI
  continua sendo o juiz final.
- Redesenhar o plano: fronteiras e escopo vem do brief. Discordou? Handoff,
  nao improviso.
- Issues `ready-for-human` — nelas o agente no maximo orienta.
