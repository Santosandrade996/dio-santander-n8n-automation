# Automação de Processos com N8N

## 📋 Índice

- [Sobre o Módulo](#sobre-o-módulo)
- [O que são Workflows de Automação](#o-que-são-workflows-de-automação)
- [Criando seu Primeiro Workflow](#criando-seu-primeiro-workflow)
- [Workflows Simples vs Complexos](#workflows-simples-vs-complexos)
- [Técnicas Avançadas](#técnicas-avançadas)
- [Exemplos Práticos](#exemplos-práticos)
- [Boas Práticas](#boas-práticas)
- [Troubleshooting](#troubleshooting)

## 🎯 Sobre o Módulo

Este módulo aborda a automação de processos utilizando N8N, desde conceitos básicos até técnicas avançadas como unificação de dados, envio de emails e criação de loops. Você aprenderá a criar workflows que automatizam tarefas repetitivas e integram diferentes ferramentas.

## 🔄 O que são Workflows de Automação

### Conceito Fundamental

Um workflow de automação é uma sequência de ações automatizadas que são executadas em resposta a um evento (trigger). No N8N, workflows são representados visualmente como um fluxo de nodes conectados.

### Componentes Básicos

```
┌──────────────┐
│   TRIGGER    │ → Evento que inicia o workflow
└──────┬───────┘
       │
       ↓
┌──────────────┐
│    AÇÃO 1    │ → Primeira ação executada
└──────┬───────┘
       │
       ↓
┌──────────────┐
│    AÇÃO 2    │ → Segunda ação executada
└──────┬───────┘
       │
       ↓
┌──────────────┐
│   RESULTADO  │ → Saída final
└──────────────┘
```

### Tipos de Nodes

1. **Trigger Nodes** - Iniciam o workflow
   - Schedule (tempo/cron)
   - Webhook (HTTP)
   - Manual Trigger
   - Email Trigger

2. **Action Nodes** - Executam operações
   - HTTP Request
   - Database operations
   - File operations
   - API calls

3. **Logic Nodes** - Controlam o fluxo
   - IF conditions
   - Switch
   - Merge
   - Loop

4. **Data Nodes** - Manipulam dados
   - Set
   - Function
   - Filter
   - Transform

## 🚀 Criando seu Primeiro Workflow

### Workflow Básico: Monitoramento de Formulário

**Objetivo:** Receber dados de um formulário e enviar notificação

#### Passo 1: Configurar o Trigger

```javascript
// Node: Webhook
{
  "httpMethod": "POST",
  "path": "formulario-contato",
  "responseMode": "responseNode",
  "options": {}
}
```

**URL gerada:** `https://seu-n8n.com/webhook/formulario-contato`

#### Passo 2: Validar os Dados

```javascript
// Node: Function - Validação
const dados = $input.first().json;

// Validações básicas
if (!dados.nome || dados.nome.trim() === '') {
  throw new Error('Nome é obrigatório');
}

if (!dados.email || !dados.email.includes('@')) {
  throw new Error('Email inválido');
}

if (!dados.mensagem || dados.mensagem.length < 10) {
  throw new Error('Mensagem muito curta (mínimo 10 caracteres)');
}

// Sanitização
return {
  json: {
    nome: dados.nome.trim(),
    email: dados.email.toLowerCase().trim(),
    mensagem: dados.mensagem.trim(),
    data: new Date().toISOString(),
    ip: $input.first().json.headers['x-forwarded-for'] || 'N/A'
  }
};
```

#### Passo 3: Salvar no Google Sheets

```javascript
// Node: Google Sheets
{
  "operation": "append",
  "sheetId": "1A2B3C4D5E",
  "range": "Contatos!A:E",
  "values": [
    "={{$json.nome}}",
    "={{$json.email}}",
    "={{$json.mensagem}}",
    "={{$json.data}}",
    "={{$json.ip}}"
  ]
}
```

#### Passo 4: Enviar Notificação

```javascript
// Node: Telegram
{
  "chatId": "123456789",
  "text": `
🔔 *Novo Contato Recebido*

👤 *Nome:* {{$json.nome}}
📧 *Email:* {{$json.email}}
💬 *Mensagem:*
{{$json.mensagem}}

🕐 {{$json.data}}
  `,
  "parseMode": "Markdown"
}
```

#### Passo 5: Responder ao Cliente

```javascript
// Node: Respond to Webhook
{
  "respondWith": "json",
  "responseBody": {
    "success": true,
    "message": "Mensagem recebida com sucesso!",
    "timestamp": "={{$json.data}}"
  }
}
```

## 📊 Workflows Simples vs Complexos

### Workflow Simples

**Características:**
- 3-5 nodes
- Fluxo linear (A → B → C)
- Sem condicionais complexas
- Uma única integração
- Execução rápida (< 10 segundos)

**Exemplo: Backup Diário**
```
Schedule Trigger → Google Drive (Read) → Dropbox (Upload) → Email (Notificação)
```

**Código do Workflow:**
```javascript
// Node 1: Schedule - Todo dia às 2h da manhã
{
  "rule": {
    "interval": [{"field": "cronExpression", "expression": "0 2 * * *"}]
  }
}

// Node 2: Google Drive
{
  "operation": "download",
  "fileId": "documento-importante"
}

// Node 3: Dropbox
{
  "operation": "upload",
  "path": "/backups/{{$now.format('YYYY-MM-DD')}}-backup.pdf"
}

// Node 4: Email
{
  "toEmail": "admin@empresa.com",
  "subject": "Backup realizado com sucesso",
  "text": "Backup diário concluído em {{$now.format('DD/MM/YYYY HH:mm')}}"
}
```

### Workflow Complexo

**Características:**
- 10+ nodes
- Múltiplos caminhos (branches)
- Condicionais e loops
- Várias integrações
- Tratamento de erros robusto
- Execução demorada (minutos)

**Exemplo: Sistema de Processamento de Pedidos**
```
Webhook
  ↓
Validação
  ↓
IF (Estoque?)
  ├─ SIM → Processar Pagamento
  │           ↓
  │         IF (Aprovado?)
  │           ├─ SIM → Gerar Nota Fiscal → Enviar Email
  │           └─ NÃO → Notificar Falha
  │
  └─ NÃO → Notificar Sem Estoque
```

**Implementação Completa:**

```javascript
// Node: Function - Validação Complexa
const pedido = $input.first().json;
const erros = [];

// Validar cliente
if (!pedido.cliente?.cpf || pedido.cliente.cpf.length !== 11) {
  erros.push('CPF inválido');
}

// Validar itens
if (!Array.isArray(pedido.itens) || pedido.itens.length === 0) {
  erros.push('Pedido sem itens');
}

// Validar valores
const total = pedido.itens.reduce((sum, item) => {
  if (!item.preco || !item.quantidade) {
    erros.push(`Item ${item.nome} com dados incompletos`);
  }
  return sum + (item.preco * item.quantidade);
}, 0);

if (total !== pedido.total) {
  erros.push('Total do pedido não confere');
}

if (erros.length > 0) {
  throw new Error(`Validação falhou: ${erros.join(', ')}`);
}

return {
  json: {
    ...pedido,
    totalCalculado: total,
    validado: true,
    timestamp: new Date().toISOString()
  }
};
```

## 🔧 Técnicas Avançadas

### 1. Unificando Dados de Múltiplas Fontes

**Cenário:** Buscar informações de cliente em diferentes sistemas

```javascript
// Node: Function - Unificação de Dados
const clienteId = $input.first().json.clienteId;

// Dados já coletados dos nodes anteriores
const dadosCRM = $('CRM').first().json;
const dadosERP = $('ERP').first().json;
const dadosEmail = $('Marketing').first().json;

// Unificar informações
const clienteCompleto = {
  id: clienteId,
  
  // Informações básicas (CRM)
  nome: dadosCRM.nome,
  email: dadosCRM.email,
  telefone: dadosCRM.telefone,
  dataCadastro: dadosCRM.created_at,
  
  // Informações financeiras (ERP)
  limiteCredito: dadosERP.limite,
  saldo: dadosERP.saldo,
  ultimaCompra: dadosERP.ultima_compra,
  totalCompras: dadosERP.total_historico,
  
  // Informações de marketing
  emailsRecebidos: dadosEmail.total_emails,
  taxaAbertura: dadosEmail.open_rate,
  ultimoClique: dadosEmail.last_click,
  
  // Análise consolidada
  score: calcularScore(dadosCRM, dadosERP, dadosEmail),
  categoria: classificarCliente(dadosERP.total_historico),
  statusRisco: avaliarRisco(dadosERP.saldo, dadosERP.limite)
};

function calcularScore(crm, erp, email) {
  let score = 0;
  
  // Pontos por histórico de compras
  score += Math.min(erp.total_historico / 1000, 50);
  
  // Pontos por engajamento
  score += email.open_rate * 30;
  
  // Pontos por tempo de cliente
  const meses = monthDiff(new Date(crm.created_at), new Date());
  score += Math.min(meses, 20);
  
  return Math.round(score);
}

function classificarCliente(totalCompras) {
  if (totalCompras > 50000) return 'VIP';
  if (totalCompras > 10000) return 'Premium';
  if (totalCompras > 1000) return 'Regular';
  return 'Novo';
}

function avaliarRisco(saldo, limite) {
  const utilizacao = (limite - saldo) / limite;
  if (utilizacao > 0.9) return 'ALTO';
  if (utilizacao > 0.7) return 'MÉDIO';
  return 'BAIXO';
}

function monthDiff(d1, d2) {
  let months = (d2.getFullYear() - d1.getFullYear()) * 12;
  months -= d1.getMonth();
  months += d2.getMonth();
  return months <= 0 ? 0 : months;
}

return { json: clienteCompleto };
```

**Estrutura do Workflow:**
```
Manual Trigger
    ↓
Split In Batches (IDs dos clientes)
    ↓
    ├─→ HTTP Request (CRM) ──┐
    ├─→ HTTP Request (ERP) ──┼─→ Merge (Wait for all branches)
    └─→ HTTP Request (Marketing) ─┘
                                  ↓
                            Function (Unificação)
                                  ↓
                            Google Sheets (Salvar)
```

### 2. Enviando Emails Personalizados

**Cenário:** Sistema de email marketing com templates dinâmicos

```javascript
// Node: Function - Preparar Email
const cliente = $input.first().json;

// Template HTML
const templateHTML = `
<!DOCTYPE html>
<html>
<head>
  <style>
    body { font-family: Arial, sans-serif; }
    .header { background: #4CAF50; color: white; padding: 20px; }
    .content { padding: 20px; }
    .footer { background: #f1f1f1; padding: 10px; text-align: center; }
    .btn { 
      background: #4CAF50; 
      color: white; 
      padding: 10px 20px; 
      text-decoration: none;
      border-radius: 5px;
    }
  </style>
</head>
<body>
  <div class="header">
    <h1>Olá, ${cliente.nome}! 👋</h1>
  </div>
  
  <div class="content">
    <p>Temos uma oferta especial para você como cliente ${cliente.categoria}!</p>
    
    <h2>Seu desconto exclusivo: ${cliente.desconto}%</h2>
    
    <p>Válido até ${formatarData(cliente.validadeDesconto)}</p>
    
    <p>
      <a href="${gerarLinkPersonalizado(cliente.id)}" class="btn">
        Ver Ofertas
      </a>
    </p>
    
    <hr>
    
    <h3>Recomendações para você:</h3>
    <ul>
      ${cliente.recomendacoes.map(prod => `
        <li>${prod.nome} - R$ ${prod.preco}</li>
      `).join('')}
    </ul>
  </div>
  
  <div class="footer">
    <p>Você está recebendo este email porque é cliente da nossa loja.</p>
    <p><a href="${gerarLinkDescadastro(cliente.email)}">Cancelar inscrição</a></p>
  </div>
</body>
</html>
`;

function formatarData(data) {
  return new Date(data).toLocaleDateString('pt-BR');
}

function gerarLinkPersonalizado(clienteId) {
  return `https://loja.com/ofertas?ref=${clienteId}&utm_source=email&utm_campaign=desconto`;
}

function gerarLinkDescadastro(email) {
  const token = Buffer.from(email).toString('base64');
  return `https://loja.com/unsubscribe?token=${token}`;
}

return {
  json: {
    para: cliente.email,
    assunto: `${cliente.nome}, seu desconto de ${cliente.desconto}% te aguarda! 🎁`,
    html: templateHTML,
    from: 'ofertas@loja.com',
    replyTo: 'contato@loja.com'
  }
};
```

**Configuração do Node de Email:**
```javascript
// Node: Send Email (Gmail/SMTP)
{
  "fromEmail": "={{$json.from}}",
  "toEmail": "={{$json.para}}",
  "subject": "={{$json.assunto}}",
  "emailType": "html",
  "message": "={{$json.html}}",
  "options": {
    "replyTo": "={{$json.replyTo}}",
    "attachments": []
  }
}
```

### 3. Criando Loops para Processamento em Massa

**Cenário:** Processar lista de produtos e atualizar preços

```javascript
// Node: Loop Over Items
const items = $input.first().json.produtos;
const batchSize = 10;
const currentIndex = $node.context.get('currentIndex') || 0;

// Pegar próximo lote
const batch = items.slice(currentIndex, currentIndex + batchSize);

// Verificar se há mais itens
const hasMore = currentIndex + batchSize < items.length;

// Salvar progresso
$node.context.set('currentIndex', currentIndex + batchSize);

return {
  json: {
    items: batch,
    hasMore: hasMore,
    currentBatch: Math.floor(currentIndex / batchSize) + 1,
    totalBatches: Math.ceil(items.length / batchSize),
    progress: Math.round((currentIndex / items.length) * 100)
  }
};
```

**Loop Completo com Controle:**

```javascript
// Node: Function - Processamento em Loop
const produtos = $input.all();
const resultados = [];

for (const item of produtos) {
  const produto = item.json;
  
  try {
    // Buscar preço do concorrente
    const precoConcorrente = await buscarPreco(produto.sku);
    
    // Calcular novo preço
    const novoPreco = calcularPrecoCompetitivo(
      produto.precoAtual,
      precoConcorrente,
      produto.margemMinima
    );
    
    // Atualizar se necessário
    if (novoPreco !== produto.precoAtual) {
      await atualizarPrecoProduto(produto.id, novoPreco);
      
      resultados.push({
        sku: produto.sku,
        nome: produto.nome,
        precoAntigo: produto.precoAtual,
        precoNovo: novoPreco,
        precoConcorrente: precoConcorrente,
        status: 'ATUALIZADO',
        timestamp: new Date().toISOString()
      });
    } else {
      resultados.push({
        sku: produto.sku,
        status: 'SEM_ALTERACAO'
      });
    }
    
    // Delay para evitar rate limit
    await new Promise(resolve => setTimeout(resolve, 1000));
    
  } catch (erro) {
    resultados.push({
      sku: produto.sku,
      status: 'ERRO',
      erro: erro.message
    });
  }
  
  // Log de progresso
  const progresso = (resultados.length / produtos.length) * 100;
  console.log(`Progresso: ${progresso.toFixed(2)}%`);
}

function calcularPrecoCompetitivo(precoAtual, precoConcorrente, margemMinima) {
  // Se concorrente for mais barato, igualar com margem mínima
  if (precoConcorrente < precoAtual) {
    const novoPreco = precoConcorrente * (1 + margemMinima);
    return Math.max(novoPreco, precoAtual * 0.9); // Máximo 10% de desconto
  }
  return precoAtual;
}

return resultados.map(r => ({ json: r }));
```

**Estrutura de Loop com Condição:**
```
Start
  ↓
Get Batch of Items
  ↓
Process Each Item ←──┐
  ↓                  │
Check if Has More    │
  ├─ YES ────────────┘
  └─ NO
      ↓
    Send Summary Email
      ↓
    End
```

## 📝 Exemplos Práticos

### Exemplo 1: Automação de Onboarding de Clientes

```javascript
// Workflow completo
{
  "nodes": [
    {
      "name": "Novo Cliente",
      "type": "webhook",
      "parameters": {
        "path": "novo-cliente"
      }
    },
    {
      "name": "Criar no CRM",
      "type": "httpRequest",
      "parameters": {
        "url": "https://crm.empresa.com/api/clientes",
        "method": "POST"
      }
    },
    {
      "name": "Email Boas-Vindas",
      "type": "emailSend",
      "parameters": {
        "subject": "Bem-vindo à nossa empresa! 🎉"
      }
    },
    {
      "name": "Criar Tarefas Follow-up",
      "type": "function",
      "parameters": {
        "functionCode": `
          const tarefas = [
            { dias: 1, titulo: "Ligar para cliente", tipo: "LIGACAO" },
            { dias: 3, titulo: "Enviar material", tipo: "EMAIL" },
            { dias: 7, titulo: "Agendar reunião", tipo: "REUNIAO" },
            { dias: 30, titulo: "Avaliação de satisfação", tipo: "PESQUISA" }
          ];
          
          return tarefas.map(t => ({
            json: {
              clienteId: $input.first().json.id,
              titulo: t.titulo,
              tipo: t.tipo,
              dataVencimento: new Date(Date.now() + t.dias * 86400000).toISOString()
            }
          }));
        `
      }
    }
  ]
}
```

### Exemplo 2: Monitoramento de Estoque com Alertas

```javascript
// Node: Schedule Trigger - A cada hora
{
  "rule": {
    "interval": [{"field": "hours", "hoursInterval": 1}]
  }
}

// Node: Function - Verificar Estoque
const produtos = await buscarProdutos();
const alertas = [];

for (const produto of produtos) {
  const estoque = produto.quantidadeEstoque;
  const estoqueMinimo = produto.estoqueMinimo;
  const estoqueIdeal = produto.estoqueIdeal;
  
  let nivel = 'NORMAL';
  let urgencia = 'BAIXA';
  
  if (estoque === 0) {
    nivel = 'ZERADO';
    urgencia = 'CRITICA';
  } else if (estoque < estoqueMinimo) {
    nivel = 'CRITICO';
    urgencia = 'ALTA';
  } else if (estoque < estoqueIdeal) {
    nivel = 'BAIXO';
    urgencia = 'MEDIA';
  }
  
  if (nivel !== 'NORMAL') {
    alertas.push({
      produtoId: produto.id,
      nome: produto.nome,
      sku: produto.sku,
      estoqueAtual: estoque,
      estoqueMinimo: estoqueMinimo,
      nivel: nivel,
      urgencia: urgencia,
      sugestaoCompra: Math.max(estoqueIdeal - estoque, 0)
    });
  }
}

// Agrupar por urgência
const alertasCriticos = alertas.filter(a => a.urgencia === 'CRITICA');
const alertasAltos = alertas.filter(a => a.urgencia === 'ALTA');
const alertasMedios = alertas.filter(a => a.urgencia === 'MEDIA');

return {
  json: {
    timestamp: new Date().toISOString(),
    total: alertas.length,
    criticos: alertasCriticos.length,
    altos: alertasAltos.length,
    medios: alertasMedios.length,
    alertas: {
      criticos: alertasCriticos,
      altos: alertasAltos,
      medios: alertasMedios
    }
  }
};
```

## ✅ Boas Práticas

### 1. Nomenclatura Clara
```javascript
// ❌ Ruim
"Node1", "Function2", "HTTP3"

// ✅ Bom
"Buscar Dados Cliente", "Validar Pedido", "Enviar Email Confirmação"
```

### 2. Tratamento de Erros
```javascript
// Em cada node crítico
{
  "continueOnFail": true,
  "onError": "continueErrorOutput"
}

// Node: Error Handler
const erro = $input.first().json;

// Registrar erro
console.error({
  workflow: $workflow.name,
  node: erro.node,
  error: erro.error.message,
  timestamp: new Date().toISOString()
});

// Notificar equipe
await enviarAlertaErro(erro);
```

### 3. Documentação
```javascript
// Adicione comentários nos Function Nodes
/**
 * Calcula o desconto baseado na categoria do cliente
 * 
 * @param {Object} cliente - Dados do cliente
 * @param {string} cliente.categoria - VIP, Premium ou Regular
 * @returns {number} Percentual de desconto (0-30)
 */
function calcularDesconto(cliente) {
  const descontos = {
    'VIP': 30,
    'Premium': 20,
    'Regular': 10
  };
  
  return descontos[cliente.categoria] || 0;
}
```

### 4. Performance
```javascript
// Use Split In Batches para grandes volumes
{
  "batchSize": 50,
  "options": {
    "reset": false
  }
}

// Adicione delays quando necessário
{
  "amount": 1000,
  "unit": "ms"
}
```

## 🐛 Troubleshooting

### Problema: Workflow não executa

**Soluções:**
1. Verifique se o workflow está ativo
2. Confira o trigger (webhook, schedule, etc.)
3. Veja os logs de execução
4. Teste manualmente com "Execute Workflow"

### Problema: Dados não passam entre nodes

**Verificar:**
```javascript
// No Function Node, sempre retorne no formato correto
return items.map(item => ({
  json: {
    // seus dados aqui
  }
}));

// ❌ Errado
return { data: "value" };

// ✅ Correto
return { json: { data: "value" } };
```

### Problema: Loop infinito

**Prevenção:**
```javascript
// Sempre adicione condição de saída
const maxIterations = 100;
let currentIteration = $node.context.get('iteration') || 0;

if (currentIteration >= maxIterations) {
  throw new Error('Limite de iterações atingido');
}

$node.context.set('iteration', currentIteration + 1);
```

## 📚 Recursos Adicionais

- [Documentação Oficial N8N](https://docs.n8n.io/)
- [N8N Community](https://community.n8n.io/)
- [Workflow Templates](https://n8n.io/workflows/)

---

**Desenvolvido com ❤️ para automatizar processos e ganhar produtividade!**