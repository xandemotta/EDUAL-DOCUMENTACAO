# Documentacao de Consumo da API

## Base URL
- Local (PM2): `http://mmsistemas.ddns.net:3101`

## Formato padrao
- `Content-Type`: `application/json`
- Autenticacao: nao possui

## Healthcheck
### `GET /`
Retorna status da API.

Exemplo de resposta:
```json
{
  "status": "ok"
}
```

## Endpoint de Produto
### `POST /produto`
Recebe dados de produto, grava no banco e retorna o JSON normalizado.

Campos obrigatorios:
- `NOMPRO` ou `nomeProduto` (string)
- `DESCRICAO` ou `descricaoProduto` (string)
- `NOMUN` ou `unidadeMedida` (string)

Campos opcionais:
- `BARPRO` ou `gtin` (string)
- `CODCLI` ou `skuCliente` (string)
- `NONCM` ou `ncm` (string)
- `ALT` ou `altura` (numero >= 0)
- `LARG` ou `largura` (numero >= 0)
- `PROF` ou `profundidade` (numero >= 0)
- `PESOLIQ` ou `pesoLiquido` (numero >= 0)
- `PESOBR` ou `pesoBruto` (numero >= 0)
- `M2` ou `m2` (numero >= 0)
- `M3` ou `m3` (numero >= 0)

Regra importante:
- Nao enviar os dois nomes do mesmo campo (exemplo: `NOMPRO` e `nomeProduto` juntos).

Exemplo de requisicao:
```json
{
  "nomeProduto": "Camisa Dry Fit",
  "descricaoProduto": "Camisa esportiva manga curta",
  "unidadeMedida": "UN",
  "pesoBruto": 0.3,
  "gtin": "7891234567890",
  "altura": 2,
  "largura": 30,
  "profundidade": 25
}
```

Exemplo de resposta:
```json
{
  "NOMPRO": "Camisa Dry Fit",
  "DESCRICAO": "Camisa esportiva manga curta",
  "NOMUN": "UN",
  "BARPRO": "7891234567890",
  "ALT": 2,
  "LARG": 30,
  "PROF": 25,
  "PESOBR": 0.3
}
```

## Endpoint de Entrada de Nota
### `POST /tarefa-entrada`
Recebe uma lista de notas, grava cabecalho e itens no banco e retorna JSON normalizado.

Estrutura obrigatoria:
- `notas` (array com pelo menos 1 item)

Campos obrigatorios por nota:
- `identificacao.numeroNfe` (string)
- `identificacao.serie` (string)
- `identificacao.chaveNfe` (string)
- `identificacao.cnpjcliente` (string)
- `identificacao.nomecliente` (string)
- `identificacao.tipomovimentacao` (string)
- `identificacao.destinatario.cnpj` (string)
- `identificacao.destinatario.nome` (string)
- `identificacao.destinatario.ie` (string)
- `identificacao.destinatario.endereco` (string)
- `identificacao.destinatario.ibgecidade` (numero >= 0)
- `identificacao.destinatario.cep` (numero >= 0)
- `itens` (array com pelo menos 1 item)
- `totais.valorProdutos` (numero >= 0)
- `totais.pesoBrutoTotal` (numero >= 0)
- `totais.pesoLiquidoTotal` (numero >= 0)
- `xml.nfe` (string)

Campos obrigatorios por item:
- `skuProduto` (string)
- `descricaoProduto` (string)
- `quantidade` (numero >= 0)
- `valorUnitario` (numero >= 0)
- `valorTotal` (numero >= 0)
- `unidadeMedida` (string)
- `pesoBruto` (numero >= 0)

Campos opcionais por item:
- `gtin` (string)
- `pesoLiquido` (numero >= 0)
- `dimensoes.largura` (numero >= 0)
- `dimensoes.altura` (numero >= 0)
- `dimensoes.profundidade` (numero >= 0)

Exemplo de requisicao:
```json
{
  "notas": [
    {
      "identificacao": {
        "numeroNfe": "12345",
        "serie": "1",
        "chaveNfe": "41260100000000000000550010000123451000012345",
        "cnpjcliente": "12345678000190",
        "nomecliente": "Cliente Exemplo LTDA",
        "tipomovimentacao": "ENTRADA",
        "destinatario": {
          "cnpj": "12345678000190",
          "nome": "Cliente Exemplo LTDA",
          "ie": "1234567890",
          "endereco": "Rua Exemplo, 100",
          "ibgecidade": 4106902,
          "cep": 80000000
        }
      },
      "itens": [
        {
          "skuProduto": "SKU-001",
          "descricaoProduto": "Produto Exemplo",
          "quantidade": 2,
          "valorUnitario": 50,
          "valorTotal": 100,
          "unidadeMedida": "UN",
          "pesoBruto": 1.2,
          "gtin": "7891234567890",
          "dimensoes": {
            "largura": 10,
            "altura": 20,
            "profundidade": 5
          }
        }
      ],
      "totais": {
        "valorProdutos": 100,
        "pesoBrutoTotal": 1.2,
        "pesoLiquidoTotal": 1
      },
      "xml": {
        "nfe": "<xml>...</xml>"
      }
    }
  ]
}
```

Exemplo de resposta:
```json
{
  "TABPRETAREFA": {
    "CGCCLI": "12345678000190",
    "NOMCLI": "Cliente Exemplo LTDA",
    "NOCID_IBGE": 4106902,
    "CEPCLI": 80000000,
    "ENDCLI": "Rua Exemplo, 100",
    "REFCLI": "12345",
    "USUARIO": "SISTEMA",
    "STATUS": "PENDENTE"
  },
  "TABPRETAREFA_ITEM": [
    {
      "NOMPRO": "Produto Exemplo",
      "CODCLI": "SKU-001",
      "QTD": 2,
      "VLRUNIT": 50,
      "TOTAL": 100
    }
  ]
}
```

## Erros
### `400 Payload invalido`
Quando faltar campo obrigatorio, tipo invalido ou valor numerico negativo.

Formato:
```json
{
  "message": "Payload invalido.",
  "errors": [
    "descricao do erro"
  ]
}
```

### `500 Erro ao inserir no banco`
Erro interno ao gravar no banco.

Formato:
```json
{
  "message": "Erro ao inserir no banco.",
  "error": "mensagem tecnica"
}
```
