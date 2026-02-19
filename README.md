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

Tabela de parametros (`query`):

| Campo | Tipo | Obrig. | Descricao | Exemplo | Validacao/Regras |
|---|---|---|---|---|---|
| `-` | - | Nao | Endpoint sem parametros de consulta | - | Nao se aplica |

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

Tabela de campos (`body`):

| Campo | Tipo | Obrig. | Descricao | Exemplo | Validacao/Regras |
|---|---|---|---|---|---|
| `nompro` | string | Sim | Nome do produto | `Produto Exemplo` | Obrigatorio, `1..120` |
| `descricao` | string | Sim | Descricao completa | `Descricao completa do produto` | Obrigatorio, `1..600` |
| `infadic` | string | Nao | Informacoes adicionais | `Descricao complementar` | Se informado: `1..500` |
| `codcli` | string | Sim | SKU/codigo do cliente (chave de upsert) | `SKU-001` | Obrigatorio, `1..50` |
| `nomun` | string | Sim | Unidade de medida | `UN` | Obrigatorio, `1..10` |
| `barpro` | string | Nao | GTIN/EAN | `7891234567890` | Se enviado: 8/12/13/14 digitos |
| `cobpro` | string | Nao | Flag de cobranca/controle | `F` | Tamanho `1`; default `F` |
| `noncm` | string | Nao | NCM do produto | `12345678` | Sem validacao de tamanho no endpoint |
| `alt` | number | Nao | Altura (cm) | `20` | Se enviado: `>= 0` |
| `larg` | number | Nao | Largura (cm) | `30` | Se enviado: `>= 0` |
| `prof` | number | Nao | Profundidade (cm) | `15` | Se enviado: `>= 0` |
| `pesoliq` | number | Nao | Peso liquido (kg) | `5.0` | Se enviado: `>= 0` |
| `pesobr` | number | Nao | Peso bruto (kg) | `5.2` | Se enviado: `>= 0` |
| `m2` | number | Nao | Area (m2) | `0.6` | Se enviado: `>= 0` |
| `m3` | number | Nao | Cubagem (m3) | `0.009` | Se enviado: `>= 0` |

Exemplo:

```json
{
  "nompro": "Produto Exemplo",
  "barpro": "7891234567890",
  "descricao": "Descricao completa do produto",
  "infadic": "Descricao complementar",
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

Campos separados por tabela/objeto:

Tabela `notas[].identificacao`

| Campo | Tipo | Obrig. | Descricao | Exemplo | Validacao/Regras |
|---|---|---|---|---|---|
| `numeroNfe` | string | Sim | Identificador da nota de entrada | `ENT-POSTMAN-123` | Campo obrigatorio (nao vazio) |
| `serie` | string | Sim | Serie da nota | `1` | Campo obrigatorio (nao vazio) |
| `chaveNfe` | string | Sim | Chave da NF-e | `3519...1234` | Exatamente 44 digitos |
| `cnpjcliente` | string | Sim | CNPJ do cliente | `00398268000123` | CNPJ valido (14 digitos) |
| `nomecliente` | string | Sim | Nome do cliente | `POLIOTTO...` | Campo obrigatorio (nao vazio) |
| `tipomovimentacao` | string | Sim | Tipo da movimentacao | `ENTRADA` | Campo obrigatorio; precisa existir na `TABOPOS` |
| `processo` | string | Nao | Controle interno do cliente | `REF001` | Se informado: `1..100` |
| `container` | string | Nao | Numero do container | `AAAA 123456-7` | Se informado: `1..13` |
| `cnpjDestinatario` | string | Sim | CNPJ do armazem destino | `11259442000173` | CNPJ valido; precisa existir na `TABEMP` |

Tabela `notas[].itens[]`

| Campo | Tipo | Obrig. | Descricao | Exemplo | Validacao/Regras |
|---|---|---|---|---|---|
| `skuProduto` | string | Sim | SKU do produto | `SKU-001` | Campo obrigatorio (nao vazio) |
| `descricaoProduto` | string | Sim | Descricao do item | `Produto Exemplo 1` | Campo obrigatorio (nao vazio) |
| `quantidade` | number | Sim | Quantidade do item | `10` | Deve ser `> 0` |
| `valorUnitario` | number | Sim | Valor unitario | `25.5` | Deve ser `>= 0` |
| `valorTotal` | number | Sim | Total do item | `255` | Deve ser `>= 0` e igual a `quantidade * valorUnitario` |
| `unidadeMedida` | string | Sim | Unidade de medida do item | `UN` | Campo obrigatorio (nao vazio) |
| `gtin` | string | Sim | GTIN/EAN do item | `7891234567890` | 8/12/13/14 digitos |
| `pesoBruto` | number | Sim | Peso bruto do item (kg) | `5.2` | Deve ser `>= 0` |
| `pesoLiquido` | number | Sim | Peso liquido do item (kg) | `5.0` | Deve ser `>= 0` |
| `dimensoes.largura` | number | Sim | Largura do item (cm) | `30` | Deve ser `>= 0` |
| `dimensoes.altura` | number | Sim | Altura do item (cm) | `20` | Deve ser `>= 0` |
| `dimensoes.profundidade` | number | Sim | Profundidade do item (cm) | `15` | Deve ser `>= 0` |

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
        "processo": "REF001",
        "container": "AAAA 123456-7",
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

Tabela de parametros (`query`):

| Campo | Tipo | Obrig. | Descricao | Exemplo | Validacao/Regras |
|---|---|---|---|---|---|
| `numeroNfe` | string | Sim* | Alias de `refcli` | `ENT-POSTMAN-123` | Obrigatório |
| `cgccli` | string | Nao | CNPJ do cliente para filtrar por `NOCLI` | `00398268000123` | Se enviado: CNPJ valido; se nao existir cliente retorna `422` |

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

Tabela `pedidos[].identificacao`:

| Campo | Tipo | Obrig. | Descricao | Exemplo | Validacao/Regras |
|---|---|---|---|---|---|
| `numeropedido` | string | Sim | Numero do pedido | `PED-API-10001` | Campo obrigatorio |
| `cgccli` | string | Sim | CNPJ do cliente | `00398268000123` | CNPJ valido; cliente deve existir |
| `tipomovimentacao` | string | Sim | Tipo da movimentacao | `SAIDA` | Campo obrigatorio; precisa existir na `TABOPOS` |
| `nomecliente` | string | Nao | Nome do cliente | `POLIOTTO...` | Sem validacao de tamanho no endpoint |
| `usuario` | string | Nao | Usuario de origem | `INTEGRACAO` | Opcional |
| `obs` | string | Nao | Observacoes do pedido | `Pedido criado via integracao` | Opcional |
| `destinatario.cnpj` | string | Sim | CNPJ do destinatario | `04700714000163` | CNPJ valido |
| `destinatario.nome` | string | Sim | Nome do destinatario | `APM TERMINALS PORTUARIOS SA` | Campo obrigatorio |
| `destinatario.ie` | string | Nao | Inscricao estadual do destinatario | `12345678` | Opcional |
| `destinatario.endereco` | string | Nao | Endereco do destinatario | `Rua Exemplo 456` | Opcional |
| `destinatario.numero` | string | Nao | Numero do endereco | `456` | Opcional |
| `destinatario.complemento` | string | Nao | Complemento do endereco | `Galpao` | Opcional |
| `destinatario.bairro` | string | Nao | Bairro | `Centro` | Opcional |
| `destinatario.ibgecidade` | number | Nao | Codigo IBGE da cidade | `1200013` | Se enviado, deve mapear em `TABCID` |
| `destinatario.cep` | string | Nao | CEP do destinatario | `88301365` | Opcional |

Tabela `pedidos[].itens[]`:

| Campo | Tipo | Obrig. | Descricao | Exemplo | Validacao/Regras |
|---|---|---|---|---|---|
| `nompro` | string | Sim | Descricao do item | `Produto Exemplo 1` | Campo obrigatorio |
| `codcli` | string | Nao | SKU do item | `SKU-001` | Se enviado: string nao vazia |
| `qtd` | number | Sim | Quantidade | `10` | Deve ser `> 0` |
| `vlrunit` | number | Sim | Valor unitario | `25.5` | Deve ser `>= 0` |
| `total` | number | Sim | Total do item | `255` | Deve ser `>= 0` e igual a `qtd * vlrunit` |

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

Tabela `pedido[].identificacao`:

| Campo | Tipo | Obrig. | Descricao | Exemplo | Validacao/Regras |
|---|---|---|---|---|---|
| `numeropedido` | string | Sim | Numero do pedido a cancelar | `PED-API-10001` | Campo obrigatorio |
| `cgccli` | string | Sim | CNPJ do cliente | `00398268000123` | CNPJ valido; cliente deve existir |
| `motivocancelamento` | string | Nao | Motivo do cancelamento | `Cancelamento solicitado` | Opcional |

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

Consulta status de separacao.

Tabela `pedido[].identificacao`:

| Campo | Tipo | Obrig. | Descricao | Exemplo | Validacao/Regras |
|---|---|---|---|---|---|
| `numeropedido` | string | Sim | Numero do pedido | `PED-API-10001` | Necessario para localizar pedido |
| `cgccli` | string | Sim | CNPJ do cliente | `00398268000123` | Cliente deve existir |

Exemplo:

```json
{
  "pedido": [
    {
      "identificacao": {
        "numeropedido": "PED-API-10001",
        "cgccli": "00398268000123"
      }
    }
  ]
}
```

Retorno inclui totais de separacao no corpo raiz:
- `statusseparacao`
- `noosmov`
- `volume`
- `pesobruto`
- `pesoliquido`
- `totalmercadoria`

Regra de retorno:
- `statusseparacao` retorna `concluido` quando existe separacao concluida; caso contrario retorna `pendente`.

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
  "message": "Status de separacao consultado",
  "data": {
    "resultados": [
      {
        "numeroPedido": "PED-API-10001",
        "status": "OK",
        "statusseparacao": "concluido",
        "noosmov": 98765,
        "volume": 1,
        "pesobruto": 12.3,
        "pesoliquido": 11.8,
        "totalmercadoria": 255
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

Tabela `pedido[].identificacao`:

| Campo | Tipo | Obrig. | Descricao | Exemplo | Validacao/Regras |
|---|---|---|---|---|---|
| `numeropedido` | string | Sim | Numero do pedido | `PED-API-10001` | Campo obrigatorio |
| `cgccli` | string | Sim | CNPJ do cliente | `00398268000123` | Necessario para localizar cliente/pedido |
| `volume` | number | Sim | Volume para conferencia | `1` | Campo obrigatorio; deve bater com separacao |
| `pesoBruto` | number | Sim | Peso bruto para conferencia | `12.3` | Campo obrigatorio; deve bater com separacao |
| `pesoLiquido` | number | Nao | Peso liquido para conferencia | `11.8` | Se enviado, deve bater com separacao |
| `xmlnfe` | string | Nao | XML da NF-e (ou referencia) | `<nfe>xml-faturamento</nfe>` | Opcional |
| `transporte` | string | Nao | Nome/descricao do transporte | `TRANSPORTADORA TESTE` | Opcional |
| `transportadora` | object | Sim | Dados da transportadora | `{ "cnpj": "...", "nome": "..." }` | `cnpj` e `nome` obrigatorios |

Tabela `pedido[].identificacao.transportadora`:

| Campo | Tipo | Obrig. | Descricao | Exemplo | Validacao/Regras |
|---|---|---|---|---|---|
| `cnpj` | string | Sim | CNPJ da transportadora | `11259442000173` | CNPJ valido |
| `nome` | string | Sim | Nome da transportadora | `TRANSPORTADORA TESTE` | Campo obrigatorio |
| `ie` | string | Nao | Inscricao estadual | `12345678` | Opcional |
| `endereco` | string | Nao | Endereco da transportadora | `Rua Transporte, 100` | Opcional |
| `ibgecidade` | number | Nao | Codigo IBGE da cidade | `1200013` | Opcional |
| `cep` | string | Nao | CEP da transportadora | `88301365` | Opcional |

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
