# API Gateway – Integração Seller Itaú Shop

Este documento serve como um guia de integração e FAQ para parceiros que desejam utilizar a **API Gateway de Integração Seller Itaú Shop**. Ele aborda conceitos, fluxos de negócio e dúvidas frequentes, auxiliando tanto o time de suporte quanto os desenvolvedores dos parceiros.

---

## Sumário

1. [Introdução](#introdução)  
2. [Glossário de Termos](#glossário-de-termos)  
3. [Principais Endpoints e Fluxos](#principais-endpoints-e-fluxos)  
4. [FAQ (Perguntas Frequentes)](#faq-perguntas-frequentes)  
5. [Exemplos de Fluxos de Integração](#exemplos-de-fluxos-de-integração)  
6. [Conclusão](#conclusão)

---

## Introdução

A **API Gateway de Integração Seller Itaú Shop** permite que vendedores (sellers) cadastrem ou atualizem SKUs, gerenciem estoque, preço, pedidos e muito mais, de forma integrada ao marketplace Itaú Shop.

Este *README* consolida informações essenciais para a **integração**:
- Conceitos e termos (SKU, EAN, refId, etc.).
- Principais endpoints (como `/suggestions`, `/products`, `/orders`, `/sellers/baseurl`).
- Fluxos comuns (cadastro, cancelamento de pedidos, etc.).
- Perguntas frequentes (FAQ).

---

## Glossário de Termos

### SKU (Stock Keeping Unit)
Identificador único de produto. Cada variação (ex.: cor, tamanho) costuma ter um SKU distinto.

### SellerSkuId
Código interno de SKU usado pelo **Seller**. Em várias rotas, aparece como parâmetro de caminho (path), por exemplo, `POST /products/{sellerSkuId}/price`.

### EAN
Código de barras universal de 13 dígitos. Caso não haja EAN, utilize `refId`.

### refId
Identificador de referência interna para o SKU (obrigatório se não houver EAN).

### CategoryFullPath
Caminho completo de categoria no formato `{departamento}/{categoria}`, por exemplo: `Eletrodomésticos/Forno`.

### ProductDescription
Descrição textual de um produto.

### Suggestions
Cadastro de produtos/SKUs enviados pelo Seller ao Itaú Shop para aprovação. Podem estar em status como `pending`, `accepted` ou `denied`.

### Pricing
Informações de preço (por exemplo, `salePrice`) e moeda. É usado tanto em `Suggestions` quanto em endpoints de atualização de preço.

### AvailableQuantity
Quantidade disponível em estoque.

### x_itau_apikey, x_itau_correlationid, authorization
*Headers* obrigatórios para autenticação e rastreamento das requisições:
- **x_itau_apikey**: Chave de API do Itaú.  
- **x_itau_correlationid**: Identificador de correlação para logs (p. ex., UUID).  
- **authorization**: Token de autenticação (formato Bearer).

### BaseURL (Sellers)
Endpoint que o Seller cadastra para informar ao Itaú Shop onde chamá-lo para determinadas ações (e.g., frete, criação de pedido no Seller).

### Rotas “seller precisa implementar”
Algumas rotas do **Seller** devem estar disponíveis para que o Itaú Shop as acione, por exemplo:
- `POST /url-seller/simulation` (cálculo de frete)  
- `POST /url-seller/orders` (criação de pedido)  
- `POST /url-seller/orders/{sellerOrderId}/approve` (aprovar pedido)  
- `POST /url-seller/orders/{sellerOrderId}/cancel` (cancelar pedido)  
- `GET /url-seller/orders/{sellerOrderId}` (consultar pedido)

---

## Principais Endpoints e Fluxos

Abaixo, os endpoints mais utilizados, organizados em categorias:

### 1. Suggestions
- **POST** `/suggestions`: cria uma *suggestion* de SKU.  
- **GET** `/suggestions`: lista as sugestões existentes (com opção de filtros `type`, `status`, etc.).  
- **GET** `/suggestions/{sellerSkuId}`: obtém detalhes de uma *suggestion* específica.  
- **PUT** `/suggestions/{sellerSkuId}`: atualiza dados de uma *suggestion*.  
- **DELETE** `/suggestions/{sellerSkuId}`: exclui a *suggestion* correspondente.

### 2. Produtos
- **PUT** `/products/inventory-info`: atualiza informações de estoque (versão atual).  
- **PUT** `/products/{sellerSkuId}/price-info`: atualiza informações de preço (versão atual).  
- **POST** `/products/{sellerSkuId}/inventory`: atualiza estoque (rota legada/deprecada).  
- **POST** `/products/{sellerSkuId}/price`: atualiza preço (rota legada/deprecada).  
- **DELETE** `/products/{sellerSkuId}`: remove a ligação entre o Seller e o SKU especificado.

### 3. Pedidos (Orders)
- **GET** `/orders/{orderId}`: consulta detalhes de um pedido.  
- **POST** `/orders/{orderId}/invoice`: fatura o pedido (informando nota fiscal, etc.).  
- **POST** `/orders/{orderId}/cancel`: cancela o pedido.  
- **POST** `/orders/{orderId}/tracking/{invoiceNumber}/sent`: inicia o rastreamento do envio.  
- **PUT** `/orders/{orderId}/tracking/{invoiceNumber}/update`: atualiza o rastreamento (entrega, datas, eventos, etc.).

### 4. Sellers
- **GET** `/sellers/baseurl`: consulta a URL base do Seller.  
- **POST** `/sellers/baseurl`: cria nova URL base.  
- **PUT** `/sellers/baseurl`: atualiza a URL base.

### 5. Rotas que o Seller deve expor
Como mencionado, são endpoints no seu sistema, chamados pelo Itaú Shop:

- `POST /url-seller/simulation`: simulação de frete.  
- `POST /url-seller/orders`: criação de pedido no Seller.  
- `POST /url-seller/orders/{sellerOrderId}/approve`: aprovar pedido já criado.  
- `POST /url-seller/orders/{sellerOrderId}/cancel`: cancelamento de pedido.  
- `GET /url-seller/orders/{sellerOrderId}`: consulta pedido diretamente no Seller.

---

## FAQ (Perguntas Frequentes)

1. **Como autenticar as requisições?**  
   Use os *headers* `x_itau_apikey`, `x_itau_correlationid` e `authorization` em cada chamada.

2. **Diferença entre `/price`/`/inventory` (legadas) e `/price-info`/`/inventory-info` (atuais)?**  
   - As rotas antigas (`POST /products/{sku}/price`, etc.) estão deprecadas.  
   - As rotas novas (`PUT /products/{sku}/price-info` e `PUT /products/inventory-info`) possuem melhorias e devem ser adotadas.

3. **Por que `height`, `width`, `length`, `weight` têm `minimum = 1`?**  
   - A especificação exige valores >= 1 (por exemplo, 1 cm). Caso precise trabalhar com valores menores ou decimais, é preciso alinhar com a equipe do Itaú.

4. **Preciso informar `refId` ou `ean` ao criar um SKU?**  
   - Sim, pelo menos um desses identificadores é obrigatório.

5. **Erro 401/403 – o que significa?**  
   - **401 (Unauthorized)**: falha na autenticação (token inválido, ausente, expirado).  
   - **403 (Forbidden)**: seu token não tem permissão para acessar aquele recurso.

6. **Como funcionam as *suggestions*?**  
   - Você envia uma nova (POST), ela fica pendente até aprovação ou reprovação.  
   - Consultas podem ser feitas via GET.  
   - Se necessário, atualize (PUT) ou exclua (DELETE).

7. **Como atualizar preço ou estoque de um SKU que já existe?**  
   - Estoque: `PUT /products/inventory-info`.  
   - Preço: `PUT /products/{sellerSkuId}/price-info`.

8. **Como cancelar um pedido?**  
   - **No marketplace**: `POST /orders/{orderId}/cancel`.  
   - **No Seller**: se o Itaú Shop cancelar, ele chamará `POST /url-seller/orders/{sellerOrderId}/cancel`.

9. **Como emitir a nota fiscal (faturamento) de um pedido?**  
   - `POST /orders/{orderId}/invoice`, enviando informações como `invoiceNumber`, `invoiceKey`, `items`, etc.

10. **Como inserir rastreamento (tracking)?**  
   - Para começar, `POST /orders/{orderId}/tracking/{invoiceNumber}/sent`.  
   - Para atualizar status de entrega ou histórico, `PUT /orders/{orderId}/tracking/{invoiceNumber}/update`.

11. **Como o Itaú Shop chama minhas rotas de Seller?**  
   - Você cadastra sua URL base em `POST /sellers/baseurl` ou `PUT /sellers/baseurl`.  
   - Então, o Itaú Shop fará requisições para os endpoints de `/url-seller/...` no seu sistema.

12. **O que é `x_itau_correlationid`?**  
   - ID de rastreamento único da requisição (muitas vezes um UUID). Ajuda na depuração de logs.

13. **Como enviar eventos de rastreamento (campo `events`)?**  
   - Cada evento inclui `city`, `state`, `description`, `date`. São repassados como um array para acompanhar o deslocamento do pedido.

14. **Existe limite de paginação em `GET /suggestions`?**  
   - Sim, até 50 registros por página. Use `_from` e `_to` para definir o intervalo.

15. **Possíveis `status` de uma Suggestion?**  
   - `pending`, `accepted`, `denied`.

---

## Exemplos de Fluxos de Integração

### 1. Cadastro de Novo SKU com Suggestions
1. **POST** `/suggestions` enviando campos como `productName`, `brandName`, `height`, etc.  
2. A *suggestion* fica `pending`.  
3. Consulte com **GET** `/suggestions/{sellerSkuId}` até obter `accepted` ou `denied`.  
4. Se necessário, atualize (PUT) ou exclua (DELETE).

### 2. Atualização de Preço e Estoque
1. **PUT** `/products/{sellerSkuId}/price-info`: corpo com `{ "salePrice": <valor em centavos> }`.  
2. **PUT** `/products/inventory-info`: corpo com `{ "skus": [ { "sellerSkuId": "ABC123", "availableQuantity": 10 } ] }`.

### 3. Fluxo de Pedido (Faturamento, Cancelamento, Rastreamento)
1. **GET** `/orders/{orderId}`: verifica status.  
2. **POST** `/orders/{orderId}/invoice`: informa nota fiscal.  
3. **POST** `/orders/{orderId}/tracking/{invoiceNumber}/sent`: adiciona tracking.  
4. **PUT** `/orders/{orderId}/tracking/{invoiceNumber}/update`: atualiza status de entrega.  
5. **POST** `/orders/{orderId}/cancel`: efetua cancelamento.

---

## Conclusão

Este *README* fornece uma visão geral e respostas rápidas para as principais questões de integração com a **API Gateway de Integração Seller Itaú Shop**. Para dúvidas adicionais ou cenários complexos, consulte a documentação completa em OpenAPI ou entre em contato com o suporte do Itaú.

**Boa integração!**
