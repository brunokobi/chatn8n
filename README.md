# 🤖 chatBruno — Assistente Virtual Multi-Agente com RAG

<div align="center">

![chatBruno Banner](https://img.shields.io/badge/chatBruno-Multi--Agente%20RAG-6C63FF?style=for-the-badge&logo=robot&logoColor=white)

[![n8n](https://img.shields.io/badge/n8n-Workflow%20Automation-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)](https://n8n.io)
[![n8n Chat UI](https://img.shields.io/badge/n8n%20Chat%20UI-Interface%20Nativa-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-langchain.chattrigger/)
[![LangChain](https://img.shields.io/badge/LangChain-RAG%20Pipeline-1C3C3C?style=for-the-badge&logo=langchain&logoColor=white)](https://langchain.com)
[![Supabase](https://img.shields.io/badge/Supabase-pgvector-3ECF8E?style=for-the-badge&logo=supabase&logoColor=white)](https://supabase.com)
[![Google Gemini](https://img.shields.io/badge/Google%20Gemini-LLM%20%2B%20Embeddings-4285F4?style=for-the-badge&logo=google&logoColor=white)](https://ai.google.dev)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-pgvector-336791?style=for-the-badge&logo=postgresql&logoColor=white)](https://postgresql.org)
[![AWS](https://img.shields.io/badge/AWS-EC2%20Self--Hosted-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)](https://aws.amazon.com)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](https://opensource.org/licenses/MIT)
[![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=flat-square)]()
[![Made by Bruno Kobi](https://img.shields.io/badge/Made%20by-Bruno%20Kobi-blueviolet?style=flat-square)](https://brunokobi.netlify.app)

<br/>

> **Assistente virtual inteligente com arquitetura Multi-Agente + RAG (Retrieval Augmented Generation), construído sobre n8n, LangChain, Supabase pgvector e Google Gemini — hospedado em instância própria na AWS EC2.**

<br/>

[🌐 Portfolio](https://brunokobi.netlify.app) · [🚀 Portfolio 3D](https://brunokobi3d.netlify.app) · [💻 GitHub](https://github.com/brunokobi) · [🔗 LinkedIn](https://linkedin.com/in/brunokobi)

</div>

<br/>

---

## 📋 Índice

- [Interface — n8n Chat UI](#-interface--n8n-chat-ui)
- [Visão Geral](#-visão-geral)
- [Arquitetura](#-arquitetura)
- [Stack Tecnológico](#-stack-tecnológico)
- [Conceitos Aplicados](#-conceitos-aplicados)
- [Estrutura Multi-Agente](#-estrutura-multi-agente)
- [Pipeline RAG](#-pipeline-rag)
- [Base de Conhecimento Vetorial](#-base-de-conhecimento-vetorial)
- [Infraestrutura](#-infraestrutura)
- [Como Executar](#-como-executar)
- [Workflows n8n](#-workflows-n8n)
- [Keep-Alive Automático](#-keep-alive-automático)
- [Autor](#-autor)

---

## 🖥️ Interface — n8n Chat UI

A interface de chat do **chatBruno** é provida nativamente pelo **n8n Chat UI**, o frontend embutido no próprio n8n ativado pelo nó **"When chat message received"** (`@n8n/n8n-nodes-langchain.chatTrigger`). Sem necessidade de construir um frontend customizado — o n8n expõe automaticamente uma janela de chat responsiva e pronta para uso, acessível via URL pública do workflow.

> O botão **"Hide chat"** visível na tela é parte da interface nativa do n8n, que permite alternar entre o editor do fluxo e a janela de conversa em tempo real — ideal para testes e demonstrações diretamente no painel.

### Fluxo completo — visão do editor n8n

![Workflow n8n — fluxo multi-agente com RAG](print.png)

*Visão do editor mostrando o nó de chat trigger, Agente Roteador (Gemini), Switch de intenção, 7 agentes especialistas, Busca Vetorial Supabase, Embeddings Retriever e Memória de Conversa compartilhada.*

### Diagrama de arquitetura dos agentes

![Diagrama de arquitetura — todos os agentes consultam RAG + memória](print2.png)

*Todos os agentes consultam RAG + memória: Chat Trigger → Agente Roteador → Switch (6 rotas + fallback) → Agentes especializados → Supabase Vector RAG (pgvector) + Memória por sessão.*

---

## 🎯 Visão Geral

O **chatBruno** é um assistente virtual de última geração que representa a confluência de múltiplas tecnologias de IA aplicada. Em vez de depender de um único agente com contexto estático no system prompt, o projeto implementa uma **arquitetura multi-agente especializada** onde cada agente é responsável por um domínio específico de conhecimento.

A base de conhecimento é armazenada como **vetores semânticos** no Supabase com extensão `pgvector`, permitindo **busca por similaridade semântica** (não apenas palavras-chave). Quando o usuário faz uma pergunta, o Agente Roteador identifica o domínio da intenção e chaveia o fluxo para um agente especialista. Este agente consome dinamicamente a base vetorial através de chamadas autônomas de ferramenta (**LangChain Tools**) sob o padrão **RAG (Retrieval Augmented Generation)**.

### Por que Multi-Agente + RAG?

| Abordagem | Limitação |
|---|---|
| System prompt estático | Contexto fixo, sem atualização dinâmica, tokens desperdiçados |
| Agente único com RAG | Responde tudo, sem especialização por domínio |
| **Multi-Agente + RAG** ✅ | Cada agente especializado busca de forma autônoma apenas o que é relevante para seu domínio |

---

## 🏗️ Arquitetura

```
                    ┌─────────────────────────────────────────────┐
                    │              AWS EC2 (Self-Hosted)          │
                    │                                             │
Usuário             │   ┌──────────┐    ┌────────────────────┐   │
│                   │   │  Chat    │    │  Agente Roteador   │   │
│ mensagem          │   │ Trigger  │───▶│(Gemini 2.5 Flash L)│   │
▼                   │   │ (Webhook)│    │ classifica intent  │   │
Interface Web ──────┼──▶│  n8n     │    └─────────┬──────────┘   │
                    │   └──────────┘              │              │
                    │                             ▼              │
                    │                    ┌────────────────┐      │
                    │                    │  Switch Router │      │
                    │                    │  7 rotas +     │      │
                    │                    │  fallback      │      │
                    │                    └───────┬────────┘      │
                    │          ┌──────────────────┼────────────┐ │
                    │          ▼                  ▼            ▼ │
                    │   ┌──────────┐       ┌──────────┐  ┌──────────┐
                    │   │  Agente  │       │  Agente  │  │  Agente  │
                    │   │  Perfil  │       │  Skills  │  │Experiênc.│
                    │   └────┬─────┘       └────┬─────┘  └────┬─────┘
                    │        └────────────────── ┼────────────┘ │
                    │                            │ (Aciona Tool)│
                    │                            ▼              │
                    │             ┌──────────────────────┐      │
                    │             │ Tool: match_documents │      │
                    │             │   busca semântica     │      │
                    │             └──────────────────────┘      │
                    └─────────────────────────────────────────── ┘
```

---

## 🛠️ Stack Tecnológico

### Orquestração & Automação

| Tecnologia | Versão | Uso |
|---|---|---|
| **n8n** | Self-hosted (AWS EC2) | Orquestração de todos os workflows e gerenciamento de estados |
| **n8n Chat UI** | Nativa (Chat Trigger) | Interface de chat embutida — frontend zero-config exposto via URL pública |
| **LangChain** (via n8n) | Latest | Pipeline RAG, roteamento estruturado e acoplamento de ferramentas |

### Inteligência Artificial

| Tecnologia | Modelo | Uso |
|---|---|---|
| **Google Gemini** | `gemini-2.5-flash-lite` | LLM do Agente Roteador e de todos os Agentes Especialistas |
| **Google Gemini Embeddings** | `gemini-embedding-001` | Vetorização de documentos da base de conhecimento (768 dim) |

### Banco de Dados Vetorial

| Tecnologia | Extensão | Uso |
|---|---|---|
| **Supabase** | `pgvector` | Armazenamento persistente de embeddings |
| **PostgreSQL** | `ivfflat index` | Busca por similaridade cosseno em consultas rpc |

### Infraestrutura

| Tecnologia | Uso |
|---|---|
| **AWS EC2** | Hospedagem self-hosted estável do ecossistema n8n |
| **Supabase Free Tier** | Banco vetorial persistente amparado por keep-alive automático |

---

## 🧠 Conceitos Aplicados

### RAG via Native Tools

RAG (Retrieval Augmented Generation) é uma técnica que combina busca em base de conhecimento externa com a capacidade generativa do LLM. Diferente de pipelines rígidos que injetam o contexto antes do modelo rodar, este projeto adota o RAG sob a forma de **Tools (Ferramentas) de Agente**:

1. O **Agente Especializado** recebe a pergunta original e decide se precisa consultar dados.
2. Caso precise, aciona a tool `buscar_conhecimento_bruno`.
3. A ferramenta **vetoriza** a query em tempo real e realiza uma busca de similaridade no banco.
4. O contexto mapeado é injetado dinamicamente para gerar uma resposta precisa e fundamentada.

```
Pergunta do Usuário → Decisão do Agente → Chamada de Tool → Busca Semântica → Contexto Retornado → Resposta Final
```

### Embeddings & Similaridade Semântica

Cada chunk de texto é transformado em um vetor de **768 dimensões** através do modelo `gemini-embedding-001`. O cálculo de proximidade conceitual é executado via **distância cosseno** no PostgreSQL:

```sql
-- Busca os 5 chunks mais relevantes para a pergunta
SELECT content, 1 - (embedding <=> query_embedding) AS similarity
FROM documents
ORDER BY embedding <=> query_embedding
LIMIT 5;
```

---

## 👥 Estrutura Multi-Agente

O sistema é composto por **8 agentes** orquestrados em paralelo:

```
┌─────────────────────────────────────────────────────────────┐
│                       AGENTE ROTEADOR                       │
│  Analisa a mensagem e retorna: PERFIL | SKILLS |            │
│  EXPERIENCIA | EDUCACAO | PROJETOS | CONTATO | GERAL        │
└─────────────────────────────────────────────────────────────┘
         │
         ▼
┌────────────────────────────────────────────────────────────────────┐
│                         AGENTES ESPECIALIZADOS                     │
│                                                                    │
│  🧑 Agente Perfil      → história, filosofia, transição de carreira│
│  ⚙️  Agente Skills      → linguagens, frameworks, ferramentas       │
│  💼 Agente Experiência  → empresas, cargos, períodos, tecnologias   │
│  🎓 Agente Educação     → formação, mestrado, pesquisas             │
│  🚀 Agente Projetos     → portfólio, detalhes técnicos, impacto     │
│  📬 Agente Contato      → links, email, redes sociais               │
│  🔄 Agente Geral        → fallback (saída extra do Roteador)        │
└────────────────────────────────────────────────────────────────────┘
         │
         ▼ (Todos os agentes acessam de forma autônoma)
┌─────────────────────────────────────────────────────────────┐
│                      RECURSOS COMPARTILHADOS                │
│  📦 Supabase Vector Store (Tool: buscar_conhecimento_bruno) │
│  🧠 Window Buffer Memory (Histórico limitado a 10 sessões)  │
│  🔢 Gemini Agentes (Pool de LLM gemini-2.5-flash-lite)      │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔍 Pipeline RAG

### Fase 1 — Ingestão (execução única)

```
Chunks de texto → Gemini Embeddings (gemini-embedding-001) → Supabase pgvector (ivfflat index)
```

### Fase 2 — Consulta (em tempo real)

```
Pergunta do Usuário → Roteador de Intenção → Agente Especialista → Acionamento de Tool RAG → Resposta Contextualizada
```

---

## 📚 Base de Conhecimento Vetorial

A base é composta por **22 chunks** organizados por categoria semântica:

| Categoria | Chunks | Conteúdo |
|---|---|---|
| `identidade` | 1 | Nome, cargo, localização |
| `resumo` | 1 | História e filosofia de vida |
| `sobre` | 1 | Paixões e objetivos pessoais |
| `skills` | 1 | Stack tecnológico completo |
| `educacao` | 5 | UFES, IFES (2x), Unisales, Técnico |
| `experiencia` | 5 | Vennx, Ambipar (2x), Directy, Ecosoft, Infovix |
| `projeto` | 6 | Presence Now, Guia Alimentar BR, Quiz, Placas, Portfolio, Portfolio 3D |
| `contato` | 1 | Links, email, telefone |
| `certificacoes` | 1 | Microsoft, Deep Learning, Scrum, NLW |

### Schema da Tabela

```sql
CREATE TABLE documents (
  id        BIGSERIAL PRIMARY KEY,
  content   TEXT NOT NULL,              -- Texto do chunk
  metadata  JSONB DEFAULT '{}',         -- categoria, fonte, empresa, etc.
  embedding VECTOR(768)                 -- Vetor gerado pelo Gemini
);

-- Índice para busca eficiente por similaridade cosseno
CREATE INDEX documents_embedding_idx
  ON documents USING ivfflat (embedding vector_cosine_ops)
  WITH (lists = 100);
```

### Função de Busca Semântica

```sql
CREATE OR REPLACE FUNCTION match_documents(
  query_embedding VECTOR(768),
  match_count     INT DEFAULT 5,
  filter          JSONB DEFAULT '{}'
) RETURNS TABLE (
  id         BIGINT,
  content    TEXT,
  metadata   JSONB,
  similarity FLOAT
) AS $$
BEGIN
  RETURN QUERY
  SELECT
    documents.id,
    documents.content,
    documents.metadata,
    1 - (documents.embedding <=> query_embedding) AS similarity
  FROM documents
  WHERE metadata @> filter
  ORDER BY documents.embedding <=> query_embedding
  LIMIT match_count;
END;
$$ LANGUAGE plpgsql;
```

---

## ☁️ Infraestrutura

### AWS EC2 (n8n Self-Hosted)

O n8n roda em uma instância EC2 própria, eliminando dependência de planos pagos de plataformas SaaS e garantindo:

- Controle total sobre os dados e payloads trafegados
- Sem limites artificiais de execução por plano de assinatura
- Customização completa de arquivos de configuração do servidor

### Supabase (Free Tier Otimizado)

Para contornar o congelamento automático do Supabase após 7 dias de inatividade, o ecossistema inclui um robô interno que gera requisições de atividade periódicas a cada 3 dias.

```
Schedule Trigger (a cada 3 dias) ──▶ HTTP GET ──▶ Projetos Supabase Ativos
```

---

## 🚀 Como Executar

### Pré-requisitos

- n8n self-hosted configurado e ativo (AWS EC2 ou Local)
- Instância Supabase ativa com extensão `pgvector` habilitada
- Google AI API Key vinculada aos serviços do Gemini

### 1. Configurar o Banco Vetorial

Execute o arquivo `setup_supabase_bruno.sql` no SQL Editor do Supabase. O script irá estruturar a tabela `documents`, configurar o índice de performance `ivfflat`, embarcar a função RPC `match_documents` e injetar os 22 registros originais.

### 2. Gerar os Embeddings (Ingestão)

Importe e execute o arquivo `ingestao_supabase_bruno.json` no seu painel n8n:

1. Vincule suas chaves de acesso na credencial **Supabase API** usando a **Service Role Key**.
2. Execute o gatilho manual uma vez.
3. Os logs confirmarão a geração dos vetores pelo modelo de embeddings.

### 3. Ativar o Chat Multi-Agente

Importe o arquivo consolidado `chatBruno_multiagente_rag.json` para o n8n:

1. Configure as credenciais de banco no nó `Busca Vetorial Supabase`.
2. Certifique-se de que a API Key do Gemini está atribuída corretamente aos nós de LLM.
3. Alterne a chave do workflow para **Active**.
4. O **n8n Chat UI** será disponibilizado automaticamente via URL pública do workflow — nenhum frontend adicional é necessário. Acesse a URL do chat trigger para interagir em tempo real.

### 4. Ativar o Keep-Alive

Importe o fluxo complementar `supabase_ping.json` e ative seu agendador para garantir estabilidade contínua do banco vetorial.

---

## 📁 Workflows n8n

| Arquivo | Descrição | Trigger |
|---|---|---|
| `chatBruno_multiagente_rag.json` | Core principal do chat com roteamento e agentes conectados à tool RAG | Webhook (Chat público) |
| `ingestao_supabase_bruno.json` | Consome dados textuais brutos e gera embeddings estruturados | Manual (Execução única) |
| `supabase_ping.json` | Script preventivo contra inatividade do banco gratuito | Schedule (A cada 3 dias) |
| `setup_supabase_bruno.sql` | Estrutura de banco de dados PostgreSQL e sementes relacionais | SQL Editor Supabase |

---

## ⏰ Keep-Alive Automático

O workflow de persistência envia uma requisição leve às rotas REST da API utilizando a chave anônima para sinalizar tráfego legítimo:

```json
{
  "schedule": "a cada 3 dias",
  "projetos": [
    "lxkobnubawhqndboxgwj (presensenow)",
    "xztrtmjhhrdxubcrgwaa (chatBruno RAG)"
  ],
  "metodo": "GET /rest/v1/ com anon key"
}
```

---

## 👨‍💻 Autor

**Bruno Kobi Valadares de Amorim**

*Desenvolvedor Full Stack | IA & Automação | Mestrando em Computação Aplicada*

[![Portfolio](https://img.shields.io/badge/Portfolio-brunokobi.netlify.app-6C63FF?style=flat-square)](https://brunokobi.netlify.app)
[![Portfolio 3D](https://img.shields.io/badge/Portfolio%203D-brunokobi3d.netlify.app-6C63FF?style=flat-square)](https://brunokobi3d.netlify.app)
[![GitHub](https://img.shields.io/badge/GitHub-brunokobi-181717?style=flat-square&logo=github)](https://github.com/brunokobi)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-brunokobi-0077B5?style=flat-square&logo=linkedin)](https://linkedin.com/in/brunokobi)

---

*Serra, Espírito Santo, Brasil · **Vamos construir o futuro juntos! 🚀***
