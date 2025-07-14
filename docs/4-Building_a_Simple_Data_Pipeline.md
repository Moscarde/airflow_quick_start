# 3 - Building a Simple Data Pipeline

Neste módulo, construímos um pipeline de dados simples, porém significativo, com Apache Airflow. O objetivo foi criar uma DAG que:

- Baixa um arquivo CSV externo
- Carrega os dados para uma tabela temporária (`employees_temp`)
- Realiza limpeza e *upsert* dos dados na tabela final (`employees`)

## Ferramentas e Operadores Utilizados

- **SQLExecuteQueryOperator**: para executar comandos SQL diretamente no Postgres
- **PostgresHook**: para carregar o CSV via `COPY` e fazer *merge* dos dados via SQL
- **Decorators `@dag` e `@task`** da TaskFlow API

## Etapas do Pipeline

1. **Setup do ambiente com Docker Compose**  
   - O ambiente Airflow foi inicializado via `docker-compose.yaml` fornecido pela documentação oficial.
   - Foram criadas as pastas `./dags`, `./logs` e `./plugins`.
   - A interface foi acessada via http://localhost:8080 (usuário e senha: `airflow`).

2. **Criação da conexão Postgres**  
   Foi criada manualmente na UI do Airflow:
   - `Conn ID`: `tutorial_pg_conn`
   - `Conn Type`: `postgres`
   - `Host`: `postgres`
   - `Database`: `airflow`
   - `Login`: `airflow`
   - `Password`: `airflow`
   - `Port`: `5432`

3. **Criação das tabelas**
   - `employees_temp`: tabela temporária para dados brutos
   - `employees`: tabela final com dados deduplicados

4. **Download e carga de dados**
   - O CSV foi baixado via `requests` e salvo em `/opt/airflow/dags/files/employees.csv`
   - Os dados foram carregados para `employees_temp` com `PostgresHook.copy_expert(...)`

5. **Limpeza e merge dos dados**
   - A função `merge_data()` realizou um `INSERT ... ON CONFLICT DO UPDATE` para atualizar a tabela `employees`.

6. **Montagem da DAG**
   A DAG foi construída utilizando a API TaskFlow, com encadeamento natural dos operadores e tasks:
   ```python
   [create_employees_table, create_employees_temp_table] >> get_data() >> merge_data()
   ```
7.  **Execução no Airflow**

    -   A DAG foi salva como `dags/process_employees.py`

    -   Foi ativada e executada via Airflow UI

    -   O processo pôde ser acompanhado via a visualização em Grade (Grid view)

Código Final
------------

O código completo da DAG pode ser encontrado no repositório:

🔗 [github.com/Moscarde/airflow-docker](https://github.com/Moscarde/airflow-docker)