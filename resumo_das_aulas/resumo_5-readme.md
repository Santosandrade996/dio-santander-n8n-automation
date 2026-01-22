# Sistema de Notificações de Artigos com N8N

## 📋 Índice

- [Sobre o Projeto](#sobre-o-projeto)
- [Arquitetura do Sistema](#arquitetura-do-sistema)
- [Pré-requisitos](#pré-requisitos)
- [Configuração Inicial](#configuração-inicial)
- [Integrações Implementadas](#integrações-implementadas)
- [Workflows Disponíveis](#workflows-disponíveis)
- [Solução de Problemas](#solução-de-problemas)
- [Boas Práticas](#boas-práticas)
- [Recursos Adicionais](#recursos-adicionais)

## 🎯 Sobre o Projeto

Este projeto demonstra a implementação completa de um sistema automatizado de notificações de artigos utilizando N8N, integrando múltiplos serviços populares como Google Sheets, Telegram e APIs REST personalizadas (UZAPI).

O sistema foi desenvolvido seguindo o curso de integração com serviços populares no N8N, abordando desde conceitos básicos até implementações avançadas com tratamento de erros e otimizações.

## 🏗️ Arquitetura do Sistema

```
┌─────────────────┐
│  Google Sheets  │ ← Fonte de dados (artigos)
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│   N8N Workflow  │ ← Processamento e lógica
└────────┬────────┘
         │
         ├─────→ ┌──────────────┐
         │       │   Telegram   │ ← Notificações
         │       └──────────────┘
         │
         └─────→ ┌──────────────┐
                 │    UZAPI     │ ← Envio WhatsApp
                 └──────────────┘
```

## 📦 Pré-requisitos

### Software Necessário

- N8N instalado (versão 1.0+)
- Node.js (versão 18+)
- Docker (opcional, para instalação containerizada)

### Contas e Credenciais

- Conta Google (para Google Sheets)
- Bot do Telegram criado via @BotFather
- Conta UZAPI (para integração WhatsApp)
- Credenciais de API para cada serviço

## ⚙️ Configuração Inicial

### 1. Instalação do N8N

#### Via NPM

```bash
npm install n8n -g
n8n start
```

#### Via Docker

```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
```

### 2. Configuração de Credenciais

#### Google Sheets (OAuth2)

1. Acesse o [Google Cloud Console](https://console.cloud.google.com)
2. Crie um novo projeto ou selecione existente
3. Ative a API do Google Sheets
4. Configure a tela de consentimento OAuth
5. Crie credenciais OAuth 2.0
6. No N8N:
   - Vá em **Credentials** → **New**
   - Selecione **Google Sheets OAuth2 API**
   - Insira Client ID e Client Secret
   - Clique em **Connect** e autorize

**Escopos necessários:**

```
https://www.googleapis.com/auth/spreadsheets
https://www.googleapis.com/auth/drive.readonly
```

#### Telegram Bot

1. Abra o Telegram e busque por `@BotFather`
2. Digite `/newbot` e siga as instruções
3. Copie o token fornecido
4. No N8N:
   - Credentials → New → Telegram API
   - Cole o Access Token
   - Teste a conexão

**Obtendo Chat ID:**

```bash
# Envie uma mensagem para o bot e execute:
curl https://api.telegram.org/bot<SEU_TOKEN>/getUpdates
```

#### UZAPI (API REST)

1. Acesse [https://uzapi.com.br/](https://uzapi.com.br/) e registre-se na plataforma
2. Obtenha sua API Key no painel de controle
3. No N8N:
   - Credentials → New → Header Auth
   - Header Name: `Authorization`
   - Header Value: `Bearer SEU_TOKEN`

## 🔗 Integrações Implementadas

### 1. Google Sheets - Integração Nativa

**Funcionalidades:**

- Leitura de planilhas
- Filtros e consultas
- Atualização de status
- Processamento em lote

**Exemplo de Configuração:**

```javascript
// Node: Google Sheets
{
  "operation": "read",
  "sheetId": "1A2B3C4D5E6F7G8H9I0J",
  "range": "Artigos!A2:E",
  "filters": {
    "status": "pendente"
  }
}
```

**Estrutura da Planilha Esperada:**

| Título | URL | Categoria | Status | Data |
|--------|-----|-----------|--------|------|
| Artigo 1 | https://... | Tech | pendente | 2026-01-20 |
| Artigo 2 | https://... | News | enviado | 2026-01-19 |

### 2. Telegram - Integração Nativa

**Funcionalidades:**

- Envio de mensagens formatadas (Markdown/HTML)
- Envio de imagens e documentos
- Botões interativos (Inline Keyboard)
- Grupos e canais

**Exemplo de Mensagem Formatada:**

```javascript
// Node: Telegram
{
  "chatId": "123456789",
  "text": `
📰 *Novo Artigo Disponível*

*Título:* {{$json["titulo"]}}
*Categoria:* {{$json["categoria"]}}
*Link:* {{$json["url"]}}

_Publicado em {{$json["data"]}}_
  `,
  "parseMode": "Markdown"
}
```

### 3. UZAPI - Integração Não-Nativa (HTTP Request)

**Funcionalidades:**

- Envio de mensagens WhatsApp
- Upload de mídia
- Grupos e listas de transmissão

**Exemplo de Request:**

```javascript
// Node: HTTP Request
{
  "method": "POST",
  "url": "https://api.uzapi.com/v1/messages/send",
  "headers": {
    "Authorization": "Bearer {{$credentials.uzapi}}",
    "Content-Type": "application/json"
  },
  "body": {
    "phone": "5521999999999",
    "message": "{{$json["mensagem"]}}",
    "instance": "minha-instancia"
  }
}
```

## 🔄 Workflows Disponíveis

### Workflow 1: Notificação Simples de Artigos

**Objetivo:** Buscar artigos pendentes e notificar via Telegram

**Estrutura:**

1. **Schedule Trigger** - Executa diariamente às 9h
2. **Google Sheets** - Busca artigos com status "pendente"
3. **IF Node** - Verifica se há artigos novos
4. **Telegram** - Envia notificação
5. **Google Sheets Update** - Marca como "enviado"

**Configuração do Schedule:**

```
Cron: 0 9 * * *
Timezone: America/Sao_Paulo
```

### Workflow 2: Sistema Completo com Múltiplos Canais

**Objetivo:** Notificar artigos via Telegram e WhatsApp com validações

**Estrutura:**

1. **Webhook** - Recebe trigger manual ou via API
2. **Google Sheets Read** - Busca dados
3. **Function Node** - Formata e valida dados
4. **Split In Batches** - Processa em lotes de 5
5. **Switch Node** - Roteia por categoria
6. **Telegram Send** - Notifica canal Tech
7. **UZAPI Send** - Notifica WhatsApp corporativo
8. **Error Handling** - Captura e registra erros
9. **Google Sheets Update** - Atualiza status

**Exemplo de Function Node:**

```javascript
// Validação e formatação
const items = $input.all();

return items.map(item => {
  const json = item.json;
  
  // Validações
  if (!json.titulo || !json.url) {
    throw new Error('Dados incompletos');
  }
  
  // Formatação
  return {
    json: {
      ...json,
      mensagem_telegram: `📰 ${json.titulo}\n${json.url}`,
      mensagem_whatsapp: `Novo artigo: ${json.titulo}\nAcesse: ${json.url}`,
      timestamp: new Date().toISOString()
    }
  };
});
```

### Workflow 3: Notificações com Retry Logic

**Recursos Avançados:**

- Tentativas automáticas em caso de falha
- Exponential backoff
- Logging detalhado
- Alertas de erro

**Configuração de Retry:**

```javascript
// Settings do Node HTTP Request
{
  "retry": {
    "maxTries": 3,
    "waitBetweenTries": 5000,
    "backoff": "exponential"
  },
  "timeout": 10000
}
```

## 🐛 Solução de Problemas

### Erros Comuns e Soluções

#### 1. Erro de Autenticação Google Sheets

**Sintoma:**

```
ERROR: The service account does not have permission to access this resource
```

**Solução:**

- Verifique se o email de serviço tem acesso à planilha
- Compartilhe a planilha com o email OAuth2
- Confirme os escopos da API no Google Cloud Console

#### 2. Timeout no Telegram

**Sintoma:**

```
ERROR: Request timeout - No response after 60000ms
```

**Solução:**

```javascript
// Adicione timeout e retry
{
  "timeout": 30000,
  "continueOnFail": true
}
```

#### 3. Rate Limit na UZAPI

**Sintoma:**

```
ERROR: 429 Too Many Requests
```

**Solução:**

- Implemente `Split In Batches` com delay
- Configure `Wait` node entre requisições

```javascript
// Node: Split In Batches
{
  "batchSize": 5,
  "options": {
    "reset": false
  }
}

// Seguido de Wait Node
{
  "amount": 2000, // 2 segundos
  "unit": "ms"
}
```

#### 4. Dados Malformados do Google Sheets

**Sintoma:**

```
ERROR: Cannot read property 'titulo' of undefined
```

**Solução:**

```javascript
// Function Node para sanitização
const items = $input.all();

return items
  .filter(item => item.json && Object.keys(item.json).length > 0)
  .map(item => ({
    json: {
      titulo: String(item.json.titulo || '').trim(),
      url: String(item.json.url || '').trim(),
      categoria: String(item.json.categoria || 'Geral').trim(),
      status: String(item.json.status || 'pendente').toLowerCase()
    }
  }));
```

### Debugging Avançado

#### Habilitar Logs Detalhados

```bash
# .env do N8N
N8N_LOG_LEVEL=debug
N8N_LOG_OUTPUT=console,file
N8N_LOG_FILE_LOCATION=/logs/n8n.log
```

#### Testando Conexões Individuais

```javascript
// Node: HTTP Request (Test)
{
  "method": "GET",
  "url": "{{$credentials.uzapi_url}}/health",
  "headers": {
    "Authorization": "Bearer {{$credentials.uzapi_token}}"
  }
}
```

## ✅ Boas Práticas

### Gerenciamento de Credenciais

1. **Nunca exponha credenciais no código**
   - Use sempre o sistema de Credentials do N8N
   - Rotacione tokens periodicamente

2. **Organize por ambiente**

   ```
   - Google_Sheets_Prod
   - Google_Sheets_Dev
   - Telegram_Bot_Prod
   - Telegram_Bot_Test
   ```

3. **Documentação de permissões**
   - Mantenha registro dos escopos necessários
   - Documente IPs autorizados

### Otimização de Workflows

1. **Processamento em Lote**
   - Use `Split In Batches` para grandes volumes
   - Limite de 100 itens por execução

2. **Cache de Dados**

   ```javascript
   // Use Set/Get nodes para cache
   // Set Node
   {
     "key": "artigos_cache",
     "value": "={{$json}}",
     "ttl": 3600 // 1 hora
   }
   ```

3. **Tratamento de Erros Robusto**

   ```javascript
   // Em cada node crítico
   {
     "continueOnFail": true,
     "onError": "continueErrorOutput"
   }
   
   // Adicione Error Trigger
   // Para capturar e notificar falhas
   ```

### Monitoramento

1. **Logs Estruturados**

   ```javascript
   // Function Node
   console.log(JSON.stringify({
     workflow: $workflow.name,
     execution: $execution.id,
     node: $node.name,
     data: $input.all().length,
     timestamp: new Date().toISOString()
   }));
   ```

2. **Alertas de Falha**
   - Configure workflow de monitoramento
   - Envie notificações em caso de erro
   - Monitore tempo de execução

3. **Métricas**
   - Taxa de sucesso por workflow
   - Tempo médio de execução
   - Volume de dados processados

## 📚 Recursos Adicionais

### Documentação Oficial

- [N8N Documentation](https://docs.n8n.io/)
- [Google Sheets API](https://developers.google.com/sheets/api)
- [Telegram Bot API](https://core.telegram.org/bots/api)

### Templates Úteis

#### Template: Notificação com Retry

```json
{
  "nodes": [
    {
      "name": "Trigger",
      "type": "n8n-nodes-base.scheduleTrigger"
    },
    {
      "name": "Get Data",
      "type": "n8n-nodes-base.googleSheets"
    },
    {
      "name": "Try Send",
      "type": "n8n-nodes-base.httpRequest",
      "retryOnFail": true,
      "maxTries": 3
    }
  ]
}
```

### Comunidade

- [N8N Community Forum](https://community.n8n.io/)
- [GitHub Issues](https://github.com/n8n-io/n8n/issues)
- [Discord Server](https://discord.gg/n8n)

## 🤝 Contribuindo

Contribuições são bem-vindas! Para contribuir:

1. Fork o projeto
2. Crie uma branch (`git checkout -b feature/nova-feature`)
3. Commit suas mudanças (`git commit -m 'Adiciona nova feature'`)
4. Push para a branch (`git push origin feature/nova-feature`)
5. Abra um Pull Request

## 📄 Licença

Este projeto é distribuído sob a licença MIT. Veja `LICENSE` para mais informações.

**Desenvolvido com ❤️ utilizando N8N**
