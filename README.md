# Documentacao de Consumo da API

## Base URL
- `http://mmsistemas.ddns.net:3101`

## Formato padrao
- `Content-Type`: `application/json`
- `Accept`: `application/json`
- Autenticacao: `Bearer Token` obrigatorio em endpoints de negocio

## Fluxo de autenticacao
1. Consumidor chama `POST /auth/login` com `cnpjCliente`, `usuario` e `senha`.
2. API retorna `tokenAcesso`.
3. Consumidor envia `Authorization: Bearer <tokenAcesso>` em todas as chamadas de negocio.

## Healthcheck
### `GET /`
Sem autenticacao.

Exemplo de resposta:
```json
{
  "status": "ok"
}
```

## Login
### `POST /auth/login`
Sem token.

Campos:

| Campo | Tipo | Obrigatorio | Descricao | Validacao/Regras |
|---|---|---|---|---|
| `cnpjCliente` | string | Sim | CNPJ do cliente | Nao vazio |
| `usuario` ou `usuário` | string | Sim | Usuario de acesso | Nao vazio |
| `senha` | string | Sim | Senha de acesso | Nao vazio |

Exemplo de requisicao:
```json
{
  "cnpjCliente": "12345678000190",
  "usuario": "integrador",
  "senha": "senhaForte@123"
}
```

Exemplo de resposta:
```json
{
  "tokenAcesso": "<JWT>",
  "tipo": "Bearer",
  "expiraEm": "8h"
}
```

## Endpoint de Produto (protegido)
### `POST /produto`
Obrigatorio header:
- `Authorization: Bearer <tokenAcesso>`

Campos:

| Campo | Tipo | Obrigatorio | Descricao | Validacao/Regras |
|---|---|---|---|---|
| `NOMPRO` ou `nomeProduto` | string | Sim | Nome do produto | Nao vazio |
| `DESCRICAO` ou `descricaoProduto` | string | Sim | Descricao do produto | Nao vazio |
| `NOMUN` ou `unidadeMedida` | string | Sim | Unidade de medida | Nao vazio |
| `BARPRO` ou `gtin` | string | Nao | Codigo de barras | Se enviado, nao vazio |
| `CODCLI` ou `skuCliente` | string | Nao | SKU do cliente | Se enviado, nao vazio |
| `NONCM` ou `ncm` | string | Nao | NCM do produto | Se enviado, nao vazio |
| `ALT` ou `altura` | number | Nao | Altura | `>= 0` |
| `LARG` ou `largura` | number | Nao | Largura | `>= 0` |
| `PROF` ou `profundidade` | number | Nao | Profundidade | `>= 0` |
| `PESOLIQ` ou `pesoLiquido` | number | Nao | Peso liquido | `>= 0` |
| `PESOBR` ou `pesoBruto` | number | Nao | Peso bruto | `>= 0` |
| `M2` ou `m2` | number | Nao | Area em m2 | `>= 0` |
| `M3` ou `m3` | number | Nao | Cubagem em m3 | `>= 0` |

Regra importante:
- Nao enviar os dois nomes do mesmo campo (exemplo: `NOMPRO` e `nomeProduto` juntos).

Exemplo de resposta:
```json
{
  "NOMPRO": "Camisa Dry Fit",
  "DESCRICAO": "Camisa esportiva manga curta",
  "NOMUN": "UN"
}
```

## Endpoint de Entrada de Nota (protegido)
### `POST /tarefa-entrada`
Obrigatorio header:
- `Authorization: Bearer <tokenAcesso>`

Campos da requisicao:

| Campo | Tipo | Obrigatorio | Descricao | Validacao/Regras |
|---|---|---|---|---|
| `notas` | array | Sim | Lista de notas | Minimo 1 item |

Campos por nota (`notas[]`):

| Campo | Tipo | Obrigatorio | Descricao | Validacao/Regras |
|---|---|---|---|---|
| `identificacao.numeroNfe` | string | Sim | Numero da NF-e | Nao vazio |
| `identificacao.serie` | string | Sim | Serie da NF-e | Nao vazio |
| `identificacao.chaveNfe` | string | Sim | Chave da NF-e | Nao vazio |
| `identificacao.cnpjcliente` | string | Sim | CNPJ do cliente | Nao vazio |
| `identificacao.nomecliente` | string | Sim | Nome do cliente | Nao vazio |
| `identificacao.tipomovimentacao` | string | Sim | Tipo de movimentacao | Nao vazio |
| `identificacao.destinatario.cnpj` | string | Sim | CNPJ do destinatario | Nao vazio |
| `identificacao.destinatario.nome` | string | Sim | Nome do destinatario | Nao vazio |
| `identificacao.destinatario.ie` | string | Sim | IE do destinatario | Nao vazio |
| `identificacao.destinatario.endereco` | string | Sim | Endereco do destinatario | Nao vazio |
| `identificacao.destinatario.ibgecidade` | number | Sim | Codigo IBGE da cidade | `>= 0` |
| `identificacao.destinatario.cep` | number | Sim | CEP do destinatario | `>= 0` |
| `itens` | array | Sim | Itens da nota | Minimo 1 item |
| `totais.valorProdutos` | number | Sim | Soma dos produtos | `>= 0` |
| `totais.pesoBrutoTotal` | number | Sim | Peso bruto total | `>= 0` |
| `totais.pesoLiquidoTotal` | number | Sim | Peso liquido total | `>= 0` |
| `xml.nfe` | string | Sim | XML da NF-e | Nao vazio |

Campos por item (`notas[].itens[]`):

| Campo | Tipo | Obrigatorio | Descricao | Validacao/Regras |
|---|---|---|---|---|
| `skuProduto` | string | Sim | SKU do produto | Nao vazio |
| `descricaoProduto` | string | Sim | Descricao do produto | Nao vazio |
| `quantidade` | number | Sim | Quantidade | `>= 0` |
| `valorUnitario` | number | Sim | Valor unitario | `>= 0` |
| `valorTotal` | number | Sim | Valor total | `>= 0` |
| `unidadeMedida` | string | Sim | Unidade de medida | Nao vazio |
| `pesoBruto` | number | Sim | Peso bruto do item | `>= 0` |
| `gtin` | string | Nao | Codigo de barras | Se enviado, nao vazio |
| `pesoLiquido` | number | Nao | Peso liquido do item | `>= 0` |
| `dimensoes.largura` | number | Nao | Largura do item | `>= 0` |
| `dimensoes.altura` | number | Nao | Altura do item | `>= 0` |
| `dimensoes.profundidade` | number | Nao | Profundidade do item | `>= 0` |

## Endpoint de Pedidos (protegido)
### `POST /pedidos`
Obrigatorio header:
- `Authorization: Bearer <tokenAcesso>`

Campos:

| Campo | Tipo | Obrigatorio | Descricao | Validacao/Regras |
|---|---|---|---|---|
| `pedidos` | array | Sim | Lista de pedidos | Minimo 1 item |
| `pedidos[].identificacao.numeropedido` | string | Sim | Numero do pedido | Nao vazio |
| `pedidos[].identificacao.cnpjcliente` | string | Sim | CNPJ do cliente | Nao vazio |
| `pedidos[].identificacao.nomecliente` | string | Sim | Nome do cliente | Nao vazio |
| `pedidos[].identificacao.tipomovimentacao` | string | Sim | Tipo de movimentacao | Deve ser `SAIDA` |
| `pedidos[].identificacao.destinatario.cnpj` | string | Sim | CNPJ do destinatario | Nao vazio |
| `pedidos[].identificacao.destinatario.nome` | string | Sim | Nome do destinatario | Nao vazio |
| `pedidos[].itens` | array | Sim | Itens do pedido | Minimo 1 item |
| `pedidos[].itens[].skuProduto` | string | Sim | SKU do item | Nao vazio |
| `pedidos[].itens[].quantidade` | number | Sim | Quantidade do item | `> 0` |
| `pedidos[].itens[].valorUnitario` | number | Sim | Valor unitario | `>= 0` |
| `pedidos[].itens[].valorTotal` | number | Sim | Valor total | `>= 0` |
| `pedidos[].itens[].unidadeMedida` | string | Sim | Unidade de medida | Nao vazio |
| `pedidos[].totais.valorProdutos` | number | Sim | Soma dos produtos | `>= 0` |

Exemplo de requisicao:
```json
{
  "pedidos": [
    {
      "identificacao": {
        "numeropedido": "12345",
        "cnpjcliente": "12345678000190",
        "nomecliente": "CLIENTE EXEMPLO LTDA",
        "tipomovimentacao": "SAIDA",
        "destinatario": {
          "cnpj": "12345678000190",
          "nome": "DESTINATARIO EXEMPLO LTDA"
        }
      },
      "itens": [
        {
          "skuProduto": "SKU-001",
          "quantidade": 10,
          "valorUnitario": 25.5,
          "valorTotal": 255,
          "unidadeMedida": "UN"
        }
      ],
      "totais": {
        "valorProdutos": 255
      }
    }
  ]
}
```

Exemplo de resposta:
```json
{
  "success": true,
  "message": "Pedidos registrados",
  "data": {
    "resultados": [
      {
        "numeroPedido": "12345",
        "status": "OK",
        "protocoloPedido": "PED-1700000000000-1"
      }
    ]
  },
  "error": null
}
```

## Endpoint de Cancelamento (protegido)
### `POST /pedidos/cancelamento`
Obrigatorio header:
- `Authorization: Bearer <tokenAcesso>`

Campos:

| Campo | Tipo | Obrigatorio | Descricao | Validacao/Regras |
|---|---|---|---|---|
| `pedido` | array | Sim | Lista de pedidos | Minimo 1 item |
| `pedido[].identificacao.numeropedido` | string | Sim | Numero do pedido | Nao vazio |
| `pedido[].identificacao.motivocancelamento` | string | Sim | Motivo do cancelamento | Nao vazio |

Exemplo de requisicao:
```json
{
  "pedido": [
    {
      "identificacao": {
        "numeropedido": "12345",
        "motivocancelamento": "Erro de destinatario"
      }
    }
  ]
}
```

Exemplo de resposta:
```json
{
  "success": true,
  "message": "Cancelamento solicitado",
  "data": {
    "resultados": [
      {
        "numeroPedido": "12345",
        "status": "CANCEL_REQUESTED"
      }
    ]
  },
  "error": null
}
```

## Endpoint de Separacao (protegido)
### `POST /pedidos/separacao`
Obrigatorio header:
- `Authorization: Bearer <tokenAcesso>`

Campos:

| Campo | Tipo | Obrigatorio | Descricao | Validacao/Regras |
|---|---|---|---|---|
| `pedido` | array | Sim | Lista de pedidos | Minimo 1 item |
| `pedido[].identificacao.numeropedido` | string | Sim | Numero do pedido | Nao vazio |
| `pedido[].identificacao.statusseparacao` | string | Sim | Status da separacao | `concluido`, `pendente`, `em_andamento` |
| `pedido[].identificacao.volume` | number | Nao | Quantidade de volumes | `>= 0` |
| `pedido[].identificacao.m3` | number | Nao | Cubagem | `>= 0` |

Exemplo de requisicao:
```json
{
  "pedido": [
    {
      "identificacao": {
        "numeropedido": "12345",
        "statusseparacao": "concluido",
        "volume": 15,
        "m3": 3
      }
    }
  ]
}
```

Exemplo de resposta:
```json
{
  "success": true,
  "message": "Status de separacao atualizado",
  "data": {
    "resultados": [
      {
        "numeroPedido": "12345",
        "status": "OK"
      }
    ]
  },
  "error": null
}
```

## Endpoint de Faturamento (protegido)
### `POST /pedidos/faturamento`
Obrigatorio header:
- `Authorization: Bearer <tokenAcesso>`

Campos:

| Campo | Tipo | Obrigatorio | Descricao | Validacao/Regras |
|---|---|---|---|---|
| `pedido` | array | Sim | Lista de pedidos | Minimo 1 item |
| `pedido[].numeropedido` | string | Sim | Numero do pedido | Nao vazio |
| `pedido[].xmlnfe` | string | Sim | XML da NF-e | Nao vazio |
| `pedido[].transportadora` | object | Nao | Dados da transportadora | Se enviado, deve ser objeto |
| `pedido[].transportadora.cnpj` | string | Nao | CNPJ da transportadora | Opcional |
| `pedido[].transportadora.nome` | string | Nao | Nome da transportadora | Opcional |
| `pedido[].transportadora.ie` | string | Nao | IE da transportadora | Opcional |
| `pedido[].transportadora.endereco` | string | Nao | Endereco da transportadora | Opcional |
| `pedido[].transportadora.ibgecidade` | number | Nao | Codigo IBGE da cidade | `>= 0` |
| `pedido[].transportadora.cep` | string | Nao | CEP da transportadora | Opcional |

Exemplo de requisicao:
```json
{
  "pedido": [
    {
      "numeropedido": "12345",
      "transportadora": {
        "cnpj": "12345678000190",
        "nome": "TRANSPORTADORA EXEMPLO LTDA"
      },
      "xmlnfe": "base64-ou-xml-string"
    }
  ]
}
```

Exemplo de resposta:
```json
{
  "success": true,
  "message": "Faturamento vinculado ao pedido",
  "data": {
    "resultados": [
      {
        "numeroPedido": "12345",
        "status": "OK",
        "chaveNfe": null
      }
    ]
  },
  "error": null
}
```

## Erros
### `400 Falha na validacao` (novos endpoints de pedidos)
```json
{
  "success": false,
  "message": "Falha na validacao",
  "data": null,
  "error": {
    "code": "VALIDATION_ERROR",
    "details": [
      "descricao do erro"
    ]
  }
}
```

### `401 Nao autorizado`
```json
{
  "message": "Token de acesso obrigatorio.",
  "error": "Envie o header Authorization: Bearer <tokenAcesso>."
}
```

```json
{
  "message": "Token de acesso invalido.",
  "error": "jwt expired"
}
```

```json
{
  "message": "Credenciais invalidas."
}
```

### `500 Erro ao inserir no banco` (`/produto` e `/tarefa-entrada`)
```json
{
  "message": "Erro ao inserir no banco.",
  "error": "mensagem tecnica"
}
```
