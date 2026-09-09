# PZaaS-LoggerAPI
Documentação da Logger API, para o projeto de Arquitetura de Serviços em Nuvem, aula ministrada por Andrews Egas.


## 1\. Visão geral

A **Logger API** é um serviço responsável por receber, armazenar e disponibilizar logs e métricas gerados pelos microsserviços de uma arquitetura distribuída.

O workflow é implementado em **n8n** e utiliza o **Supabase** como camada de persistência.

### Responsabilidades

* Receber logs estruturados dos serviços.
* Validar a chave de acesso das requisições.
* Validar os campos obrigatórios dos logs e métricas.
* Normalizar os dados recebidos.
* Armazenar logs e métricas no Supabase.
* Permitir consulta de logs por filtros e paginação.
* Permitir consulta de métricas por filtros e paginação.
* Disponibilizar um endpoint de health check.
* Permitir o controle e consulta do status do serviço através do Chaos Monkey.
* Permitir o rastreamento de operações por `orderId`.

---

## 2\. Arquitetura

```text
Microsserviço
     |
     | HTTP
     v
+------------------+
|    Logger API    |
|      (n8n)       |
+------------------+
     |
     +--------------------+
     |                    |
     v                    v
  Validação           Normalização
     |                    |
     +---------+----------+
               |
               v
          +---------+
          | Supabase|
          +---------+
          | logs    |
          | metrics |
          +---------+
