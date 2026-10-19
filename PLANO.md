# PLANO — Multiagente e Orquestração (Formação LOOP-R Engenharia, curso 3 de 6) · 19/10/2026

Versões em que tudo foi rodado: Claude Code **2.1.303**, Codex CLI **0.163.0**, Node 24, bash, git. Docs oficiais lidas em 19/10/2026: Claude Code `sub-agents`, `agents` (comparação), `agent-teams`, `worktrees`, `agent-view`; Codex `cli/reference`; Anthropic "How we built our multi-agent research system" (13/06/2025).
Projeto de treino do aluno: um clone de `inematds/inema-agent-runtime` (kit 0.3.0, commit 5cb431b) em `~/treino-multi` — traz o time de três papéis, o `verificar.mjs`, a ponte `codex-exec.sh` e o `colisao.mjs`. A fila caseira vem de `inematds/jornada-agentes`.
Saídas de terminal com selo "real": capturadas em 19/10/2026 no scratchpad com os mesmos blocos das aulas (agente falso por variável ou por PATH; `TMPDIR` aponta a pasta de ensaio). Três chamadas reais pela assinatura: subagente explorador (aula 2), gerador Claude (aula 4) e revisão pelo Codex (aula 7).
Números: só do post da Anthropic (4× e 15× tokens; 90,2% na avaliação interna de pesquisa deles) e das docs (qualitativo: "significativamente mais tokens"). Não há medição local de multiagente; os tempos dos ensaios são de agente falso (sleep) e servem só para ver o paralelismo.

Fora do escopo (dono em outro curso): critério de pronto e avaliador cético a fundo (curso 1); loop headless, flock e tetos de ciclo (curso 2); hooks como guarda, sandbox como segurança, tríade letal (curso 4); promoção/rollback (curso 5).

| # | slug | título | objetivo (≤160) | diagrama | prática → pronto quando |
|---|---|---|---|---|---|
| 1 | quando | Quando NÃO usar multiagente | decidir entre um agente, sequência, paralelo ou híbrido pela matriz de sinais, antes de abrir o segundo agente. | pedido → simples? → sinais S × P → sequencial / paralelo / híbrido (+ erro caro centraliza) | `python3 decide.py tarefas.txt \| grep -c "^decisão:"` → `4`; suas 3 tarefas → `3` decisões, `0` avisos |
| 2 | subagentes | Subagentes: contexto limpo, retorno curto | criar um subagente só de leitura com retorno de até 12 linhas e conferir que a resposta dele cabe na conversa principal. | conversa principal → pedido curto → subagente (janela própria, ferramentas restritas) → resumo | conferidor → `explorador · tools: Read, Grep, Glob · model: inherit · escreve: não`; chamada real → `grep -c "^CONFIANÇA"` → `1` |
| 3 | fanout | Orquestrador-trabalhadores e fan-out | rodar um fan-out com N trabalhadores, teto por item e consolidação curta, refazendo só o item reprovado. | orquestrador → lista → N trabalhadores (timeout) → arquivos → consolidado | ensaio N=1 ~10 s × N=3 ~4 s; `OK: 4 · FALHA: 1`; 2ª rodada → `pula … (aprovado)` × 4 |
| 4 | avaliador | Gerador × avaliador | ligar um gerador a um avaliador separado que só olha o artefato e devolve parecer, com teto de rodadas. | pedido → gerador → artefato → avaliador (verificar.mjs) → aprovado / parecer → gerador | ensaio → `rodada 2 · avaliador: 2/2 critérios OK → APROVADO`; teimoso → `PARADO: 3 rodadas sem aprovação` (2) |
| 5 | isolamento | Isolamento: worktrees e sandboxes | dar a cada agente que escreve um worktree próprio e a cada revisor um sandbox só de leitura, e provar a diferença. | mesmo checkout (perde gravação) × worktree por agente (merge) × sandbox read-only / workspace-write | A) `versao = 1` (perdeu) B) `versao = 2` + `cor = verde`; `git worktree list \| wc -l` → `3`; sandbox → `bloqueado fora da pasta` |
| 6 | filas | Filas, retomada e colisão entre sessões | interromper e retomar uma fila, provocar a colisão de duas filas na mesma pasta e travar com flock e com a guarda de colisão. | entrada → rodando → feito/falhou; 2ª cópia sem trava × com flock; sessão B → guarda pergunta | `execuções: 5 para 4`; duas cópias → `8 para 4`; com flock → `4 para 4`; guarda → `"permissionDecision":"ask"` |
| 7 | claude-codex | Time Claude + Codex: roteamento por papel | rotear cada papel para o nível e a ferramenta certos e fazer o Codex revisar, só lendo, o que o Claude fez. | papéis × níveis (menor/executor/topo/super) × Claude/Codex; ponte codex-exec.sh read-only | ensaio → `PONG` (0) e `sandbox recusado pela POLITICA` (2); revisão real → última linha `APROVADO` ou `FALTA:` + reconciliação |
| 8 | time | Projetar o time do seu loop | escrever o time.md do seu loop (papel, nível, ferramentas, dono dos arquivos, isolamento, retorno, verificação, teto) e rodá-lo. | decisão → papéis → isolamento → verificação → teto (mapa + gerador) | `python3 confere-time.py time.md` → `papéis: N · problemas: 0` (0); time rodado → `APROVADO` |

Simulação: `simfila` na aula 6 (interromper e retomar; uma tarefa falha na 1ª execução).
Vídeo publicado: "Claude e Codex com o mesmo contexto" (7 min 36 s, só PT) na aula 7.
Imagens: capa + inicio + 8 heroes pelo Codex (uma por vez), sem texto; cartões dos 6 cursos reaproveitados do curso 0.
Próximo curso da ordem: `https://inematds.github.io/loop-r-eng-seguranca-v72/` (curso 4; ainda não publicado em 19/10).

Achados nas fontes (não alteradas):
- `jornada-agentes/ferramentas/fila-caseira.sh`: duas cópias na mesma pasta executam cada tarefa duas vezes (o `mv` que falha não pula a tarefa, e a retomada de uma cópia devolve à entrada o que a outra está rodando). Contorno ensinado: `flock -n`.
- `inema-agent-runtime/.claude/agents/revisor.md`: diz "não altere arquivos", mas tem `Bash` — pode escrever. É política por instrução, não trava (curso 4).
- `inema-agent-runtime/runtime/ROTEAMENTO.md` fixa nomes de modelo; o curso usa só os níveis.
