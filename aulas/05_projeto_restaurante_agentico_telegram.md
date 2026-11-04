# Aula 05 — Projeto: Restaurante Agêntico no Telegram

📓 Notebook: [`05_projeto_restaurante_agentico_telegram.ipynb`](../notebooks/05_projeto_restaurante_agentico_telegram.ipynb)

## Objetivo

Juntar tudo o que foi visto em um **sistema multiagente real**: o atendente virtual do *Restaurante Sabor da Casa*, que funciona como um bot no **Telegram**. O cliente consulta o cardápio, faz e altera pedidos e tira dúvidas, e cada tipo de pedido é atendido por um agente especialista.

## Arquitetura

```
                  Cliente (Telegram)
                         │
                         ▼
     ┌──── Atendente do Restaurante (triagem) ────┐
     │   guardrail de entrada + guardrail de saída │
     └───────┬──────────────┬──────────────┬──────┘
             │ handoff      │ handoff      │ handoff
             ▼              ▼              ▼
      FAQ Restaurante  Especialista   Especialista
      (FileSearchTool) em Cardápio    em Pedidos
                       (menu_tools)   (menu + order_tools)

   Analista de Dados (CodeInterpreterTool) → só uso administrativo
```

## 1. Modelagem e dados

- **`cardapio.csv`**: itens com id, nome, categoria, preço e disponibilidade.
- **`pedidos.csv`**: pedidos com id, usuário, item, quantidade, preço e status.
- **`faq_restaurante.txt`**: horário, endereço, formas de pagamento, regiões de entrega e políticas.
- **`CustomerContext`** (Pydantic): guarda o `user_id` de quem está falando. Esse **contexto** é passado para as ferramentas e não fica visível para o modelo, o que garante que cada cliente só veja os próprios pedidos.

## 2. Ferramentas (function tools)

| Grupo | Ferramentas |
|-------|-------------|
| Cardápio (`menu_tools`) | `consultar_cardapio`, `buscar_item_cardapio`, `consultar_preco` |
| Pedidos (`order_tools`) | `fazer_pedido`, `consultar_meus_pedidos`, `alterar_pedido`, `calcular_total_pedido`, `consultar_status_pedido`, `solicitar_atendimento_humano` |

As ferramentas de pedido recebem `ctx: RunContextWrapper[CustomerContext]` e filtram tudo pelo `user_id`. Elas também têm **regras de negócio no código**, e não só no prompt:
- a quantidade precisa ser positiva;
- o item precisa existir e estar disponível;
- só pedidos com status `ABERTO` podem ser alterados.

Regras críticas ficam no código porque o modelo pode errar, mas o código sempre aplica a regra.

## 3. Guardrails com agentes

Na Aula 04 os guardrails eram listas de palavras. Aqui **cada guardrail é um agente** que classifica a mensagem e devolve uma saída estruturada (`is_unsafe`, `reason`):

- **Entrada**: marca como insegura uma mensagem que pede cartão ou senha, tenta fazer o sistema ignorar as regras (*prompt injection*) ou foge do escopo do restaurante. Perguntas sobre alergia são permitidas, com recomendação de confirmar com um humano.
- **Saída**: bloqueia respostas que pedem credenciais, que dizem ter criado um pedido **sem informar o `pedido_id`** ou que garantem risco zero para alergias.

Assim o sistema entende a **intenção** da mensagem, não só palavras-chave.

## 4. Agentes especialistas

- **FAQ Restaurante**: responde com `FileSearchTool` sobre a FAQ indexada em um vector store. Se não encontrar a resposta, diz isso e oferece atendimento humano.
- **Especialista em Cardápio**: é obrigado a usar as ferramentas antes de informar preço ou item.
- **Especialista em Pedidos**: cria, consulta e altera pedidos, e nunca diz que fez algo sem chamar a ferramenta correspondente.
- **Analista de Dados**: usa `CodeInterpreterTool` com os CSVs em um contêiner, para análises administrativas (ex.: reajustar em 5% o preço das bebidas e comparar antes e depois). **Não** fica disponível para os clientes.

## 5. Triagem e memória

O **Atendente do Restaurante** é a porta de entrada: aplica os guardrails e faz **handoff** para o especialista certo.

A função `conversar(user_id, mensagem)` junta tudo:
- cria uma `SQLiteSession` por usuário (`restaurante_{user_id}`), salva em `restaurante_sessions.db`;
- passa o `CustomerContext` para o `Runner`;
- trata cada tipo de falha com uma mensagem amigável: guardrail de entrada, guardrail de saída e erro genérico.

Os testes mostram a memória funcionando (*"No pedido que acabei de fazer, deixe apenas 1 unidade"*), o isolamento entre clientes diferentes e o bloqueio de uma tentativa de prompt injection.

## 6. Integração com Telegram

1. Criar o bot no **@BotFather** e salvar o token no Secret `TELEGRAM_BOT_TOKEN`.
2. Usar a biblioteca `python-telegram-bot`, com os handlers:
   - `/start`: mensagem de boas-vindas;
   - `/help`: exemplos de perguntas;
   - mensagens de texto: chamam `conversar(f"telegram_{user.id}", texto)`.
3. O `id` do usuário no Telegram vira o `user_id`, então cada pessoa tem a própria sessão e os próprios pedidos.
4. Respostas longas são quebradas em blocos de 4000 caracteres (limite do Telegram).
5. No Colab, o bot roda com **polling assíncrono** e `nest_asyncio`, para não conflitar com o event loop do notebook. Há células separadas para **iniciar** e **parar** o bot.

## O que este projeto ensina

- Dividir responsabilidades entre **agentes especialistas**.
- Colocar regras críticas **no código** (ferramentas), e não só no prompt.
- Usar **contexto** para separar dados por usuário.
- Montar guardrails "inteligentes" com agentes classificadores.
- Levar um agente para um **canal real** de atendimento.

## Conceitos-chave

sistema multiagente · triagem e handoff · `RunContextWrapper` e contexto · regras de negócio em ferramentas · guardrails baseados em agentes · `FileSearchTool` · `CodeInterpreterTool` · `SQLiteSession` · bot do Telegram · polling assíncrono
