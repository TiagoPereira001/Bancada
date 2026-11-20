# Handoff da Baylight

> **Escrito a 20/11/2026**, para quem pegar neste projeto a seguir (uma pessoa, ou outra sessão do Claude Code). Diz onde estamos, o que já foi feito, em que estado está cada coisa, o que falta e o que eu faria agora.

> **Nota posterior, no mesmo dia:** o produto passou a chamar-se **Baylight** (domínio `baylight.dev`) e este documento foi atualizado com o novo nome. Onde diz "o repositório chama-se Bancada" refere-se ao nome do repositório no GitHub, que só muda quando o Tiago o renomear. O CI vermelho da secção 3 foi corrigido no commit `0113d81` (ver o "Registo" do `CLAUDE.md`). O que o handoff diz do commit `9cb3331` ser em nome do Claude é verdade e não se reescreve.
>
> Tudo o que aqui está como "verificado" foi confirmado **hoje**, a correr os comandos num ambiente Linux limpo (clone de `dev`, commit `b440141`). O que não foi confirmado está dito como tal. Quando este documento e o código discordarem, **manda o código**.
>
> Leitura recomendada, por ordem: este ficheiro, [`CLAUDE.md`](CLAUDE.md) (regras e memória), [`docs/PROJETO-BAYLIGHT.md`](docs/PROJETO-BAYLIGHT.md) (o projeto inteiro), [`AI.md`](AI.md) (mapa do código), [`docs/adr/0001-arquitetura-para-durar.md`](docs/adr/0001-arquitetura-para-durar.md) (para onde vai).

---

## 1. Em 30 segundos

- **Baylight**: folhas de obra digitais para oficinas, feitas para o chão da oficina (tablet partilhado, PIN, quadro como o tablier de um carro). Multi-oficina: cada oficina só vê os seus dados. Dono e autor: **Tiago Dias Pereira**. Nasceu na oficina Duarte & Raposo (Canhoso, Covilhã); desde 18/11/2026 é um **projeto pessoal**.
- **Funciona**, com dados de demonstração: API Node 22 + Express 5 + SQL Server 2022, frontend React 19 + Vite 8 (PWA). 54 testes passam.
- **Nunca foi usada numa oficina com dados reais** e **não está publicada**. O ensaio na oficina está preparado, mas ainda não se fez.
- **O CI está vermelho** em `main` e `dev` desde 18/11 (ver a secção 3). É a primeira coisa a resolver.
- A arquitetura para o futuro está **decidida e por fazer**: PostgreSQL com isolamento por oficina na própria base de dados, TypeScript, camadas, auditoria, migrações, produção no servidor pessoal do Tiago (Coolify).

## 2. Regras de trabalho (resumo; o detalhe está no `CLAUDE.md`)

- **Português de Portugal em tudo** (código, interface, commits, documentos, respostas). **Sem travessões** (—) nos textos: usar dois pontos, vírgulas ou parênteses.
- **Git:** só `main` (estável) e `dev` (trabalho). Trabalha-se **diretamente na `dev`**, sem ramos por tarefa. A `main` recebe a `dev` por PR e **o merge é do Tiago**. **Commits só em nome do Tiago, sem linhas de coautoria. Nada de `push --force`.** O histórico foi reescrito a 18/11 para cumprir isto: os hashes que aparecem em documentos antigos (por exemplo, `523068b`) já não existem.
- **Nunca inventar** clientes, testemunhos, métricas, preços nem funcionalidades. "Por fazer" é "por fazer". Dados de demonstração são sempre fictícios.
- **Segredos nunca** no código, no git ou no chat. As passwords geram-se ao correr.
- **Regras do código que não se partem** (detalhe no `AI.md`): cada oficina só vê os seus dados; autoria vem da sessão; nada se apaga a sério; dinheiro só em SQL com `DECIMAL`; SQL sempre parametrizado; CSP rigorosa; a Baylight **não emite faturas**; nenhuma porta aberta à rede por omissão.
- **Antes de dar uma tarefa por pronta:** `cd backend && npm test`, `cd frontend && npm run lint && npm run build`, e, para mexidas no visual, ver no browser nos dois temas e em 390, 1180 e 1440 px. **A documentação faz parte do pronto** (atualizar `CLAUDE.md`, `AI.md`, `PRODUCT.md` e `docs/PROJETO-BAYLIGHT.md` na mesma tarefa).

## 3. Onde estamos (estado verificado a 20/11/2026)

| | Estado | Como foi visto |
|---|---|---|
| Git | `main`, `dev` e o meu `HEAD` no mesmo commit, `b440141` (PR #20, `dev` para `main`, 18/11). Sem PRs abertos. O repositório no GitHub chama-se agora **`Bancada`** (o remoto local ainda diz `Projeto_final_UBI`; o GitHub redireciona) | `git`, GitHub |
| Testes da API | **54 em 54 passam** (20,8 s), contra um SQL Server 2022 num container | corrido hoje |
| Frontend | `lint` sem erros; `build` ok (JavaScript inicial de 261 KB, 80,5 KB gzip no ficheiro principal) | corrido hoje |
| `npm audit` do frontend | 0 vulnerabilidades | corrido hoje |
| **`npm audit` do backend** | **falha: 5 vulnerabilidades (3 altas, 2 moderadas)** | corrido hoje, igual ao CI |
| **CI no GitHub** | **vermelho**: as três últimas corridas falharam (PR #20, e os *pushes* para `dev` e `main`). Falha o passo `npm audit --omit=dev --audit-level=high` do trabalho "API", por isso os testes da API ficam "skipped". Os trabalhos "Frontend" e "Infraestrutura" estão verdes | GitHub Actions, corridas 44341613006, 44341881263 e 44501148335 |

**O CI vermelho, em pormenor.** As vulnerabilidades são novas (a 30/09 o CI passava) e vêm de dependências indiretas:
- `body-parser` 2.0.0 a 2.3.0 (**alta**, negação de serviço com bytes a mais depois de um corpo comprimido), via `express`;
- `@azure/msal-node` até 7.0.2 (**alta**, pode revelar parâmetros do fluxo de autorização), via `@azure/identity`, via o driver do SQL Server;
- `qs` até 6.16.0 e `ip-address` até 10.7.3 (moderadas, negação de serviço).

O `npm audit` diz que há correção com `npm audit fix`. **Não a apliquei** (não me pediram código) e, por isso, também não corri os testes com as versões novas.

## 4. O que já fizemos

### Antes desta conversa
- **Jun a jul/2026:** primeira fase (requisitos, modelo de dados, SQL Server em Docker), API de colaboradores, clientes e autocaravanas, login com JWT, início do frontend (não arrancava).
- **26/09/2026:** revisão completa e passagem a **multi-oficina**: 6 tabelas com chaves estrangeiras compostas por oficina, sessões em cookies `httpOnly`, modo bancada com PIN, registo público; frontend refeito (quadro-tablier, nova entrada pela matrícula, folha, histórico, clientes, veículos, equipa, definições, temas claro e escuro); testes automáticos e CI.

### Nesta conversa (setembro e outubro de 2026)
- **27/09:** documentos de base (`CLAUDE.md`, `PRODUCT.md`, `DESIGN.md`, `AI.md`); relatório da UC e documento das ferramentas (LaTeX); `LICENSE` de todos os direitos reservados e aviso no README; **auditoria completa** (`docs/auditoria.md`, 82 verificações, nenhum achado crítico nem alto); **revisão de código e dívida técnica** (`docs/revisao-codigo.md`, 3 bugs confirmados e 12 itens de dívida); guia de arranque para macOS e Windows no README.
- **28/09:** interface revista com o plugin `frontend-design` (menos "cara de IA": raios por objeto, visor no tablier, dados da folha em caixas); **`Iniciar Baylight.command`** (duplo clique no Mac); **reauditoria** (corrigiu QA-005, SEC-009, REL-005 e REL-006).
- **28 e 29/09:** **`docs/infraestrutura.md`**: proposta de publicação (Caddy, SQL Server Express, cópias cifradas com `age`, restauro, retrocesso) **validada numa cópia em `localhost`**, com `docker-compose.prod.yml` e `infra/`. Nada disto está em uso.
- **30/09:** crítica de design do quadro (28/40) e os cinco passos do plano, todos entregues:
  1. `layout`: o ecrã de descanso do tablet mostra o tablier grande e só de leitura, e há um aviso 30 s antes do bloqueio por inatividade;
  2. `harden`: "Entregue" pede dois toques, tratamento de erros e uma corrida real corrigida no `useRecurso`;
  3. `adapt`: tablier maior no tablet partilhado, para se ler a 1 ou 2 m;
  4. `quieter`: botão "Sair do tablet" neutro e um só botão âmbar por ecrã no quadro vazio;
  5. `polish`: mensagem do PIN que já não empurra as teclas, quadro sem ligação diz de quando são os números, filtro e pesquisa sobrevivem a ir ver uma folha, títulos das páginas, alvos de toque, 320 px sem scroll de lado.
- **01/10:** **modo oficina** (`Iniciar Baylight na oficina.command`, `BANCADA_REDE=1`): o tablet abre a Baylight pelo Wi-Fi a partir do Mac, a porta só se abre nessa corrida. Guia do ensaio em `docs/ensaio-oficina.md` (guião de 11 tarefas). Contexto: o tablet da oficina é **Android** e o diagnóstico liga por **Bluetooth**, por isso não mexe no Wi-Fi.

### Feito pelo Tiago a 18/11/2026
- **ADR 0001** (arquitetura alvo) e **`docs/PROJETO-BAYLIGHT.md`** (o projeto inteiro num só documento, com análise de produto), **`docs/linkedin.md`** (textos para o perfil e uma publicação).
- **Faturação:** pesquisa dos programas certificados com API (InvoiceXpress, Moloni, Vendus, TOConline, Cegid Jasmin/Primavera). Direção recomendada: integrar **um** deles atrás de uma "porta" de faturação. Preços **não** verificados.
- **Ramos simplificados** (só `main` e `dev`), commits só em nome do Tiago, README e `AI.md` atualizados. O PR #20 passou a `dev` para a `main`.

## 5. Estado de cada coisa

| Área | Estado | Nota |
|---|---|---|
| API (Node 22, Express 5, SQL Server 2022) | **funciona, 54 testes verdes** | CI vermelho por dependências (secção 3) |
| Isolamento entre oficinas | **feito**, em três sítios (API, chaves estrangeiras compostas, testes) | no alvo passa para Row-Level Security |
| Modo bancada (PIN) | **feito** | PIN bloqueia 5 minutos ao fim de 5 falhas; o bloqueio por inatividade (5 min) **só existe no ecrã**, no servidor a sessão dura 12 h |
| Frontend (React 19, Vite 8, PWA) | **feito**, revisto em design até ao `polish` | verificado só no Chromium com toques simulados, não num tablet |
| Lançadores do Mac | **feitos**, testados em Linux com terminal simulado | Mac, Docker Desktop e Android **não testados** |
| Documentação de produto, design e código | **feita** | ver incoerências na secção 7 |
| Auditoria e revisão de código | **feitas, achados quase todos abertos** | secção 6 |
| Proposta de infraestrutura (`docs/infraestrutura.md`) | **validada numa cópia** | descreve a versão **SQL Server**: fica em parte ultrapassada pelo ADR (Postgres, Coolify). Ainda não está em uso |
| Ensaio na oficina | **preparado, não realizado** | guia em `docs/ensaio-oficina.md` |
| Arquitetura alvo (ADR 0001) | **decidida, por fazer** | nada está no código |
| Faturação por programa certificado | **só pesquisa** | por decidir; a Baylight continua a não emitir faturas |
| Publicação online / servidor | **não existe** | o servidor do autor (Time4VPS, Coolify) está fora deste repositório, em `~/Desktop/Server` |
| Texto legal (termos, privacidade, acordo RGPD) | **não existe** | obrigatório antes do registo público e de dados reais |
| Relatório LaTeX da UC (`Relatorio/`) | **feito em setembro**, PDFs compilados | capturas de ecrã anteriores à revisão de 28/09; **não sei se já foi entregue**, não lhe toquei |
| Utilizadores reais, métricas, preços | **nenhum** | e nada disto se inventa |

## 6. O que falta fazer

### Já conhecido, por prioridade

**A. Agora**
1. **Pôr o CI verde:** corrigir o `npm audit` do backend (secção 3).

**B. Antes do ensaio (pequeno, útil já)**
2. **Impor o bloqueio por inatividade no servidor.** Hoje os 5 minutos são só do ecrã; se o Android descartar a página em segundo plano (depois do diagnóstico, por exemplo) e a recarregar, a sessão por PIN continua em nome do mecânico anterior, até 12 h.
3. **REL-003:** sem rede, ao recarregar a página aparece o ecrã de entrada em vez de uma mensagem de "sem ligação". O Wi-Fi da oficina vai falhar.
4. **REL-001:** sem `errorElement`, um erro mostra o ecrã do React Router em inglês.
5. **BUG-01 (alta):** guardar as notas de uma folha apaga o que um colega gravou noutro dispositivo (o pedido envia os três campos, não só o que mudou).
6. **Teste em casa** do modo oficina com o telemóvel, na véspera (ele só foi testado em Linux).

**C. O ensaio (é o Tiago que o faz)**
7. Seguir `docs/ensaio-oficina.md`, escrever as notas no próprio dia e passá-las a limpo num documento à parte, **com a data e as condições**. Só o que for medido entra no relatório e nos textos públicos.

**D. Depois: o caminho do ADR (fases)**
8. **Fase 1, Fundação:** TypeScript, camadas, **PostgreSQL com RLS**, migrações versionadas, auditoria, testes de dinheiro e de isolamento. Pronto quando os testes estiverem verdes, os **totais iguais ao cêntimo** e a interface não mudar. Antes de migrar, **guardar os totais e o IVA de um conjunto de folhas calculados pelo SQL Server atual**.
9. **Fase 2, produção privada:** Coolify, imagem no CI/GHCR, cópias cifradas fora da VM com restauro testado, monitorização, a Duarte & Raposo como piloto.
10. **Fase 3, público:** domínio, email (recuperação de password), texto legal, registo aberto.
11. **Fase 4, crescer:** faturação certificada por API, notificações ao cliente, fotografias, auditoria visível, orçamentos.

### Achados abertos de auditorias (para não perder)
- **Revisão de código** (`docs/revisao-codigo.md`): BUG-02 (o bloqueio do PIN contorna-se com pedidos em simultâneo), BUG-03 (Definições não atualiza depois de desligar os tablets), DT-001 (a regra do dinheiro escrita em 10 sítios) e mais 11 itens de dívida. **Muitos desaparecem ou mudam com a reescrita do ADR**: convém decidir quais se corrigem agora e quais se levam como casos de teste para a versão nova.
- **Auditoria** (`docs/auditoria.md`): REL-001 e REL-003 (acima) e outros menores; SEC-010 (o container da API recebe a password do `sa`) fica resolvido só na proposta de produção.
- **Decisões de design em aberto** (da crítica do quadro): um só âmbar na Folha e na nova entrada (o segmento escolhido e o "Guardar notas"), gravação automática das notas, e o nome da página principal (o separador diz "Oficina", o resto diz "Quadro").
- **Pequenos:** avisos de erro com `role="status"` em vez de `alert`; a mensagem do PIN em telemóvel ainda empurra o teclado uma linha quando é comprida; a barra de topo do ecrã de descanso parte o nome da oficina em duas linhas a 390 px.
- **Relatório:** refazer as capturas de `Relatorio/Anexos/ecras/` e recompilar o PDF (se ainda interessar).

## 7. Incoerências que encontrei hoje (nenhuma corrigida)

1. **CI vermelho** (secção 3). O PR #20 entrou na `main` com o CI vermelho; não sei se foi visto.
2. **52 testes, mas são 54:** `docs/PROJETO-BAYLIGHT.md` (secção 5) e o ADR 0001 dizem 52. O código, o `README.md`, o `AI.md` e o `CLAUDE.md` dizem 54, e hoje contei 54 e passaram os 54.
3. **Nome do repositório:** no GitHub é `Bancada`; o `README.md` (selo do CI, `git clone`, `cd Projeto_final_UBI`) e o remoto local ainda dizem `Projeto_final_UBI`. O GitHub redireciona, por isso funciona, mas os guias de arranque ficam com o nome antigo. `docs/linkedin.md` já usa o novo.
4. **`CLAUDE.md`, "Estado atual":** o título ainda diz 01/10/2026 e o texto descreve o trabalho "por ramos" (PRs do ramo `claude/ecstatic-lamport-81qsk8`, que já não existe). O "Registo" já tem a entrada de 18/11. Também diz duas coisas opostas sobre commits: "terminam com as linhas de atribuição que o ambiente indicar" e "sem linhas de coautoria". **Segui a segunda**, que é a do Tiago e a mais recente.
5. **`docs/infraestrutura.md`** descreve SQL Server Express + Caddy; o ADR escolhe PostgreSQL + Coolify. A proposta é histórica e os scripts de `infra/` e o `docker-compose.prod.yml` seguem a versão SQL Server.
6. **Sessão:** `docs/PROJETO-BAYLIGHT.md` diz que "a sessão termina ao fim de 5 minutos parada". É verdade só no ecrã (ponto 2 da secção 6).

## 8. O que eu faria agora

Nada disto foi feito: **só o ponto 1 mexe no que é urgente**, e peço-te o "sim" antes de alterar código.

1. **Pôr o CI verde (pequeno, urgente).** Na `dev`: `npm audit fix` no backend, correr os 54 testes e o `audit` outra vez, ver que a API arranca e que o login e as folhas continuam a funcionar, e só então enviar. Risco: a atualização do `express`/`body-parser` ou do `qs` mudar algum comportamento de leitura de pedidos; os testes cobrem boa parte, e vê-se logo.
2. **Corrigir as incoerências da secção 7** (52 para 54, nome do repositório nos guias, título do "Estado atual", a contradição sobre as linhas de coautoria): é só documentação, num commit à parte.
3. **Fazer as quatro correções que o ensaio vai testar** (pontos 2 a 5 da secção 6), uma de cada vez, com um teste automático por correção:
   - bloqueio por inatividade no servidor;
   - REL-003;
   - REL-001;
   - BUG-01.
   Faria estas e não mais, porque tudo o que é só da camada de dados vai ser reescrito na Fase 1. Cada uma fica como **caso de teste** que a versão nova também tem de passar.
4. **Esperar pelo ensaio** e só depois começar a Fase 1, para o que aprendermos com os mecânicos entrar no desenho. Entretanto, o único trabalho de Fase 1 que não perde nada e que já se pode fazer: **o conjunto de folhas com os totais e o IVA calculados pelo SQL Server atual** (o "ouro" a que a migração tem de ser igual ao cêntimo).
5. **Escrever as notas do ensaio** contigo, no próprio dia ou no seguinte, e transformá-las numa lista de problemas por gravidade, com a captura ao lado.

## 9. Perguntas para o Tiago

1. Posso corrigir já o CI (ponto 1 da secção 8)? E o resto da secção 8 vai por esta ordem?
2. **Quando é o ensaio** e já tens o Mac e o tablet em casa para o teste da véspera?
3. O repositório **mudou de nome de propósito** para `Bancada`? Atualizo os guias do README?
4. **Que programa de faturação usa a Duarte & Raposo hoje?** É a pergunta que decide qual integrar primeiro (secção 9 do `PROJETO-BAYLIGHT.md`).
5. Os BUG-02, BUG-03 e a dívida técnica: corrijo agora ou levo-os como testes para a reescrita?
6. Os P2 de design (âmbar, notas automáticas, "Oficina" ou "Quadro"): quando quiseres decidir.
7. O relatório LaTeX da UC: já foi entregue? Mexo-lhe ou deixo como está?

## 10. Como retomar (comandos)

```bash
# SQL Server para os testes (as passwords geram-se na hora; nada se escreve no repositório)
nohup dockerd > /tmp/dockerd.log 2>&1 &            # só se o Docker não estiver a correr
export DB_ADMIN_PASSWORD="Aa1!$(openssl rand -hex 10)" DB_PASSWORD="Bb2!$(openssl rand -hex 10)"
docker run -d --name bancada_sql_teste --platform linux/amd64 -e ACCEPT_EULA=Y \
  -e MSSQL_SA_PASSWORD="$DB_ADMIN_PASSWORD" -p 127.0.0.1:1433:1433 mcr.microsoft.com/mssql/server:2022-latest

cd backend  && npm ci && npm test                  # 54 testes (recria a BD Bancada_Teste)
cd frontend && npm ci && npm run lint && npm run build
cd backend  && npm audit --omit=dev --audit-level=high     # é o que o CI corre
docker rm -f bancada_sql_teste                     # no fim
```

- **Para ver a app:** `docker compose --profile app up -d --build` (ou `Iniciar Baylight.command` no Mac) e `http://localhost:3000`. Os dados de demonstração criam-se com `npm run db:seed`; as passwords e os PINs só aparecem no terminal. **Ensaio na oficina:** `Iniciar Baylight na oficina.command`.
- **Armadilhas desta máquina de trabalho** (estão no `CLAUDE.md`, "Ambiente das sessões na nuvem"): `pkill -f` com um padrão que também aparece no comando mata a própria shell; `docker build` não confia na CA do ambiente (`npm ci` falha dentro da imagem), por isso testa-se numa cópia com a CA no Dockerfile; o limite de logins (8 em 15 min) obriga a reiniciar a API entre percursos no browser; o Chromium usa `executablePath: '/opt/pw-browsers/chromium'`.

## 11. Mapa rápido

| Para... | Ver |
|---|---|
| o produto | `PRODUCT.md`, `docs/PROJETO-BAYLIGHT.md` |
| as regras e a API | `AI.md` |
| a memória e as decisões | `CLAUDE.md` |
| o visual | `DESIGN.md`, `frontend/src/styles/tokens.css`, `.impeccable/` |
| a arquitetura futura | `docs/adr/0001-arquitetura-para-durar.md` |
| o ensaio | `docs/ensaio-oficina.md` |
| achados e dívida | `docs/auditoria.md`, `docs/revisao-codigo.md` |
| publicar (versão SQL Server) | `docs/infraestrutura.md`, `docker-compose.prod.yml`, `infra/` |
| o LinkedIn | `docs/linkedin.md` |
| o código | `backend/` (rotas em `routes/`, esquema em `database/schema.sql`), `frontend/src/` (páginas, componentes, `lib/`) |

## 12. O que não foi verificado

- Um **Mac a sério** (`route`, `ipconfig`, `caffeinate`, o pedido da firewall), o **Docker Desktop** e um **tablet Android** com o Chrome dele.
- Um **leitor de ecrã** (só se correu o axe, que dá 0 violações).
- O **tablier a 1,5 m** num tablet montado (o ângulo é uma conta, não uma medição).
- Tudo o que precisa de **servidor, domínio e certificado**; **preços** de qualquer fornecedor ou programa de faturação; a **lista oficial da AT** (a contagem de "2.552 programas" vem de uma fonte secundária, sem data).
- Que as correções do `npm audit` não partem nada: não foram aplicadas.
