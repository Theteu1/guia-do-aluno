# Guia do Aluno — Operação Oceânica (`Ocean 1.1.0-pilot`)

Bem-vindo(a) ao desafio **Operação Oceânica (`Ocean 1.1.0-pilot`)**! Este guia reúne os fundamentos arquiteturais, os protocolos de comunicação, as frentes de investigação e os critérios de entrega necessários para conectar a solução multiagente da sua equipe ao servidor oficial do simulador.

> **Espírito do Desafio (Modo Descoberta):** Neste laboratório, não há receitas prontas. O simulador se comporta como um sistema ciberfísico real e auditável: suas leis físicas, restrições financeiras e eventos oceânicos devem ser descobertos pela sua equipe por meio de observação de telemetria, consulta à documentação da API (`/docs`), formulação de hipóteses, experimentação iterativa e inspeção do *ledger* criptográfico.

---

## Sumário

1. [A Missão, a Frota e o Método de Engenharia](#1-a-missão-a-frota-e-o-método-de-engenharia)
2. [Servidores Oficiais por Equipe (`t01` a `t05`) e Acesso à Documentação](#2-servidores-oficiais-por-equipe-t01-a-t05-e-acesso-à-documentação)
3. [Identidades, Topologia Própria de Agentes e Fronteira de Segurança](#3-identidades-topologia-própria-de-agentes-e-fronteira-de-segurança)
4. [Observabilidade: REST, SSE, WebSocket e Webhooks HMAC](#4-observabilidade-rest-sse-websocket-e-webhooks-hmac)
5. [Ciclo de Fases do Tick, Protocolo Multiagente e Performance HTTPS](#5-ciclo-de-fases-do-tick-protocolo-multiagente-e-performance-https)
6. [Catálogo das 7 Ações Permitidas e Pré-Condições Físicas](#6-catálogo-das-7-ações-permitidas-e-pré-condições-físicas)
7. [Frentes de Investigação Marítima e Econômica](#7-frentes-de-investigação-marítima-e-econômica)
8. [Hard Gates e Diagnóstico de Erros HTTP](#8-hard-gates-e-diagnóstico-de-erros-http)
9. [Entrega no Repositório da Equipe e Documentação da Solução](#9-entrega-no-repositório-da-equipe-e-documentação-da-solução)
10. [Roteiro Sugerido de Evolução](#10-roteiro-sugerido-de-evolução)

---

## 1. A Missão, a Frota e o Método de Engenharia

Sua equipe coordena o Centro de Operações Autônomas de **4 cargueiros** distintos (`S1-FAROL`, `S2-MARE`, `S3-CORAL` e `S4-ATLAS`). Ao longo de **72 horas simuladas**, divididas em **`432` ticks discretos de 10 minutos** (`tick 0` até `tick 431`), sua frota precisa transportar dezenas de milhares de toneladas de carga até quatro portos de destino sob orçamento fechado em moeda virtual (`D$`).

Princípios inegociáveis da arquitetura:
- **Zero comando humano sobre os navios:** Operadores humanos jamais pilotam embarcações. Apenas agentes autenticados podem submeter propostas, revisões e comandos operacionais.
- **Governança multiagente real (*Maker-Checker*):** Todo comando mutável exige a cooperação comprovada de **pelo menos 2 agentes distintos** (`propositor != revisor`). Os dois agentes pré-criados (`navigation` e `safety`) servem apenas como ponto de partida: antes do `tick 0`, sua equipe tem **total liberdade criativa** para projetar e registrar sua própria organização de agentes especialistas (`POST /api/v1/identities/agents`, nos papéis `"navigation"`, `"safety"`, `"economy"` e `"coordinator"`).
- **Determinismo e Rastreabilidade:** Toda mensagem aceita ou rejeitada recebe hash canônico SHA-256, assinatura digital Ed25519 e entra em um *ledger* imutável selado por árvore de Merkle.

### Como evoluir com método científico: `conformance` vs. `official`
O servidor da sua equipe oferece dois modos de cenário selecionáveis via `POST /api/v1/round/reset`:
- **Modo `conformance` (Bancada de Laboratório):** Todos os eventos e sorteios usam uma semente determinística fixa. Se você rodar a mesma estratégia duas vezes, obterá exatamente o mesmo centavo e o mesmo milésimo de tonelada. Use este modo para isolar variáveis, testar hipóteses e medir ganhos reais!
- **Modo `official` (Prova de Generalização):** Os fenômenos móveis do oceano mudam de janela temporal e a semente criptográfica é verificada pós-rodada via *commit-reveal*. Agentes que "decoram" números de ticks falham aqui; agentes orientados a estados e eventos mantêm alta performance!

---

## 2. Servidores Oficiais por Equipe (`t01` a `t05`) e Acesso à Documentação

**Não é necessário subir nem instalar o servidor localmente.** Cada equipe já conta com uma instância dedicada e isolada do simulador Ocean 1.1 rodando na nuvem via HTTPS:

| Equipe (`team_id`) | URL Base do Servidor (`OCEAN_BASE_URL`) | Documentação Interativa (Swagger / OpenAPI) |
| :--- | :--- | :--- |
| `team-01` | `https://t01.beatbot.com.br` | `https://t01.beatbot.com.br/docs` |
| `team-02` | `https://t02.beatbot.com.br` | `https://t02.beatbot.com.br/docs` |
| `team-03` | `https://t03.beatbot.com.br` | `https://t03.beatbot.com.br/docs` |
| `team-04` | `https://t04.beatbot.com.br` | `https://t04.beatbot.com.br/docs` |
| `team-05` | `https://t05.beatbot.com.br` | `https://t05.beatbot.com.br/docs` |

### O que você encontra diretamente no link da sua equipe
Substituindo `t0X` pelo subdomínio do seu grupo (ex.: `t01.beatbot.com.br`):
- **Centro de Controle Ao Vivo (`https://t0X.beatbot.com.br/`):** Painel visual em tempo real conectado via SSE mostrando estados dos 4 navios, progresso na rota, velocidade atual/alvo, nível dos tanques vs. reserva mínima, caixa em `D$`, selo Merkle do *ledger* e feed de decisões/telemetria.
- **Documentação Completa da API (`https://t0X.beatbot.com.br/docs`):** Especificação Swagger / OpenAPI interativa com todos os endpoints, exemplos de payloads JSON, schemas, parâmetros e códigos de resposta. O JSON OpenAPI bruto também está em `/openapi.json`.
- **Healthcheck (`https://t0X.beatbot.com.br/health`):** Confirma que o servidor e o pacote normativo estão operacionais (`"status": "ok"`, `"package_valid": true`).
- **Contratos Normativos (caso presentes no repositório em `round-package/`):** Catálogo fechado de ações (`catalog/actions.json`), erros (`catalog/errors.json`), regras JCS/Merkle (`integrity/canonicalization.json`) e pesos de pontuação (`scoring/scoring.json`).

### Regras da Interface Web durante uma Partida
- **Sem recurso de Pausa:** Uma partida iniciada **não pode ser pausada nem retomada manualmente**.
- **Condução exclusiva pelos agentes:** Assim que os seus agentes iniciam uma rodada via API, todos os controles manuais de condução na interface (`▶ Iniciar`, `+1 Tick`, `+25 Ticks`, `↺ Reset`, seleção de cenário e velocidade) ficam **automaticamente bloqueados**.
- **Botão único `⏹ Parar`:** Durante o jogo ativo, a única ação manual disponível no painel é **`⏹ Parar`** (`POST /api/v1/round/stop`), que **aborta e encerra definitivamente** a partida atual (selando o *ledger* e gerando a pontuação até aquele ponto, sem possibilidade de retomar do meio — qualquer nova tentativa exige reiniciar do `tick 0`).

### Configurando a conexão dos seus agentes (`.env`)
No código da sua equipe, configure as variáveis de ambiente apontando para o subdomínio do seu time (nunca faça commit de chaves privadas de LLM no Git!):
```env
OCEAN_BASE_URL=https://t01.beatbot.com.br
OCEAN_DEFAULT_TEAM_ID=team-01
```
*(Se o repositório da sua equipe incluir `starter/hello_ocean.py`, você pode validar a conectividade imediatamente rodando `python starter/hello_ocean.py` após instalar `httpx`).*

---

## 3. Identidades, Topologia Própria de Agentes e Fronteira de Segurança

Consulte a qualquer momento `GET https://t0X.beatbot.com.br/api/v1/identities` para inspecionar os atores ativos da sua equipe, suas chaves públicas Ed25519, escopos e hashes congelados. Por padrão, substituindo `<team_id>` pelo código do seu grupo (ex.: `team-01`), o servidor inicia com três identidades base:

- **Agente de Navegação (`<team_id>-navigation-agent`)** — `agent` / `navigation` · `tok-<team_id>-navigation-agent-poc` · escopos: `telemetry:read`, `events:read`, `heartbeat:write`, `decision:write`, `command:write`
- **Agente de Segurança (`<team_id>-safety-agent`)** — `agent` / `safety` · `tok-<team_id>-safety-agent-poc` · escopos: `telemetry:read`, `events:read`, `heartbeat:write`, `decision:write`, `command:write`
- **Operador Humano (`<team_id>-operator`)** — `human` / `operator` · `tok-<team_id>-operator-poc` · escopos: `telemetry:read`, `events:read`, `intervention:write`

### 3.1. Cadastrando a Organização Multiagente da Sua Equipe (`POST /api/v1/identities/agents`)
Os 2 agentes iniciais acima **não são obrigatórios**! Enquanto o servidor estiver em `IDLE` antes da abertura do `tick 0` (*pre-round freeze*), sua equipe pode registrar **de `2` a `16` agentes próprios** (em LangGraph, CrewAI, AgentFlow, AutoGen ou Python puro) e receber na resposta suas chaves Ed25519 e tokens Bearer exclusivos:
- **Papéis válidos (`role`):** `"navigation"`, `"safety"`, `"economy"` e `"coordinator"`.
- **RBAC fino (`scopes` e `assigned_ships`):** você pode atribuir subconjuntos de navios (`"assigned_ships": ["S1-FAROL", "S2-MARE"]`) ou remover `"command:write"` de um agente auditor para criar um revisor puro que aprova/veta propostas sem poder disparar comandos!
- **Persistência no `reset` e congelamento na largada:** os agentes customizados são **preservados automaticamente** ao chamar `POST /api/v1/round/reset` (use `DELETE /api/v1/identities/agents` ou `"reset_identities": true` apenas se quiser voltar aos 2 agentes padrão). Após abrir o `tick 0`, alterar agentes retorna `409 STATE_VERSION_CONFLICT`.
- **Subconjunto aderente (`multi_agent_proof`):** o apurador exige que **pelo menos `2` agentes** mantenham `>= 90%` de *heartbeats*, `>= 10` transações `committed` e `>= 2` navios — agentes especialistas extras (ex.: um `economy` dedicado ao abastecimento inicial) não derrubam sua nota!

```bash
curl -s -X POST https://t01.beatbot.com.br/api/v1/identities/agents \
  -H "Content-Type: application/json" \
  -d '{
    "replace_defaults": true,
    "agents": [
      {"actor_id": "team-01-fleet-coordinator", "role": "coordinator", "version": "coord-1.0.0"},
      {"actor_id": "team-01-bunker-economist", "role": "economy", "version": "econ-1.0.0"},
      {"actor_id": "team-01-route-navigator", "role": "navigation", "version": "nav-1.0.0"},
      {
        "actor_id": "team-01-safety-guardian",
        "role": "safety",
        "version": "safety-1.0.0",
        "scopes": ["telemetry:read", "events:read", "heartbeat:write", "decision:write"]
      }
    ]
  }' | python3 -m json.tool
```

Em todas as chamadas de escrita (`/api/v1/heartbeats`, `/api/v1/decisions`, `/api/v1/commands`, `/api/v1/interventions`), envie sempre os dois cabeçalhos correspondentes ao mesmo ator:
```http
Authorization: Bearer tok-team-01-navigation-agent-poc
X-Actor-Id: team-01-navigation-agent
```

> **ALERTA CRÍTICO (`HG_HUMAN_BYPASS`):** Jamais utilize o token `<team_id>-operator` para chamar `POST /api/v1/commands`! O gateway bloqueará a chamada com HTTP `403` (`HUMAN_BYPASS_ATTEMPT`) e registrará imediatamente o *hard gate* desclassificatório **`HG_HUMAN_BYPASS`**, zerando a nota final da rodada (`0.0000`). Credenciais humanas existem apenas para `/api/v1/interventions` (`ANNOTATE`, `REQUEST_DIAGNOSTIC`, `PAUSE_REQUEST`, `RESTART_AGENT`), que também devem ser evitadas em operação nominal para preservar `100%` de autonomia.

---

## 4. Observabilidade: REST, SSE, WebSocket e Webhooks HMAC

### 4.1. Endpoints REST de Consulta (detalhes completos em `/docs`)
- `GET /api/v1/state` — fotografia consolidada da rodada (`status`, `round_active`, `tick`), frota (`ships`), última telemetria conhecida, resumo contábil e status do *ledger*.
- `GET /api/v1/telemetry?ship_id=...&tick=...&limit=50` — histórico recente de pacotes de telemetria emitidos.
- `GET /api/v1/events?since_tick=0` — avisos antecipados (`_FORECAST`) e eventos ativos publicamente visíveis.
- `GET /api/v1/economy` — saldo em caixa (`cash_cent`), bloqueios de compra (`purchase_holds_remaining_cent`), reserva operacional obrigatória (`operational_reserve_cent`), saldo livre para novas compras (`available_cent`) e lançamentos contábeis (`postings`).
- `GET /api/v1/ledger?offset=0&limit=100` — entradas assinadas do razão e verificação da cadeia SHA-256 / Merkle.
- `GET /api/v1/result` — boletim oficial completo assinado pelo apurador independente (disponível após concluir os `432` ticks ou chamar `POST /api/v1/round/finalize`).

### 4.2. Canais de Streaming em Tempo Real
Todos os canais suportam filtro opcional por `event_types` (ex.: `telemetry,event,transaction,round_completed`) e por `ship_id`:
1. **Server-Sent Events (`GET /api/v1/stream`):**
   ```bash
   curl -N "https://t01.beatbot.com.br/api/v1/stream?event_types=telemetry,event,transaction"
   ```
2. **WebSocket Seguro (`wss://t0X.beatbot.com.br/ws/telemetry`):** entrega assíncrona contínua de envelopes JSON numerados por `seq` sem precisar abrir portas na máquina local da equipe.
3. **Webhooks HTTP/HTTPS com HMAC-SHA256 (`POST /api/v1/webhooks`):**
   - Caso queira receber *push* HTTP do servidor remoto, cadastre uma URL acessível publicamente pelo servidor (ex.: via Cloudflare Tunnel / ngrok ou host público) informando um `secret`.
   - Cada entrega trará `X-Ocean-Event`, `X-Ocean-Seq` e `X-Ocean-Signature-SHA256` (`HMAC-SHA256(secret, raw_body_bytes)` em hexadecimal).

---

## 5. Ciclo de Fases do Tick, Protocolo Multiagente e Performance HTTPS

Sua solução pode operar o simulador em **passo a passo determinístico** (`open-tick` -> agentes atuam -> `close-tick`, recomendado para orquestradores de agentes e LLMs) ou em **tempo contínuo** (`POST /api/v1/round/start` com `speed_ms`). Em ambos os modos, cada tick (`0` a `431`) obedece rigorosamente à mesma sequência de fases:

1. **Abertura do Tick (`POST /api/v1/round/open-tick` com `{"tick": t}`):**
   - No `tick 0`, registra os lançamentos econômicos de abertura (estoque inicial de carga e combustível).
   - Processa avisos e eventos do cenário, conclui serviços finalizados (creditando receitas de cargas entregues), encaminha navios na fila de atracação quando o berço está livre, emite a telemetria do tick e abre a janela de submissão (`window_open = true`).
2. **Pulso Vital Encadeado (`POST /api/v1/heartbeats` — Fase 40):**
   - Os agentes ativos da equipe devem enviar `1` *heartbeat* por tick enquanto `window_open == true`.
   - **Encadeamento obrigatório:** No primeiro *heartbeat* (`tick 0`), envie `"previous_heartbeat_message_id": null`. Nos ticks seguintes, envie **exatamente** o `heartbeat.message_id` retornado pelo servidor no último *heartbeat* aceito daquele mesmo agente! Quebrar esse elo gera `INTEGRITY_FAILURE`.
   - **Meta de elegibilidade:** Pelo menos `2` agentes elegíveis precisam manter cobertura válida em **`>= 90%` dos `432` ticks**.
3. **Cadeia Causal de Decisão e Comando (`proposal -> review -> command`):**
   Sempre que seus agentes decidirem executar uma ação em um navio naquele tick:
   - **Passo A — Proposta (`POST /api/v1/decisions`, ex.: agente `navigation`):** Envie `decision_kind: "proposal"` contendo `ship_id`, `decision_id`, `agent_role`, `policy_version`, `input_state_version` (igual ao `state_version` atual do navio), `input_refs` (`[{"message_id": ..., "payload_sha256": ...}]` da telemetria ou estado lido), `action`, `action_hash`, `materiality` (`{"kind": "service" | "event" | "plan_change", "key": "..."}`), `reason_codes` e `constraints_checked`. Guarde `decision.message_id` e `decision.integrity.payload_sha256` retornados!
   - **Passo B — Revisão Cruzada (`POST /api/v1/decisions`, outro agente, ex.: `safety`):** Um segundo agente distinto audita a proposta de forma independente e envia `decision_kind: "review"` com `proposal_message_id` (apontando para a mensagem da proposta), `proposal_agent_id`, `input_refs` (referenciando a proposta), o **mesmo `action_hash`** e `review_result: "approved"` (ou `"rejected"`, caso o revisor vete a manobra antes de enviar comando!).
   - **Passo C — Comando Atômico (`POST /api/v1/commands`, token de agente com `command:write`):** Somente após `proposal` e `review` aprovada estarem registradas (dica: se for comandar mais de um navio no mesmo tick, registre todas as decisões primeiro e envie todos os comandos em seguida!), submeta o comando referenciando `proposal_decision_id`, `review_decision_id`, `action_hash`, `expected_state_version == state_version`, `valid_from_tick == expires_at_tick == tick` e `idempotency_key`.
4. **Fechamento Físico e Econômico (`POST /api/v1/round/close-tick` com `{"tick": t}`):**
   - Bombeia lotes de combustível em abastecimento, aplica rampa de velocidade, integra avanço espacial e consumo, avança contadores de serviço, debita folha de tripulação/manutenção/eventuais multas, grava checkpoint e fecha a janela (`window_open = false`).
5. **Apuração Final (`POST /api/v1/round/finalize` após concluir os `432` ticks):**
   - Encerra *holds* pendentes, verifica toda a árvore de Merkle, executa o *commit-reveal* (no modo `official`), calcula as notas nas 5 dimensões e disponibiliza o boletim oficial completo em `GET /api/v1/result`.

### Regras Exatas de `action_hash`, `idempotency_key` e Cotas
- **`action_hash`:** SHA-256 hexadecimal de `b"ocean:action:v1\n" + JCS(action)` conforme **RFC 8785** (chaves ordenadas sem espaços; veja a implementação de referência em `starter/hello_ocean.py`).
- **`idempotency_key`:** Deve obrigatoriamente seguir o padrão `{team_id}:{round_id}:{ACTION_NAME}:{command_id}` (ex.: `team-01:round-2026-01:SET_SPEED:cmd-0001`). Reenviar exatamente o mesmo payload com a mesma chave retorna `duplicate` sem penalidade; reutilizar a mesma chave mudando qualquer campo gera `409 IDEMPOTENCY_CONFLICT`.
- **Limites operacionais:** Máximo de **`1` comando `committed` por navio por tick**, máximo de **`8` tentativas de comando mutável por equipe por tick** (`429 RATE_LIMITED` acima disso) e corpo HTTP limitado a **`64 KiB`** (`413 PAYLOAD_TOO_LARGE`).
- **Prova Multiagente (`multi_agent_proof`):** Exige simultaneamente `>= 2` agentes elegíveis com `>= 90%` de *heartbeats*, `>= 10` transações `committed` por agente, `>= 2` navios atendidos por cada agente e `>= 10` cadeias materiais creditadas (`credited_chains >= 10`, sem oscilações artificiais / *churn*).

### Boas Práticas de Performance HTTPS (~133 ms RTT até o Servidor)
Como os servidores `t01`–`t05.beatbot.com.br` estão hospedados em nuvem internacional (~133 ms de RTT), pequenos cuidados de rede aceleram drasticamente uma rodada de 432 ticks:
1. **Use sempre sessão HTTPS Keep-Alive persistente (`httpx.Client` ou `requests.Session`):** Crie o cliente HTTP **uma única vez** fora do loop de ticks e reutilize-o do `tick 0` ao `431`. Evitar novos handshakes TCP + TLS 1.3 a cada requisição reduz o tempo por chamada de **~425 ms para ~134 ms** e aproveita compressão `gzip` automática!
2. **Envie os *heartbeats* dos agentes em paralelo:** Como cada *heartbeat* depende apenas do `previous_heartbeat_message_id` do **próprio** agente, dispare os *heartbeats* de todos os agentes simultaneamente via `ThreadPoolExecutor` (sobre o mesmo `httpx.Client` / `requests.Session`) ou `asyncio.gather` (`httpx.AsyncClient`). Assim, todos os agentes batem ponto em ~1 único RTT (~134 ms total)!

---

## 6. Catálogo das 7 Ações Permitidas e Pré-Condições Físicas

Consulte os schemas detalhados em `https://t0X.beatbot.com.br/docs` (e em `round-package/catalog/actions.json` se disponível no repositório):

- **`SET_ROUTE`** — `{"route_id": "direct"}` ou `"protected"`
  - Permitida apenas em `AT_ORIGIN` e antes do primeiro movimento do navio.
- **`SET_SPEED`** — `{"speed_kn": "12.50"}` (string decimal entre `"0.00"` e `"18.00"`, sempre em passos de `"0.50"`, formato `^\d+\.\d0$`)
  - Permitida em `AT_ORIGIN`, `UNDERWAY`, `HOLDING` ou `AT_BUNKER_STATION` (proibida enquanto houver serviço ativo como `BUNKERING`, `REPAIRING` ou `UNLOADING`).
- **`HOLD`** — `{"reason_code": "MOTIVO_OPERACIONAL"}` (regex `^[A-Z][A-Z0-9_]{2,63}$`)
  - Permitida em `UNDERWAY` ou `HOLDING`. Define velocidade-alvo zero respeitando a rampa física de desaceleração por tick até atingir `0.00 kn` (`HOLDING`).
- **`BUY_FUEL`** — `{"quantity_t": 15}` (inteiro `>= 5`, múltiplo exato de `5`)
  - Permitida em `AT_ORIGIN` ou `AT_BUNKER_STATION` (na rota protegida). Exige navio totalmente parado (`effective_speed_kn == "0.00"`), espaço livre no tanque, fornecedor disponível na janela atual e saldo livre (`available_cent`) suficiente acima da reserva operacional obrigatória. O combustível entra em lotes a cada fechamento de tick enquanto o navio permanece em `BUNKERING`.
- **`CANCEL_FUEL`** — `{"order_id": "ord-..."}`
  - Permitida em `BUNKERING`. Cancela a fração ainda não transferida da ordem e libera o saldo retido correspondente.
- **`REPAIR`** — `{"fault_id": "..."}`
  - Permitida apenas com o navio totalmente parado (`effective_speed_kn == "0.00"`), em estado compatível (`HOLDING`, `AT_ORIGIN`, `AT_BUNKER_STATION`, `WAITING_BERTH`, `SAFE_STOP`) e quando houver avaria mecânica ativa no navio.
- **`SAFE_STOP`** — `{"reason_code": "CONTENCAO_EXTREMA"}`
  - Permitida em `HOLDING`. **Atenção:** é um estado terminal de preservação que encerra a viagem competitiva daquele navio, gerando custos elevados de custódia. Use apenas em emergência real incontornável.

> **Ações Proibidas (`forbidden_actions`):** `BUY_CARGO`, `EMERGENCY_COMMAND`, `DEPLOY`, `CHANGE_CONFIG` e `ROLLBACK`. Submeter qualquer uma dessas cinco ações aciona imediatamente o *hard gate* desclassificatório **`HG_UNAUTHORIZED_COMMAND`**.

---

## 7. Frentes de Investigação Marítima e Econômica

Para construir agentes realmente competitivos sem depender de sorte, investigue com atenção estas três camadas do simulador:

### 7.1. Unidades Exatas e Dinâmica Hidrodinâmica
- **Sem ponto flutuante impreciso:** Massas de carga/combustível e distâncias náuticas aparecem em milésimos inteiros (`_milli`, onde `1000` equivale a `1,000 t` ou `1,000 nm`). Valores monetários aparecem sempre em centavos inteiros de dólar simulado (`_cent`, onde `100` equivale a `D$ 1,00`).
- **Inércia e Rampa de Velocidade:** Embarcações carregadas não aceleram nem freiam instantaneamente em um único tick. Observe na telemetria quanto `effective_speed_kn` varia por tick após um `SET_SPEED` ou `HOLD` — isso é vital sempre que uma ação exigir `effective_speed_kn == "0.00"`!
- **Consumo Não-Linear vs. Prazo Contratual (`deadline_tick`):** A queima horária de combustível cresce de forma acentuada (cúbica) com a velocidade, além do consumo basal de hotelaria de bordo. Navegar rápido demais devora combustível e encarece a operação; navegar devagar demais pode estourar o `deadline_tick` de entrega (lembrando que a descarga portuária `UNLOADING` também consome vários ticks após a chegada!).
- **Reserva Mínima Intocável (`reserve_min_milli`):** Todo navio possui um volume mínimo obrigatório de segurança no tanque. Fechar um único tick abaixo de `reserve_min_milli` (ou secar o tanque em `STRANDED`) gera pesadas multas e aciona `HG_NEGATIVE_OR_UNSAFE_FUEL`.

### 7.2. Fluxo de Caixa Fechado (`D$`) e Composição da Nota
- Inspecione `GET /api/v1/economy` desde o `tick 0`: entenda quanto do caixa inicial é consumido pelos lançamentos de abertura, como o servidor calcula `operational_reserve_cent` (protegendo folha de tripulação, manutenção por tick, tarifas portuárias e provisões futuras) e quanto sobra em `available_cent` para compras de combustível.
- Observe também como o preço da tonelada de combustível varia entre o porto de origem (ao longo do tempo) e a estação intermediária da rota protegida, e note que óleo excedente não utilizado no fim da viagem não se converte em receita na margem líquida.
- As 5 dimensões oficiais da nota final são: **Segurança (`25%`)**, **Economia (`25%`)**, **Serviço (`20%`)**, **Engenharia e Resiliência (`20%`)** e **Observabilidade e Governança (`10%`)**.

### 7.3. Eventos Oceânicos, Crises Mecânicas e Degradação de Telemetria
Durante as 72 horas de travessia, o oceano apresentará perturbações reais que exigem leitura contínua de `/api/v1/events` e dos indicadores de cada navio em `/api/v1/state`:
- **Clima Severo e Trechos Expostos:** Tempestades impõem limites seguros de velocidade e aumentam o consumo quando o navio atravessa segmentos geográficos expostos da rota. Exceder a velocidade segura sob tempestade degrada a qualidade da carga, gera multas e reduz notas de segurança e resiliência.
- **Restrições Logísticas e Portuárias:** Fornecedores intermediários e berços de descarga podem sofrer janelas temporárias de indisponibilidade. Antecipar avisos (`_FORECAST`) permite ajustar o ritmo de navegação em vez de queimar combustível à toa para esperar fila!
- **Avarias Mecânicas e Escalonamento:** Caso um navio apresente falha de motor ativa (`e04_active`), observe os limites de velocidade impostos, os prazos de resposta (`trace_due_tick` e `safe_action_due_tick`) e a necessidade de desacelerar até parar antes de iniciar `REPAIR`, evitando multas de exposição prolongada ou escalonamento severo da falha.
- **Lacunas de Comunicação (`state_age_ticks` / `telemetry_gap_ticks`):** Nem toda anomalia aparece anunciada de antemão em `/api/v1/events`! Monitore continuamente `state_age_ticks` em cada navio: se pacotes de telemetria falharem consecutivamente e a idade do estado ultrapassar o limite seguro (`state_age_ticks > 2`), o estado público permanecerá congelado na última leitura confirmada e o servidor bloqueará comandos agressivos (como `SET_SPEED`), aceitando apenas contenção segura (`HOLD` ou `SAFE_STOP`) até o restabelecimento do sinal.

---

## 8. Hard Gates e Diagnóstico de Erros HTTP

### 8.1. Os Principais Hard Gates (Evite a Qualquer Custo!)
Cada comando rejeitado reduz sua métrica de validade de comandos em Segurança, e violações graves acionam *Hard Gates*:
- **`HG_HUMAN_BYPASS`** — Desclassificação imediata (`score_final = 0`): uso de token humano em rota de comando de navio ou intervenção inválida pós-congelamento.
- **`HG_UNAUTHORIZED_COMMAND`** — Desclassificação imediata (`score_final = 0`): envio de qualquer ação listada em `forbidden_actions`.
- **`HG_NEGATIVE_OR_UNSAFE_FUEL`** — Zera toda a dimensão Segurança (`-25` pts): fechar qualquer tick com combustível abaixo de `reserve_min_milli` ou entrar em `STRANDED`.
- **`HG_SINGLE_AGENT`** — Zera Engenharia/Resiliência (`-20` pts) e retira elegibilidade ao ranking: falhar nos mínimos de `multi_agent_proof` (2 agentes, `>= 90%` *heartbeats*, `>= 10` transações e `>= 2` navios por agente, `>= 10` `credited_chains`).
- **`HG_UNAUDITABLE`** — Zera Observabilidade/Governança (`-10` pts) e retira elegibilidade: acumular `3` ou mais tentativas de comandos órfãos (`proposal`/`review` ausentes, do mesmo agente ou com `action_hash` divergente).

### 8.2. Mapa Rápido de Respostas de Erro HTTP
- **`401 UNAUTHENTICATED`** — Ausência de `Authorization: Bearer ...`, token inválido ou divergente do `X-Actor-Id`.
- **`403 FORBIDDEN_SCOPE` / `HUMAN_BYPASS_ATTEMPT`** — Papel sem escopo para a rota, navio de outra equipe ou ação proibida.
- **`409 STATE_VERSION_CONFLICT`** — Mais de 1 comando para o mesmo navio no tick, `expected_state_version` desatualizado, ou tentativa de comando não-conservador com telemetria defasada (`state_age_ticks > 2`).
- **`409 COMMAND_EXPIRED` / `SERVICE_IN_PROGRESS` / `IDEMPOTENCY_CONFLICT`** — Envio fora da janela aberta (`window_open == false`), navio ocupado em serviço (`BUNKERING`/`REPAIRING`/`UNLOADING`), ou reuso indevido de `idempotency_key` com payload diferente.
- **`422 INVALID_SCHEMA` / `SHIP_NOT_STOPPED` / `OPERATIONAL_RESERVE_REQUIRED` / `SUPPLIER_UNAVAILABLE` / `INTEGRITY_FAILURE`** — Formato inválido (ex.: velocidade fora do passo `.00`/`.50`, `idempotency_key` fora do padrão), navio ainda em movimento ao tentar abastecer/reparar, compra invadindo a reserva financeira, fornecedor fechado ou erro em `action_hash`/cadeia de *heartbeat*.

---

## 9. Entrega no Repositório da Equipe e Documentação da Solução

Cada equipe possui **seu próprio repositório oficial criado pelo instrutor**. Além da auditoria criptográfica que já fica gravada automaticamente no servidor da equipe (`t01`–`t05.beatbot.com.br`), a avaliação da banca será feita sobre **o que sua equipe subir no repositório do time**.

Certifique-se de versionar e fazer `git push` de **três entregas obrigatórias**:

### 9.1. Código-Fonte Completo da Solução Multiagente
- Todo o código dos agentes, orquestrador de ticks, políticas de decisão/revisão, cálculo de `action_hash` e cliente HTTP.
- Arquivo de dependências (`requirements.txt` ou `pyproject.toml`), `.env.example` (sem segredos/chaves de API reais!) e instruções claras no `README.md` de como executar a solução apontando para `https://t0X.beatbot.com.br`.

### 9.2. Relatório / Diário de Engenharia e Processo de Organização Multiagente
Documente no repositório (em `docs/diario-engenharia.md` ou diretamente no `README.md`) **o que a equipe fez, o que descobriu e como chegou ao desenho final de agentes e ao score alcançado**:
1. **Arquitetura Final de Agentes:** quantos agentes sua equipe definiu, quais papéis (`navigation`, `safety`, `economy`, `coordinator`), escopos RBAC e divisão de navios escolheram, como funciona o fluxo *Maker-Checker* (proposta -> auditoria/veto -> comando) e por que chegaram a essa organização.
2. **Descobertas do Domínio:** principais parâmetros físicos, hidrodinâmicos, econômicos e regras de resposta a eventos (`E01`–`E06` e quedas de telemetria) que a equipe mapeou investigando o simulador.
3. **Linha do Tempo de Iterações e Evolução do Score:** quais hipóteses testaram em `conformance`, quais erros/rejeições encontraram no `/api/v1/ledger`, quais ajustes destravaram cada ganho de nota e como validaram a generalização no modo `official`.

### 9.3. Evidência do Resultado Final Alcançado
- Salve no repositório (ex.: `results/final_result.json`) o retorno de `GET https://t0X.beatbot.com.br/api/v1/result` da melhor rodada oficial da equipe e registre no documento o `score_final`, as notas nas 5 dimensões, o status de `multi_agent_proof` e o `merkle_root_sha256` correspondente.

### Modelo Sugerido para `docs/diario-engenharia.md`

```markdown
# Diário de Engenharia e Arquitetura Multiagente — Equipe `team-0X`

## 1. Visão Geral da Organização Final de Agentes
- **Servidor da Equipe:** `https://t0X.beatbot.com.br`
- **Topologia Registrada (`/api/v1/identities/agents`):**
  - [Liste cada agente, papel, escopos, navios sob responsabilidade e função na tomada de decisão]
- **Dinâmica de Cooperação e Governança (Maker-Checker):**
  - [Explique como um agente propõe, como outro audita/veta independentemente e como evitaram conflitos e Hard Gates]

## 2. Principais Descobertas Físicas, Econômicas e Operacionais
- **Física e Consumo:** [Rampas de aceleração/frenagem, velocidades ótimas, reservas mínimas]
- **Gestão de Caixa (`D$`) e Bunker:** [Reserva operacional, estratégia de compra na origem vs. rota protegida]
- **Tratamento de Eventos e Crises:** [Tempestades, indisponibilidades portuárias, falhas mecânicas e perda de telemetria]

## 3. Evolução Iterativa até o Score Final

### Iteração #01 — [Título curto do experimento]
- **Modo:** `conformance` | `official`
- **Hipótese & Mudança:** [O que testamos na organização ou política dos agentes?]
- **Diagnóstico (`/api/v1/ledger`):** [Quais erros ou gargalos apareceram e como corrigimos?]
- **Score Obtido:** `score_final = XX.XXXX` (Seg: `XX` | Eco: `XX` | Srv: `XX` | Eng: `XX` | Gov: `XX`)

### Iteração Final — Consolidação em `official`
- **Score Final Alcançado:** `XX.XXXX` (`rank_eligible = true`)
- **Prova Multiagente (`multi_agent_proof`):** `compliant = true`, `credited_chains = XX`
- **Selo Criptográfico (`merkle_root_sha256`):** `...`
```

---

## 10. Roteiro Sugerido de Evolução

1. **Marco 1 — Conectividade Remota e Pulso Limpo (`conformance`):**
   - Explore `https://t0X.beatbot.com.br/` e `/docs`, configure sessão HTTPS Keep-Alive persistente e construa um loop capaz de percorrer os `432` ticks mantendo os *heartbeats* encadeados em paralelo com `>= 90%` de cobertura (`previous_heartbeat_message_id`), sem disparar comandos inválidos.
2. **Marco 2 — Primeira Travessia Completa Multiagente (`proposal -> review -> command`):**
   - Estruture sua organização inicial de agentes cooperantes (propositor != revisor), valide `action_hash` e `idempotency_key`, abasteça a frota respeitando `available_cent` e conduza os `4` navios até concluir a descarga nos portos de destino sem acionar *Hard Gates*.
3. **Marco 3 — Refinamento da Organização Multiagente e Otimização (`conformance`):**
   - Analise `/api/v1/economy`, `/api/v1/events`, `/api/v1/ledger` e `/api/v1/result` para identificar onde houve perda de margem, excesso de consumo cúbico, filas portuárias, avarias mecânicas ou quedas de telemetria (`state_age_ticks`). Evolua a especialização dos seus agentes e registre cada descoberta no Diário de Engenharia!
4. **Marco 4 — Validação no Cenário `official` e Entrega no Repositório:**
   - Execute a rodada em `scenario_mode: "official"` **sem fixar ticks manualmente no código**, salve o resultado oficial (`GET /api/v1/result`) e suba no repositório da equipe o código completo junto com a documentação de todo o processo até o score final!

Bons ventos, decisões auditáveis e excelente engenharia para a sua equipe!
