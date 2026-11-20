# Baylight no LinkedIn

> Textos prontos para publicar a Baylight no LinkedIn: a secção **Projetos** do perfil e uma **publicação**. Escritos para serem verdadeiros: a Baylight está em desenvolvimento, ainda não foi usada numa oficina com dados reais e não tem clientes nem métricas. **Antes de publicar, confirmar** que o que aqui está continua certo (ver a secção 5 do [`PROJETO-BAYLIGHT.md`](PROJETO-BAYLIGHT.md)) e trocar o que tiver mudado.
>
> Atualizado: 18/11/2026.

---

## 1. Secção "Projetos" do perfil

**Nome do projeto:** Baylight: folhas de obra digitais para oficinas

**Data:** desde 2026 (em curso)

**Associado a:** Universidade da Beira Interior (onde o projeto começou; hoje é um projeto pessoal)

**Ligação:** https://github.com/TiagoPereira001/Bancada

**Descrição** (cabe no limite do LinkedIn):

> A Baylight substitui as folhas de obra em papel das oficinas por uma folha digital pensada para o chão da oficina.
>
> Nasceu na oficina onde trabalho, a partir das folhas de obra reais: folhas que se perdiam, contas feitas à mão e pouca certeza sobre quem fez o quê.
>
> O que faz hoje:
> • Tablet partilhado: cada mecânico entra com o nome e um PIN, e a sessão fecha sozinha quando o tablet fica parado.
> • Quadro da oficina como o tablier de um carro: uma luz por estado da reparação, legível a dois metros.
> • Folha de obra com peças, horas e outros custos, com totais e IVA calculados ao cêntimo no servidor.
> • Histórico de clientes e veículos, autocaravanas incluídas (chassis e célula habitacional).
> • Várias oficinas na mesma plataforma, cada uma só com os seus dados, com o isolamento coberto por testes.
> • Aplicação web instalável (PWA), com tema claro e escuro.
>
> Tecnologia: React, Node.js e Express, base de dados relacional, Docker, testes automáticos e CI no GitHub Actions.
>
> Próximo passo: passar para PostgreSQL com isolamento por oficina na própria base de dados, TypeScript e auditoria, e pô-la a correr no meu servidor, com HTTPS e cópias de segurança testadas.
>
> Estado: em desenvolvimento, ainda sem uso real numa oficina.

**Competências a associar:** React · Node.js · Express.js · SQL · PostgreSQL · Docker · Testes de software · Integração contínua (CI/CD) · Design de interfaces (UI/UX) · Progressive Web Apps

---

## 2. Publicação

> Uma imagem ajuda: `docs/imagens/quadro-tablet.png` (o quadro no tablet). Usar só capturas com os dados fictícios do `db:seed`.

**Texto:**

> Trabalho numa oficina de mecânica e autocaravanas, e todas as reparações começavam numa folha de papel.
>
> Folhas que se sujavam, se perdiam, e no fim obrigavam a fazer as contas à mão. Para saber o que se fez a um carro no ano passado, era procurar numa pilha.
>
> Por isso comecei a construir a Baylight: folhas de obra digitais, feitas para o chão da oficina e não para o escritório.
>
> Três decisões que tomei e porquê:
>
> 🔧 Um tablet para todos. Na oficina não há um computador por mecânico. Cada um entra no mesmo tablet com o nome e um PIN, e a sessão fecha sozinha quando ninguém mexe. Tudo o que regista fica em nome dele.
>
> 🚦 O quadro é o tablier de um carro. Uma luz por estado (aberta, em curso, a aguardar peças, pronta), legível a dois metros, com as mãos ocupadas. Tocar numa luz mostra esses carros.
>
> 🧮 As contas batem certo ao cêntimo. Totais e IVA calculados no servidor, com decimais exatos, nunca no browser. A Baylight não emite faturas (em Portugal isso exige software certificado), mas deixa tudo pronto para as emitir.
>
> Está em desenvolvimento e ainda não foi usada com dados reais. O próximo passo é um ensaio na oficina com os mecânicos, e depois levá-la para o meu servidor com PostgreSQL, isolamento entre oficinas na própria base de dados e cópias de segurança testadas.
>
> Se trabalha numa oficina, ou com software para oficinas, gostava de ouvir o que acha. 👇
>
> #desenvolvimentoweb #react #nodejs #postgresql #oficina #projetopessoal

---

## 3. Regras para atualizar estes textos

- Só o que está feito e verificado; o que vem a seguir diz-se como "próximo passo".
- Sem números inventados (oficinas, utilizadores, horas poupadas). Quando houver dados do ensaio ou do piloto, entram com a data.
- Sem nomes nem dados de clientes reais nas imagens.
- A Duarte & Raposo só aparece pelo nome com autorização do dono da oficina.
- Português de Portugal, sem travessões.
