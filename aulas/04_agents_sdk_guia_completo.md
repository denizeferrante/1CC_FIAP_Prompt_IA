# Aula 04 — OpenAI Agents SDK (guia completo)

📓 Notebook: [`04_agents_sdk_guia_completo.ipynb`](../notebooks/04_agents_sdk_guia_completo.ipynb)

## Objetivo

Aprofundar o Agents SDK visto na Aula 03, com exemplos mais robustos e boas práticas. Os destaques são as três **hosted tools** (busca na web, busca em documentos e execução de código) e os dois padrões de delegação entre agentes.

## 0. Configuração segura

A chave é lida dos Secrets do Colab e validada **sem ser exibida**:

```python
api_key = userdata.get("OPENAI_API_KEY")
if not api_key:
    raise ValueError("Adicione OPENAI_API_KEY aos Secrets do Google Colab.")
```

## 1. `Agent` e `Runner`

Um `Agent` reúne nome, instruções, modelo e capacidades; o `Runner` executa o ciclo até chegar a uma resposta final. Em notebooks se usa `await Runner.run(...)`; em scripts Python, `asyncio.run(main())`.

## 2. Function tools com validação

A ferramenta `consultar_clima(cidade)` é uma versão mais completa que a da Aula 03:
- recebe o **nome da cidade** e descobre as coordenadas pela API de geocoding;
- usa `timeout` e `raise_for_status()` para tratar falhas de rede;
- devolve um **dicionário estruturado** (temperatura, sensação térmica, vento, horário).

A **docstring** e os **type hints** são importantes: é com eles que o modelo entende quando e como usar a ferramenta.

## 3. Hosted tools

Rodam na infraestrutura da OpenAI e você não precisa implementá-las.

### 3.1 `WebSearchTool`
Pesquisa informações atuais. As instruções pedem para **citar fontes**, diferenciar a data de publicação da data do acontecimento e avisar quando houver informações conflitantes.

### 3.2 `FileSearchTool`: RAG pronto
É o RAG da Aula 02, só que gerenciado pela OpenAI:

1. Criar um **vector store** e enviar o documento com `upload_and_poll`, que espera a indexação terminar.
2. Passar o `vector_store.id` para o agente.

```python
FileSearchTool(vector_store_ids=[vector_store.id], max_num_results=5, include_search_results=True)
```

O documento de exemplo é a política de trocas da TechMais. A aula testa três situações:
- uma pergunta direta ("Qual o prazo para trocas?");
- uma comparação entre duas regras, em tabela;
- uma pergunta **fora da base** (telefone da loja). O comportamento certo é dizer que não encontrou, e não inventar.

**Exercício:** recriar o RAG da Aula 02 usando o Agents SDK.

### 3.3 `CodeInterpreterTool`
Permite ao modelo **escrever e executar Python** em um contêiner isolado. É útil para contas, estatística e análise de dados, porque o modelo deixa de "chutar" números.

- **Exemplo 1**: cálculo de *lift* percentual, mostrando fórmula, substituição e resultado.
- **Exemplo 2**: estatísticas de vendas semanais (média, mediana, desvio padrão, coeficiente de variação e valores atípicos).
- **Exemplo 3**: análise de um **CSV** enviado para um contêiner, com pandas.

> Executar código reduz erros de conta, mas não garante que o **método** escolhido esteja certo. O resultado ainda precisa ser conferido.

### 3.4 Combinando as duas
Um agente pode buscar uma regra no documento (prazo de 5 dias úteis) e fazer contas com ela (37 solicitações ÷ 5 dias). A instrução pede para informar a **origem** de cada conclusão: o que veio do documento e o que foi calculado.

## 4. Várias ferramentas

O agente "Guia urbano" combina clima (API), sugestão de atividade (regra Python) e busca web (só quando o usuário pede notícias ou eventos). Instruções claras ajudam o modelo a escolher a ferramenta certa.

## 5. Delegação entre agentes

| Padrão | Como funciona | Quem dá a resposta final |
|--------|---------------|--------------------------|
| **Handoff** | O especialista **assume** a conversa | O especialista |
| **Agents as tools** | O coordenador **consulta** especialistas como se fossem ferramentas | O coordenador |

- **Handoff**: a triagem encaminha para cobrança ou reembolso. O campo `handoff_description` explica quando escolher cada um, e `resultado.last_agent` mostra quem terminou o atendimento.
- **Agents as tools**: o coordenador consulta um especialista técnico e um de negócio com `.as_tool(...)` e escreve uma decisão final que junta as duas visões.

## 6. Guardrails

- **Input**: bloqueia mensagens com senha, CPF ou cartão de crédito. Com `run_in_parallel=False`, a verificação termina **antes** de o agente começar, economizando uma chamada ao modelo.
- **Output**: bloqueia respostas que contenham `SEGREDO_INTERNO`.
- O bloqueio é tratado capturando a exceção específica (`InputGuardrailTripwireTriggered`), o que separa um bloqueio de segurança de um erro de rede ou de código.

Guardrails **complementam**, mas não substituem, autenticação, autorização e supervisão humana.

## 7. Memória com sessões

O exemplo usa **duas sessões diferentes** de propósito: a segunda pergunta ("Em que país ele fica?") vai para outra sessão, então o agente **não sabe** do que se trata. Isso mostra que a memória fica isolada por sessão, ou seja, por usuário.

## Boas práticas (resumo do notebook)

- Instruções e descrições de ferramentas específicas.
- Type hints, docstrings, validação e tratamento de erros.
- Nada de chaves, senhas ou dados pessoais no código ou nos prompts.
- Ferramentas com o **menor privilégio** necessário.
- Testar perguntas **sem resposta** na base (File Search).
- Conferir método, unidades e resultados (Code Interpreter).
- Guardrails e supervisão humana em ações sensíveis.
- Inspecionar *traces* e criar avaliações antes de ir para produção.

## Conceitos-chave

hosted tools · `WebSearchTool` · `FileSearchTool` · vector store · `CodeInterpreterTool` · contêiner · handoff × agents as tools · `handoff_description` · guardrails · sessões isoladas · menor privilégio
