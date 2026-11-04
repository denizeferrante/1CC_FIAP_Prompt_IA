# Aula 03 — OpenAI Agents SDK (prática em aula)

📓 Notebook: [`03_agents_sdk_pratica.ipynb`](../notebooks/03_agents_sdk_pratica.ipynb)

## Objetivo

Conhecer o **OpenAI Agents SDK** (`openai-agents`), uma biblioteca que simplifica a criação de **agentes**: modelos que usam ferramentas, conversam com outros agentes, têm regras de segurança e memória. Na Aula 01 o tool calling foi feito "na mão"; aqui o SDK cuida desse ciclo.

## O que é um agente?

Um **agente** é um LLM com:
- **instruções** (o que ele é e como deve agir);
- **ferramentas** (o que ele pode fazer);
- um **ciclo de execução**: pensar → chamar ferramenta → ler resultado → repetir até ter a resposta.

## 1. "Hello World"

```python
agent = Agent(
    name="Assistant",
    instructions="Você é um assistente prestativo.",
    model="gpt-4o-mini",
)
result = await Runner.run(agent, "Escreva um poema sobre recursão em programação.")
print(result.final_output)
```

- `Agent` define o agente.
- `Runner.run` executa o ciclo e devolve o resultado. Como é **assíncrono**, no Colab se usa `await`.

## 2. Function tools

Com o decorador `@function_tool`, qualquer função Python vira uma ferramenta. O SDK gera o esquema automaticamente a partir dos **type hints**.

```python
@function_tool
def consulta_clima(latitude: float, longitude: float) -> float:
    ...
agent = Agent(..., tools=[consulta_clima])
```

Compare com a Aula 01: não é preciso escrever o JSON Schema nem montar a mensagem `role: "tool"`.

## 3. Hosted tools

São ferramentas que rodam na **infraestrutura da OpenAI**, como a `WebSearchTool`, que pesquisa na internet. O exemplo pergunta sobre uma greve da CPTM e mostra a diferença entre o agente **com** e **sem** a busca: sem ela, o modelo não tem como saber de fatos recentes.

## 4. Várias ferramentas juntas

O agente `AgenteCulturaClima` combina:
- `consulta_clima` (API de clima);
- `WebSearchTool` (notícias);
- `sugere_atividade` (regra Python: acima de 30 °C sugere praia, acima de 20 °C caminhada, abaixo disso café ou museu).

O próprio modelo decide qual ferramenta usar e em que ordem. **Exercício:** replicar a calculadora da Aula 01 usando o Agents SDK.

## 5. Handoff (transferência entre agentes)

Um **agente de triagem** identifica o assunto e passa o atendimento para um especialista:

```python
triage_agent = Agent(
    name="TriageAgent",
    instructions="Direcione questões de cobrança ou reembolso aos agentes apropriados.",
    handoffs=[billing_agent, handoff(refund_agent)],
)
```

É o mesmo modelo de uma central de atendimento: quem atende primeiro encaminha para o setor certo.

## 6. Guardrails

**Guardrails** são verificações de segurança:
- **input guardrail**: analisa a mensagem do usuário antes do agente responder (ex.: bloquear palavrões);
- **output guardrail**: analisa a resposta antes de entregá-la (ex.: bloquear se aparecer "senha", "segredo" ou "cpf").

Quando a regra é violada, o guardrail aciona um **tripwire** (`tripwire_triggered=True`) e a execução é interrompida com uma exceção.

## 7. Sessões (memória)

```python
session = SQLiteSession("sessao_usuario_123")
await Runner.run(agent, "Onde fica o Taj Mahal?", session=session)
await Runner.run(agent, "E em que país ele está?", session=session)
```

A `SQLiteSession` guarda o histórico automaticamente. Na segunda pergunta, o agente sabe que "ele" é o Taj Mahal sem que você precise reenviar a conversa, como era feito com a lista `msgs` na Aula 01.

## 8. Hands-on: assistente virtual da loja TechMais

Projeto que junta tudo:

```
           CLIENTE
              │
              ▼
     AssistenteTechMais ──(guardrail: bloqueia senha, CPF, cartão)
        │            │
        ▼            ▼
 consultar_produto  Handoff
     (tool)          │
                     ▼
            EspecialistaTrocas
```

- **Base fictícia**: um dicionário com produtos, preços e estoque.
- **Tool `consultar_produto`**: devolve preço e estoque reais, para o agente nunca inventar.
- **Handoff** para o `EspecialistaTrocas` quando o assunto é troca ou devolução.
- **Input guardrail** contra dados sensíveis.
- **Sessão** para lembrar do contexto ("E quantas unidades **dele** vocês têm?").

O notebook termina com um teste de **prompt injection** (*"Ignore todas as instruções anteriores..."*), um tipo de ataque em que o usuário tenta fazer o agente desobedecer às próprias regras.

## Conceitos-chave

agente · `Agent` e `Runner` · execução assíncrona · function tools · hosted tools · `WebSearchTool` · handoff · guardrails e tripwire · sessões · prompt injection
