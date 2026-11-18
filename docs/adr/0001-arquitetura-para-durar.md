# ADR 0001: Arquitetura para a Bancada durar e crescer

- **Estado:** aceite (18/11/2026). **Implementação: por fazer.** Nada disto está no código ainda.
- **Decisor:** Tiago Dias Pereira.
- **Âmbito:** todo o repositório (API, frontend, base de dados, produção).

## Contexto

A Bancada é um **projeto pessoal**. O `README.md` e o `CLAUDE.md` ainda a descrevem como projeto de uma unidade curricular: essa parte está desatualizada e é corrigida à parte.

Quero que a Bancada tenha vida longa e espaço para crescer (mais oficinas, mais funcionalidades), e que corra no meu servidor pessoal: uma VM da Time4VPS (3 vCPU, 8 GB de RAM, x86_64, Ubuntu 24.04) com o Coolify, onde já corre a MyNotes. A oficina tem internet fiável, por isso a Bancada continua a ser uma aplicação web (PWA) e não uma app só no tablet.

O que o código tem hoje (ramo `dev`, 18/11/2026):

- **Bom e para manter:** isolamento entre oficinas, sessões que se podem revogar (`Versao_Sessao`), contas e IVA calculados em SQL com `DECIMAL`, folha entregue fechada, modo bancada (nome + PIN), registo público de oficinas, PWA, 52 testes e CI.
- **O que limita o crescimento:**
  1. **SQL Server 2022.** Precisa de pelo menos 2 GB de RAM para arrancar em Linux (documentação da Microsoft). Sem `MSSQL_PID`, arranca como Developer, que não está licenciado para produção; a edição Express é gratuita mas limitada (memória e 10 GB por base).
  2. O SQL vive dentro das rotas (cerca de 2300 linhas no `backend`), sem camada de serviços.
  3. O filtro por oficina é escrito à mão: `ID_Oficina` aparece 65 vezes nas rotas. Basta faltar uma vez para uma oficina ver dados de outra.
  4. Não há registo de quem mudou o quê.
  5. JavaScript sem tipos.
  6. Não há migrações versionadas da base de dados (o `db-setup` não mexe no que já existe).
- **Dados:** só há dados de teste (`db:seed`). Não há dados reais a migrar.

## Decisão

### 1. Monólito modular

Uma aplicação, um deploy, uma base de dados, dividida por dentro em módulos: oficinas, equipa e sessões, clientes, veículos, folhas de obra, contas. Sem microserviços.

### 2. Camadas

```text
rotas finas (HTTP) -> serviços (validam, autorizam, auditam) -> repositórios (SQL) -> Postgres
                       regras puras: totais, IVA, estados da folha (testáveis sem base de dados)
```

Cada regra de negócio existe num só sítio (por exemplo, "uma folha entregue não se edita") e serve a interface e tudo o que vier depois (notificações, integrações, trabalhos em segundo plano).

### 3. TypeScript de ponta a ponta

API e frontend em TypeScript, com contratos validados (zod) partilhados entre os dois. A migração faz-se por módulos, com os testes verdes a cada passo.

### 4. PostgreSQL com o isolamento garantido pela base de dados

- Troca do SQL Server pelo **PostgreSQL**.
- Todas as tabelas de dados de uma oficina têm `oficina_id`.
- **Row-Level Security** do Postgres: a base de dados recusa linhas de outra oficina, mesmo que o código se esqueça do filtro. A oficina da sessão passa para a base em cada transação.
- A aplicação liga-se com um utilizador **sem** poderes para contornar a RLS. As migrações correm com outro utilizador.
- Testes de isolamento: uma oficina nunca lê nem escreve dados de outra, nem pela API nem diretamente no SQL com o utilizador da aplicação.

### 5. Dinheiro sem erros

`NUMERIC` na base de dados (ou cêntimos inteiros) e funções puras para totais e IVA, nunca números de vírgula flutuante. **Antes da migração**, guardam-se os totais e o IVA de um conjunto de folhas calculados pelo SQL Server atual; a versão em Postgres só avança com esses totais iguais ao cêntimo.

### 6. Auditoria

Tabela `audit_event` (quem, o quê, quando, valor antigo e novo) escrita pelos serviços. É a base do histórico de cada folha e de qualquer disputa sobre horas ou preços.

### 7. Migrações versionadas

Uma ferramenta de migrações (Drizzle, como na MyNotes) e a regra de **só acrescentar**: nunca apagar ou renomear no mesmo deploy em que o código deixa de usar.

### 8. Login próprio

O login continua na Bancada: email e password (bcrypt), PIN do modo bancada, sessões em cookie `httpOnly`. **Os utilizadores das oficinas nunca passam pelo Authentik do meu servidor** e nunca se misturam com os da MyNotes: vivem noutra base de dados, noutra pilha. Acrescenta-se:

- recuperação de password por email (precisa de SMTP);
- MFA para gestores;
- limites de tentativas onde ainda não existem.

### 9. Portas para o que vem a seguir

Tudo o que é externo entra por uma interface (porta) com um adaptador:

- notificações ao cliente ("o carro está pronto") por email, SMS ou WhatsApp;
- fotografias de danos e peças, guardadas fora da VM (armazenamento compatível com S3);
- exportação para o programa de faturação da oficina (a Bancada continua a **não** emitir faturas: isso exige software certificado pela AT);
- trabalhos em segundo plano com uma fila em Postgres (`pg-boss`).

Nada disto existe hoje. As portas garantem que, quando vier, as regras de negócio não mudam.

### 10. Aplicação web instalável (PWA)

Continua a ser uma PWA (já existe com o `vite-plugin-pwa`): instala-se no tablet, abre em ecrã inteiro, a "casca" fica em cache e as respostas da API nunca ficam em cache. Sem app nativa e sem escrita offline.

### 11. Produção no meu servidor

- Recurso próprio no Coolify (app + Postgres), com rede e volume próprios e **nenhuma porta publicada** no anfitrião.
- Imagem construída no CI e publicada no GHCR; deploy automático a partir da `main`.
- Cópias cifradas fora da VM, com restauro testado.
- Monitor externo em `/api/ready`.
- Um ambiente de *staging* a partir da `dev`, só por túnel.
- Primeiro só por túnel; público num domínio próprio na fase 3. Nessa altura decide-se o proxy da VM (hoje é o Caddy da MyNotes que serve as portas 80 e 443).

### 12. Legal antes de abrir

Antes do registo público a sério: termos de serviço, política de privacidade e um acordo de tratamento de dados com as oficinas (passo a ser subcontratante no RGPD, porque as oficinas guardam dados dos clientes delas).

## Ordem

| Fase | O quê | Pronto quando |
|---|---|---|
| 1. Fundação | TypeScript, camadas, Postgres com RLS, migrações, testes de dinheiro e de isolamento, CI | testes verdes; totais iguais ao cêntimo; a interface não muda |
| 2. Produção privada | Coolify, cópias, monitorização; a Duarte & Raposo como piloto | restauro testado; acesso só por túnel ou restrito |
| 3. Público | domínio, email, texto legal, registo aberto | recuperação de password a funcionar; texto legal revisto |
| 4. Crescer | auditoria na interface, notificações, fotografias, exportação para a faturação, subscrições se fizer sentido | cada uma com a sua decisão e os seus testes |

## Consequências

- **Bom:** menos RAM (o Postgres gasta uma fração do SQL Server); sem limites de licença; isolamento entre oficinas garantido pela base de dados; dinheiro testado; código com tipos; operação igual à da MyNotes (as mesmas cópias, o mesmo deploy, a mesma monitorização).
- **Custo:** reescrever o acesso a dados (esquema, consultas e testes), a camada de serviços e a passagem a TypeScript. Aceito o esforço.
- **Obrigatório a partir daqui:** nenhuma consulta de dados de oficina sem RLS ativa; dinheiro só em `NUMERIC` ou cêntimos; migrações só a acrescentar; cada fase fecha com testes verdes.
- **Por verificar:** quanta RAM gasta a Bancada na VM com Postgres; o desenho exato das políticas de RLS com o modo bancada.

## Alternativas rejeitadas

| Alternativa | Porque não |
|---|---|
| Manter o SQL Server (Express) | 2 GB de RAM no mínimo numa VM partilhada, limites da Express, e duas formas de fazer cópias e restauros no mesmo servidor |
| App só no tablet, sem nada online | o tablet passa a ser o único sítio dos dados; cópias e sincronização ficam mais difíceis; o gestor deixa de ver a oficina de fora; acaba o multi-oficina. A oficina tem internet fiável |
| App nativa | a PWA já se instala e abre em ecrã inteiro; nada no produto precisa de hardware que o browser não dê |
| Login pelo Authentik do meu servidor | o modo bancada (PIN no tablet partilhado) e o registo público não encaixam no OIDC, e os utilizadores das oficinas misturar-se-iam com os da MyNotes |
| Microserviços, Kubernetes, escrita offline | complexidade sem um problema real que a justifique |

## Ligações

- [`AI.md`](../../AI.md) (mapa e regras), [`PRODUCT.md`](../../PRODUCT.md) (produto), [`docs/infraestrutura.md`](../infraestrutura.md).
- Servidor (fora deste repositório): `~/Desktop/Server`, a começar pelo `AGENTS.md`.
