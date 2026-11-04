# Aula 01 — Introdução à API da OpenAI

📓 Notebooks: [`01_introducao_api_openai.ipynb`](../notebooks/01_introducao_api_openai.ipynb) e a variação [`01b_introducao_api_openai_variacao_voz.ipynb`](../notebooks/01b_introducao_api_openai_variacao_voz.ipynb)

## Objetivo

Aprender a conversar com um modelo de linguagem (LLM) por código, em vez de usar a interface do ChatGPT. A aula começa com uma chamada simples e chega a um assistente por voz e a um modelo que chama funções Python (tool calling).

## 1. Primeira chamada

O ponto de partida é o cliente `OpenAI`, criado com a chave guardada nos Secrets do Colab:

```python
client = OpenAI(api_key=userdata.get('OPENAI_API_KEY'))

response = client.responses.create(
    model="gpt-4o-mini",
    input="o que é a vida?",
    temperature=0,
    max_output_tokens=500,
)
print(response.output_text)
```

- **`model`**: qual modelo responde (aqui o `gpt-4o-mini`, barato e rápido).
- **`temperature`**: controla a aleatoriedade. Com `0` a resposta fica mais previsível; valores maiores deixam o texto mais criativo.
- **`max_output_tokens`**: limite de tamanho da resposta.

Um teste curioso da aula é perguntar *"Quantos 'o' tem a palavra capivara?"*. O modelo costuma errar, porque lê o texto em **tokens** (pedaços de palavras), e não letra por letra. Esse problema volta no fim da aula, resolvido com tool calling.

## 2. Roles: system, user e assistant

Na API de Chat Completions, a conversa é uma lista de mensagens, cada uma com um papel:

| Role | Função |
|------|--------|
| `system` | Define o comportamento do modelo: personalidade, regras, formato. |
| `user` | O que o usuário pergunta. |
| `assistant` | Respostas anteriores do modelo (histórico ou exemplos). |

Exemplo da aula: com o system prompt *"responda como um mineiro"*, a mesma pergunta técnica sobre JavaScript ganha outro tom. **Exercício:** alterar o role `system` e observar a mudança.

## 3. Engenharia de prompt: zero-shot e few-shot

- **Zero-shot**: você só descreve a tarefa, sem exemplos. Ex.: *"Classifique o sentimento como positivo, negativo ou neutro"* e depois envia *"este restaurante é bom"*.
- **Few-shot**: você mostra alguns exemplos de pergunta e resposta antes da pergunta real, usando pares `user`/`assistant`. O modelo segue o padrão demonstrado.

**Exercício:** usar few-shot para "inverter" a lógica (positivo → Negativo, neutro → Positivo, negativo → Neutro). Isso mostra que os exemplos podem ensinar uma regra que contraria o senso comum.

## 4. Saídas estruturadas

Em vez de texto livre, o modelo devolve um objeto com campos definidos, descritos com **Pydantic**:

```python
class CalendarEvent(BaseModel):
    name: str
    date: str
    participants: list[str]

completion = client.beta.chat.completions.parse(
    model="gpt-4o-mini",
    messages=[...],
    response_format=CalendarEvent,
)
event = completion.choices[0].message.parsed
```

Isso é essencial quando a resposta vai alimentar outro sistema (banco de dados, agenda, formulário). A aula também passa a data de hoje no prompt para o modelo conseguir interpretar "sexta-feira".

**Exercícios:** extrair nome, CPF e telefone de uma frase; extrair nome, telefone, data e hora de um pedido de agendamento.

## 5. Modelos multimodais (imagem)

O `gpt-4o-mini` também entende imagens. Basta mandar no `content` uma parte de texto e outra do tipo `image_url`. O exemplo usa a imagem de um cardápio.

**Exercícios:**
1. Extrair pratos, preços e ingredientes do cardápio.
2. Descrever uma obra de arte.
3. Combinar imagem e saída estruturada para extrair estilo, palavras-chave e cores predominantes da obra.

## 6. (Re)construindo o ChatGPT e um assistente por voz

Um chatbot é basicamente um **loop** que guarda o histórico na lista `msgs`: a cada rodada entra a mensagem do usuário, o modelo responde e a resposta também é adicionada ao histórico. É assim que o modelo "lembra" da conversa.

O caso prático é o **assistente da academia Vitality Fitness Center**: todas as informações da empresa (modalidades, horários, público) vão no system prompt, e o modelo responde só com base nelas.

A aula então adiciona voz, com três funções:

| Função | O que faz | Modelo |
|--------|-----------|--------|
| `transcribe_speech` | **STT** (speech-to-text): transcreve o áudio do cliente | `gpt-4o-mini-transcribe` |
| `resposta_ia` | gera a resposta em texto | `gpt-4o-mini` |
| `generate_speech` | **TTS** (text-to-speech): transforma a resposta em áudio | `gpt-4o-mini-tts` |

O TTS aceita `instructions` de estilo, como *"responda de forma empolgada como se você fosse um mineiro"*. Os áudios do "cliente" também são gerados por TTS, para simular uma conversa completa.

> **Variação 01b:** o mesmo conteúdo, com outro assistente de voz ("AcademIA", com sotaque sulista) e um loop `while` em vez de `for`.

## 7. Tool calling (chamada de ferramentas)

LLMs não sabem dados em tempo real (clima, cotações, resultados de jogos) e erram em contas e contagens. Com **tool calling**, o modelo não executa nada sozinho: ele **pede** para o seu código chamar uma função e depois usa o resultado.

O fluxo da aula, com a função `get_weather` (API Open-Meteo):

1. Você descreve a ferramenta em JSON Schema (nome, descrição, parâmetros).
2. Envia a pergunta ("tempo em SP") junto com `tools=[...]`.
3. O modelo responde com um `tool_call`, contendo a latitude e a longitude que ele escolheu.
4. Seu código executa `get_weather(...)` de verdade.
5. Você devolve o resultado como uma mensagem `role: "tool"` e chama o modelo de novo.
6. O modelo escreve a resposta final em linguagem natural.

**Exercícios:**
1. Agente que resolve contas chamando uma calculadora Python (`eval`).
2. Agente que conta caracteres (ex.: quantos "o" tem "capivara"), resolvendo o problema de tokenização do início da aula.

## Conceitos-chave

`temperature` · tokens · roles · system prompt · zero-shot · few-shot · saídas estruturadas · Pydantic · multimodalidade · STT/TTS · histórico de conversa · tool calling
