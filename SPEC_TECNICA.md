# 📋 SPEC TÉCNICA — Dashboard Controle Financeiro Familiar

> Documento para agente de código criar o `index.html` completo do zero.
> Deploy já configurado: GitHub Pages em `https://joaocmatos.github.io/controle-financeiro/`
> Qualquer commit na branch `main` faz deploy automático.

---

## 🎯 Objetivo do produto

Dashboard financeiro pessoal para famílias de baixa e média renda brasileiras.
Preço de venda: R$ 10–15. Público: famílias com orçamento apertado.
Entrega máxima de valor com zero fricção técnica para o usuário final.

---

## 📦 Entregável

**UM único arquivo:** `index.html`
- Self-contained: HTML + CSS + JavaScript em um só arquivo
- Sem dependências locais (só CDNs externos)
- Funciona no GitHub Pages sem configuração adicional

---

## 🔗 Dependências externas (CDN)

```html
<!-- Chart.js para gráficos -->
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>

<!-- Fontes Google -->
<link href="https://fonts.googleapis.com/css2?family=DM+Sans:wght@400;500;700&family=Space+Mono:wght@400;700&display=swap" rel="stylesheet">
```

---

## 🎨 Design System

### Paleta de cores

```css
--teal-darkest:  #085041;   /* header, títulos */
--teal-dark:     #0F6E56;   /* textos secundários */
--teal-main:     #1D9E75;   /* botões primários, active */
--teal-light:    #5DCAA5;   /* bordas destaque */
--teal-pale:     #9FE1CB;   /* bordas padrão */
--teal-bg:       #E1F5EE;   /* backgrounds cards */
--teal-page:     #f6fdf9;   /* fundo da página */

--red-bg:        #FCEBEB;   /* alertas, despesas */
--red-border:    #E24B4A;   /* bordas alerta */
--red-text:      #791F1F;   /* texto alerta */

--amber-bg:      #FAEEDA;   /* avisos */
--amber-border:  #EF9F27;   /* bordas aviso */
--amber-text:    #633806;   /* texto aviso */

--text-primary:  #2C2C2A;
--text-muted:    #888780;
```

### Tipografia

```css
font-family: 'DM Sans', sans-serif;       /* corpo, labels, UI */
font-family: 'Space Mono', monospace;     /* valores monetários (KPIs) */
```

### Cores por categoria de gasto

```js
const categoryColors = {
  'Moradia':      '#D85A30',
  'Alimentação':  '#639922',
  'Filhos':       '#7F77DD',
  'Saúde':        '#378ADD',
  'Transporte':   '#EF9F27',
  'Assinaturas':  '#D4537E',
  'Lazer':        '#1D9E75',
  'Vestuário':    '#E24B4A',
  'Educação':     '#888780',
  'Outros':       '#B4B2A9'
};
```

---

## 💾 Sistema de persistência — localStorage + URL params

### Prioridade de leitura do ID da planilha

1. `localStorage.getItem('planilha_id')`
2. `new URLSearchParams(window.location.search).get('id')`
3. → Mostrar modal de onboarding

### Funções obrigatórias

```js
function obterIdPlanilha() {
  const stored = localStorage.getItem('planilha_id');
  if (stored) return stored;

  const urlId = new URLSearchParams(window.location.search).get('id');
  if (urlId) {
    localStorage.setItem('planilha_id', urlId);
    window.history.replaceState({}, '', window.location.pathname); // limpa URL
    return urlId;
  }

  return null;
}

function extrairIdDaUrl(input) {
  input = input.trim();
  if (!input.includes('/')) return input;             // já é o ID
  const match = input.match(/\/d\/([a-zA-Z0-9-_]+)/);
  if (match) return match[1];
  return input;
}

async function validarIdPlanilha(id) {
  const url = id.startsWith('2PACX-')
    ? `https://docs.google.com/spreadsheets/d/e/${id}/pub?output=csv`
    : `https://docs.google.com/spreadsheets/d/${id}/export?format=csv&gid=0`;
  try {
    const r = await fetch(url);
    return r.ok;
  } catch {
    return false;
  }
}
```

---

## 🔌 Conexão com Google Sheets

### Dois formatos de planilha suportados

```js
// Formato 1: planilha normal compartilhada publicamente
// ID exemplo: 18FBhbdMxxfp6UYZlNOg52AZMCLGt4ctz
const csvUrl = `https://docs.google.com/spreadsheets/d/${id}/export?format=csv&gid=0`;

// Formato 2: planilha publicada na web (Arquivo > Publicar na Web > CSV)
// ID exemplo: 2PACX-1vS-w-QoVFx7zbQCmfM6zdcQfw_8fqWTJVKHRKv1Mvdg...
const csvUrl = `https://docs.google.com/spreadsheets/d/e/${id}/pub?output=csv`;

// Detecção automática:
const csvUrl = id.startsWith('2PACX-')
  ? `https://docs.google.com/spreadsheets/d/e/${id}/pub?output=csv`
  : `https://docs.google.com/spreadsheets/d/${id}/export?format=csv&gid=0`;
```

### Parsing do CSV

O CSV vem com valores em formato brasileiro. Função obrigatória:

```js
function parseValor(valorStr) {
  if (!valorStr) return 0;
  // Remove "R$", espaços, pontos de milhar, troca vírgula por ponto
  const cleaned = valorStr
    .replace(/R\$\s?/g, '')
    .replace(/\s/g, '')
    .replace(/\./g, '')
    .replace(',', '.');
  return parseFloat(cleaned) || 0;
}
```

### Colunas da planilha (cabeçalho — linha 1)

| Coluna | Nome exato | Tipo | Exemplo |
|--------|-----------|------|---------|
| A | `Data da compra` | string dd/mm/aaaa | `01/05/2025` |
| B | `Responsável` | string | `João`, `Maria`, `Família` |
| C | `Tipo` | enum | `Receita`, `Despesa`, `Poupança`, `Investimento` |
| D | `Categoria` | enum | `Moradia`, `Alimentação`, etc. |
| E | `Subcategoria` | string livre | `Aluguel`, `Mercado` |
| F | `Descrição` | string livre | `Aluguel apartamento` |
| G | `Valor (R$)` | monetário BR | `R$ 1.800` ou `1800` ou `1.800,00` |
| H | `Forma de pagamento` | enum | `Débito`, `Crédito`, `PIX`, `Dinheiro` |
| I | `Status` | enum | `Pago`, `Pendente`, `Agendado` |
| J | `Recorrente?` | enum | `Sim`, `Não` |

### Parse do CSV linha a linha

```js
async function fetchData() {
  const csvUrl = sheetId.startsWith('2PACX-')
    ? `https://docs.google.com/spreadsheets/d/e/${sheetId}/pub?output=csv`
    : `https://docs.google.com/spreadsheets/d/${sheetId}/export?format=csv&gid=0`;

  const response = await fetch(csvUrl);
  if (!response.ok) throw new Error('Planilha não encontrada. Verifique o ID e as permissões.');

  const text = await response.text();
  const lines = text.split('\n');
  const headers = lines[0].split(',').map(h => h.trim().replace(/"/g, ''));

  const data = [];
  for (let i = 1; i < lines.length; i++) {
    if (!lines[i].trim()) continue;
    const values = lines[i].split(',').map(v => v.trim().replace(/"/g, ''));
    if (values.length < headers.length) continue;

    const row = {};
    headers.forEach((header, idx) => { row[header] = values[idx]; });

    // Só inclui linhas com Tipo e Valor preenchidos
    if (row['Tipo'] && row['Valor (R$)']) {
      data.push(row);
    }
  }
  return data;
}
```

---

## 🗂️ Estrutura de navegação

```
[📊 Visão Geral]  [🔍 Análise]  [💡 Dicas]
```

Navegação por tabs sem reload de página. Páginas são divs com `display:none`/`display:block`.

---

## 📊 Página 1: Visão Geral

### KPIs (4 cards no topo)

```
┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│   RECEITAS   │ │   DESPESAS   │ │    SALDO     │ │   POUPADO    │
│  R$ 10.700   │ │   R$ 2.640   │ │   R$ 8.060   │ │    R$ 500    │
│              │ │              │ │  ✓ Positivo  │ │              │
└──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘
  bg: teal-bg     bg: red-bg       bg: teal-bg       bg: amber-bg
```

**Cálculo dos KPIs:**
- Receitas = soma de linhas onde `Tipo === 'Receita'` ou `Tipo === 'Investimento'`
- Despesas = soma de linhas onde `Tipo === 'Despesa'`
- Saldo = Receitas − Despesas
- Poupado = soma de linhas onde `Tipo === 'Poupança'`

**Status do Saldo:**
- `> 0` → "✓ Positivo" (cor teal)
- `< 0` → "⚠ Negativo" (cor red) + gerar alerta vermelho no topo
- `=== 0` → "Zerado"

### Alertas dinâmicos (acima dos KPIs)

Gerar automaticamente se:
1. **Saldo negativo** → alerta vermelho: "Atenção: Saldo negativo! Suas despesas ultrapassaram as receitas em R$ X"
2. **Contas pendentes** → alerta âmbar: "Você tem R$ X em contas não pagas"
3. **Mais de 80% da renda gasta** → alerta âmbar: "Você já gastou X% da sua renda este mês"

### Filtros

```
[Filtros:] [Todos os meses ▼] [Todos ▼] [Todas categorias ▼] [Todos status ▼]
```

Todos os filtros são `<select>` populados dinamicamente a partir dos dados.
Ao mudar qualquer filtro → recalcular KPIs, gráficos e tabela com `filteredData`.

**Meses** → extrair de `Data da compra`, ordenar decrescente, exibir como "Mai/2025"
**Responsável** → valores únicos da coluna `Responsável`
**Categoria** → valores únicos da coluna `Categoria`
**Status** → fixo: Pago, Pendente, Agendado

### Gráficos (lado a lado, altura máxima 300px)

```
┌─────────────────────────────────────┐ ┌────────────────────┐
│   Despesas por categoria            │ │  Gastos por pessoa │
│   [doughnut chart]                  │ │  [pie chart]       │
│                    ● Moradia        │ │  ● Família         │
│                    ● Alimentação    │ │  ● João            │
└─────────────────────────────────────┘ └────────────────────┘
  grid: 2fr 1fr
```

**CRÍTICO para Chart.js:**
```js
// Sempre usar maintainAspectRatio: false + height CSS fixo no container
options: {
  responsive: true,
  maintainAspectRatio: false,  // OBRIGATÓRIO
}

// CSS do container:
.chart-card { height: 300px; }
canvas { max-height: 240px !important; }
```

### Tabela de lançamentos

Mostrar os 15 mais recentes. Colunas: Data | Responsável | Descrição | Categoria | Valor | Status

Status como badge colorido:
- `Pago` → bg teal-bg, texto teal-darkest
- `Pendente` → bg amber-bg, texto amber-text
- `Agendado` → bg `#EEEDFE`, texto `#3C3489`

---

## 🔍 Página 2: Análise

### Gráfico de evolução mensal (linha)

- Eixo X: últimos 6 meses (do mais antigo para mais recente)
- Duas linhas: Receitas (teal) e Despesas (red)
- Fill área abaixo de cada linha
- Tooltip mostra valor em R$
- `maintainAspectRatio: false`, altura 280px

### Top 5 maiores gastos

Cards empilhados verticalmente. Cada card:
```
┌──────────────────────────────────────────────────────┐
│ 1. Aluguel apartamento                   R$ 1.800    │
│    Moradia • 02/05/2025                              │
│ ████████████████████████░░░░░░░░░ (barra proporcional) │
└──────────────────────────────────────────────────────┘
```
- Ordenar por valor decrescente
- Barra de progresso: o maior gasto = 100%, demais proporcionais
- Usar `filteredData` (respeita filtros da Visão Geral)

---

## 💡 Página 3: Dicas Personalizadas

Cards de dica baseados nos dados do usuário. Gerados automaticamente.

### Dicas que SEMPRE aparecem

```
📊 Regra 50-30-20
"Tente: 50% em necessidades (moradia, comida), 30% em desejos (lazer)
e 20% em poupança. Isso equilibra sua vida financeira."
```

### Dicas condicionais (verificar com dados reais)

```js
// 1. Categoria que mais pesa
const catMax = Object.keys(porCategoria).reduce((a, b) =>
  porCategoria[a] > porCategoria[b] ? a : b
);
const pct = ((porCategoria[catMax] / totalDespesas) * 100).toFixed(0);
// → "💡 {catMax} representa {pct}% dos seus gastos. [dica específica da categoria]"

// 2. Se cartão de crédito > 40% das despesas
// → "💳 Cuidado com o cartão de crédito..."

// 3. Se saldo positivo
// → "🎯 Você está poupando! Em 6 meses terá R$ X de reserva."
// Se saldo negativo
// → "⚠️ Despesas maiores que receitas..."

// 4. Se Alimentação > 30% das despesas
// → "🍽️ Economize na alimentação..."
```

### Dicas por categoria (texto da dica específica)

```js
const dicasPorCategoria = {
  'Moradia':      'Moradia é essencial, mas veja se consegue negociar aluguel ou reduzir água e luz.',
  'Alimentação':  'Fazer lista e cozinhar em casa reduz muito. Evite delivery toda semana.',
  'Transporte':   'Use transporte público, bike ou caronas quando possível.',
  'Lazer':        'Dá pra se divertir gastando pouco: parque, praia, filme em casa.',
  'Filhos':       'Economize comprando material escolar em atacado ou reutilizando.',
  'Saúde':        'Veja farmácias populares e genéricos mais baratos.',
  'Assinaturas':  'Cancele serviços que você não usa todo mês.',
  'Vestuário':    'Evite compras por impulso. Compre em promoções.',
  'Educação':     'Veja se há cursos gratuitos online que atendam sua necessidade.',
  'Outros':       'Analise se esses gastos são necessários ou compras por impulso.'
};
```

---

## 🪟 Modal de Onboarding

Aparece quando:
- Primeiro acesso (sem localStorage e sem URL param)
- Usuário clica em "⚙️ Trocar" no header

```
┌─────────────────────────────────────────┐
│ header gradiente teal                   │
│ Bem-vindo!                              │
│ Vamos conectar sua planilha             │
├─────────────────────────────────────────┤
│ 📋 Compartilhe sua planilha             │
│    Clique em "Compartilhar" e mude      │
│    para "Qualquer pessoa com o link     │
│    pode VER"                            │
│                                         │
│ 🔗 Cole o ID ou link                    │
│    Pode colar o link completo           │
│                                         │
│ [input: Cole aqui...]                   │
│ Exemplo: .../d/1ABC123/edit             │
│                                         │
│ [mensagem de erro se der]               │
│                                         │
│ [      Salvar e continuar      ]        │
│           ❓ Ajuda                      │
└─────────────────────────────────────────┘
```

### Fluxo do botão "Salvar":

```js
async function salvarPlanilhaId() {
  const input = document.getElementById('inputPlanilhaId').value;
  if (!input) { /* mostrar erro */ return; }

  const id = extrairIdDaUrl(input);

  // Desabilitar botão e mostrar "Validando..."
  btnSalvar.disabled = true;
  btnSalvar.textContent = 'Validando...';

  const valido = await validarIdPlanilha(id);

  if (!valido) {
    // Mostrar erro: "Verifique se o ID está correto e se a planilha está
    // compartilhada como 'Qualquer pessoa com o link pode VER'"
    btnSalvar.disabled = false;
    btnSalvar.textContent = 'Salvar';
    return;
  }

  localStorage.setItem('planilha_id', id);
  sheetId = id;
  document.getElementById('modalOnboarding').style.display = 'none';
  init(); // carregar dados
}
```

---

## 🔄 Fluxo de inicialização

```js
async function init() {
  sheetId = obterIdPlanilha();

  if (!sheetId) {
    // Mostrar modal
    document.getElementById('modalOnboarding').style.display = 'flex';
    return;
  }

  // Mostrar loading
  document.getElementById('loadingState').style.display = 'block';
  document.getElementById('mainContent').style.display = 'none';

  try {
    allData = await fetchData();
    filteredData = allData;

    document.getElementById('loadingState').style.display = 'none';
    document.getElementById('mainContent').style.display = 'block';

    populateFilters(allData);
    updateDashboard(filteredData);
    setupFilterListeners();

  } catch (error) {
    document.getElementById('loadingState').style.display = 'none';
    document.getElementById('errorState').style.display = 'block';
    document.getElementById('errorMessage').textContent = error.message;
  }
}

window.addEventListener('DOMContentLoaded', init);
```

---

## 📐 Layout e CSS

### Estrutura HTML principal

```html
<div class="container">
  <header>...</header>
  <nav class="nav-tabs">...</nav>

  <div id="loadingState">...</div>
  <div id="errorState">...</div>

  <div id="mainContent">
    <div class="page active" id="page-visao-geral">...</div>
    <div class="page" id="page-analise">...</div>
    <div class="page" id="page-dicas">...</div>
  </div>
</div>

<!-- Modal fora do container, position:fixed -->
<div id="modalOnboarding" class="modal-overlay">...</div>
```

### CSS crítico

```css
/* Página */
body { background: #f6fdf9; font-family: 'DM Sans', sans-serif; }
.container { max-width: 1400px; margin: 0 auto; padding: 20px; }

/* Header */
header {
  background: #085041;
  color: #E1F5EE;
  padding: 20px;
  border-radius: 12px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

/* Tabs */
.nav-tab.active { background: #1D9E75; color: #fff; }

/* Cards de gráfico — ALTURA FIXA OBRIGATÓRIA */
.chart-card {
  background: #fff;
  border: 0.5px solid #9FE1CB;
  border-radius: 12px;
  padding: 20px;
  height: 300px;           /* OBRIGATÓRIO para Chart.js */
}
.chart-card canvas {
  max-height: 240px !important;  /* OBRIGATÓRIO */
}

/* KPIs */
.kpi-value {
  font-family: 'Space Mono', monospace;
  font-size: 28px;
  font-weight: 700;
}

/* Modal */
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(8, 80, 65, 0.85);
  backdrop-filter: blur(4px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

/* Responsivo */
@media (max-width: 768px) {
  .charts-grid { grid-template-columns: 1fr; }
  header { flex-direction: column; text-align: center; }
}
```

---

## ⚠️ Armadilhas conhecidas — EVITAR

### 1. SyntaxError com template literals dentro de Chart.js

Evitar template literals com backtick dentro de callbacks do Chart.js.
Usar concatenação de string ao invés:

```js
// ❌ PROBLEMÁTICO:
label: function(ctx) {
  return `${ctx.label}: ${formatCurrency(ctx.parsed)}`
}

// ✅ SEGURO:
label: function(ctx) {
  return ctx.label + ': ' + formatCurrency(ctx.parsed);
}
```

### 2. Chaves extras em funções que criam Charts

```js
// ❌ ERRADO — chave extra no final:
charts.x = new Chart(ctx, { ... })};   // <- } extra

// ✅ CORRETO:
charts.x = new Chart(ctx, { ... });
```

### 3. Valores monetários brasileiros

```js
// ❌ ERRADO — parseFloat não entende formato BR:
parseFloat("R$ 1.800,00")  // → NaN

// ✅ CORRETO — usar parseValor():
parseValor("R$ 1.800,00")  // → 1800
```

### 4. maintainAspectRatio

```js
// ❌ ERRADO — gráfico fica enorme ou vazio:
maintainAspectRatio: true

// ✅ CORRETO — sempre false + altura CSS no container:
maintainAspectRatio: false
```

### 5. Trocar page sem recriar charts

Ao trocar de aba, destruir instâncias anteriores antes de criar novas:
```js
if (charts.evolucao) charts.evolucao.destroy();
charts.evolucao = new Chart(ctx, { ... });
```

---

## ✅ Checklist de validação

Antes de subir o arquivo, verificar:

- [ ] `parseValor()` existe e é chamada em TODOS os `row['Valor (R$)']`
- [ ] Nenhum `parseFloat(row['Valor (R$)'])` direto
- [ ] Todos os Charts têm `maintainAspectRatio: false`
- [ ] `.chart-card` tem `height: 300px` no CSS
- [ ] `canvas` tem `max-height: 240px !important` no CSS
- [ ] `fetchData()` detecta `2PACX-` e usa URL correta
- [ ] `validarIdPlanilha()` idem
- [ ] Modal some após salvar ID válido
- [ ] Botão "⚙️ Trocar" limpa localStorage e reabre modal
- [ ] Filtros repopulam ao trocar planilha
- [ ] Nenhum `parseFloat` direto em valores monetários
- [ ] Charts destruídos antes de recriar (`.destroy()`)
- [ ] Sem chaves `}` ou parênteses `)` extras em nenhuma função
- [ ] Template literals verificados (especialmente dentro de callbacks)

---

## 🚀 Deploy

Repositório: `https://github.com/joaocmatos/controle-financeiro`
Branch deploy: `main`
GitHub Pages URL: `https://joaocmatos.github.io/controle-financeiro/`
Deploy: automático a cada push na main

**Arquivo que deve existir na raiz do repo:**
- `index.html` — o dashboard completo
- `README.md` — instruções para o cliente
- `planilha_financeira_familiar.xlsx` — template da planilha

---

## 🧪 Como testar localmente

1. Abrir `index.html` no navegador
2. Colar no modal: `2PACX-1vS-w-QoVFx7zbQCmfM6zdcQfw_8fqWTJVKHRKv1Mvdg3Dhqelc865Zy9Uevz0cBbA`
3. Deve mostrar dados de teste da planilha João/Maria/Família
4. Verificar no console do navegador: zero erros

**Dados de teste disponíveis na planilha:**
- João — Salário R$ 6.500 (Receita, Pago)
- Maria — Salário R$ 4.200 (Receita, Pago)
- Família — Resgate CDB R$ 800 (Investimento, Pago)
- Família — Aluguel R$ 1.800 (Moradia, Pago)
- Maria — Mercado R$ 380 (Alimentação, Pago)
- João — Gasolina R$ 210 (Transporte, Crédito, Pendente)
- Família — Consulta pediatra R$ 250 (Saúde, Crédito, Pendente)
- Família — Reserva emergência R$ 500 (Poupança, Pago)
