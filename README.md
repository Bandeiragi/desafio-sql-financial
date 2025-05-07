# Desafio Técnico de SQL - Análise de Dados Financeiros

## ✅ Etapa 1 – Configuração do Ambiente

### 🔧 Dataset criado
- **Nome:** `case_financial_record`

### 📁 Tabelas importadas
As seguintes tabelas foram importadas para o BigQuery a partir dos arquivos fornecidos:

- `amount`
- `company`
- `financial_record`
- `coa`

### 📌 Observações
- Todas as tabelas foram importadas a partir dos arquivos disponíveis no Google Drive.
- O arquivo coa.csv (sobre índices e níveis de classificação) estava disponível apenas em formato CSV, impossibilitando o download e também não sendo possível abrir com o Planilhas Google, o que estava impedindo a importação no BigQuery.
- Solicitei a versão do arquivo em Google Sheets à Tech Lead e consegui subir a tabela no BigQuery.
- Nenhuma transformação, limpeza de dados ou junção entre tabelas foi realizada nesta etapa.
- O objetivo desta fase foi apenas a **configuração do ambiente no BigQuery**, conforme instruções.
- O print do ambiente está com a data do dia 07/05 pois renomeei o nome das tabelas para ficar de acordo com o arquivo original.

### Prints das tabelas no BigQuery:
![Captura de Tela 2025-05-07 às 11 06 28](https://github.com/user-attachments/assets/417f4847-2a99-4f1e-90fb-0d31e608a3b9)


---

## ✅ Etapa 2 – Consultas SQL

### Questão 1 
###  Qual é o total de registros financeiros sem data de pagamento?
SELECT
  COUNT (id) AS total_registros_financeiros
FROM 
  `case_financial_record.financial_record` fr
WHERE
fr.paid_at IS NULL

### Print Resultado Questão 1
![Captura de Tela 2025-05-06 às 17 06 00](https://github.com/user-attachments/assets/470740c1-caa3-47a3-b2e1-4d51abafc25a)

### 📌 Observações questão 1
- Nessa querie fiz a contagem dos ids vindos da tabela financial_record onde a data de pagamento fosse = a Nula

### ✅ Resposta Questão 1
- O total de registros financeiros sem data de pagamento é de 168

### Questão 2 
### Quais são os 5 maiores valores de registro financeiro pagos por metodo de pagamento?
SELECT
  payment_method AS Metodo_Pagamento,
  payment_method_name AS Nome_Metodo_Pagamento,
  value AS valor,
  COUNT (id) AS total_registros_financeiros
FROM 
  `case_financial_record.financial_record` fr
GROUP BY
payment_method, payment_method_name, value
ORDER BY
fr.value DESC

### Print Resultado Questão 2
![Captura de Tela 2025-05-06 às 17 09 49](https://github.com/user-attachments/assets/80a70882-489b-4a02-8926-43ca0dcddb11)

### 📌 Observações questão 2
- Nessa querie selecionei o método de pagamento, o nome do método, os valores e fiz uma contagem dos id para entender o valor total dos registros.
- Peguei as informações da tabela financial_record
- agrupei as colunas de método de pagamento, o nome do método e valor
- e ordenei por ordem decrescente para visualizar as 5 maiores valores de registro

### ✅ Resposta Questão 2
- O 5 maiores valores de registros financeiros por método de pagamento são:
Boleto = R$2.894.945,0
PIX = R$1.000.613,0
PIX R$809.865,0
Depósito = R$614.736,0
Boleto = R$599.850,0

### Questão 3
### Qual é a porcentagem de registros financeiros realizados por ano?
SELECT
  EXTRACT(YEAR FROM created_at) AS ano,
  COUNT(*) AS total_registros,
  ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER(), 2) AS porcentagem
FROM
  `case_financial_record.financial_record`
GROUP BY
  ano
ORDER BY
  ano;

### Print Resultado Questão 3
![Captura de Tela 2025-05-07 às 10 51 07](https://github.com/user-attachments/assets/3facd8e9-0c75-44be-a61a-d13f2b896a06)


### 📌 Observações questão 3
- Extrai na coluna created_at apenas o ano
- fiz uma contagem do total de registros
- fiz uma soma de todas as contagem de grupo
- dividi o número de grupo pelo total
- multipliquei por 100 para virar percentual
- e arredondei para duas casas decimais
- peguei os dados da tabela financial_record
- agrupei por ano e ordenei também por ano 

### ✅ Resposta Questão 3
A porcentagem de registros financeiro realizados por ano é de:
2023 = 2,93%, com um total de registros de 44
2024 = 76,53%, com um total de registros de 1148
2025 = 20,53%, com um total de registros de 308


### Questão 4
### Qual empresas tem lançamentos de seguro e quais são as suas classificações?
SELECT DISTINCT
  c.fantasy_name AS empresa,
  co.coa_account_name AS classificacao_contabil,
  fr.name AS tipo_lancamento
FROM
  `case_financial_record.financial_record` fr
JOIN `case_financial_record.amount` a ON fr.id = a.financial_record_id
JOIN `case_financial_record.company` c ON a.company_id = c.id
LEFT JOIN `case_financial_record.coa` co ON a.id = co.amount_id
WHERE
  fr.name LIKE '%seguro%'
ORDER BY
  c.fantasy_name, co.coa_account_name;

### Print Resultado Questão 5
![Captura de Tela 2025-05-07 às 10 41 48](https://github.com/user-attachments/assets/6dec671f-e1b5-471b-bb3c-b2a55d2d305f)

### 📌 Observações questão 4
- Iniciei a querie com uma seleção distinta do nome fantasia, da classificação e do tipo do lançamento
- tive que fazer as tabelas se relacionarem pelos id para conseguir trazer as informações das classificações
- onde usei como filtro o que continha a nomenclatura 'seguro'e oredenei pelo nome fantasia e pela classificação

### ✅ Resposta Questão 4
As empresas que tem lançamento de seguro, com suas classificações são:
Empresa 55, classificação RECEITA e lançamento de Seguro
Empresa 75, classificação RECEITA e lançamento de seguro


### Questão 5
### Qual é o valor total e a classificação dos lançamentos de conta de luz para empresas do plano de contas do tipo SERVICO?
SELECT
  c.fantasy_name AS empresa,
  co.coa_account_name AS classificacao_lancamento,
  SUM(a.value) AS valor_total_luz,
  COUNT(a.id) AS quantidade_lancamentos
FROM
  `case_financial_record.financial_record` fr
JOIN `case_financial_record.amount` a ON fr.id = a.financial_record_id
JOIN `case_financial_record.company` c ON a.company_id = c.id
JOIN `case_financial_record.coa` co ON a.id = co.amount_id
WHERE
  c.coa_name LIKE '%SERVICO%'  
  AND (fr.name LIKE '%luz%')
GROUP BY
  c.fantasy_name, co.coa_account_name

### Print Resultado Questão 5
![Captura de Tela 2025-05-07 às 10 10 55](https://github.com/user-attachments/assets/cfd17240-8f68-46c6-82d1-7160e4af04f0)

### 📌 Observações
- Rodei essa querie simples onde trouxe a soma dos valores e a contagem dos lançamentos, fazendo uma junção das tabelas pelos Ids correspondentes de cada uma onde tive o filtro do tipo do plano de contas SERVICO e tivesse também a classificação dos lançamentos contendo a palavra luz.
- O resultado da querie não devolveu nenhuma resposta, então realizei outro teste verificando se exite registro que atenda a todas as condições com essa querie:
SELECT
  COUNT(*) AS registros_completos
FROM
  `case_financial_record.financial_record` fr
JOIN `case_financial_record.amount` a ON fr.id = a.financial_record_id
JOIN `case_financial_record.company` c ON a.company_id = c.id
JOIN `case_financial_record.coa` co ON a.id = co.amount_id
WHERE
  c.coa_name LIKE '%SERVICO%'
  AND fr.name LIKE '%luz%';
e o retorno foi = 0. Então concluo que não há registro nas bases com ambas as condições (Plano de Contas tipo SERVICO e Classificação de Contas tipo LUZ). 
### Print do Segundo teste para a Questão 5
![Captura de Tela 2025-05-07 às 10 18 36](https://github.com/user-attachments/assets/4443d799-a9d5-495d-901f-ab993807c7de)

### ✅ Resposta Questão 5
O valor total para empresas do Plano de Contas do tipo Servico e classificação de lançamento de contas de luz é zero. Não há registros nos históricos desses filtros em conjunto.
