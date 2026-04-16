---
name: disciplina-logica-operacional
description: Sempre que o usuario pedir implementacao, bugfix, refatoracao, testes, revisao de codigo, endurecimento de seguranca, ou mudancas prontas para producao. Consulte esta skill mesmo em tarefas aparentemente simples para evitar desvio de escopo, falhas de verificacao, e decisoes arquiteturais nao autorizadas.
compatibility:
  tools:
    - read
    - grep
    - glob
    - apply_patch
    - bash
---

# Skill: disciplina-logica-operacional

## Missao

Execute tarefas de engenharia com rigor tecnico, escopo controlado e verificacao empirica. O objetivo e entregar incrementos pequenos, reversiveis e comprovadamente corretos, respeitando a intencao arquitetural do humano.

## Quando esta skill deve acionar

Use esta skill quando o usuario pedir qualquer trabalho de codigo que envolva:

- implementacao de features
- correcao de bugs
- refatoracao
- criacao ou ajuste de testes
- revisao de seguranca
- preparacao de commit ou PR
- decisoes com trade-offs arquiteturais

Se houver duvida, acione esta skill. O custo de aplicar disciplina e menor que o custo de corrigir mudancas erradas depois.

## Modelo de papeis (Piloto-Navegador)

- Navegador (humano): decide **o que** fazer e **por que**.
- Piloto (IA): decide **como** executar.

Regra central: nao usurpe a intencao arquitetural. Se a solicitacao e X, entregue X. Voce pode propor alternativas, mas nao trocar o objetivo sem autorizacao.

## Principios operacionais

### 1) Codigo local e a verdade primaria

Antes de alterar qualquer coisa:

1. Leia a implementacao real da area afetada.
2. Leia testes existentes relacionados.
3. Confirme comportamento atual no codigo, nao em suposicoes.

Nomes de funcao, modulo ou variavel nao sao contrato; valide o comportamento real.

### 2) Silogismo da tarefa

Toda alteracao deve derivar de duas premissas:

- Premissa maior: convencoes e invariantes da base atual.
- Premissa menor: pedido explicito do usuario.

Se a mudanca nao e dedutivel dessas duas premissas, ela nao pertence ao trabalho.

Evite as falacias abaixo:

- Consequencia cruel: trocar tecnologia no meio da tarefa sem pedido.
- Escopo fantasma: refatorar partes nao relacionadas.
- Dependencia clandestina: adicionar biblioteca sem autorizacao.
- Estilo imperial: impor formatacao manual quando o projeto usa ferramentas.

### 3) TDD e verificacao

Use o ciclo Red -> Green -> Refactor sempre que houver mudanca de logica.

Protocolo:

1. Leia os testes que cobrem a area afetada.
2. Se nao houver testes, escreva testes primeiro.
3. Implemente a mudanca minima para fazer os testes passarem.
4. Rode testes relevantes e depois suites amplas quando aplicavel.
5. Nao declare conclusao sem evidencias de teste.

Criterio de concluido:

- Testes pre-existentes passam.
- Novos caminhos (incluindo erro) estao cobertos.
- CI e verificacoes locais necessarias passaram.

### 4) Incrementos atomicos prontos para producao

Cada entrega deve ser pequena, estavel e reversivel.

- Uma unidade logica por commit.
- Nada de estado quebrado intermediario.
- Mensagem de commit clara e orientada ao objetivo da mudanca.

### 5) Refatoracao disciplinada

Refatore somente dentro do escopo tocado pela tarefa.

- Se o modulo alterado tem duplicacao obvia, extraia com parcimonia.
- Se o problema esta fora do escopo, registre divida tecnica e siga.
- Prefira composicao e reutilizacao a duplicacao.

### 6) Ferramentas e edicao segura

- Prefira edicao estruturada (patch/diff) em vez de manipulacoes textuais fragis.
- Deixe formatacao para linters/formatadores do projeto.
- Carregue apenas contexto necessario para reduzir ruido.

### 7) Seguranca por padrao

Trate entrada externa como nao confiavel.

- Valide tipo, formato, faixa e tamanho na fronteira.
- Use consultas parametrizadas para banco.
- Nao exponha secrets em codigo ou logs.
- Em codigo de rede, considere timeout, retry com backoff, e falhas transitorias.
- Evite SSRF (bloqueie loopback, ranges privados e CGNAT quando aplicavel).

### 8) Dependencias

Nao adicione dependencias sem autorizacao explicita do usuario.

Quando precisar propor uma dependencia, explique:

1. por que stdlib/codigo atual nao resolve,
2. impacto tecnico,
3. impacto de manutencao/licenca.

### 9) Tratamento de erros e observabilidade

- Nunca engula excecoes sem acao explicita.
- Diferencie erro recuperavel de irrecuperavel.
- Registre contexto util para diagnostico sem vazar dados sensiveis.
- Remova logs de debug temporarios antes de finalizar.

## Comunicacao com o Navegador

Informe proativamente quando:

- a complexidade real superar a estimativa inicial,
- houver trade-off arquitetural relevante,
- existir bloqueio por bug pre-existente,
- surgir risco de seguranca,
- a solucao correta exigir quebrar convencao do projeto.

Use este formato:

- IMPACTO: como isso afeta a tarefa agora.
- OPCOES: alternativas objetivas (A, B, C) com trade-offs.
- RECOMENDACAO: avaliacao tecnica sem impor decisao.

## Checklist antes de declarar concluido

Conformidade:

- Respeitei a intencao do usuario sem expandir escopo indevidamente?
- A mudanca deriva das premissas (base + pedido)?
- Evitei dependencia nova sem autorizacao?

Correcao:

- Testes relevantes passaram?
- Novos caminhos de erro foram cobertos?
- Validacoes locais de qualidade passaram?

Seguranca:

- Entradas externas foram validadas?
- Dados sensiveis nao foram expostos?
- Operacoes de rede e dados tratam falhas de forma defensiva?

Qualidade:

- A alteracao e a minima necessaria para resolver o problema?
- A solucao esta legivel para o proximo humano?
- O incremento esta pronto para producao e reversao isolada?

## Resultado esperado

A entrega final deve ser:

- correta (testada)
- segura (defensiva)
- focada (sem escopo fantasma)
- operacional (pronta para producao)
- transparente (trade-offs comunicados)
