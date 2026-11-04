# Prompt & Artificial Intelligence

Notebooks da disciplina **Prompt & Artificial Intelligence** (1º ano — FIAP).
O material vai da primeira chamada à API da OpenAI até um sistema multiagente integrado ao Telegram, passando por engenharia de prompt, saídas estruturadas, modelos multimodais, voz, RAG e o OpenAI Agents SDK.

## Conteúdo

> 📚 A explicação detalhada de cada aula está na pasta [`aulas/`](aulas/README.md).

| # | Notebook | Tema | Principais tópicos |
|---|----------|------|--------------------|
| 01 | [`01_introducao_api_openai.ipynb`](notebooks/01_introducao_api_openai.ipynb) | Introdução à API da OpenAI | Roles (system/user), zero-shot e few-shot, saídas estruturadas (Pydantic), visão computacional (cardápios e obras de arte), texto para voz e transcrição de áudio, chatbot customizado para empresa, tool calling |
| 01b | [`01b_introducao_api_openai_variacao_voz.ipynb`](notebooks/01b_introducao_api_openai_variacao_voz.ipynb) | Variação do notebook 01 | Mesmo conteúdo com outra versão do assistente por voz (sotaque regional e chatbot "AcademIA") |
| 02 | [`02_rag_langchain_embeddings.ipynb`](notebooks/02_rag_langchain_embeddings.ipynb) | RAG e Word Embeddings | Data loaders, text splitters, embeddings, vector store, LangChain + LangGraph, LangSmith; exercícios com RAG sobre o conteúdo das aulas e sobre o Manual do Candidato FIAP |
| 03 | [`03_agents_sdk_pratica.ipynb`](notebooks/03_agents_sdk_pratica.ipynb) | Agents SDK — prática em aula | "Hello World", function tools, hosted tools, handoffs, guardrails, sessões; hands-on: assistente virtual para loja |
| 04 | [`04_agents_sdk_guia_completo.ipynb`](notebooks/04_agents_sdk_guia_completo.ipynb) | Agents SDK — guia completo | `Agent` e `Runner`, function tools, `WebSearchTool`, `FileSearchTool`, `CodeInterpreterTool`, handoff, agents as tools, guardrails, memória com sessões, boas práticas |
| 05 | [`05_projeto_restaurante_agentico_telegram.ipynb`](notebooks/05_projeto_restaurante_agentico_telegram.ipynb) | Projeto: Restaurante Agêntico | Modelagem de dados, ferramentas e guardrails, agentes especialistas, agente de triagem com memória de sessão, bot no Telegram |

## Tecnologias

- Python 3 e Google Colab
- [OpenAI API](https://platform.openai.com/docs) (Chat Completions, Responses, áudio e visão)
- [OpenAI Agents SDK](https://openai.github.io/openai-agents-python/) (`openai-agents`)
- LangChain, LangGraph e LangSmith
- Pydantic, pandas, BeautifulSoup
- python-telegram-bot

## Como executar

Os notebooks foram feitos para o **Google Colab**.

1. Abra o notebook no Colab (**Arquivo → Abrir notebook → GitHub** e cole a URL deste repositório).
2. No menu lateral, abra **Secrets** (ícone de chave 🔑) e cadastre as chaves necessárias:

   | Secret | Usado em |
   |--------|----------|
   | `OPENAI_API_KEY` | Todos os notebooks |
   | `LANGSMITH_API_KEY` | 02 (opcional, para rastreamento) |
   | `TELEGRAM_BOT_TOKEN` | 05 (crie o bot pelo [@BotFather](https://t.me/BotFather)) |

3. Execute as células em ordem. A primeira célula de cada notebook instala as dependências com `pip`.

Para rodar localmente (fora do Colab), troque `userdata.get("NOME")` por `os.environ["NOME"]` e defina as variáveis de ambiente (ou um arquivo `.env`, que já está no `.gitignore`).

> ⚠️ Nunca escreva chaves de API diretamente no código. Use os Secrets do Colab ou variáveis de ambiente.

## Estrutura

```
.
├── aulas/                  # explicação de cada aula
│   ├── README.md
│   ├── 01_introducao_api_openai.md
│   ├── 02_rag_langchain_embeddings.md
│   ├── 03_agents_sdk_pratica.md
│   ├── 04_agents_sdk_guia_completo.md
│   └── 05_projeto_restaurante_agentico_telegram.md
├── notebooks/              # notebooks do Colab
│   ├── 01_introducao_api_openai.ipynb
│   ├── 01b_introducao_api_openai_variacao_voz.ipynb
│   ├── 02_rag_langchain_embeddings.ipynb
│   ├── 03_agents_sdk_pratica.ipynb
│   ├── 04_agents_sdk_guia_completo.ipynb
│   └── 05_projeto_restaurante_agentico_telegram.ipynb
├── .gitignore
└── README.md
```

## Autora

Denize Ferrante — estudante de TI na FIAP.
