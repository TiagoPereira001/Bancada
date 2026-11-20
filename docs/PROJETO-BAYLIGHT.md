# Baylight: o projeto inteiro num só documento

> **Para quem é:** qualquer pessoa ou agente de IA que chegue a este repositório sem contexto. Em 10 minutos fica a saber o que é a Baylight, porque existe, o que já faz, o que **não** faz, o que está decidido, para onde vai e que regras não se podem partir.
>
> **Atualizado:** 20/11/2026 (nome Baylight, antes Bancada). **Autor e dono:** Tiago Dias Pereira. **Língua:** português de Portugal (código, interface e documentos).
>
> Este documento resume e liga. O detalhe vive nos ficheiros de origem, que mandam quando houver diferença: [`PRODUCT.md`](../PRODUCT.md) (produto), [`AI.md`](../AI.md) (mapa do código e regras), [`DESIGN.md`](../DESIGN.md) (visual), [`docs/adr/0001-arquitetura-para-durar.md`](adr/0001-arquitetura-para-durar.md) (arquitetura alvo) e, acima de todos, o próprio código.

---

## 1. Numa frase

**A Baylight substitui as folhas de obra em papel das oficinas por uma folha digital feita para o chão da oficina:** os mecânicos registam peças e horas num tablet partilhado enquanto trabalham, e o gestor sabe a cada momento que carros estão na oficina, em que estado, quanto vale o trabalho e quem o fez.

## 2. O problema

Muitas oficinas pequenas ainda trabalham com folhas de obra em papel. Na **Duarte & Raposo** (Canhoso, Covilhã), oficina de mecânica, eletricidade e autocaravanas onde o projeto nasceu, isso traz problemas todos os dias:

- é difícil saber o que se fez a um carro no ano passado;
- as folhas sujam-se e perdem-se pela oficina;
- no fim faz-se as contas à mão para passar a fatura;
- nem sempre se sabe quem fez o quê.

Os programas que existem para oficinas são pensados para o escritório (ERP, faturação, stock). Ninguém os usa de luvas, ao lado do carro, num tablet que passa de mão em mão.

## 3. A solução e o que a torna diferente

- **Feita para o chão da oficina.** Tablet partilhado: cada mecânico entra com o nome e um PIN, e a sessão termina sozinha se o tablet ficar parado. Alvos de toque grandes, para dedos com luvas. Poucos passos: se o digital for mais lento do que o papel, está errado.
- **O quadro da oficina é um tablier de carro:** uma luz por estado da reparação (aberta, em curso, a aguardar peças, pronta), legível a um ou dois metros. Tocar numa luz filtra a lista.
- **As contas batem certo ao cêntimo.** Totais e IVA calculados no servidor, com decimais exatos. A taxa de IVA fica gravada em cada folha quando abre.
- **Quem fez fica registado.** A autoria de cada folha e de cada linha vem da sessão de quem está a trabalhar, nunca de um campo que se possa escolher.
- **Nasceu de folhas reais**, por isso sabe coisas que só uma oficina sabe: autocaravanas com marca do chassis e marca da célula habitacional, clientes e matrículas estrangeiras.
- **Faz uma coisa bem feita.** Não é um ERP nem um programa de faturação: é a folha de obra.
- **Várias oficinas, dados separados.** Qualquer oficina se regista sozinha; cada uma só vê o que é seu.

## 4. Para quem é

| Quem | Onde | O que precisa |
|---|---|---|
| **Mecânicos** | tablet partilhado na bancada ou na parede | dar entrada a um carro em poucos toques, registar peças e horas, mudar o estado, deixar conselhos para o cliente |
| **Gestor** (dono ou responsável) | sobretudo no computador | saber o que está na oficina, histórico de clientes e veículos, gerir a equipa e os PINs, ver os valores com IVA |
| **Oficinas novas** | registo público | registar-se sozinhas, sem falar com ninguém |
| **Clientes das oficinas** (indiretos) | não usam a Baylight | contas certas, histórico do carro, conselhos de manutenção, aviso quando o carro está pronto |

## 5. O que existe hoje (verificado no código)

- Registo público de oficinas e entrada com email e password.
- **Modo bancada:** tablet partilhado, nome + PIN, PIN bloqueado ao fim de 5 falhas, sessão termina ao fim de 5 minutos parada, o gestor desliga todos os tablets de uma vez.
- **Quadro da oficina** (tablier), pesquisa por matrícula, cliente ou número, valores do mês para o gestor.
- **Nova entrada** a partir da matrícula, com veículo e cliente criados no mesmo passo quando são novos.
- **Folha de obra:** estado com um toque, linhas de peças, mão de obra e outros custos, totais por categoria, subtotal, IVA e total, quilómetros, observações, conselhos; folha entregue fechada (só um gestor reabre, com dois toques).
- Histórico de folhas, clientes e veículos; nada se apaga a sério (arquiva-se).
- Equipa (gestores e mecânicos), definições (dados da oficina, IVA, tablets).
- Temas claro e escuro; tablet, computador e telemóvel; **instalável como app (PWA)**.
- Lançador com duplo clique no macOS e um **modo oficina** para ensaios com um tablet pelo Wi-Fi ([`ensaio-oficina.md`](ensaio-oficina.md)).
- 52 testes automáticos contra uma base de dados real, incluindo o isolamento entre oficinas; CI no GitHub Actions.

**Estado honesto:** em desenvolvimento. **Ainda não foi usada numa oficina com dados reais** e não está publicada online. Só há dados de demonstração (`npm run db:seed`).

## 6. O que NÃO existe (não apresentar como feito)

- emissão de faturas (ver a secção 9) e integração com programas de faturação;
- funcionamento sem rede;
- notificações ao cliente;
- fotografias nas folhas;
- orçamentos e stock de peças;
- folha imprimível para o cliente;
- recuperação de password por email;
- auditoria completa (quem mudou o quê);
- versão publicada online com HTTPS;
- testes com os mecânicos da Duarte & Raposo (há um plano: [`ensaio-oficina.md`](ensaio-oficina.md));
- utilizadores reais, testemunhos, métricas de uso, preços ou outros clientes. **Nada disto se inventa.**

## 7. Decisões tomadas (não reabrir sem razão)

| Decisão | Escolha | Desde |
|---|---|---|
| Nome | **Baylight** (antes Bancada), domínio `baylight.dev`. "Bancada" ficou como nome do conceito (modo bancada) e de nomes técnicos | 20/11/2026 |
| Natureza | **projeto pessoal** do Tiago (nasceu como projeto de uma cadeira da licenciatura em Informática Web, Móvel e na Nuvem, UBI) | 18/11/2026 |
| Âmbito | plataforma para qualquer oficina; a Duarte & Raposo é o primeiro cliente, não a marca | origem |
| Novas oficinas | registo público (a oficina e o primeiro gestor) | origem |
| Dispositivo dos mecânicos | tablet partilhado (referência 1180 × 820, deitado), modo bancada com nome + PIN | origem |
| Ecrã principal | tablier de luzes, uma por estado | origem |
| Identidade visual | grafite + âmbar, a "luz indicadora", pictogramas próprios (carro, chave de bocas, pistão, bandeira de xadrez, chave); claro e escuro | origem |
| Faturação | a Baylight **não emite** faturas; calcula totais e IVA | origem |
| Tipo de aplicação | **aplicação web instalável (PWA)**, não app nativa nem app só no tablet: a oficina tem internet fiável, e um tablet sozinho seria o único sítio dos dados | 18/11/2026 |
| Login | **próprio da Baylight** (email e password, PIN); os utilizadores das oficinas nunca passam pelo Authentik do servidor do autor e nunca se misturam com os da MyNotes | 18/11/2026 |
| Arquitetura alvo | [ADR 0001](adr/0001-arquitetura-para-durar.md): monólito modular, TypeScript, **PostgreSQL com Row-Level Security por oficina** (sai o SQL Server), camadas, auditoria, migrações, produção no Coolify | 18/11/2026 |
| Onde corre | no servidor pessoal do autor (Time4VPS, Coolify), como **segundo projeto**, depois da MyNotes | 18/11/2026 |
| Ramos | só **`main`** (estável) e **`dev`** (trabalho); sem ramos por tarefa | 18/11/2026 |
| Commits | só em nome do Tiago, sem linhas de coautoria | 18/11/2026 |
| Direitos | todos os direitos reservados ao autor ([`LICENSE`](../LICENSE)); código público só para consulta | origem |

## 8. Arquitetura

### Hoje

```text
Tablet / computador / telemóvel (React 19 + Vite, PWA)
        │  mesma origem, cookies httpOnly SameSite=Strict
        ▼
API Node.js 22 + Express 5  (o SQL vive dentro das rotas)
        │  SQL parametrizado, filtrado à mão por ID_Oficina
        ▼
SQL Server 2022 (Docker)
```

Bom e para manter: isolamento entre oficinas, sessões revogáveis (`Versao_Sessao`), contas em `DECIMAL`, folha entregue fechada, testes. O que limita: SQL Server (pesado e com licença), SQL nas rotas, filtro por oficina escrito à mão, sem auditoria, sem tipos, sem migrações.

### Alvo ([ADR 0001](adr/0001-arquitetura-para-durar.md), aceite, por fazer)

```text
PWA (React + TypeScript)
        ▼
rotas finas → serviços (validam, autorizam, auditam) → repositórios → PostgreSQL
               regras puras: totais, IVA, estados        Row-Level Security por oficina
        ▼
portas para o exterior: faturação certificada, notificações, fotografias, fila (pg-boss)
```

Os pontos que fazem a diferença:

1. **Isolamento pela própria base de dados** (Row-Level Security): mesmo que o código se esqueça de um filtro, uma oficina não vê dados de outra. É a garantia que deixa abrir o registo a muitas oficinas com tranquilidade.
2. **Dinheiro testado:** os totais e o IVA do sistema atual ficam guardados como casos de teste; a versão nova só avança com eles iguais ao cêntimo.
3. **Auditoria:** quem mudou o quê e quando, a base do histórico de cada folha.
4. **Portas e adaptadores:** cada integração (faturação, SMS, WhatsApp, email, fotografias) entra por uma interface, sem mexer nas regras de negócio.

### Produção

No servidor pessoal do autor: uma VM da Time4VPS (3 vCPU, 8 GB, x86_64, Ubuntu 24.04, Vilnius, UE) com o **Coolify**, onde já corre a MyNotes. A Baylight é um recurso próprio (app + o seu Postgres), com rede e volume próprios, imagem feita no CI e publicada no GHCR, deploy automático a partir da `main`, cópias cifradas fora da VM com restauro testado, monitorização externa. Primeiro só por túnel; público num domínio próprio quando estiver pronta. O estado do servidor vive fora deste repositório, em `~/Desktop/Server` (começar pelo `AGENTS.md`).

## 9. Faturas

**Em Portugal, uma fatura tem de ser emitida por software certificado pela Autoridade Tributária** (Portaria n.º 363/2010 e legislação seguinte). Por isso a Baylight não emite faturas.

**Direção recomendada (ainda por decidir):** integrar com um programa de faturação **já certificado** que tenha API. Na folha entregue, um botão "Emitir fatura" envia o cliente, as linhas e o IVA; o programa certificado emite a fatura; a Baylight guarda o número e a ligação para o PDF. A oficina usa a conta dela nesse programa e a responsabilidade fiscal fica do lado de quem é certificado. Encaixa no ADR 0001 como uma porta ("faturação") com um adaptador por programa.

**Rejeitado:** certificar a própria Baylight como software de faturação. É um processo formal e contínuo (assinatura, numeração, comunicação à AT, SAF-T, QR, ATCUD) que transforma o projeto num programa de faturação, que é exatamente o que a Baylight decidiu não ser.

**Pesquisa de 18/11/2026** (fontes indicadas; preços **não** verificados):

- **Quantos há:** a AT mantém a lista oficial de programas certificados no Portal das Finanças (ligação no [gov.pt](https://www.gov.pt/servicos/consultar-o-programa-de-faturacao-certificado), página atualizada a 29/07/2026). A lista não se deixou contar automaticamente. Uma fonte secundária (Doutor Finanças, data não indicada) fala de **2.552 programas certificados**. Ordem de grandeza: milhares, quase todos sem API pública.
- **A obrigação:** a Portaria n.º 340/2013 obriga quem emite faturas a usar exclusivamente programas certificados pela AT, com algumas exceções (por exemplo, de volume de negócios) que mudaram ao longo dos anos: confirmar a versão em vigor no Diário da República antes de escrever qualquer texto para clientes.
- **Com API pública documentada** (confirmado nas páginas oficiais dos próprios programas):

| Programa | Documentação da API | Notas |
|---|---|---|
| InvoiceXpress | [docs.invoicexpress.com](https://docs.invoicexpress.com/) | API REST com chave por conta; a resposta de uma fatura traz `atcud`, `saft_hash` e a ligação para o documento |
| Moloni | [moloni.pt/dev](https://www.moloni.pt/dev/) | faturas, recibos, notas de crédito e de débito, faturas simplificadas |
| Vendus | [vendus.pt/ws](https://www.vendus.pt/ws/) | API v1.1 com chave; mais virado para ponto de venda |
| TOConline | [api-docs.toconline.pt](https://api-docs.toconline.pt/) | da Ordem dos Contabilistas Certificados; segundo uma fonte secundária, só se subscreve através de um contabilista |
| Cegid Jasmin / Primavera | [developers.ila.cegid.com](https://developers.ila.cegid.com/en/cloud/rose/api-documentation/) | API REST da Cegid; produtos mais completos (vendas, compras, contabilidade) |

- **Recomendação para a primeira integração:** começar por **um** destes (InvoiceXpress ou Moloni parecem os mais simples para uma oficina pequena), com o adaptador atrás da porta "faturação", e só acrescentar outro quando uma oficina real o pedir. Antes de escolher: perguntar à Duarte & Raposo que programa usa hoje, e confirmar preços e limites da API na página oficial.

## 10. Caminho

| Fase | O quê | Pronto quando |
|---|---|---|
| 0. Ensaio | meio dia na Duarte & Raposo com dados fictícios ([plano](ensaio-oficina.md)) | as observações estão escritas, com data |
| 1. Fundação | TypeScript, camadas, PostgreSQL com RLS, migrações, testes de dinheiro e de isolamento | testes verdes; totais iguais ao cêntimo; a interface não muda |
| 2. Produção privada | Coolify, cópias, monitorização; a Duarte & Raposo como piloto | restauro testado; acesso restrito |
| 3. Público | domínio, email (recuperação de conta), texto legal, registo aberto | recuperação de password a funcionar; texto legal revisto |
| 4. Crescer | faturação certificada por API, notificações ao cliente, fotografias, auditoria visível, orçamentos | cada uma com a sua decisão e os seus testes |

## 11. A Baylight como produto (para um dia a vender)

> Esta secção é uma **análise**, não uma promessa. Não há clientes pagantes, preços nem métricas; os números de mercado e de preços têm de ser confirmados antes de usar.

**Proposta de valor, em linguagem de oficina:**
- "Sabe sempre que carros tem na oficina, em que estado e quanto vale o trabalho."
- "Acabaram as folhas perdidas e as contas à mão: os totais e o IVA saem certos."
- "Sabe quem fez cada trabalho e o histórico de cada carro em segundos."
- "Os seus mecânicos usam-na de luvas, no tablet que já tem."

**Para quem vender primeiro:** oficinas pequenas e independentes (poucos mecânicos, ainda no papel), começando pela região onde há uma referência real (a Duarte & Raposo, depois do piloto). Nichos onde a Baylight já é mais forte: oficinas de autocaravanas e oficinas com clientes estrangeiros.

**Modelo de negócio provável:** subscrição mensal por oficina (software como serviço), com o servidor, as cópias e as atualizações incluídos. Um período de experiência gratuito encaixa no registo público que já existe. **Preço por definir**, depois de falar com oficinas e de ver o que a concorrência cobra.

**O que tem de existir antes de cobrar a alguém:**
1. As fases 1 a 3 do caminho (dados isolados pela base de dados, cópias testadas, HTTPS, recuperação de conta).
2. **Texto legal:** termos de serviço, política de privacidade e um **acordo de tratamento de dados** (as oficinas guardam dados dos clientes delas na Baylight; o autor passa a ser subcontratante no RGPD).
3. Faturação por integração com um programa certificado (secção 9): é a primeira pergunta de uma oficina.
4. Forma de faturar as próprias subscrições (atividade aberta nas Finanças e um programa certificado).
5. Suporte e um compromisso de disponibilidade realista para um só autor.
6. Pelo menos um piloto real, com o que se aprendeu escrito, antes de qualquer testemunho.

**Riscos:**
- concorrência de programas de gestão de oficinas já estabelecidos;
- um só autor (suporte, férias, bus factor), mitigado por documentação como esta e por automatização;
- dependência de um único servidor, mitigada por cópias fora da VM e por a pilha ser portátil (Docker Compose, Postgres);
- obrigações fiscais e de privacidade, que não se resolvem com código.

## 12. Regras para qualquer IA (e pessoa) que trabalhe aqui

1. **Ler primeiro** o [`AI.md`](../AI.md) (mapa e regras), depois este documento; para trabalho visual, o [`DESIGN.md`](../DESIGN.md) e o [`PRODUCT.md`](../PRODUCT.md).
2. **Cada oficina só vê os seus dados.** Nenhuma consulta de dados de oficina sem o filtro (hoje) ou sem RLS ativa (no alvo). Qualquer alteração ao isolamento leva teste.
3. **Dinheiro ao cêntimo:** nunca somar dinheiro com números de vírgula flutuante; totais e IVA no servidor.
4. **Autoria pela sessão,** nunca por um campo do pedido.
5. **Nada se apaga a sério;** arquiva-se.
6. **Não emitir faturas** nem prometer que a Baylight o faz.
7. **Não inventar** clientes, testemunhos, métricas, preços nem funcionalidades. "Por fazer" é "por fazer".
8. **Segredos nunca** no código, no git ou em conversas (`.env` fora do git; as passwords antigas do histórico público nunca se reutilizam).
9. **Português de Portugal** em tudo, **sem travessões** nos textos.
10. **Git:** só `main` e `dev`; trabalha-se na `dev`; a `main` recebe a `dev` por PR e o merge é do Tiago. Commits só em nome do Tiago, sem linhas de coautoria. Nada de `push --force`.
11. **Documentação faz parte do pronto:** quando algo muda, atualizar este documento, o `AI.md`, o `CLAUDE.md` e o `PRODUCT.md` na mesma tarefa.

## 13. Onde está cada coisa

| Preciso de... | Ver |
|---|---|
| o produto em detalhe | [`PRODUCT.md`](../PRODUCT.md) |
| o mapa do código, a API, os comandos | [`AI.md`](../AI.md) |
| o visual | [`DESIGN.md`](../DESIGN.md) |
| a arquitetura alvo | [`docs/adr/0001-arquitetura-para-durar.md`](adr/0001-arquitetura-para-durar.md) |
| a proposta de infraestrutura (versão SQL Server) | [`docs/infraestrutura.md`](infraestrutura.md) |
| o plano de ensaio na oficina | [`docs/ensaio-oficina.md`](ensaio-oficina.md) |
| auditorias e revisão de código | [`docs/auditoria.md`](auditoria.md), [`docs/revisao-codigo.md`](revisao-codigo.md) |
| o texto para o LinkedIn | [`docs/linkedin.md`](linkedin.md) |
| a memória das sessões do Claude Code | [`CLAUDE.md`](../CLAUDE.md) |
