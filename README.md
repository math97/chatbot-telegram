# Bot de Clima no Telegram com N8N

> Desafio Prático da Fase 2 - Agentes e Automações | Pós-graduação Rocketseat

---

## Sumário / Table of Contents

- [Versão em Português](#versão-em-português)
- [English Version](#english-version)

---

# Versão em Português

## Descrição

Este projeto é um chatbot para Telegram que informa a temperatura atual de qualquer cidade do mundo. O bot utiliza a API do OpenWeather para obter dados meteorológicos e, opcionalmente, o Google Gemini para gerar respostas mais naturais e humanizadas.

### Fluxo do Workflow

```
[Telegram Trigger] → [Set Queue Variable] → [OpenWeather API] → [Validate Response]
                                                                        ↓
                                            ┌───────────────────────────┴───────────────────────────┐
                                            ↓                                                       ↓
                                    [Edit Fields]                                    [Send Error Message]
                                            ↓
                                    [Basic LLM Chain (Gemini)]
                                            ↓
                            ┌───────────────┴───────────────┐
                            ↓                               ↓
                    [Send Response]                 [Fallback Code]
                                                            ↓
                                                    [Send Response]
```

## Funcionalidades

- Recebe o nome de uma cidade via Telegram
- Consulta a API do OpenWeather para obter a temperatura atual
- Formata a resposta com tom amigável usando IA (opcional)
- Possui fallback determinístico caso o Gemini não esteja disponível
- Tratamento de erros para cidades não encontradas

## Pré-requisitos

- [N8N](https://n8n.io/) instalado (self-hosted ou cloud)
- Bot do Telegram criado via [@BotFather](https://t.me/BotFather)
- Conta na [OpenWeather](https://openweathermap.org/api) com API Key
- (Opcional) Conta no [Google AI Studio](https://aistudio.google.com/) para o Gemini

## Variáveis e Credenciais Necessárias

### Variáveis de Ambiente

| Variável | Descrição | Obrigatório |
|----------|-----------|-------------|
| `OPENWEATHER_API_KEY` | Chave de API do OpenWeather | Sim |

### Credenciais N8N (Settings > Credentials)

| Credencial | Tipo no N8N | Descrição | Obrigatório |
|------------|-------------|-----------|-------------|
| Telegram Bot | Telegram API | Token do bot obtido via BotFather | Sim |
| Google Gemini | Google PaLM API | Chave de API do Google AI Studio | Não (opcional) |

## Como Importar e Configurar

### 1. Importar o Workflow

1. Acesse sua instância do N8N
2. Clique em **"Import from File"** ou **"Import from URL"**
3. Selecione o arquivo `workflow-chatbot-telegram.json`

### 2. Configurar Credenciais do Telegram

1. No N8N, vá em **Settings > Credentials**
2. Clique em **"Add Credential"**
3. Selecione **"Telegram API"**
4. Insira:
   - **Access Token**: Seu `TELEGRAM_BOT_TOKEN` obtido do BotFather
5. **Reconecte as credenciais nos nós do workflow:**
   - Clique em cada nó do Telegram (Telegram Trigger, Send Error Message, Send Weather Response)
   - Selecione a credencial que você acabou de criar

### 3. Configurar Variável de Ambiente do OpenWeather

O workflow utiliza uma variável de ambiente para a API key do OpenWeather. Configure-a no N8N:

**Para N8N Self-Hosted (Docker):**
```bash
# No seu docker-compose.yml ou variáveis de ambiente
OPENWEATHER_API_KEY=sua_api_key_aqui
```

**Para N8N Self-Hosted (npm):**
```bash
export OPENWEATHER_API_KEY=sua_api_key_aqui
```

**Para N8N Cloud:**
1. Vá em **Settings > Variables**
2. Adicione uma nova variável:
   - **Key**: `OPENWEATHER_API_KEY`
   - **Value**: Sua chave de API do OpenWeather

> **Nota**: O workflow já está configurado para usar `{{ $env.OPENWEATHER_API_KEY }}` automaticamente.

### 4. Configurar Google Gemini (Opcional)

O nó Gemini está localizado após a validação da resposta do OpenWeather:

1. No N8N, vá em **Settings > Credentials**
2. Clique em **"Add Credential"**
3. Selecione **"Google PaLM API"** (usado para Gemini)
4. Insira sua `GOOGLE_GEMINI_API_KEY`
5. No workflow, clique no nó **"Google Gemini Chat Model"**
6. Selecione a credencial configurada

**Nota**: Se as credenciais do Gemini não estiverem configuradas, o workflow utilizará automaticamente o **Fallback Code** para gerar uma resposta simples.

### 5. Ativar o Workflow

1. Salve todas as alterações
2. Clique no toggle para ativar o workflow
3. O webhook do Telegram será registrado automaticamente

## Como Testar

### Via Telegram

1. Abra o Telegram e encontre seu bot pelo username configurado no BotFather
2. Envie uma mensagem com o nome de uma cidade e código do País:
   ```
   São Paulo, BR
   ```

### Exemplos de Entrada e Saída

**Entrada:**
```
Rio de Janeiro, BR
```

**Saída (com Gemini):**
```
Olha, o pessoal no Rio de Janeiro está curtindo um clima bem agradável! 🇧🇷
No momento, faz 28°C. Perfeito para uma praia! 🏖️💧
```

**Saída (Fallback):**
```
🌤️ A temperatura em Rio de Janeiro é de 28°C.
```

**Entrada inválida:**
```
cidadeinexistente123
```

**Saída de erro:**
```
❌ Cidade não encontrada. Use o formato Cidade,UF (ex.: São Paulo, BR).
```

## Estrutura dos Nós

| Nó | Descrição |
|----|-----------|
| Telegram Trigger | Recebe mensagens do usuário |
| Set Queue Variable | Normaliza o texto (lowercase, trim) |
| OpenWeather API | Consulta a API de clima |
| Validate Response | Verifica se a cidade foi encontrada |
| Edit Fields | Extrai cidade, país e temperatura |
| Basic LLM Chain | Formata resposta com Gemini |
| Google Gemini Chat Model | Modelo de IA para respostas naturais |
| Fallback Code | Resposta simples sem IA |
| Send Weather Response | Envia resposta ao usuário |
| Send Error Message | Envia mensagem de erro |

## Solução de Problemas

| Problema | Solução |
|----------|---------|
| Bot não responde | Verifique se o workflow está ativo e as credenciais do Telegram estão corretas |
| "Cidade não encontrada" para cidades válidas | Tente adicionar o código do país (ex: "Paris, FR") |
| Erro de API do OpenWeather | Verifique se a API key está correta e ativa |
| Gemini não funciona | O fallback será usado automaticamente; verifique as credenciais se quiser usar o Gemini |

---

# English Version

## Description

This project is a Telegram chatbot that provides current temperature information for any city in the world. The bot uses the OpenWeather API to fetch weather data and, optionally, Google Gemini to generate more natural and humanized responses.

### Workflow Flow

```
[Telegram Trigger] → [Set Queue Variable] → [OpenWeather API] → [Validate Response]
                                                                        ↓
                                            ┌───────────────────────────┴───────────────────────────┐
                                            ↓                                                       ↓
                                    [Edit Fields]                                    [Send Error Message]
                                            ↓
                                    [Basic LLM Chain (Gemini)]
                                            ↓
                            ┌───────────────┴───────────────┐
                            ↓                               ↓
                    [Send Response]                 [Fallback Code]
                                                            ↓
                                                    [Send Response]
```

## Features

- Receives city name via Telegram
- Queries OpenWeather API for current temperature
- Formats response with friendly tone using AI (optional)
- Has deterministic fallback when Gemini is unavailable
- Error handling for cities not found

## Prerequisites

- [N8N](https://n8n.io/) installed (self-hosted or cloud)
- Telegram bot created via [@BotFather](https://t.me/BotFather)
- [OpenWeather](https://openweathermap.org/api) account with API Key
- (Optional) [Google AI Studio](https://aistudio.google.com/) account for Gemini

## Required Variables and Credentials

### Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `OPENWEATHER_API_KEY` | OpenWeather API key | Yes |

### N8N Credentials (Settings > Credentials)

| Credential | N8N Type | Description | Required |
|------------|----------|-------------|----------|
| Telegram Bot | Telegram API | Bot token obtained via BotFather | Yes |
| Google Gemini | Google PaLM API | Google AI Studio API key | No (optional) |

## How to Import and Configure

### 1. Import the Workflow

1. Access your N8N instance
2. Click on **"Import from File"** or **"Import from URL"**
3. Select the `workflow-chatbot-telegram.json` file

### 2. Configure Telegram Credentials

1. In N8N, go to **Settings > Credentials**
2. Click **"Add Credential"**
3. Select **"Telegram API"**
4. Enter:
   - **Access Token**: Your `TELEGRAM_BOT_TOKEN` from BotFather
5. **Reconnect credentials in workflow nodes:**
   - Click on each Telegram node (Telegram Trigger, Send Error Message, Send Weather Response)
   - Select the credential you just created

### 3. Configure OpenWeather Environment Variable

The workflow uses an environment variable for the OpenWeather API key. Configure it in N8N:

**For N8N Self-Hosted (Docker):**
```bash
# In your docker-compose.yml or environment variables
OPENWEATHER_API_KEY=your_api_key_here
```

**For N8N Self-Hosted (npm):**
```bash
export OPENWEATHER_API_KEY=your_api_key_here
```

**For N8N Cloud:**
1. Go to **Settings > Variables**
2. Add a new variable:
   - **Key**: `OPENWEATHER_API_KEY`
   - **Value**: Your OpenWeather API key

> **Note**: The workflow is already configured to use `{{ $env.OPENWEATHER_API_KEY }}` automatically.

### 4. Configure Google Gemini (Optional)

The Gemini node is located after the OpenWeather response validation:

1. In N8N, go to **Settings > Credentials**
2. Click **"Add Credential"**
3. Select **"Google PaLM API"** (used for Gemini)
4. Enter your `GOOGLE_GEMINI_API_KEY`
5. In the workflow, click on the **"Google Gemini Chat Model"** node
6. Select the configured credential

**Note**: If Gemini credentials are not configured, the workflow will automatically use the **Fallback Code** to generate a simple response.

### 5. Activate the Workflow

1. Save all changes
2. Click the toggle to activate the workflow
3. The Telegram webhook will be registered automatically

## How to Test

### Via Telegram

1. Open Telegram and find your bot by the username configured in BotFather
2. Send a message with a city name and country code:
   ```
   New York, US
   ```

### Input and Output Examples

**Input:**
```
Tokyo, JP
```

**Output (with Gemini):**
```
Hey there! The folks in Tokyo are enjoying some nice weather! 🇯🇵
Right now, it's 22°C. Perfect weather for a walk! 🚶‍♂️
```

**Output (Fallback):**
```
🌤️ A temperatura em Tokyo é de 22°C.
```

**Invalid input:**
```
nonexistentcity123
```

**Error output:**
```
❌ Cidade não encontrada. Use o formato Cidade,UF (ex.: São Paulo, BR).
```

## Node Structure

| Node | Description |
|------|-------------|
| Telegram Trigger | Receives user messages |
| Set Queue Variable | Normalizes text (lowercase, trim) |
| OpenWeather API | Queries weather API |
| Validate Response | Checks if city was found |
| Edit Fields | Extracts city, country, and temperature |
| Basic LLM Chain | Formats response with Gemini |
| Google Gemini Chat Model | AI model for natural responses |
| Fallback Code | Simple response without AI |
| Send Weather Response | Sends response to user |
| Send Error Message | Sends error message |

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Bot doesn't respond | Check if workflow is active and Telegram credentials are correct |
| "City not found" for valid cities | Try adding country code (e.g., "Paris, FR") |
| OpenWeather API error | Check if API key is correct and active |
| Gemini not working | Fallback will be used automatically; check credentials if you want to use Gemini |

---

## License

This project was created as part of the practical challenge for the Rocketseat postgraduate program.

## Author

Created for the "Desafio Prático da Fase 2 - Bot de Clima no Telegram com N8N" challenge from the "Agentes e Automações" module.
