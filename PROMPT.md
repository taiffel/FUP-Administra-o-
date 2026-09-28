# Prompt — Central de Follow-up do CS Stegia

> Cole este prompt no Claude Code para gerar a página. Antes, preencha a seção **"Identidade visual Stegia"** com as cores, fontes e logo do site (ou deixe o Claude Code extrair de https://stegia.com.br/ se a rede do ambiente permitir acesso ao site).

---

## Contexto

Trabalho na **Stegia** (https://stegia.com.br/), uma empresa de automação parceira do **Pipefy** (plataforma de gestão de processos no estilo Kanban). Sou da área de **Sucesso do Cliente (CS)**.

No dia a dia eu:

- Acompanho a carteira de clientes da Stegia, garantindo bom retorno e boa comunicação.
- Faço a **ponte entre o cliente e o time operacional** (quem constrói e entrega as automações no Pipefy).
- Faço **follow-ups** frequentes: cobrança de retorno, validação de entregas, pedidos de informação, agendamento de reuniões.
- Faço **handoffs**: passo demandas do cliente para o operacional, para o financeiro ou para o comercial, e acompanho até a resolução.
- Faço **gestão de projetos**: acompanho etapas, prazos e pendências das implantações de cada cliente.

## O problema que preciso resolver

1. **Follow-up sem resposta cai no esquecimento.** Mando uma mensagem, o cliente não responde, e eu esqueço de cobrar de novo.
2. **Demandas que dependem de outras áreas se perdem.** O cliente pede algo que preciso levar ao financeiro (ou ao operacional), e eu perco o controle de quem está com a bola e do que já respondi.
3. **Confundo clientes.** Sem um histórico centralizado, misturo o que foi combinado com um cliente e com outro.

Quero um **"coworking" pessoal**: uma página única onde eu abro de manhã, vejo exatamente o que preciso fazer hoje, e registro tudo o que acontece com cada cliente.

## O que construir

Um **único arquivo `index.html`** (HTML + CSS + JavaScript puro, sem build, sem backend), que abra direto no navegador e salve os dados no **`localStorage`**, com **exportar/importar backup em JSON**. Interface **em português do Brasil**, datas no formato `dd/mm/aaaa` e fuso `America/Sao_Paulo`.

### Entidades (modelo de dados)

- **Cliente**: nome da empresa, CNPJ (opcional), segmento, status (`Onboarding`, `Em implantação`, `Ativo`, `Em risco`, `Pausado`, `Encerrado`), saúde (verde/amarelo/vermelho), responsável operacional da Stegia, link do pipe/organização no Pipefy, observações fixas ("o que eu preciso lembrar sempre sobre este cliente"), data de início.
- **Contato**: nome, cargo, e-mail, telefone/WhatsApp, marcação de contato principal. Um cliente tem vários contatos.
- **Follow-up**: cliente, contato, canal (e-mail, WhatsApp, reunião, ligação, Slack/Teams), assunto, o que foi enviado/perguntado, data de envio, **próxima cobrança** (data), status (`Aguardando cliente`, `Respondido`, `Sem retorno — escalar`, `Concluído`), número de tentativas.
- **Demanda / Handoff**: cliente, título, descrição, **área responsável** (`Operacional`, `Financeiro`, `Comercial`, `Suporte Pipefy`, `Interno CS`), pessoa responsável, prioridade (baixa/média/alta/urgente), prazo, status (`Nova`, `Enviada para área`, `Em andamento`, `Aguardando cliente`, `Aguardando área`, `Retornar ao cliente`, `Concluída`), e o **retorno que preciso dar ao cliente**.
- **Projeto**: cliente, nome (ex.: "Implantação pipe de Compras"), etapas/marcos com datas e checkbox, status, % de conclusão calculado pelas etapas.
- **Registro na linha do tempo**: toda criação, mudança de status e anotação vira um item datado no histórico do cliente.

### Telas

1. **Hoje (tela inicial)** — o coração da ferramenta:
   - Blocos: **Atrasados** (vermelho), **Para hoje**, **Próximos 7 dias**.
   - Mistura follow-ups com próxima cobrança vencida/hoje, demandas com prazo, e demandas em `Retornar ao cliente`.
   - Cada item mostra o **nome do cliente em destaque** (para não confundir), o assunto, há quantos dias está sem resposta e botões rápidos: **"Cobrei de novo"** (registra nova tentativa e empurra a próxima cobrança), **"Respondeu"**, **"Concluir"**, **"Abrir cliente"**.
   - Contadores no topo: follow-ups aguardando, demandas abertas por área, clientes em risco.

2. **Clientes** — lista/grade com busca e filtros (status, saúde, responsável). Cada card mostra saúde, último contato ("há 12 dias") e pendências abertas. Destacar clientes **sem contato há mais de X dias** (configurável, padrão 15).

3. **Ficha do cliente** — tudo sobre um cliente em um só lugar:
   - Cabeçalho com nome, status, saúde, link do Pipefy e as **observações fixas** bem visíveis.
   - Abas: **Linha do tempo**, **Follow-ups**, **Demandas**, **Projetos**, **Contatos**.
   - Botão para adicionar nota rápida na linha do tempo.

4. **Demandas / Handoffs** — **quadro Kanban** (lembrar o Pipefy) com colunas pelos status da demanda, arrastar e soltar, filtros por área e por cliente. Alternar para visão em lista.

5. **Follow-ups** — lista de todos os follow-ups, filtrável por status, com destaque para "Sem retorno".

6. **Modelos de mensagem** — textos prontos com variáveis (`{cliente}`, `{contato}`, `{assunto}`, `{data}`) para 1º follow-up, 2ª cobrança, retorno do financeiro, envio para validação, agendamento de reunião. Botão **"Copiar"** já com as variáveis preenchidas a partir de um follow-up ou demanda.

7. **Configurações** — cadência padrão de follow-up, limite de dias sem contato, lista de responsáveis/áreas editável, exportar/importar JSON, limpar dados (com confirmação).

### Regras de negócio (automações da página)

- **Cadência de follow-up**: ao registrar um follow-up, sugerir a próxima cobrança automaticamente (padrão: +2 dias úteis na 1ª tentativa, +3 na 2ª, +5 na 3ª). Pular sábados e domingos.
- **Escalonamento**: após **3 tentativas sem resposta**, mudar o status para `Sem retorno — escalar` e sugerir marcar o cliente como `Em risco`.
- **Retorno ao cliente**: quando uma demanda de outra área muda para `Concluída` ou `Retornar ao cliente`, criar automaticamente uma pendência "Dar retorno ao cliente" na tela Hoje.
- **Saúde automática (sugestão)**: amarelo se houver follow-up sem resposta há mais de 7 dias ou demanda atrasada; vermelho se houver escalonamento ou mais de 30 dias sem contato. Eu posso sobrescrever manualmente.
- Toda ação relevante gera registro na linha do tempo do cliente.

### Usabilidade

- **Busca global** (atalho `/` ou `Ctrl+K`) por cliente, contato, follow-up ou demanda.
- **Botão "+ Novo"** sempre visível com opções: cliente, follow-up, demanda, nota. No formulário de follow-up/demanda, o cliente é escolhido por campo com autocompletar.
- Modais/painéis laterais para criar e editar, sem recarregar a página.
- Cor e/ou inicial do cliente (avatar) consistentes em toda a interface, para diferenciar visualmente os clientes.
- Responsivo (funciona no notebook e no celular) e com bom contraste.
- Carregar **dados de exemplo** (3 clientes fictícios) na primeira abertura, com botão para apagá-los.

## Identidade visual Stegia

Seguir o padrão visual do site **https://stegia.com.br/**: mesma paleta, tipografia, estilo de botões, cantos, espaçamentos e tom de voz. A interface deve parecer uma ferramenta interna da Stegia.

Preencher antes de gerar (ou extrair do site):

- Logo (URL ou arquivo): `__________`
- Cor primária: `#______`
- Cor secundária: `#______`
- Cor de destaque / CTA: `#______`
- Fundo e superfícies: `#______` / `#______`
- Texto principal: `#______`
- Fonte de títulos: `__________` (Google Fonts, se possível)
- Fonte de texto: `__________`
- Estilo geral (ex.: moderno, tecnológico, cantos arredondados, sombras suaves): `__________`

Regras de uso:

- Definir as cores como variáveis CSS em `:root` para ficar fácil ajustar.
- Cores de status (atrasado, hoje, ok) devem continuar legíveis e distintas das cores da marca.
- Cabeçalho com o logo da Stegia e o título **"Central de CS"**.

## Entregável

- Um arquivo `index.html` autocontido, funcionando ao abrir no navegador.
- Código organizado e comentado em português, fácil de manter.
- Ao final, um resumo curto de como usar e de como fazer backup dos dados.
