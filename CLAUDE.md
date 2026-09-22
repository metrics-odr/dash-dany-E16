# CLAUDE.md — Dashboard de Controle de Tráfego Pago (TEMPLATE)

> Este arquivo é lido automaticamente pelo Claude Code ao abrir o repositório.
> Este é o repositório **TEMPLATE**, genérico — ainda não configurado para
> nenhum cliente. A engine (`build/template.html`, `build/build.py`,
> `ia-worker/worker.js`) é genérica e não deve ser editada por cliente; os
> valores do cliente ficam em **`build/config.py`** (config do funil) e
> **`config.js`** (metadados de publicação no GitHub). Veja também `README.md`
> para a visão geral e o passo a passo completo de publicação.

---

## ✅ CHECKLIST DE NOVO CLIENTE

Ordem para colocar um cliente novo no ar. Cada item aponta o arquivo e o marcador.

1. [ ] **Config do cliente** — copie `build/config.example.py` para
   `build/config.py` e preencha (comentado campo a campo no próprio arquivo):
   - `SPREADSHEET_ID`, `GID_META`, `GID_SALES` (planilha do cliente)
   - `TAX_FACTOR`, `MAIN_PRODUCT_PREFIX`, `COUNT_ALL_AS_PAID` (regras de negócio)
   - `UPSELL_PRODUCT_PREFIX`/`UPSELL_SPLIT_VALUE`/`UPSELL_USL_LABEL`/`UPSELL_DSL_LABEL`
     (OPCIONAL — só se o funil tiver upsell/downsell pós-compra do mesmo produto)
   - `CLIENT_NAME`, `CLIENT_SUB`, `TAX_LABEL`, `MAIN_PRODUCT` (rótulos exibidos)
   - `CAC_TARGET`, `ROAS_TARGET`, `REPORT_BAND_LOW`, `REPORT_BAND_HIGH` (metas da aba Relatórios)
2. [ ] **Este arquivo (`CLAUDE.md`)** — preencher a seção "Fontes de dados" abaixo
   com a planilha real do cliente (abas/colunas) e ajustar a regra de
   atribuição/produto principal se for diferente do padrão do template.
3. [ ] **`README.md`** — preencher: título, nome do produto principal (o resto
   já lê de `config.js`/`build/config.py`).
4. [ ] **`config.js`** (raiz do repo) — copie `config.example.js` para
   `config.js` e preencha `GITHUB_USERNAME`, `GITHUB_REPOSITORY`,
   `PROJECT_NAME`. Depois substitua manualmente os placeholders
   `metrics-odr`/`dash-dany-E16` que aparecem em `SETUP-CRON.md` e
   `README.md` pelos mesmos valores (são docs Markdown estáticos, a
   substituição não é automática).
5. [ ] **`SETUP-CRON.md`** — depois do passo acima, gere um **token
   fine-grained novo** (GitHub → Settings → Developer settings → Fine-grained
   tokens) escopado só para esse repositório, permissão Actions: Read and
   write; cadastre no cron-job.org (nunca commitar o token).
6. [ ] **GitHub Pages** — confirmar que o workflow `.github/workflows/deploy.yml`
   está na branch `main` e que o Pages foi habilitado (ele se autoconfigura na
   1ª execução via `actions/configure-pages`).
7. [x] **Routine do Briefing do Gestor** — configure uma Routine do Claude Code (ou
   equivalente) que rode **23:59 BRT** (padrão do template — ver nota abaixo), leia
   `build/relatorios_metrics.json` e escreva `build/relatorios.json` seguindo
   `build/GUIA-RELATORIOS.md` (ver "Aba Relatórios" abaixo). É o que preenche o
   Briefing do Gestor; o resto da aba (cards, Saúde do funil, Top/Piores anúncios)
   roda 100% no navegador, sem essa Routine.
   **Configurado para este cliente às 07:00 BRT (pedido explícito, não o padrão do
   template)** — o workflow `gerar-relatorios-metrics.yml` foi ajustado para rodar
   06:50 BRT. Trade-off aceito: o período "hoje" no briefing reflete só a madrugada
   até o horário de corte, não o dia quase completo (ver nota em
   `build/GUIA-RELATORIOS.md`).
8. [ ] *(OPCIONAL/legado — não é preciso para o cliente ir ao ar)* **Worker da IA
   Insights** — o dashboard não tem mais uma aba de geração de insights ao vivo
   (foi substituída pela Routine acima + o card "Saúde do funil", que não usa IA);
   `ia-worker/` fica no repo só como infraestrutura reaproveitável, caso um cliente
   específico volte a precisar de geração ao vivo. Se for usar: criar um Worker na
   Cloudflare, ajustar `ia-worker/wrangler.toml` (`name = "..."`, troque o
   placeholder `nomecliente-ia-insights`), cadastrar os secrets
   `CLOUDFLARE_API_TOKEN`/`CLOUDFLARE_ACCOUNT_ID`/`ANTHROPIC_API_KEY`/
   `INSIGHTS_PASSWORD` e preencher `IA_WORKER_URL` em `build/config.py`. Passo a
   passo completo em `SETUP-IA.md`.

---

## O que é

Dashboard de **Controle de Tráfego Pago** — app de BI estático (HTML/CSS/JS + Chart.js
via CDN) publicado no **GitHub Pages**, que cruza o gerenciador **Meta Ads** com a lista
de **Compradores** e se atualiza a cada ~30 min (build na nuvem via GitHub Actions,
disparado pelo cron-job.org). **Somente leitura** das planilhas.

- **URL pública:** `https://metrics-odr.github.io/dash-dany-E16/`
  (preencha `config.js` — ver checklist acima)
- **Cliente/projeto:** Dany Sakugawa — Lançamento Pago (`build/config.py`: `CLIENT_NAME`/`CLIENT_SUB`)
- **Tipo de funil:** Masterclass E16 — lançamento pago, sem VSL (o lead é a venda do
  ingresso) — `Anúncio → Página → Checkout → Compra`:
  `Gasto → Impressões → Cliques → Page Views → Checkouts → Vendas → Faturamento`
- **Sigla do funil:** `E16` (prefixo `26-E16` em todo `Campaign Name` do Meta Ads)

## Fontes de dados (Google Sheets)

Planilha: "Dany | E16 | Planilha Central de Lançamento Pago"
(`SPREADSHEET_ID` em `build/config.py`) — Meta Ads e Compradores ficam na mesma
planilha (leitura via export CSV).

| Aba | gid | Colunas |
|-----|-----|----------------|
| **Meta Ads** | `GID_META` (`1059708846`) | Day · Campaign Name · Ad Set Name · Ad Name · Amount Spent · Impressions · Link Clicks · Landing Page Views · Checkouts Initiated · Campaign Status · Ad Status · Ad Set Status · Ad ID · Creative Instagram Permalink |
| **Compradores** ("Ingressos") | `GID_SALES` (`1836439885`) | Data do pedido · Aprovado em · Pagamento · Produto · Valor · Situacao · Nome · Email · Telefone · Pagina · UTM Source · UTM Medium · UTM Campaign · UTM Term · UTM Content · Marcadores · Campanha (src) · Criativo (src) · Oferta · Pedido · Hotmart ID · src bruto · sck bruto · xcod |

**Regras confirmadas para este cliente:**
- **Coluna de receita**: `Valor` (não há coluna "Faturamento" separada — ticket único
  do ingresso, ~R$29). O alias `faturamento > valor` de `build/build.py` cai no
  `Valor` normalmente.
- **Coluna de status de pagamento**: `Situacao` (valores vistos: `Aprovada`/
  `Cancelado`) é confiável → `COUNT_ALL_AS_PAID = False` em `build/config.py`. Como
  o cabeçalho é `Situacao` (não `Status`), o alias genérico de status em
  `build/build.py` foi ampliado para incluir `"situacao"` (mudança genérica, não
  específica deste cliente).
- **Identificador do anúncio**: `UTM Content` (ex.: `BTS | VD_222`) — confirmado
  batendo com `Ad Name` real do Meta. `UTM Term` carrega o **posicionamento**
  (`Instagram_Stories`/`Instagram_Feed`), não o anúncio — não usar para o match.
- **Produto principal**: string única na coluna `Produto` — `MasterClass
  Best-Seller de Verdade` (`MAIN_PRODUCT_PREFIX`). Não há upsell/downsell pós-compra
  neste funil (`UPSELL_PRODUCT_PREFIX` vazio).

URL de export CSV: `https://docs.google.com/spreadsheets/d/{SPREADSHEET_ID}/export?format=csv&gid={GID}`

### Métricas do funil VSL (`build.py` + `template.html`)
`Gasto → Impressões → Cliques → Page Views → Checkouts → Vendas → Faturamento`

Gasto · Impressões · CPM · Cliques · CPC · CTR · Page Views · CPV · CR (Cliques/PageViews) ·
Checkouts · CPIC · VisCHK (Checkouts/PageViews) · Vendas · CAC (Gasto/Vendas) ·
ConvCHK (Vendas/Checkouts) · Faturamento · ROAS (Faturamento/Gasto) · Ticket (Faturamento/Vendas).

### Produto principal / atribuição
- **Produto principal** = `MAIN_PRODUCT_PREFIX` (definido em `build/config.py`). Base de
  **Vendas / CAC / ConvCHK / Ticket**.
- **Faturamento / ROAS** = soma de **todos os produtos** do funil (orderbumps/upsells).
- Uma venda entra no funil se: é o produto principal **OU** a combinação **`UTM Campaign`
  + `UTM Content`** (campanha + anúncio) casa com uma linha real do Meta (captura
  orderbumps/upsells que carregam a UTM do anúncio). O match exige campanha **e**
  anúncio juntos — nomes de anúncio (`AD01`, `AD02`...) podem se repetir entre campanhas
  diferentes; casar só pelo nome do anúncio atribuiria a venda à campanha errada. Quando
  casa, a venda herda a campanha/conjunto **reais do Meta** (fica na mesma linha do
  gasto nas tabelas). Vendas de outros funis (UTM/produto não relacionados) ficam de
  fora. Só conta status pago.
- **Desambiguação de conjunto quando o mesmo anúncio roda em mais de 1 conjunto**:
  campanha+anúncio às vezes não é suficiente — o mesmo `Ad Name` pode rodar
  simultaneamente em conjuntos diferentes na mesma campanha (ex.: teste de público
  reaproveitando o mesmo criativo — confirmado neste cliente na campanha "Teste
  Publicos": `BTS | VD_240` roda em 4 conjuntos ao mesmo tempo). Nesse caso o
  `build.py` tenta desambiguar pelo `UTM Medium`/`UTM Term` da própria venda: a
  planilha deste cliente é inconsistente sobre qual dos dois carrega o nome real
  do conjunto (varia por página/template de tracking — em `master-v1b2` é o `UTM
  Term`; em outras é o `UTM Medium`), então o build confere os dois contra os
  conjuntos candidatos. Sem match exato em nenhum dos dois, a venda fica marcada
  como `(múltiplos conjuntos)` em vez de ser creditada a um conjunto arbitrário
  (campanha/anúncio/Vendas totais continuam corretos; só o conjunto fica
  indefinido nesse caso raro).
- Se não houver coluna de Receita, não há Receita/ROAS/Ticket — ajuste o texto desta
  seção se o cliente novo tiver uma regra diferente.
- **Upsell/downsell pós-compra (OPCIONAL)**: se o funil do cliente tiver, logo após
  a compra do produto principal, uma oferta de upsell e, se recusada, um downsell
  mais barato do MESMO produto (as duas aparecem com o texto IDÊNTICO na coluna
  Produto), preencha `UPSELL_PRODUCT_PREFIX`/`UPSELL_SPLIT_VALUE`/
  `UPSELL_USL_LABEL`/`UPSELL_DSL_LABEL` em `build/config.py` (comentado em
  `config.example.py`). O build separa as duas ofertas pelo valor da venda; elas
  entram no Faturamento/ROAS mas não em Vendas/CAC/ConvCHK/Ticket, e são
  atribuídas ao Meta Ads herdando a campanha/anúncio da compra do produto
  principal do MESMO comprador (casando por e-mail), já que upsell/downsell
  normalmente não carregam UTM própria. Deixe os campos vazios se o funil não
  tiver esse tipo de oferta (comportamento inalterado).
- **Indicativo ATIVO/PAUSADO** (bolinha antes do nome nas tabelas de Campanhas/
  Conjuntos/Anúncios da aba Meta Ads): OPCIONAL — aparece automaticamente se a
  planilha do cliente tiver colunas de status por linha (aliases `campaign status`/
  `ad set status`/`ad status` em `build/build.py`). Resolvido no navegador
  (`latestStatusByDim()` em `build/app.js`), escopado pela mesma seleção de
  campanha/conjunto/anúncio (drill-down) das tabelas — nunca agregue esse status
  por nome sozinho no `build.py`: nomes de anúncio/conjunto podem se repetir em
  campanhas diferentes como anúncios DISTINTOS, e agregar por nome vaza o status
  de um anúncio ativo numa campanha para um anúncio pausado com o mesmo nome em
  outra.
- **HR/BR/ER (retenção de vídeo)**: OPCIONAL — se a planilha do cliente tiver as
  colunas de video views (3s/50%/95%), essas 3 taxas aparecem automaticamente na
  tabela de Anúncios (entre CPM e CTR), calculadas em `derive()`
  (`build/app.js`) sobre Impressões.
- **Busca por nome** nas tabelas de Campanhas/Conjuntos/Anúncios (campo de texto
  ao lado do título) e **heatmap na coluna Vendas** (além de Gasto/Faturamento/
  ROAS) já vêm por padrão no template, sem configuração.

### Imposto Meta Ads
Toggle ON aplica o `TAX_FACTOR` (definido em `build/config.py`) sobre os custos do Meta.

### Convenções de campanha
`Campaign Name = utm_campaign`, `Ad Set Name = utm_medium`, `Ad Name = utm_content`
(⚠️ **confira se não é** `utm_term` no caso deste cliente — essa coluna costuma
carregar o **posicionamento** do anúncio, não o nome dele — ver "Pontos de atenção"
acima). O match com o Meta (campo `meta`, usado pela aba Meta Ads) exige
`utm_campaign`+`utm_content` batendo com uma linha real do Meta; quando casa, a venda
herda a campanha/conjunto reais do Meta (para o gasto e a venda caírem na mesma linha
das tabelas).

## IA Insights (legado)

O dashboard **não tem mais uma aba própria de geração de insights ao vivo**. O que
existia (botão "Gerar insights" chamando um Worker via POST) foi substituído por:
o card **"Saúde do funil"** (nota 0-100 calculada no navegador, sem IA — ver "Aba
Relatórios" abaixo) e o **Briefing do Gestor** (texto pré-gerado 1×/dia por uma
Routine, sem chamada de API no navegador). O backend (`ia-worker/worker.js`,
`ia-worker/wrangler.toml`, `IA_WORKER_URL` em `build/config.py`) continua no repo
como infraestrutura reaproveitável para quem quiser voltar a ter geração ao vivo —
ver `SETUP-IA.md` para o passo a passo de deploy (Cloudflare Worker + GitHub
Actions), item 8 (opcional) do checklist acima.

## Arquitetura / arquivos

A dashboard é montada a partir de **arquivos separados** (visual x lógica), costurados
pelo `build.py` no `render()` — assim dá para mexer só em cor/layout sem tocar na lógica:

```
build/build.py             # ENGINE: lê os 2 CSVs (read-only), emite meta[]/sales[] e COSTURA os arquivos abaixo
build/config.py            # CONFIG DO CLIENTE (copie de config.example.py e preencha)
build/config.example.py    # modelo comentado de build/config.py
build/template.html        # esqueleto HTML (placeholders __STYLES__ / __APP_JS__ / __DATA_JSON__)
build/identidade-visual.css # ⭐ TODAS as cores (temas claro/escuro, paleta de gráficos, heatmap). Edite AQUI p/ mexer só em cor.
build/estilos.css          # layout/componentes (CSS não-cor)
build/app.js               # lógica + renderização (gráficos/heatmap leem as cores via CSS vars)
.github/workflows/deploy.yml         # roda build.py e publica no Pages
.github/workflows/deploy-worker.yml  # publica o Worker de IA (legado/opcional — ver "IA Insights (legado)")
.github/workflows/gerar-relatorios-metrics.yml # 06:50 BRT (este cliente): busca as planilhas e commita relatorios_metrics.json
ia-worker/worker.js    # backend de IA legado/opcional (ENGINE — não editar por cliente; não usado pela UI atual)
ia-worker/wrangler.toml # nome do Worker (preencher por cliente, placeholder nomecliente-ia-insights)
build/relatorios.json  # briefings do Gestor por período (aba Relatórios) — VERSIONADO
build/relatorios_metrics.json # números por período (gerado pelo Actions, lido pela Routine) — VERSIONADO
build/gerar_relatorios.py # calcula as métricas por período (rodado pelo Actions, não pela Routine)
build/GUIA-RELATORIOS.md  # passo a passo da Routine que regenera os briefings + particularidades deste funil
build/METODO-ODR-ANALISE.md # ⭐ método de raciocínio (Método ODR) p/ redigir os briefings — genérico, não editar por cliente
dist/index.html        # saída gerada (gitignored; o Actions reconstrói)
GUIA-REPLICACAO.md     # engine explicada + solução dos problemas de publicação
config.js               # metadados de publicação (GitHub) — copie de config.example.js
SETUP-CRON.md          # valores do cron-job.org (owner/repo com placeholders)
SETUP-IA.md            # passo a passo do backend de IA legado/opcional (ver "IA Insights (legado)")
```

### Aba Relatórios / "Insights de IA" (relatórios automáticos do funil)
Última aba do menu (nome exibido "Insights de IA", id interno `page-rel`).
Reaproveita os filtros de data da topbar e os dados já embutidos (`meta[]`/
`sales[]`) — tudo calculado no navegador (custo zero): card **Saúde do funil**
(nota 0-100 — ver abaixo), visão por campanha, **Top 5 / Piores 5 anúncios**
(Top 5 exige um mínimo de 3 vendas no período; com link do criativo via coluna
*Creative Instagram Permalink* → `ad_links`, se a planilha do cliente tiver essa
coluna), cards **Visão Geral Total** (todas as vendas) e **Tráfego** (só Meta
Ads), e o **Briefing do Gestor** (texto pré-gerado por IA). **Código de cor**
(vermelho/amarelo/verde/ciano) em **CAC** e **ROAS** (visão por campanha/Top-
Piores), conforme `CAC_TARGET`/`ROAS_TARGET` em `build/config.py` (desempenho =
ROAS `valor/meta`, CAC `meta/valor`).

**Saúde do funil** — nota 0-100 calculada no navegador (sem IA), ancorada numa
meta de **CAC** editável direto no card (persistida em `localStorage`, ou
`CAC_TARGET` de `build/config.py` como valor inicial) e numa meta de **ROAS**
(`ROAS_TARGET`); os demais componentes (CTR, CR, VisCHK, ConvCHK) usam como piso
85% da taxa histórica da própria conta (todo o período, sem filtro de data).
ROAS pesa mais na nota (35%) por ser o único componente que garante lucro, não
só desempenho operacional.

O **Briefing do Gestor** (texto interpretativo por período) é **pré-gerado por IA**
e lido de `build/relatorios.json` — **sem chamada de API no navegador nem créditos
da Anthropic**. Regeneração em **2 etapas diárias** (o sandbox do agente não alcança
o Google Sheets, só o runner do GitHub Actions — ver "problemas conhecidos" #4):
**Padrão do template: 23:50 BRT** o workflow `gerar-relatorios-metrics.yml` busca
as planilhas e commita `build/relatorios_metrics.json` (só números); **23:59 BRT**
uma **Routine do Claude Code** lê esse arquivo, migra o texto que estava em "hoje"
para "ontem" e redige os 9 briefings do zero seguindo `build/GUIA-RELATORIOS.md`,
commitando `relatorios.json`. Rodar no fim do dia (não de manhã) garante que "hoje"
seja analisado com o dia quase completo.
**Este cliente pediu 07:00 BRT** (workflow ajustado para 06:50 BRT) — trade-off
aceito de "hoje" refletir só a madrugada até o corte; ver nota em
`build/GUIA-RELATORIOS.md`. Se o JSON não existir, a aba mostra tudo menos o
briefing (cards/tabelas seguem funcionando).

O `build.py` **não agrega**: exporta as linhas cruas e toda a lógica (filtros, KPIs,
tabelas, gráficos, heatmap, imposto, tema) roda no navegador.

Teste local:
`python build/build.py --meta-file meta.csv --sales-file sales.csv --out dist/index.html`

## Publicação — problemas conhecidos e soluções

1. **Push com integração somente‑leitura:** se `git push`/MCP derem `403 Resource not
   accessible by integration`, faça push com o **PAT do usuário** direto ao github.com
   (`git push https://x-access-token:<TOKEN>@github.com/<owner>/<repo>.git main:main`).
   **Nunca** grave o token no `.git/config` (use a URL efêmera).
2. **cron-job.org só funciona na `main`:** `workflow_dispatch` só existe na branch padrão.
3. **Pages liga sozinho:** `actions/configure-pages@v5` com `enablement: true`
   (+ `permissions: {pages: write, id-token: write}`).
4. **Proxy do sandbox:** o agente NÃO alcança `docs.google.com`, `*.github.io` nem a API
   REST de Actions/Pages e nem `api.cloudflare.com` — mas o runner do Actions alcança
   tudo. Teste dados com CSV local; deploys da Cloudflare passam pelo GitHub Actions.
5. **Token exposto no chat:** revogar e gerar um novo (fine‑grained, só Actions: r/w no repo).
6. **"Senha incorreta" no backend de IA legado (se estiver em uso):** normalmente
   indica que os secrets `ANTHROPIC_API_KEY`/`INSIGHTS_PASSWORD` não estão
   cadastrados como Secrets do repositório no GitHub — o workflow `deploy-worker.yml`
   os reaplica no Worker a cada deploy; sem eles cadastrados, o Worker fica sem
   senha válida. Só se aplica a quem reativou o Worker (ver "IA Insights (legado)").
7. **Briefing do Gestor não aparece:** confirme que `build/relatorios.json` existe e
   que a Routine (item 7 do checklist) está rodando 1×/dia; sem esse arquivo, a aba
   mostra tudo (cards, Saúde do funil, tabelas) menos o briefing.
8. **Venda não aparece na aba Meta Ads (ou aparece na campanha errada):** confirme
   qual coluna UTM da planilha do cliente carrega o identificador real do anúncio do
   Meta (`Ad Name`) — nem sempre é `UTM Content`; `UTM Term` costuma carregar o
   **posicionamento** (`Instagram_Reels`/`Feed`/`Stories`), não o nome do anúncio.
   Casar pela coluna errada zera as atribuições. Além disso, nomes de anúncio podem se
   repetir entre campanhas diferentes — o match precisa ser **campanha+anúncio juntos**
   (`UTM Campaign`+`UTM Content`), senão a venda pode ser atribuída à campanha errada.
   Confira o valor real do `Ad Name` na API/painel do Meta e compare com as colunas
   UTM antes de mexer no alias.
