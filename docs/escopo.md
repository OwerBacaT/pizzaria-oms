# Escopo do MVP — Pizzeria Order Management System (OMS)

> **Status:** v0.2 — regras com valores padrão de mercado, pendentes de validação com a operação real da pizzaria (ver seção 10). A v0.2 incorpora as regras RN-27 e RN-28, descobertas na modelagem.

## 1. Visão

O OMS garante que cada pedido saia do cliente e chegue ao caixa, à cozinha e à entrega **sem se perder, sem erro de montagem e sem erro de cobrança**, dando ao dono visibilidade do que foi vendido no dia.

## 2. Objetivos do MVP

- Registrar pedidos de balcão e entrega em poucos cliques.
- Exibir a fila da cozinha em tempo real, sem recarregar a página.
- Controlar o ciclo de vida de cada pedido com histórico auditável.
- Registrar as formas de pagamento de cada pedido.
- Gerar o fechamento do dia.

## 3. Stack definida

| Camada | Tecnologia |
|---|---|
| Backend | Python + FastAPI |
| Banco de dados | PostgreSQL |
| ORM / Migrations | SQLAlchemy + Alembic |
| Validação | Pydantic |
| Tempo real | WebSockets (FastAPI nativo) |
| Frontend | JavaScript + React + Vite |
| Infraestrutura | Docker Compose (dev), Render/Railway (deploy) |

## 4. Perfis de usuário

| Perfil | Permissões |
|---|---|
| **Admin (dono)** | Acesso total: cardápio, usuários, bairros e taxas, relatórios, cancelamento em qualquer etapa |
| **Atendente** | Criar e editar pedidos, cadastrar clientes, registrar pagamentos, cancelar até `EM_PREPARO` |
| **Cozinha** | Visualizar a fila e alterar status de preparo (`EM_PREPARO`, `NO_FORNO`, `PRONTO`) |
| **Entregador** | Visualizar pedidos prontos para entrega, marcar `SAIU_PARA_ENTREGA` e `ENTREGUE` |

## 5. Histórias de usuário

- **US-01** — Como atendente, quero montar um pedido escolhendo tamanho, sabores, borda e adicionais, para registrar exatamente o que o cliente pediu.
- **US-02** — Como atendente, quero buscar o cliente pelo telefone, para não redigitar o endereço de quem já pediu antes.
- **US-03** — Como atendente, quero ver o total calculado automaticamente, incluindo a taxa de entrega, para não errar a cobrança.
- **US-04** — Como cozinha, quero ver os pedidos novos aparecendo na tela assim que são criados, em ordem de chegada.
- **US-05** — Como cozinha, quero avançar o status de um pedido com um toque, para que atendimento e entrega saibam o andamento.
- **US-06** — Como entregador, quero ver os pedidos prontos com endereço, valor e forma de pagamento, para levar o troco certo.
- **US-07** — Como dono, quero cadastrar e pausar sabores, para refletir o que está disponível no dia.
- **US-08** — Como dono, quero ver o fechamento do dia, para conferir o caixa.

## 6. Regras de negócio

As regras são numeradas para serem referenciadas em código, testes e commits.

### 6.1 Cardápio

- **RN-01** — Os produtos se dividem em **pizzas** e **itens simples** (bebidas, sobremesas, porções). Itens simples têm preço fixo.
- **RN-02** — Cada pizza é vendida em um **tamanho**, e cada tamanho define o número máximo de sabores:

  | Tamanho | Máx. de sabores |
  |---|---|
  | Broto | 1 |
  | Média | 2 |
  | Grande | 3 |
  | Família | 4 |

- **RN-03** — Cada sabor tem um preço por tamanho.
- **RN-04** — Pizza com mais de um sabor é cobrada pelo **preço do sabor mais caro** naquele tamanho.
- **RN-05** — A **borda recheada** é opcional, com preço definido por tamanho.
- **RN-06** — **Adicionais** têm preço próprio e são somados ao item. **Observações** são texto livre e não alteram o preço.
- **RN-07** — Sabores e produtos podem ser **pausados** (indisponíveis temporariamente) sem serem excluídos. Itens pausados não aparecem para seleção em novos pedidos.
- **RN-08** — Produtos nunca são excluídos fisicamente se já constarem em algum pedido (exclusão lógica), preservando o histórico.

### 6.2 Pedido

- **RN-09** — Todo pedido é de um dos tipos: **RETIRADA** ou **ENTREGA**.
- **RN-10** — Pedido de **ENTREGA** exige cliente com telefone e endereço completo, incluindo bairro.
- **RN-11** — Cada pedido recebe um **número sequencial diário** (#001, #002...), reiniciado a cada dia de operação (RN-27). O ID interno do banco não é exibido a usuários.
- **RN-12** — Os preços são **gravados no item do pedido no momento da venda**. Alterações posteriores no cardápio não modificam pedidos já registrados.
- **RN-13** — O total do pedido é: soma dos itens + adicionais + bordas + taxa de entrega.
- **RN-14** — O pedido só pode ser editado enquanto estiver em `RECEBIDO`.

### 6.3 Ciclo de vida (máquina de estados)

```
RECEBIDO → EM_PREPARO → NO_FORNO → PRONTO ─┬→ RETIRADO                      (retirada)
                                           └→ SAIU_PARA_ENTREGA → ENTREGUE  (entrega)

CANCELADO: a partir de RECEBIDO ou EM_PREPARO (atendente/admin)
           a partir de NO_FORNO ou PRONTO (somente admin)
```

- **RN-15** — Somente as transições acima são válidas. Qualquer outra é rejeitada pela API.
- **RN-16** — O cancelamento exige **motivo obrigatório**.
- **RN-17** — Toda mudança de status registra **status anterior, novo status, usuário e data/hora**.
- **RN-18** — `RETIRADO`, `ENTREGUE` e `CANCELADO` são estados finais.

### 6.4 Entrega

- **RN-19** — A **taxa de entrega é definida por bairro**, cadastrada pelo admin.
- **RN-20** — Pedidos para bairros não cadastrados não podem ser criados como ENTREGA.

### 6.5 Pagamento

- **RN-21** — Formas aceitas: **DINHEIRO**, **PIX**, **CRÉDITO** e **DÉBITO**.
- **RN-22** — Um pedido pode ter **mais de uma forma de pagamento**, e a soma deve ser igual ao total do pedido. A soma só é exigida para **finalizar** o pedido (`RETIRADO` ou `ENTREGUE`), já que em entregas o pagamento costuma ser registrado na volta do entregador.
- **RN-23** — Pagamento em dinheiro aceita o campo **"troco para"**, e o sistema calcula o troco.
- **RN-24** — No MVP o pagamento é apenas **registrado**, sem integração com gateway ou maquininha.

### 6.6 Fechamento do dia

- **RN-25** — O fechamento exibe: total vendido, total por forma de pagamento, quantidade de pedidos, quantidade e valor de cancelamentos e ticket médio.
- **RN-26** — Pedidos cancelados não entram no total vendido nem no ticket médio.

### 6.7 Dia de operação e registro do pedido

- **RN-27** — O pedido pertence a um **dia de operação**. Pedidos criados antes da hora de corte (padrão 06:00, configurável) pertencem ao dia anterior. O número diário (RN-11) e o fechamento (RN-25) usam o dia de operação.
- **RN-28** — O **endereço de entrega e a taxa do bairro** são gravados no pedido no momento da criação. Alterações no cadastro do cliente ou do bairro não afetam pedidos existentes.

## 7. Requisitos não funcionais

- **RNF-01** — Um novo pedido deve aparecer na tela da cozinha em até 2 segundos após a criação.
- **RNF-02** — As telas de atendimento e cozinha devem funcionar em tablet e desktop.
- **RNF-03** — Autenticação por login e senha, com senhas armazenadas com hash e acesso controlado por perfil.
- **RNF-04** — A API deve ter documentação automática (OpenAPI/Swagger).
- **RNF-05** — As regras de negócio (RN) devem ter testes automatizados.
- **RNF-06** — O ambiente de desenvolvimento deve subir com um único comando (`docker compose up`).

## 8. Fora do escopo do MVP

Registrados para fases futuras. O modelo de dados não deve impedir sua implementação.

- Pedidos feitos pelo cliente via site ou landing page
- Recebimento de pedidos via WhatsApp
- Integração com iFood e outros marketplaces
- Pagamento online e integração com maquininha
- Emissão de nota fiscal
- Controle de estoque por ingrediente
- App dedicado do entregador e rastreamento
- Programa de fidelidade e cupons
- Atendimento em mesas (salão)

## 9. Critérios de aceite do MVP

O MVP está pronto quando, em um dia de operação simulado:

1. O admin cadastra o cardápio completo, os bairros e os usuários.
2. O atendente cria pedidos de retirada e de entrega, incluindo pizza com vários sabores, borda e adicionais, com o total correto.
3. A cozinha recebe os pedidos em tempo real e conduz cada um até `PRONTO`.
4. O entregador finaliza as entregas.
5. Um pedido é cancelado com motivo e não entra no faturamento.
6. O fechamento do dia confere com a soma manual dos pedidos.

## 10. Premissas a validar com a operação real

| # | Premissa | Valor adotado |
|---|---|---|
| P-01 | Tamanhos e limite de sabores | Broto 1 / Média 2 / Grande 3 / Família 4 (RN-02) |
| P-02 | Preço de pizza com vários sabores | Sabor mais caro (RN-04) |
| P-03 | Cálculo da taxa de entrega | Por bairro (RN-19) |
| P-04 | Canais de entrada de pedidos no MVP | Balcão e telefone/WhatsApp, digitados pelo atendente |
| P-05 | Uso do sistema | Operação real da pizzaria + portfólio |
| P-06 | Hora de corte do dia de operação | 06:00, a confirmar com o horário real de fechamento (RN-27) |

## 11. Glossário

| Termo | Definição |
|---|---|
| **OMS** | Order Management System — sistema de gestão de pedidos |
| **KDS** | Kitchen Display System — tela da cozinha com a fila de pedidos |
| **PDV** | Ponto de venda — tela do atendente para registrar pedidos |
| **Sabor** | Receita de pizza (ex.: calabresa), com preço por tamanho |
| **Item do pedido** | Uma linha do pedido: uma pizza montada ou um item simples |
| **Ticket médio** | Total vendido ÷ quantidade de pedidos não cancelados |
| **Dia de operação** | Expediente da pizzaria, que pode atravessar a meia-noite (RN-27) |
