# PZaaS - Logger API

API responsável pelo recebimento e armazenamento de **logs** e **métricas** da plataforma PZaaS.

O serviço utiliza **n8n** para orquestração dos fluxos e **Supabase** para armazenamento dos dados.

---

## 🔐 Autenticação

Todas as requisições devem enviar a API Key no header:

```http
x-api-key: turma2026
```

O `Content-Type` deve ser informado como:

```http
Content-Type: application/json
```

### Respostas de autenticação

| Código | Situação |
|---|---|
| `401` | API Key não informada |
| `403` | API Key inválida |

---

# 📝 Logs

## POST `/v1/log`

Registra um novo log no sistema.

### Headers

```http
x-api-key: turma2026
Content-Type: application/json
```

Opcionalmente, pode ser enviado o identificador do pedido:

```http
x-pedido-id: <id-do-pedido>
```

O header `x-pedido-id` é **opcional**. Quando informado, seu valor é utilizado para preencher o campo `orderId` do log. Quando não informado, `orderId` será `null`.

### Body

```json
{
  "service": "payment",
  "action": "process_payment",
  "status": "success",
  "level": "INFO",
  "message": "Pagamento processado com sucesso",
  "timestamp": "2026-09-08T18:00:00.000Z",
  "metadata": {
    "method": "pix"
  }
}
```

### Campos

| Campo | Obrigatório | Tipo | Descrição |
|---|---|---|---|
| `service` | Sim | string | Serviço responsável pelo evento |
| `action` | Sim | string | Ação realizada |
| `status` | Sim | string | Status da operação |
| `level` | Sim | string | Nível do log |
| `message` | Sim | string | Mensagem descritiva |
| `timestamp` | Não | string | Data/hora do evento |
| `metadata` | Não | object | Informações adicionais |

O campo `orderId` **não é enviado no body**. Caso seja necessário associar o log a um pedido, utilize o header `x-pedido-id`.

### Resposta

```json
{
  "id": "uuid-do-log",
  "message": "Log registrado com sucesso"
}
```

---

# 🔎 Consulta de Logs

## GET `/v1/logs`

Consulta os logs registrados no sistema.

### Headers

```http
x-api-key: turma2026
```

### Parâmetros opcionais

| Parâmetro | Descrição |
|---|---|
| `id` | Identificador do evento/log |
| `orderId` | Identificador do pedido |
| `service` | Serviço responsável pelo log |
| `level` | Nível do log |
| `status` | Status da operação |
| `page` | Número da página |
| `limit` | Quantidade de registros por página |

### Exemplo

```http
GET /v1/logs?service=payment&level=INFO&page=1&limit=20
```

---

# ❤️ Health Check

## GET `/v1/health-log`

Verifica a disponibilidade do serviço Logger.

### Headers

```http
x-api-key: turma2026
```

Esse endpoint permite verificar se o Logger está disponível para receber requisições.

---

# 📊 Métricas

O Logger também possui endpoints específicos para **registro e consulta de métricas**.

## POST `/v1/metric`

Registra uma nova métrica.

### Headers

```http
x-api-key: turma2026
Content-Type: application/json
```

### Body

```json
{
  "metricName": "payment_processing_time",
  "value": 245.5,
  "unit": "ms",
  "service": "payment",
  "orderId": "12345",
  "timestamp": "2026-09-08T18:00:00.000Z",
  "metadata": {
    "method": "pix"
  }
}
```

### Campos

| Campo | Obrigatório | Tipo | Descrição |
|---|---|---|---|
| `metricName` | Sim | string | Nome da métrica |
| `value` | Sim | number | Valor da métrica |
| `service` | Sim | string | Serviço responsável |
| `unit` | Não | string | Unidade de medida |
| `orderId` | Não | string | Identificador do pedido |
| `timestamp` | Não | string | Data/hora da métrica |
| `metadata` | Não | object | Informações adicionais |

### Exemplo

```json
{
  "metricName": "orders_processed",
  "value": 150,
  "service": "order",
  "unit": "count"
}
```

---

# 🔎 Consulta de Métricas

## GET `/v1/metrics`

Consulta as métricas armazenadas.

### Headers

```http
x-api-key: turma2026
```

### Parâmetros opcionais

| Parâmetro | Descrição |
|---|---|
| `metricId` | Identificador da métrica |
| `metricName` | Nome da métrica |
| `service` | Serviço responsável |
| `orderId` | Identificador do pedido |
| `page` | Número da página |
| `limit` | Quantidade de registros por página |

### Exemplo

```http
GET /v1/metrics?service=payment&metricName=payment_processing_time&page=1&limit=20
```

---

# 🐒 Chaos Monkey

O Logger possui endpoints para simular indisponibilidade do serviço.

Essa funcionalidade permite testar o comportamento dos demais serviços quando o Logger estiver indisponível.

## GET `/v1/chaos-status`

Consulta o estado atual do Chaos Monkey do Logger.

### Headers

```http
x-api-key: turma2026
```

O endpoint informa se o serviço Logger está habilitado ou desabilitado.

## POST `/v1/alter-chaos`

Altera o estado de disponibilidade do Logger.

### Headers

```http
x-api-key: turma2026
Content-Type: application/json
```

### Body

Para desabilitar:

```json
{
  "enabled": false
}
```

Para habilitar novamente:

```json
{
  "enabled": true
}
```

### Funcionamento

Quando `enabled` é `false`, o Logger fica indisponível para as operações normais, permitindo simular uma falha do serviço.

Quando `enabled` é `true`, o Logger volta a aceitar as operações normalmente.

Os endpoints de Chaos permanecem disponíveis mesmo quando o Logger está desabilitado, permitindo consultar e alterar seu estado.

---

# ⚠️ Indisponibilidade do Serviço

Quando o Logger estiver desabilitado pelo Chaos Monkey, os endpoints normais do serviço retornam:

```http
503 Service Unavailable
```

Os endpoints de Chaos continuam disponíveis para permitir a reativação do serviço.

---

# 🗄️ Armazenamento

Os dados são armazenados no **Supabase**.

O serviço utiliza tabelas para armazenamento dos logs, métricas e controle do estado do serviço.

### Controle de disponibilidade

A tabela `service_status` possui:

| Campo | Tipo | Descrição |
|---|---|---|
| `service` | text | Nome do serviço |
| `enabled` | boolean | Indica se o serviço está habilitado |
| `updated_at` | timestamptz | Data/hora da última alteração |

O Logger utiliza o registro:

```text
service = logger
```

para controlar sua disponibilidade.

---

# 🔄 Arquitetura

O fluxo geral do Logger funciona da seguinte forma:

```text
Cliente / Outro Serviço
        │
        ▼
      n8n
        │
        ├── Autenticação (x-api-key)
        │
        ├── Verificação do status do serviço
        │
        ├── Validação dos dados
        │
        └── Processamento
              │
              ▼
          Supabase
              │
              ▼
          Resposta
```

Para o Chaos Monkey:

```text
Cliente
   │
   ▼
POST /v1/alter-chaos
   │
   ▼
Atualização do service_status
   │
   ├── enabled = true
   │
   └── enabled = false
```

---

# 📌 Endpoints

| Método | Endpoint | Função |
|---|---|---|
| `POST` | `/v1/log` | Registra um log |
| `GET` | `/v1/logs` | Consulta logs |
| `GET` | `/v1/health-log` | Verifica a saúde do Logger |
| `POST` | `/v1/metric` | Registra uma métrica |
| `GET` | `/v1/metrics` | Consulta métricas |
| `GET` | `/v1/chaos-status` | Consulta o estado do Chaos |
| `POST` | `/v1/alter-chaos` | Habilita/desabilita o Logger |

---

# 🚨 Códigos HTTP

| Código | Significado |
|---|---|
| `200` | Requisição processada com sucesso |
| `201` | Recurso criado |
| `400` | Dados da requisição inválidos |
| `401` | API Key não informada |
| `403` | API Key inválida |
| `503` | Serviço Logger indisponível |

---

# 🛠️ Tecnologias

- **n8n** — Orquestração dos workflows
- **Supabase** — Banco de dados e armazenamento
- **PostgreSQL** — Banco utilizado pelo Supabase
- **REST API** — Comunicação entre os serviços
- **JSON** — Formato das requisições e respostas

---

# 📁 Responsabilidade do Serviço

O Logger é responsável por:

- Receber logs dos demais serviços;
- Armazenar logs;
- Consultar logs;
- Receber métricas;
- Armazenar métricas;
- Consultar métricas;
- Associar logs e métricas aos pedidos quando aplicável;
- Disponibilizar um endpoint de health check;
- Permitir a simulação de indisponibilidade através do Chaos Monkey;
- Disponibilizar uma API padronizada para integração com os demais serviços da plataforma PZaaS.
