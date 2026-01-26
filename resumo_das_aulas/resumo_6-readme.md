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

### Exemplo Visual de Workflow

![Workflow N8N - Automação de Aprovação](./imagens/imagem20.png)

*Exemplo de workflow de aprovação de documentos: O sistema verifica novos documentos no Google Drive ou formulários submetidos, solicita aprovação via Slack, verifica a resposta e envia email de confirmação ou feedback dependendo do resultado.*

### Componentes Básicos de um Workflow

**Estrutura:**
```
TRIGGER → AÇÃO 1 → AÇÃO 2 → CONDIÇÃO → RESULTADO
```

1. **TRIGGER (Gatilho)** - Evento que inicia o workflow
2. **AÇÃO 1** - Primeira operação executada
3. **AÇÃO 2** - Segunda operação executada
4. **CONDIÇÃO** - Verifica e decide o caminho
5. **RESULTADO** - Saída final do processo

### Tipos de Nodes

#### 1. **Trigger Nodes (Gatilhos)**
Iniciam o workflow automaticamente:
- ⏰ **Schedule** - Executa em horários programados (cron)
- 🌐 **Webhook** - Recebe requisições HTTP
- 👆 **Manual Trigger** - Execução manual
- 📧 **Email Trigger** - Monitora emails

#### 2. **Action Nodes (Ações)**
Executam operações específicas:
- 🌍 **HTTP Request** - Faz chamadas a APIs
- 💾 **Database** - Operações em bancos de dados
- 📁 **File Operations** - Manipula arquivos
- 🔌 **API Integrations** - Conecta com serviços

#### 3. **Logic Nodes (Lógica)**
Controlam o fluxo de execução:
- ❓ **IF** - Condições simples (se/senão)
- 🔀 **Switch** - Múltiplas condições
- 🔗 **Merge** - Combina dados de diferentes caminhos
- 🔄 **Loop** - Repete ações

#### 4. **Data Nodes (Manipulação de Dados)**
Transformam e processam informações:
- 📝 **Set** - Define valores
- ⚡ **Function** - JavaScript personalizado
- 🔍 **Filter** - Filtra dados
- 🔄 **Transform** - Modifica estrutura de dados

## 🚀 Criando seu Primeiro Workflow

### Workflow Básico: Monitoramento de Formulário Web

**Objetivo:** Receber dados de um formulário e enviar notificação

**Fluxo:**
```
Webhook → Validação → Google Sheets → Telegram → Email → Resposta
```

#### Passo 1: Configurar o Webhook (Trigger)

Crie um node **Webhook** para receber os dados do formulário:
- **HTTP Method:** POST
- **Path:** formulario-contato
- **Response Mode:** Response Node

Isso gerará uma URL como: `https://seu-n8n.com/webhook/formulario-contato`

#### Passo 2: Validar os Dados

Adicione um node **Function** para validar:
- Verificar se nome está preenchido
- Validar formato de email
- Checar tamanho mínimo da mensagem
- Sanitizar dados (remover espaços extras, converter para lowercase)

#### Passo 3: Salvar no Google Sheets

Configure o node **Google Sheets**:
- **Operation:** Append
- **Spreadsheet:** Selecione sua planilha
- **Sheet:** Nome da aba (ex: "Contatos")
- **Columns:** Nome, Email, Mensagem, Data, IP

#### Passo 4: Enviar Notificação no Telegram

Configure o node **Telegram**:
- **Chat ID:** ID do seu grupo/canal
- **Message:** Template com dados do formulário
- **Parse Mode:** Markdown (para formatação)

#### Passo 5: Responder ao Cliente

Use o node **Respond to Webhook**:
- **Response Type:** JSON
- **Body:** Mensagem de sucesso com timestamp

## 📊 Workflows Simples vs Complexos

### Workflow Simples

**Características:**
- ✅ 3-5 nodes apenas
- ✅ Fluxo linear (A → B → C → D)
- ✅ Uma única integração principal
- ✅ Execução rápida (< 10 segundos)
- ✅ Fácil manutenção

**Exemplo: Backup Automático Diário**

**Fluxo:**
```
Schedule (2h AM) → Google Drive (Download) → Dropbox (Upload) → Email (Notificação)
```

**Como funciona:**
1. Todo dia às 2h da manhã o workflow é acionado
2. Baixa documentos importantes do Google Drive
3. Faz upload no Dropbox como backup
4. Envia email de confirmação

---

### Workflow Complexo

**Características:**
- 🎯 10+ nodes
- 🎯 Múltiplos caminhos (branches)
- 🎯 Condicionais complexas
- 🎯 Várias integrações simultâneas
- 🎯 Tratamento de erros robusto
- 🎯 Execução pode levar minutos

**Exemplo: Sistema de Processamento de Pedidos**

**Fluxo Principal:**
```
Webhook (Novo Pedido)
  ↓
Validar Dados
  ↓
Verificar Estoque → [SIM] → Processar Pagamento → [APROVADO] → Gerar NF → Email/Telegram
                 → [NÃO] → Notificar Sem Estoque
                         → [RECUSADO] → Notificar Falha
```

**Como funciona:**
1. Recebe pedido via webhook
2. Valida todos os dados do cliente e produtos
3. Consulta estoque disponível
4. Se tem estoque, processa pagamento
5. Se pagamento aprovado, gera nota fiscal
6. Envia confirmação ao cliente e notifica equipe
7. Atualiza CRM e salva histórico
8. Trata erros em cada etapa

## 🔧 Técnicas Avançadas

### 1. Unificando Dados de Múltiplas Fontes

**Objetivo:** Criar uma visão 360° do cliente buscando informações de diferentes sistemas

**Sistema de Unificação:**
```
Buscar Cliente (ID: 12345)
  ↓
  ├── CRM (Dados Cadastrais)
  ├── ERP (Dados Financeiros)
  └── Marketing (Dados de Engajamento)
  ↓
Merge (Unificar Tudo)
  ↓
Function (Calcular Score)
  ↓
Classificar Cliente (VIP/Premium/Regular)
  ↓
Salvar Perfil Completo
```

**Dados Coletados:**
- **CRM:** Nome, email, telefone, data de cadastro
- **ERP:** Limite de crédito, saldo, histórico de compras
- **Marketing:** Taxa de abertura de emails, último clique, engajamento

**Análise Gerada:**
- Score do cliente (0-100)
- Classificação (VIP, Premium, Regular, Novo)
- Avaliação de risco (Alto, Médio, Baixo)

### 2. Enviando Emails Personalizados em Massa

**Objetivo:** Sistema de email marketing com templates dinâmicos por categoria de cliente

**Fluxo do Sistema:**
```
Google Sheets (Lista de Clientes)
  ↓
Loop para Cada Cliente
  ↓
Verificar Categoria
  ↓
  ├── VIP → Template com 30% desconto
  ├── Premium → Template com 20% desconto
  └── Regular → Template com 10% desconto
  ↓
Personalizar Email (Nome, Produtos, Ofertas)
  ↓
Enviar Email (SMTP/Gmail)
  ↓
Marcar como Enviado na Planilha
  ↓
Se houver erro → Registrar + Retry
```

**Personalização Automática:**
- Nome do cliente
- Categoria e desconto correspondente
- Produtos recomendados baseados no histórico
- Link personalizado com tracking
- Data de validade da oferta

**Template HTML Dinâmico inclui:**
- Header com nome do cliente
- Desconto exclusivo destacado
- Lista de recomendações personalizadas
- Botão de call-to-action com link rastreável
- Footer com opção de descadastro

### 3. Criando Loops para Processamento em Massa

**Objetivo:** Processar grandes listas de produtos e atualizar preços automaticamente

**Sistema de Loop Otimizado:**
```
Schedule (Diário 6h AM)
  ↓
Buscar Produtos (Database)
  ↓
Split in Batches (Lotes de 50)
  ↓
Para Cada Produto:
  ├── Buscar Preço Concorrente (API)
  ├── Calcular Novo Preço
  ├── Atualizar Banco de Dados
  └── Registrar Log
  ↓
Wait 2 segundos (Evitar Rate Limit)
  ↓
Próximo Lote → Repetir até terminar
  ↓
Relatório Final + Notificação Telegram
```

**Otimizações Implementadas:**
- **Processamento em Lotes:** 50 produtos por vez
- **Delays Controlados:** 2 segundos entre lotes
- **Log Completo:** Histórico de todas as alterações
- **Controle de Erros:** Continua mesmo se um item falhar
- **Relatório Final:** Quantos atualizados, erros, tempo total

**Lógica de Precificação:**
- Se concorrente mais barato → Igualar com margem mínima
- Máximo de 10% de desconto
- Respeitar margem mínima configurada
- Registrar todas as mudanças

## 📝 Exemplos Práticos

### Exemplo 1: Onboarding Automático de Novos Clientes

**Fluxo Completo:**
```
Webhook (Novo Cliente)
  ↓
Validar Dados (CPF, Email, Nome)
  ↓
[Dados OK?]
  ├── SIM → Criar no CRM
  │         ↓
  │       Email Boas-Vindas
  │         ↓
  │       Telegram (Notificar Vendedor)
  │         ↓
  │       Criar Tarefas Follow-up:
  │         ├── Day 1: Ligar Cliente
  │         ├── Day 3: Enviar Material
  │         ├── Day 7: Agendar Reunião
  │         └── Day 30: Pesquisa NPS
  │
  └── NÃO → Retornar Erro
```

**Benefícios:**
- Cliente recebe boas-vindas imediatamente
- Vendedor é notificado em tempo real
- Tarefas criadas automaticamente no cronograma ideal
- Zero trabalho manual

---

### Exemplo 2: Monitoramento de Estoque com Alertas Inteligentes

**Sistema de Verificação:**
```
Schedule (A Cada Hora)
  ↓
Buscar Todos os Produtos
  ↓
Para Cada Produto, Verificar:
  ├── Estoque ZERADO → Email URGENTE + Telegram
  ├── Estoque < Mínimo → Alerta de Atenção
  ├── Estoque < Ideal → Lista para Próxima Compra
  └── Estoque Normal → Sem Ação
  ↓
Dashboard com Resumo
  ↓
Gráfico de Status por Categoria
```

**Níveis de Alerta:**
- 🚨 **CRÍTICO:** Estoque zerado - urgência alta
- ⚠️ **BAIXO:** Abaixo do mínimo - urgência média
- 📊 **ATENÇÃO:** Abaixo do ideal - urgência baixa
- ✅ **NORMAL:** Sem necessidade de ação

**Ações Automáticas:**
- Email para equipe de compras (críticos)
- Notificação Telegram (baixos)
- Relatório consolidado diário
- Sugestão de quantidade a comprar

---

### Exemplo 3: Sistema de Backup Multi-Cloud

**Redundância Tripla:**
```
Schedule (Diário 2h AM)
  ↓
Google Drive (Listar Arquivos)
  ↓
[Arquivo modificado nas últimas 24h?]
  ├── SIM → Download
  │         ↓
  │       Upload Paralelo:
  │         ├── Dropbox
  │         ├── OneDrive
  │         └── Amazon S3
  │         ↓
  │       Verificar Integridade (3 locais)
  │         ↓
  │       [Backup OK em todos?]
  │         ├── SIM → Log Sucesso
  │         └── NÃO → Alerta Erro
  │
  └── NÃO → Próximo Arquivo
  ↓
Relatório Diário + Dashboard
```

**Segurança:**
- Backup em 3 locais diferentes
- Verificação de integridade
- Histórico de versões
- Alertas de falha imediatos

## ✅ Boas Práticas

### 1. Nomenclatura Clara de Nodes

**❌ Ruim:**
- "Node1"
- "Function2"
- "HTTP3"

**✅ Bom:**
- "Buscar Dados do Cliente"
- "Validar Pedido"
- "Enviar Email de Confirmação"
- "Atualizar Status no CRM"

### 2. Organização Visual

- **Agrupe nodes relacionados** usando cores ou notas
- **Documente condições** nos IF/Switch
- **Use notas** para explicar lógicas complexas
- **Mantenha fluxo da esquerda para direita**

### 3. Tratamento de Erros Robusto

**Em cada node crítico configure:**
- **Continue on Fail:** true (para não parar o workflow)
- **Retry on Fail:** true (tentar novamente)
- **Max Tries:** 3 (número de tentativas)

**Adicione nodes de tratamento:**
- Error Handler para capturar erros
- Log de erros em arquivo/banco
- Notificação para equipe
- Alternativas (fallback)

### 4. Otimização de Performance

**Use Split in Batches para grandes volumes:**
- Processe em lotes de 50-100 itens
- Adicione delays entre lotes (1-2 segundos)
- Evite rate limits de APIs

**Cache quando possível:**
- Use Set/Get nodes para armazenar dados temporários
- Evite chamadas desnecessárias a APIs
- Reutilize dados já buscados

### 5. Documentação

- Adicione comentários nos Function Nodes
- Documente credenciais e suas permissões
- Mantenha README do workflow
- Registre mudanças importantes

## 🐛 Troubleshooting

### Problemas Comuns e Soluções

| Problema | Causa Provável | Solução |
|----------|---------------|---------|
| **Workflow não executa** | Trigger desativado | Ativar workflow no botão superior direito |
| **Dados não passam entre nodes** | Formato de retorno incorreto | Sempre retornar `{ json: {...} }` em Functions |
| **Loop infinito** | Sem condição de saída | Adicionar contador e limite máximo de iterações |
| **Timeout em requisições** | Muitas chamadas simultâneas | Usar Split in Batches + Wait entre lotes |
| **Credenciais expiradas** | Token OAuth vencido | Renovar autorização nas credenciais |
| **Erro "Cannot read property"** | Dados ausentes/nulos | Validar dados antes de usar com `?.` ou verificações |
| **Rate Limit da API** | Muitas requisições rápidas | Implementar delays e respeitar limites |

### Fluxo de Diagnóstico

**Quando algo der errado:**

1. **Verificar Executions (Histórico)**
   - Veja qual node falhou
   - Analise o erro específico
   - Confira os dados de entrada

2. **Testar Manualmente**
   - Execute o workflow manualmente
   - Teste cada node individualmente
   - Verifique as conexões entre nodes

3. **Verificar Credenciais**
   - Confirme se estão válidas
   - Teste a conexão
   - Renove se necessário

4. **Consultar Logs**
   - Ative log detalhado se necessário
   - Veja console do navegador
   - Confira logs do servidor N8N

5. **Simplificar para Isolar**
   - Desative nodes não essenciais
   - Teste com dados mockados
   - Isole o problema

## 📚 Recursos Adicionais

### Documentação Oficial
- [N8N Documentation](https://docs.n8n.io/) - Documentação completa
- [N8N Community](https://community.n8n.io/) - Fórum da comunidade
- [Workflow Templates](https://n8n.io/workflows/) - Templates prontos
- [Node Reference](https://docs.n8n.io/integrations/) - Referência de todos os nodes

### Aprendizado
- [Canal Oficial N8N no YouTube](https://www.youtube.com/@n8n-io)
- [N8N Academy](https://academy.n8n.io/) - Cursos gratuitos
- [Blog N8N](https://blog.n8n.io/) - Artigos e tutoriais

### Templates Prontos para Usar

Acesse [n8n.io/workflows](https://n8n.io/workflows) para encontrar centenas de workflows prontos, incluindo:

- 📧 **Automações de Email Marketing**
- 📱 **Integrações com Redes Sociais**
- 💾 **Processamento de Dados e ETL**
- 🔔 **Sistemas de Notificações**
- 🛒 **E-commerce e Vendas**
- 📊 **Relatórios Automáticos**
- 🎫 **Gestão de Tickets**
- 📅 **Agendamentos e Lembretes**

---

**Desenvolvido com ❤️ para automatizar processos e ganhar produtividade!**