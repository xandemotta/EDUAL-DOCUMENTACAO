# API INFOCWB - Guia de Consumo

Data de referencia: 2026-02-19

## 1. Base URL

- Producao: `http://mmsistemas.ddns.net:3111`

## 2. Autenticacao

Todos os endpoints (exceto `GET /health`) exigem Bearer Token:

```http
Authorization: Bearer <seu_token>
```

Headers recomendados:

```http
Content-Type: application/json
Accept: application/json
Authorization: Bearer <seu_token>
```

## 3. Padrao de resposta

Sucesso:

```json
{
  "success": true,
  "message": "OK",
  "data": {},
  "error": null
}
```

Erro:

```json
{
  "success": false,
  "message": "Falha na validacao",
  "data": null,
  "error": {
    "code": "VALIDATION_ERROR",
    "details": [
      { "field": "campo", "message": "mensagem" }
    ],
    "traceId": "uuid"
  }
}
```

Observacao:
- Para regras de negocio, `error.code` vem como `BUSINESS_RULE_ERROR` (HTTP 422).

## 4. Healthcheck

### GET `/health`

Sem autenticacao.

Exemplo de retorno:

```json
{
  "success": true,
  "message": "OK",
  "data": { "service": "online" },
  "error": null
}
```

## 5. Endpoints de consumo

### 5.1 POST `/produtos`

Upsert de produto (chave por `codcli`).

Campos obrigatorios:
- `nompro`
- `descricao`
- `codcli`
- `nomun`

Exemplo:

```json
{
  "nompro": "Produto Exemplo",
  "barpro": "7891234567890",
  "descricao": "Descricao completa do produto",
  "codcli": "SKU-001",
  "noncm": "12345678",
  "alt": 20,
  "larg": 30,
  "prof": 15,
  "pesoliq": 5.0,
  "pesobr": 5.2,
  "m2": 0.6,
  "m3": 0.009,
  "nomun": "UN"
}
```

Exemplo de retorno (`200 OK`):

```json
{
  "success": true,
  "message": "Produto cadastrado/atualizado com sucesso",
  "data": {
    "codcli": "SKU-001",
    "nopro": 12345,
    "noun": 1,
    "nomun": "UN",
    "status": "UPDATED"
  },
  "error": null
}
```

Exemplo de erro (`400 Bad Request`):

```json
{
  "success": false,
  "message": "Falha na validacao",
  "data": null,
  "error": {
    "code": "VALIDATION_ERROR",
    "details": [
      {
        "field": "nompro",
        "message": "Obrigatorio, 1..120"
      }
    ],
    "traceId": "7f5a9f58-b0fd-4dd8-a2f0-5bf7ebf16af8"
  }
}
```

### 5.2 POST `/entradas`

Registra entrada por nota (`notas[]`).

Campos obrigatorios em `notas[].identificacao`:
- `numeroNfe`
- `serie`
- `chaveNfe` (44 digitos)
- `cnpjcliente` (14 digitos)
- `nomecliente`
- `tipomovimentacao`
- `cnpjDestinatario` (14 digitos)

Campos obrigatorios em `notas[].itens[]`:
- `skuProduto`
- `descricaoProduto`
- `quantidade` (> 0)
- `valorUnitario` (>= 0)
- `valorTotal` (>= 0 e igual a `quantidade * valorUnitario`)
- `gtin` (8/12/13/14 digitos)
- `pesoBruto` (>= 0)
- `pesoLiquido` (>= 0)
- `dimensoes.largura` (>= 0)
- `dimensoes.altura` (>= 0)
- `dimensoes.profundidade` (>= 0)

Exemplo:

```json
{
  "notas": [
    {
      "identificacao": {
        "numeroNfe": "ENT-POSTMAN-123",
        "serie": "1",
        "chaveNfe": "35191111111111111111550010000012345678901234",
        "cnpjcliente": "00398268000123",
        "nomecliente": "POLIOTTO IMPORTACAO E EXPORTACAO DE PLASTICOS LTDA",
        "tipomovimentacao": "ENTRADA",
        "cnpjDestinatario": "11259442000173",
        "usuario": "INTEGRACAO"
      },
      "itens": [
        {
          "skuProduto": "SKU-001",
          "descricaoProduto": "Produto Exemplo 1",
          "quantidade": 10,
          "valorUnitario": 25.5,
          "valorTotal": 255,
          "unidadeMedida": "UN",
          "gtin": "7891234567890",
          "pesoBruto": 5.2,
          "pesoLiquido": 5,
          "dimensoes": {
            "largura": 30,
            "altura": 20,
            "profundidade": 15
          }
        }
      ]
    }
  ]
}
```

Exemplo de retorno (`200 OK`):

```json
{
  "success": true,
  "message": "Entrada registrada",
  "data": {
    "resultados": [
      {
        "nopr": 123456,
        "refcli": "ENT-POSTMAN-123",
        "chaveNfe": "35191111111111111111550010000012345678901234",
        "status": "OK"
      }
    ]
  },
  "error": null
}
```

Exemplo de erro (`422 Unprocessable Entity`):

```json
{
  "success": false,
  "message": "Regra de negocio invalida",
  "data": null,
  "error": {
    "code": "BUSINESS_RULE_ERROR",
    "details": [
      {
        "field": "identificacao.cnpjcliente",
        "message": "Cliente nao encontrado para cnpjcliente=00398268000123"
      }
    ],
    "traceId": "f8d203bc-cd94-4d14-ad8a-8ec2f789fbd4"
  }
}
```

Erros comuns: `400` (validacao), `422` (cliente/armazem nao encontrados, regras de itens).


### 5.5 GET `/entradas`

Consulta entrada por `refcli` (ou `numeroNfe` como alias).

Query params:
- `refcli` (obrigatorio, quando `numeroNfe` nao for enviado)
- `numeroNfe` (alias de `refcli`; use um ou outro)
- `cgccli` (opcional; se enviado e nao existir cliente, retorna `422`)

Exemplo:

```http
GET /entradas?cgccli=00398268000123&refcli=ENT-POSTMAN-123
```

Exemplo equivalente com alias:

```http
GET /entradas?cgccli=00398268000123&numeroNfe=ENT-POSTMAN-123
```

Exemplo de retorno (`200 OK`):

```json
{
  "success": true,
  "message": "Entrada consultada",
  "data": {
    "resultados": [
      {
        "NOPRETAREFA": 123456,
        "USUARIO": null,
        "NOCLI": 10,
        "NOEMP": 1,
        "DATAREG": "2026-02-19T14:00:00.000Z",
        "REFCLI": "ENT-POSTMAN-123",
        "STATUS": "EM_PROCESSO",
        "OBS": null,
        "DESTINATARIO": null,
        "TRANSPORTE": null
      }
    ]
  },
  "error": null
}
```

Exemplo de erro (`400 Bad Request`):

```json
{
  "success": false,
  "message": "Falha na validacao",
  "data": null,
  "error": {
    "code": "VALIDATION_ERROR",
    "details": [
      {
        "field": "refcli|numeroNfe",
        "message": "Campo obrigatorio"
      }
    ],
    "traceId": "0580b933-c69f-46b5-9de0-2e38990de5f3"
  }
}
```

### 5.6 POST `/pedidos`

Cria pedido de separacao.

Campos obrigatorios em `pedidos[].identificacao`:
- `numeropedido`
- `cgccli` (CNPJ cliente)
- `tipomovimentacao`
- `destinatario.cnpj`
- `destinatario.nome`

Campos obrigatorios em `pedidos[].itens[]`:
- `nompro`
- `qtd`
- `vlrunit`
- `total`

Exemplo:

```json
{
  "pedidos": [
    {
      "identificacao": {
        "numeropedido": "PED-API-10001",
        "cgccli": "00398268000123",
        "nomecliente": "POLIOTTO IMPORTACAO E EXPORTACAO DE PLASTICOS LTDA",
        "tipomovimentacao": "SAIDA",
        "destinatario": {
          "cnpj": "04700714000163",
          "nome": "APM TERMINALS PORTUARIOS SA",
          "ie": "12345678",
          "endereco": "Rua Exemplo 456",
          "numero": "456",
          "complemento": "Galpao",
          "bairro": "Centro",
          "ibgecidade": 1200013,
          "cep": "88301365"
        },
        "usuario": "INTEGRACAO"
      },
      "itens": [
        {
          "nompro": "Produto Exemplo 1",
          "codcli": "SKU-001",
          "qtd": 10,
          "vlrunit": 25.5,
          "total": 255
        }
      ]
    }
  ]
}
```

Exemplo de retorno (`200 OK`):

```json
{
  "success": true,
  "message": "Pedidos registrados",
  "data": {
    "resultados": [
      {
        "numeroPedido": "PED-API-10001",
        "nopr": 123457,
        "status": "OK"
      }
    ]
  },
  "error": null
}
```

Exemplo de erro (`400 Bad Request`):

```json
{
  "success": false,
  "message": "Falha na validacao",
  "data": null,
  "error": {
    "code": "VALIDATION_ERROR",
    "details": [
      {
        "field": "pedidos",
        "message": "Array obrigatorio com ao menos 1 pedido"
      }
    ],
    "traceId": "f2f2f307-ed40-455a-96f9-c0f7432f2e95"
  }
}
```

### 5.7 POST `/pedidos/cancelamento`

Solicita cancelamento do pedido.

Campos obrigatorios:
- `identificacao.numeropedido`
- `identificacao.cgccli`

Campo opcional:
- `identificacao.motivocancelamento`

Exemplo:

```json
{
  "pedido": [
    {
      "identificacao": {
        "numeropedido": "PED-API-10001",
        "cgccli": "00398268000123",
        "motivocancelamento": "Cancelamento solicitado"
      }
    }
  ]
}
```

Exemplo de retorno (`200 OK`):

```json
{
  "success": true,
  "message": "Cancelamento solicitado",
  "data": {
    "resultados": [
      {
        "numeroPedido": "PED-API-10001",
        "status": "CANCELADO"
      }
    ]
  },
  "error": null
}
```

Exemplo de erro (`422 Unprocessable Entity`):

```json
{
  "success": false,
  "message": "Regra de negocio invalida",
  "data": null,
  "error": {
    "code": "BUSINESS_RULE_ERROR",
    "details": [
      {
        "field": "pedido",
        "message": "Pedido ja em separacao (NOOSMOV=123). Necessario validar regra com Marcio."
      }
    ],
    "traceId": "3263e5e7-627a-4f3c-b8ab-d8f790d6f933"
  }
}
```

Regra importante:
- Se o pedido ja estiver em separacao, o cancelamento retorna `422`.

### 5.8 POST `/pedidos/separacao`

Atualiza status de separacao.

Campos usados:
- `identificacao.numeropedido`
- `identificacao.cgccli`
- `identificacao.statusseparacao`

Exemplo:

```json
{
  "pedido": [
    {
      "identificacao": {
        "numeropedido": "PED-API-10001",
        "cgccli": "00398268000123",
        "statusseparacao": "concluido"
      }
    }
  ]
}
```

Retorno inclui totais de separacao no corpo raiz (quando encontrados):
- `statusseparacao`
- `noosmov`
- `volume`
- `pesobruto`
- `pesoliquido`
- `totalmercadoria`

Exemplo de retorno (`200 OK`):

```json
{
  "success": true,
  "statusseparacao": "concluido",
  "noosmov": 98765,
  "volume": 1,
  "pesobruto": 12.3,
  "pesoliquido": 11.8,
  "totalmercadoria": 255,
  "message": "Status de separacao atualizado",
  "data": {
    "resultados": [
      {
        "numeroPedido": "PED-API-10001",
        "status": "OK"
      }
    ]
  },
  "error": null
}
```

Exemplo de erro (`422 Unprocessable Entity`):

```json
{
  "success": false,
  "message": "Regra de negocio invalida",
  "data": null,
  "error": {
    "code": "BUSINESS_RULE_ERROR",
    "details": [
      {
        "field": "pedido",
        "message": "Pedido nao encontrado: PED-API-10001"
      }
    ],
    "traceId": "ed10228e-e78d-4e6e-bdd7-f4381cfe89fa"
  }
}
```

### 5.9 POST `/pedidos/faturamento`

Vincula faturamento ao pedido.

Campos obrigatorios:
- `identificacao.numeropedido`
- `identificacao.volume` (numero)
- `identificacao.pesoBruto` (numero)

Campos recomendados:
- `identificacao.cgccli`
- `identificacao.pesoLiquido`
- `identificacao.xmlnfe`
- `identificacao.transportadora`

Exemplo:

```json
{
  "pedido": [
    {
      "identificacao": {
        "numeropedido": "PED-API-10001",
        "cgccli": "00398268000123",
        "volume": 1,
        "pesoBruto": 12.3,
        "pesoLiquido": 11.8,
        "xmlnfe": "<nfe>xml-faturamento</nfe>",
        "transportadora": {
          "cnpj": "11259442000173",
          "nome": "TRANSPORTADORA TESTE"
        }
      }
    }
  ]
}
```

Exemplo de retorno (`200 OK`):

```json
{
  "success": true,
  "message": "Faturamento vinculado ao pedido",
  "data": {
    "resultados": [
      {
        "numeroPedido": "PED-API-10001",
        "status": "OK"
      }
    ]
  },
  "error": null
}
```

Exemplo de erro (`422 Unprocessable Entity`):

```json
{
  "success": false,
  "message": "Regra de negocio invalida",
  "data": null,
  "error": {
    "code": "BUSINESS_RULE_ERROR",
    "details": [
      {
        "field": "identificacao.volume",
        "message": "Volume divergente para pedido=PED-API-10001: payload=1 sistema=2"
      }
    ],
    "traceId": "d61e5e3e-ab17-4ca0-89dd-59c934089570"
  }
}
```

Regras importantes:
- Precisa existir separacao concluida para o pedido.
- `volume` e `pesoBruto` do payload devem bater com os totais do sistema.
- Se `pesoLiquido` for enviado, tambem precisa bater.
- Em schema sem coluna `PEDIDO` em `TABPRETAREFA`, este endpoint retorna `422`.

## 6. Fluxo recomendado de consumo

Fluxo de saida (pedido):
1. `POST /pedidos`
2. `POST /pedidos/separacao`
3. `POST /pedidos/faturamento`

Fluxo de entrada:
1. `POST /entradas`
2. (opcional) `POST /entradas/itens`
3. (opcional) `PATCH /entradas/status`
4. (opcional) `GET /entradas`
