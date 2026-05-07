# skills

Skill pack para governanca de agentes de IA em tarefas de engenharia de software. Define o contrato comportamental entre o humano (**Navegador**) e o agente (**Piloto**).

## Skills

| Skill | Diretorio | Governo |
|---|---|---|
| [autonomous-work](skills/autonomous-work/SKILL.md) | `skills/autonomous-work/` | Postura — quando agir vs. quando perguntar |
| [disciplina-logica-operacional](skills/disciplina-logica-operacional/SKILL.md) | `skills/disciplina-logica-operacional/` | Metodo — como executar com rigor tecnico |
| git (planejada) | — | Permissao — quando tocar o historico do usuario |

As tres skills operam juntas sem sobreposicao: `autonomous-work` define o *quando*, `disciplina-logica-operacional` define o *como*, e a skill de git define o *limite* sobre recursos escassos do usuario.

## Principios centrais

- **Silogismo da tarefa:** toda alteracao deriva de duas premissas — convencoes da base + pedido do usuario. O que nao segue e escopo fantasma.
- **Recurso escasso como limite:** permissao e devida so onde a acao do Piloto altera algo do Navegador (git history, banco, credenciais). Fora disso, age.
- **Verificacao empirica:** TDD (Red-Green-Refactor) como protocolo, nao como boas praticas opcionais.
- **Incrementos atomicos:** cada entrega pequena, estavel, revertivel, sem estado quebrado intermediario.
- **Seguranca por padrao:** validar na fronteira, falhar de forma segura, nunca engolir excecoes.

## Arquivos de suporte

- `entities.json` — registro do projeto (Piloto)
- `mempalace.yaml` — configuracao de salas para o sistema de memoria MemPalace

## Instalacao

```bash
npx skills add jadsongmatos/skills
```

| Acao | Comando |
|---|---|
| Instalar tudo (interativo) | `npx skills add jadsongmatos/skills` |
| Instalar skill especifica | `npx skills add jadsongmatos/skills --skill autonomous-work` |
| Instalar tudo sem prompt | `npx skills add jadsongmatos/skills --all` |
| Listar skills do repo | `npx skills add jadsongmatos/skills --list` |
| Instalar para agente especifico | `npx skills add jadsongmatos/skills -a claude-code` |
| Instalar globalmente | `npx skills add jadsongmatos/skills -g` |

## Licenca

GNU GPLv3