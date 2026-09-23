# MVP — Pontos de Cultura nos municípios do Rio de Janeiro

Projeto desenvolvido para o MVP da disciplina de Banco de Dados da Pós-Graduação em Ciência de Dados e Analytics da PUC-Rio.

## 1. Objetivo

O projeto busca responder à seguinte pergunta:

> Como os registros de Pontos de Cultura presentes na base da Rede Cultura Viva se distribuem entre os municípios do estado do Rio de Janeiro?

A proposta aproxima engenharia de dados e políticas culturais, organizando uma base pública para produzir uma visão territorial simples, rastreável e adequada para consulta.

## 2. Fonte dos dados

- **Fonte principal:** Ministério da Cultura — [Pontos de Cultura](https://dados.cultura.gov.br/dataset/pontos-de-cultura)
- **Arquivo de origem:** `pontosdecultura-redeculturaviva.csv`
- **Referência territorial:** [API de Localidades do IBGE](https://servicodados.ibge.gov.br/api/docs/localidades)
- **Data da consulta:** 22/09/2026
- **Ambiente de processamento:** Databricks Free Edition, com PySpark e Delta Lake

O arquivo original contém dados pessoais, como nome, CPF e contato. Por esse motivo, ele **não foi publicado neste repositório**. Para o processamento foi utilizada uma cópia reduzida, contendo apenas os campos necessários à análise territorial: identificador do registro, estado, município e tipo de agente.

## 3. Arquitetura da solução

Foi adotada a arquitetura em camadas **Bronze, Silver e Gold**:

| Camada | Finalidade |
| --- | --- |
| Bronze | Preservar os valores territoriais recebidos no arquivo, sem alterar seu conteúdo. |
| Silver | Selecionar os registros do RJ, padronizar nomes de localidades e associá-los aos códigos oficiais do IBGE. |
| Gold | Agrupar os registros validados por município, criando uma tabela pronta para consulta e visualização. |

### Fluxo do pipeline

1. O CSV reduzido foi enviado a um Volume do Unity Catalog.
2. A leitura considerou separador `;` e codificação `utf-8-sig`.
3. A camada Bronze recebeu 7.257 registros; 789 estavam identificados com o estado RJ.
4. A camada Silver verificou e padronizou os nomes municipais com base na referência oficial do IBGE.
5. A camada Gold reuniu 787 registros territoriais validados em 76 municípios.
6. Uma consulta SQL ordenou os municípios pela quantidade de registros e gerou uma visualização em barras.

## 4. Modelagem e catálogo

As tabelas foram persistidas em formato Delta no catálogo `workspace`, esquema `default`:

- `workspace.default.bronze_pontos_cultura`
- `workspace.default.silver_pontos_rj`
- `workspace.default.gold_pontos_por_municipio`

A tabela Gold possui uma linha por município validado:

| Campo | Descrição |
| --- | --- |
| `codigo_ibge` | Código oficial do município no IBGE. |
| `municipio_ibge` | Nome oficial do município. |
| `quantidade_pontos` | Quantidade de registros de Pontos de Cultura associados ao município. |

## 5. Tratamento e qualidade dos dados

Na camada Silver, cada registro recebeu uma regra que explica o tratamento aplicado:

| Regra | Registros |
| --- | ---: |
| Nome municipal já oficial | 622 |
| Padronização de grafia | 149 |
| Distrito associado ao município correspondente | 9 |
| Correção de grafia | 7 |
| Localização conflitante | 2 |

Os dois registros conflitantes indicavam `Guarulhos (SP)` junto da UF `RJ`. Para evitar uma atribuição territorial incorreta, eles permaneceram identificados como conflito e não foram incluídos na agregação Gold.

## 6. Resultados

Foram validados **787 registros**, distribuídos por **76 municípios**. Os dez municípios com as maiores contagens no arquivo são:

| Posição | Município | Registros |
| ---: | --- | ---: |
| 1 | Rio de Janeiro | 358 |
| 2 | Niterói | 37 |
| 3 | Duque de Caxias | 34 |
| 4 | Nova Iguaçu | 33 |
| 5 | São Gonçalo | 23 |
| 6 | Petrópolis | 20 |
| 7 | Belford Roxo | 16 |
| 8 | São João de Meriti | 16 |
| 9 | Nova Friburgo | 13 |
| 10 | Volta Redonda | 13 |

O resultado mostra forte concentração dos registros na capital. Entretanto, essa diferença deve ser interpretada como uma característica do arquivo analisado, e não como prova isolada da oferta cultural real ou atual de cada município.

## 7. Consulta SQL

```sql
SELECT
  municipio_ibge,
  quantidade_pontos
FROM workspace.default.gold_pontos_por_municipio
ORDER BY quantidade_pontos DESC, municipio_ibge
LIMIT 10;
```

## 8. Limitações

- A data de criação do cadastro não representa necessariamente o início das atividades culturais.
- A ausência de um município no arquivo não comprova ausência de Pontos de Cultura no território.
- As contagens descrevem a versão consultada da base e não necessariamente a rede atual.
- O projeto não analisa população, alcance, financiamento, situação de atividade ou impacto dos Pontos de Cultura.
- Não foram criadas linhas com valor zero para municípios ausentes, pois isso produziria uma conclusão não sustentada pela fonte.

## 9. Reprodutibilidade

O notebook `MVP_Pontos_Cultura_RJ_Databricks 2026-09-23 12_05_59.ipynb` contém:

- explicação das etapas;
- criação das camadas Bronze, Silver e Gold;
- verificações de qualidade;
- persistência das tabelas Delta;
- resultados da execução.

Para reproduzir o pipeline, é necessário obter a base no portal do Ministério da Cultura, gerar uma cópia somente com os quatro campos territoriais usados e atualizar no notebook o caminho do arquivo armazenado no Volume do Databricks.

## 10. Autoavaliação

O MVP cumpriu o objetivo de construir um pipeline completo em nuvem, documentando a carga, o tratamento, a validação e a disponibilização dos dados para consulta. Como principal aprendizado, o projeto mostrou que a padronização territorial não deve ser uma substituição automática: conflitos precisam permanecer explícitos para que a análise não crie informações falsas.

Como evolução futura, seria relevante acrescentar indicadores populacionais do IBGE, situação atual dos Pontos de Cultura e recortes regionais. Isso permitiria comparar números absolutos com medidas proporcionais e aprofundar a discussão sobre concentração e acesso territorial à cultura.

## Autor

Caíque Cahon Monteiro  
Pós-Graduação em Ciência de Dados e Analytics — PUC-Rio
