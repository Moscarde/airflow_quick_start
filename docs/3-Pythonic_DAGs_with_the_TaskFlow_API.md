# Pythonic DAGs com a TaskFlow API

> Baseado no guia oficial: https://airflow.apache.org/docs/apache-airflow/stable/tutorial/taskflow.html

## Visão Geral

A TaskFlow API, introduzida no Airflow 2.0, oferece uma maneira moderna e pythonica de escrever workflows. Em vez de criar DAGs com operadores tradicionais, você escreve funções Python decoradas com `@task` e o Airflow gerencia a criação das tasks, dependências e passagem de dados automaticamente.

---

## Pipeline Exemplo: ETL (Extract → Transform → Load)

- Define-se a DAG usando o decorador `@dag`.
- Cada etapa do pipeline é uma função decorada com `@task`.
- As funções podem retornar valores que são automaticamente passados para as próximas tasks (via XComs, gerenciados internamente).
- Exemplo básico:
  ```python
  @dag(...)
  def tutorial_taskflow_api():
      @task()
      def extract():
          return {"1001": 301.27, "1002": 433.21, "1003": 502.22}

      @task(multiple_outputs=True)
      def transform(order_data: dict):
          total = sum(order_data.values())
          return {"total_order_value": total}

      @task()
      def load(total_order_value: float):
          print(f"Total order value is: {total_order_value:.2f}")

      order_data = extract()
      order_summary = transform(order_data)
      load(order_summary["total_order_value"])
  tutorial_taskflow_api()
```

* * * * *

Definindo a DAG com `@dag`
--------------------------

-   Use o decorador `@dag` para definir a DAG.

-   Não é mais necessário atribuí-la a uma variável global, Airflow detecta automaticamente.

-   Configurações importantes: `schedule`, `start_date`, `catchup`, `tags`.

* * * * *

Definindo Tasks com `@task`
---------------------------

-   Cada task é uma função Python comum decorada.

-   O retorno da função é passado automaticamente para a próxima task (não precisa usar XCom manualmente).

-   `multiple_outputs=True` indica que o retorno é um dict, cujas chaves viram XComs separadas.

-   Parâmetros de funções e retorno facilitam o fluxo de dados entre tasks.

* * * * *

Execução e Dependências
-----------------------

-   As chamadas das funções decoradas configuram automaticamente as dependências entre tasks.

-   Exemplo:

````python
order_data = extract()
order_summary = transform(order_data)
load(order_summary["total_order_value"])

```

* * * * *

Comparação com a abordagem tradicional
--------------------------------------

-   Antes, usava-se `PythonOperator` e manipulação manual de XComs e dependências.

-   Agora, TaskFlow API abstrai isso, tornando o código mais limpo e fácil de manter.

* * * * *

Passagem automática de XComs
----------------------------

-   Valores retornados são armazenados como XComs e visíveis na UI.

-   Ainda é possível usar `xcom_pull` manualmente para operadores tradicionais.

* * * * *

Configuração de Retentativas e Parâmetros
-----------------------------------------

-   Pode configurar `retries` diretamente no decorator:

```python
@task(retries=3)
def my_task(): ...

```

- Permite reutilizar tasks decoradas em vários DAGs com .override() para alterar parâmetros como task_id, retries etc.

* * * * *

Padrões Avançados
-----------------

-   Suporte para múltiplos ambientes de execução:

    -   Virtualenvs dinâmicos

    -   Python externo

    -   Contêineres Docker

    -   Pods Kubernetes

-   Sensores com `@task.sensor` para espera eficiente.

-   Mistura com operadores tradicionais possível.

-   Templating avançado com argumentos Jinja e contexto Airflow.

* * * * *

Controle Condicional de Execução
--------------------------------

-   Use `@task.run_if()` ou `@task.skip_if()` para controlar execução condicional das tasks.