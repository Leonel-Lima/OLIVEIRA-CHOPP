# PROMPT — BACKEND SISTEMA OLIVEIRA CHOPP

## CONTEXTO
Preciso desenvolver o backend completo de um sistema de lanchonete/bar chamado **Oliveira Chopp**.
O frontend já está 100% pronto em HTML/CSS/JS puro (5 telas).
O servidor é Ubuntu 24.04 no IP **204.168.149.123**.
Já existe um projeto Node.js rodando no servidor em `/opt/tendas-narandiba/` gerenciado pelo PM2.

---

## SERVIDOR
- **OS:** Ubuntu 24.04.3 LTS
- **IP:** 204.168.149.123
- **Gerenciador de processos:** PM2
- **Já instalado:** Node.js, PM2, MySQL (verificar se está ativo)
- **Diretório do novo projeto:** `/opt/oliveira-chopp/`
- **Porta sugerida:** 3001 (verificar se está livre)

---

## TELAS DO SISTEMA (frontend já pronto)

### 1. `cardapio_oliveira_chopp.html` — Cliente (mobile via QR Code)
- Cliente escaneia QR da mesa e abre o cardápio
- Visualiza itens com emoji, nome, descrição e preço
- Adiciona/remove itens com controle de quantidade
- Campo de observações no pedido
- Envia pedido para a cozinha/bar
- URL com parâmetro de mesa: `cardapio.html?mesa=01`

### 2. `admin_oliveira_chopp.html` — Administrador
- Cadastrar, editar, excluir itens do cardápio
- Ativar/desativar itens
- Gerenciar categorias
- Visualizar resumo do cardápio

### 3. `barcozinha_oliveira_chopp.html` — Bar & Cozinha
- **Aba Bar:** recebe pedidos de bebidas (Chopps, Cervejas, Refrigerantes)
- **Aba Cozinha:** recebe pedidos de comidas (Churrasquinho, Porções)
- Kanban com 3 colunas: Novo → Em Preparo → Pronto
- Botão "Confirmar Entrega" remove da fila e vai para histórico
- Timer por pedido (verde < 5min, amarelo < 10min, vermelho > 10min)
- Relógio em tempo real
- Histórico do dia por setor

### 4. `caixa_oliveira_chopp.html` — Caixa
- Visão geral das mesas (ocupadas/livres)
- Ver pedidos por mesa
- Aplicar desconto (% ou R$)
- Formas de pagamento: Dinheiro (com troco), Cartão, Pix
- Imprimir comanda
- Fechar conta e liberar mesa
- Contador de vendas do dia

### 5. `estoque_oliveira_chopp.html` — Estoque
- Quantidade atual de cada item
- Alerta de estoque baixo (banner vermelho automático)
- Barra de progresso por item
- Entrada de mercadoria
- Baixa manual (venda, perda, consumo)
- Histórico de movimentações

---

## CATEGORIAS DO CARDÁPIO
| ID | Nome | Setor |
|----|------|-------|
| 1 | Chopps | Bar |
| 2 | Cervejas | Bar |
| 3 | Churrasquinho | Cozinha |
| 4 | Porções | Cozinha |
| 5 | Refrigerantes | Bar |

**Regra de roteamento:** categorias 1, 2, 5 → Bar | categorias 3, 4 → Cozinha

---

## ITENS INICIAIS DO CARDÁPIO
| Nome | Categoria | Preço |
|------|-----------|-------|
| Chopp Pilsen 300ml | Chopps | R$ 8,00 |
| Chopp Pilsen 500ml | Chopps | R$ 12,00 |
| Chopp Escuro 300ml | Chopps | R$ 9,00 |
| Cerveja Lata | Cervejas | R$ 7,00 |
| Cerveja Long Neck | Cervejas | R$ 10,00 |
| Churrasquinho de Queijo | Churrasquinho | R$ 5,00 |
| Churrasquinho de Carne | Churrasquinho | R$ 6,00 |
| Churrasquinho de Frango | Churrasquinho | R$ 6,00 |
| Churrasquinho Kafta | Churrasquinho | R$ 7,00 |
| Porção de Fritas | Porções | R$ 18,00 |
| Refrigerante Lata | Refrigerantes | R$ 5,00 |
| Água Mineral | Refrigerantes | R$ 3,00 |

---

## MESAS
- 8 mesas numeradas de 01 a 08
- Cada mesa tem um QR Code único
- QR aponta para: `http://204.168.149.123:3001/cardapio?mesa=01`

---

## REQUISITOS DO BACKEND

### Tecnologia
- **Runtime:** Node.js
- **Framework:** Express.js
- **Banco de dados:** MySQL
- **Tempo real:** Socket.IO (WebSocket)
- **QR Code:** biblioteca `qrcode` do npm

### Funcionalidades obrigatórias

#### 1. API REST
```
GET    /api/cardapio          → listar itens ativos
GET    /api/cardapio/admin    → listar todos os itens (admin)
POST   /api/cardapio          → criar item
PUT    /api/cardapio/:id      → editar item
DELETE /api/cardapio/:id      → excluir item
PATCH  /api/cardapio/:id/toggle → ativar/desativar item

GET    /api/categorias        → listar categorias
POST   /api/categorias        → criar categoria
DELETE /api/categorias/:id    → excluir categoria

POST   /api/pedidos           → criar pedido (cliente)
GET    /api/pedidos           → listar pedidos ativos
GET    /api/pedidos/mesa/:num → pedidos de uma mesa
PATCH  /api/pedidos/:id/status → atualizar status
GET    /api/pedidos/historico → histórico do dia

GET    /api/mesas             → listar mesas e status
POST   /api/mesas/:num/fechar → fechar conta da mesa

GET    /api/estoque           → listar estoque
POST   /api/estoque/entrada   → entrada de mercadoria
POST   /api/estoque/baixa     → baixa manual
GET    /api/estoque/historico → histórico de movimentações

GET    /api/qrcode/:mesa      → gerar QR Code da mesa
```

#### 2. WebSocket (Socket.IO)
Eventos em tempo real:
- `novo_pedido` → emitido quando cliente faz pedido (bar/cozinha recebem)
- `pedido_atualizado` → quando status muda (caixa atualiza mesa)
- `pedido_entregue` → quando confirma entrega (caixa libera mesa)
- `estoque_baixo` → quando item fica abaixo do mínimo

#### 3. Banco de dados MySQL
Tabelas necessárias:
- `categorias` (id, nome, emoji, setor)
- `cardapio` (id, nome, categoria_id, preco, emoji, descricao, ativo)
- `mesas` (id, numero, status)
- `pedidos` (id, mesa_num, status, obs, criado_em, setor)
- `pedido_itens` (id, pedido_id, item_id, nome, qty, preco)
- `estoque` (id, item_id, nome, categoria_id, emoji, qty, min_qty)
- `estoque_movimentacoes` (id, item_id, nome, tipo, qty, obs, criado_em)
- `vendas` (id, mesa_num, total, desconto, forma_pgto, criado_em)

#### 4. Separação automática de pedidos
Quando cliente envia pedido com itens mistos (bebida + comida):
- Sistema separa automaticamente em 2 pedidos
- Pedido 1 → setor BAR (categorias 1, 2, 5)
- Pedido 2 → setor COZINHA (categorias 3, 4)
- Ambos vinculados ao mesmo número de mesa
- WebSocket notifica cada setor individualmente

---

## ESTRUTURA DE ARQUIVOS SUGERIDA
```
/opt/oliveira-chopp/
├── server.js          → servidor principal Express + Socket.IO
├── package.json
├── .env               → variáveis de ambiente (DB, porta)
├── db/
│   ├── connection.js  → conexão MySQL
│   └── seed.sql       → dados iniciais
├── routes/
│   ├── cardapio.js
│   ├── pedidos.js
│   ├── mesas.js
│   ├── estoque.js
│   └── qrcode.js
├── public/
│   ├── cardapio_oliveira_chopp.html
│   ├── admin_oliveira_chopp.html
│   ├── barcozinha_oliveira_chopp.html
│   ├── caixa_oliveira_chopp.html
│   └── estoque_oliveira_chopp.html
└── qrcodes/           → QR codes gerados das mesas
```

---

## VARIÁVEIS DE AMBIENTE (.env)
```
PORT=3001
DB_HOST=localhost
DB_USER=root
DB_PASS=SENHA_AQUI
DB_NAME=oliveira_chopp
```

---

## FLUXO COMPLETO DE UM PEDIDO
1. Cliente escaneia QR da mesa (ex: mesa 03)
2. Abre `cardapio.html?mesa=03` no celular
3. Escolhe itens, adiciona observação e clica "Enviar Pedido"
4. Frontend faz `POST /api/pedidos` com os itens e mesa
5. Backend separa em pedido BAR e pedido COZINHA
6. Socket.IO emite `novo_pedido` para bar e cozinha
7. Bar/cozinha veem o pedido aparecer em tempo real na coluna "Novos"
8. Clicam "Iniciar Preparo" → status muda → caixa vê mesa ocupada
9. Clicam "Marcar Pronto" → garçom é avisado
10. Garçom entrega e clica "Confirmar Entrega"
11. Caixa fecha a conta → `POST /api/mesas/03/fechar`
12. Mesa volta para status "livre"
13. Estoque é baixado automaticamente

---

## OBSERVAÇÕES IMPORTANTES
- O servidor já tem outros projetos rodando (tendas-narandiba na porta 3000)
- Usar PM2 para gerenciar o processo: `pm2 start server.js --name oliveira-chopp`
- Configurar nginx como proxy reverso se necessário
- Os HTMLs precisam ser adaptados para consumir a API real (substituir dados mockados)
- Prioridade: fazer funcionar primeiro, otimizar depois
- Não precisa de autenticação/login por enquanto (fase 1)
