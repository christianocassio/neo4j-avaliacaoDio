<p align="center">
  <img src="documentation/img/bloom-icon-grey.png" width="90" alt="Ícone do Neo4j Bloom">
</p>

<h1 align="center">Neo4j – Avaliação DIO</h1>

<p align="center">
  Exploração visual de grafos com <strong>Neo4j Bloom</strong>, usando uma base de detecção de fraudes em compras online
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Neo4j-2026.01-018BFF?logo=neo4j&logoColor=white" alt="Neo4j">
  <img src="https://img.shields.io/badge/Cypher-query-008CC1" alt="Cypher">
  <img src="https://img.shields.io/badge/DIO-desafio-E4105D" alt="DIO">
</p>

---

## 📌 Sobre o projeto

Este repositório faz parte do desafio de projeto da [DIO](https://www.dio.me/) sobre **bancos de dados em grafo com Neo4j**.

Ele parte do exemplo oficial [`neo4j-graph-examples/bloom`](https://github.com/neo4j-graph-examples/bloom). A base simula compras em lojas virtuais e serve para investigar **padrões suspeitos de fraude**, como:

- vários cartões usados a partir do mesmo IP;
- um mesmo login comprando em várias lojas;
- endereços de entrega compartilhados entre compras sem relação.

| Métrica | Quantidade |
|---|---|
| Nós | **30.960** |
| Relacionamentos | **29.731** |

---

## 🧩 Modelo de dados

![Modelo do grafo](documentation/img/model.png)

O nó central é a **`Purchase`** (compra). Ele se liga a:

| Nó | Significado | Relacionamento |
|---|---|---|
| `Login` | conta que fez a compra | `(:Purchase)-[:WITH_LOGIN]->(:Login)` |
| `Shop` | loja onde o login comprou | `(:Login)-[:FOR_SHOP]->(:Shop)` |
| `IP` | endereço IP de origem | `(:Purchase)-[:FROM_IP]->(:IP)` |
| `State` | estado onde o IP está | `(:IP)-[:LOCATED_IN]->(:State)` |
| `BankCard` | cartão usado no pagamento | `(:Purchase)-[:WITH_CARD]->(:BankCard)` |
| `DeliveryAddress` | endereço de entrega | relacionamento de entrega (veja a imagem acima) |

### Exemplo visual no Bloom

![Exemplo no Bloom](documentation/img/example.png)

---

## 🚀 Como rodar

### 1. Pré-requisitos

- [Neo4j Desktop](https://neo4j.com/download/) **ou** uma instância no [Neo4j Aura](https://console.neo4j.io/)
- Neo4j **5.x** ou **2026.01** (também tem dumps para 3.5, 4.0 e 4.3)

### 2. Carregar a base

Os dumps ficam na pasta [`data/`](data/):

| Arquivo | Versão do Neo4j |
|---|---|
| `bloom-2026.01.dump` | 2026.01 |
| `bloom-50.dump` | 5.x |
| `bloom-43.dump` | 4.3 |
| `bloom-40.dump` | 4.0 |
| `bloom-35.dump` | 3.5 |

**Opção A – Neo4j Desktop**
1. Arraste o `.dump` para a seção **Files** do projeto.
2. Nas opções do arquivo, escolha **Create new DBMS from dump**.

**Opção B – Linha de comando**

```shell
# Neo4j 5.x / 2026.x (o arquivo precisa se chamar <nome-do-banco>.dump)
mkdir -p /tmp/dump && cp data/bloom-2026.01.dump /tmp/dump/neo4j.dump
bin/neo4j-admin database load --from-path=/tmp/dump neo4j --overwrite-destination

# Neo4j 4.x
bin/neo4j-admin load --from=data/bloom-43.dump --database=neo4j --force
```

**Opção C – Neo4j Aura**
Faça o upload do dump pela tela de importação do [console do Aura](https://console.neo4j.io/#import-instructions).

### 3. Importar a perspectiva do Bloom

No Neo4j Bloom, importe um dos arquivos da pasta [`bloom/`](bloom/):

- `bloom-fraud-data.bloom-perspective.json`
- `bloom-fraud-detection.bloom-perspective.json`

---

## 🔎 Consultas Cypher de exemplo

**Valores das compras feitas a partir de um IP**

```cypher
MATCH (i:IP {ip: '168.166.144.243'})-[:FROM_IP]-(p:Purchase)
RETURN p.amount AS valor
```

**IPs usados com mais de um cartão (possível fraude)**

```cypher
MATCH (i:IP)<-[:FROM_IP]-(:Purchase)-[:WITH_CARD]->(c:BankCard)
WITH i, count(DISTINCT c) AS cartoes
WHERE cartoes > 1
RETURN i.ip AS ip, cartoes
ORDER BY cartoes DESC
LIMIT 10
```

**Compras por estado**

```cypher
MATCH (p:Purchase)-[:FROM_IP]->(:IP)-[:LOCATED_IN]->(s:State)
RETURN s AS estado, count(p) AS compras
ORDER BY compras DESC
```

**Ver o modelo do banco**

```cypher
CALL db.schema.visualization()
```

---

## 💻 Exemplos de código

Conexão com o banco e execução de consultas nas linguagens abaixo:

| Linguagem | Arquivo |
|---|---|
| Python | [`code/python/example.py`](code/python/example.py) |
| JavaScript | [`code/javascript/example.js`](code/javascript/example.js) |
| Java | [`code/java/Example.java`](code/java/Example.java) |
| C# | [`code/csharp/Example.cs`](code/csharp/Example.cs) |
| Go | [`code/go/example.go`](code/go/example.go) |

---

## 📁 Estrutura

```
.
├── bloom/            # Perspectivas do Neo4j Bloom (.json)
├── code/             # Exemplos de conexão em 5 linguagens
├── data/             # Dumps do banco por versão do Neo4j
├── documentation/    # Guia do Neo4j Browser, imagens e README original
└── README.md
```

---

## 🙏 Créditos

- Base e exemplo original: [neo4j-graph-examples/bloom](https://github.com/neo4j-graph-examples/bloom), por William Lyon e equipe Neo4j. O README original em inglês está em [`documentation/README-original.adoc`](documentation/README-original.adoc).
- Adaptação para o desafio DIO: **[Christiano Miranda](https://github.com/christianocassio)**
