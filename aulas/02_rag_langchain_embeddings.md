# Aula 02 — RAG, Generative AI e Word Embeddings

📓 Notebook: [`02_rag_langchain_embeddings.ipynb`](../notebooks/02_rag_langchain_embeddings.ipynb)

## Objetivo

Fazer um LLM responder com base em **documentos próprios** (PDFs, páginas web), e não só no que ele aprendeu no treinamento. A técnica é o **RAG**, sigla de *Retrieval-Augmented Generation* (geração aumentada por recuperação).

## Por que RAG?

Um LLM sozinho:
- não conhece documentos internos ou recentes;
- pode **alucinar**, ou seja, inventar respostas com aparência confiável.

O RAG resolve isso em duas etapas: primeiro **recupera** os trechos relevantes dos seus documentos e depois **gera** a resposta usando esses trechos como contexto.

```
Pergunta ──► Busca no vector store ──► Trechos relevantes ──► LLM ──► Resposta
```

## O que são embeddings?

Um **embedding** é uma representação de um texto como um vetor de números. Textos com significado parecido ficam com vetores próximos, mesmo que usem palavras diferentes ("carro" e "automóvel", por exemplo). É isso que permite uma **busca semântica**, por significado, e não só por palavra exata.

Na aula, o modelo de embeddings é o `text-embedding-ada-002`, da OpenAI.

## Passo a passo do pipeline

### 1. Modelo e embeddings

```python
llm = init_chat_model("gpt-4o-mini", model_provider="openai")
embeddings = OpenAIEmbeddings(model="text-embedding-ada-002")
vector_store = InMemoryVectorStore(embeddings)
```

O **vector store** é o "banco de dados" que guarda os vetores e permite buscar os mais parecidos com a pergunta. Aqui ele fica em memória, mas em produção usam-se bancos como Chroma, FAISS ou Pinecone.

### 2. Carregar documentos (data loaders)

A aula usa duas fontes:
- `PyPDFLoader`: lê um PDF (`ia_intro.pdf`);
- `WebBaseLoader`: baixa e lê uma página web (artigo da IBM sobre IA generativa).

### 3. Dividir em pedaços (chunking)

```python
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,     # tamanho de cada pedaço, em caracteres
    chunk_overlap=200,   # sobreposição entre pedaços
    add_start_index=True,
)
```

Documentos grandes são quebrados em pedaços menores porque o modelo tem limite de contexto e porque trechos pequenos deixam a busca mais precisa. A **sobreposição** evita cortar uma ideia no meio.

### 4. Indexar

`vector_store.add_documents(all_splits)` calcula o embedding de cada pedaço e guarda no vector store.

### 5. Prompt de RAG

O prompt vem pronto do **LangSmith Hub** (`rlm/rag-prompt`) e é adaptado para responder *"NAO SEI"* quando o contexto não tiver a resposta. Essa instrução é a principal defesa contra alucinação.

### 6. Orquestrar com LangGraph

O fluxo vira um grafo de dois nós, que compartilham um estado:

```python
class State(TypedDict):
    question: str
    context: List[Document]
    answer: str
```

- **`retrieve`**: faz `similarity_search` com a pergunta e guarda os documentos encontrados em `context`;
- **`generate`**: monta o prompt com pergunta e contexto e chama o LLM.

```python
graph = StateGraph(State).add_sequence([retrieve, generate])
```

### 7. Testar

A aula pergunta *"o que é machine learning?"* e uma questão de múltipla escolha sobre riscos do RAG. A resposta certa, aliás, é justamente **recuperar contexto irrelevante ou ambíguo**: se a busca traz o trecho errado, a resposta final piora.

## Exercícios

1. Criar um RAG capaz de resolver uma **prova de IA** sobre todos os assuntos vistos nas aulas, usando pelo menos dois tipos de data loader, e retornar a resposta junto com os documentos recuperados.
2. Criar um RAG para candidatos ao **vestibular da FIAP**, usando o Manual do Candidato como base, e testar com *"Qual o valor da taxa de inscrição?"*.

## Conceitos-chave

RAG · alucinação · embeddings · busca semântica · vector store · data loaders · chunking (`chunk_size` e `chunk_overlap`) · LangChain · LangGraph · LangSmith
