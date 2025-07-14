# Building Your First Workflow with Apache Airflow

> Baseado no guia oficial: https://airflow.apache.org/docs/apache-airflow/stable/tutorial/fundamentals.html

## Executando uma Task Isoladamente via Linha de Comando

Durante o desenvolvimento e teste de workflows no Apache Airflow, é comum a necessidade de executar uma task específica isoladamente para validar sua lógica sem disparar toda a DAG.

### Comando `airflow tasks test`

O Airflow fornece o comando `airflow tasks test` para rodar uma task isolada, sem alterar o estado da DAG ou dependências.

**Sintaxe:**

```bash
airflow tasks test <dag_id> <task_id> <execution_date>
```

-   `<dag_id>`: Identificador da DAG.

-   `<task_id>`: Nome da task a ser executada.

-   `<execution_date>`: Data da execução no formato `YYYY-MM-DD`.


### Exemplo Prático

No tutorial padrão, para executar a task `print_date` da DAG `tutorial` para a data `2015-06-01`:

```python
import textwrap
from datetime import datetime, timedelta
from airflow.providers.standard.operators.bash import BashOperator
from airflow.sdk import DAG

with DAG(
    "tutorial",
    default_args={
        "depends_on_past": False,
        "retries": 1,
        "retry_delay": timedelta(minutes=5),
    },
    description="A simple tutorial DAG",
    schedule=timedelta(days=1),
    start_date=datetime(2021, 1, 1),
    catchup=False,
    tags=["example"],
) as dag:

    t1 = BashOperator(
        task_id="print_date",
        bash_command="date",
    )

    t2 = BashOperator(
        task_id="sleep",
        depends_on_past=False,
        bash_command="sleep 5",
        retries=3,
    )

    templated_command = textwrap.dedent("""
    {% for i in range(5) %}
        echo "{{ ds }}"
        echo "{{ macros.ds_add(ds, 7) }}"
    {% endfor %}
    """)

    t3 = BashOperator(
        task_id="templated",
        depends_on_past=False,
        bash_command=templated_command,
    )

    t1 >> [t2, t3]
```

Entendendo o Arquivo de Definição do DAG
----------------------------------------

Este script Python é uma configuração que define a estrutura do DAG no Airflow. O objetivo principal não é processar dados, mas definir os objetos DAG e tarefas rapidamente para que o Airflow possa verificar mudanças com frequência.

### Importações

-   Importamos bibliotecas padrão do Python (`datetime`, `timedelta`, `textwrap`).

-   Importamos operadores Airflow, como o `BashOperator`, que executa comandos bash.

-   Importamos o objeto `DAG` para instanciar nosso workflow.

### Argumentos Padrão (`default_args`)

Um dicionário que define parâmetros padrão para todas as tasks, como:

-   `depends_on_past`: se a tarefa depende da execução anterior.

-   `retries`: número de tentativas em caso de falha.

-   `retry_delay`: tempo entre tentativas.

Estes valores podem ser sobrescritos para cada tarefa individualmente.

* * * * *

Criando o DAG
-------------

-   `dag_id`: identificador único da DAG.

-   `description`: texto descritivo.

-   `schedule`: intervalo de execução (ex.: `timedelta(days=1)` para execução diária).

-   `start_date`: data de início da execução.

-   `catchup`: controla se execuções passadas devem ser "recuperadas".

-   `tags`: etiquetas para categorizar DAGs.

* * * * *

Entendendo os Operadores
------------------------

Operadores representam unidades de trabalho. O `BashOperator` executa comandos shell. Outros operadores incluem PythonOperator, KubernetesPodOperator, etc. Todos herdam do `BaseOperator`, que define argumentos comuns.

* * * * *

Definindo Tarefas
-----------------

-   Cada tarefa é uma instância de um operador com um `task_id` único.

-   Parâmetros podem ser específicos ou herdados de `default_args`.

-   Exemplo: `print_date` imprime a data atual; `sleep` pausa 5 segundos.

* * * * *

Uso de Jinja Templating
-----------------------

Airflow suporta Jinja para templating, permitindo construir comandos dinâmicos com variáveis e macros.

Exemplo:

```python
templated_command = """
{% for i in range(5) %}
    echo "{{ ds }}"
    echo "{{ macros.ds_add(ds, 7) }}"
{% endfor %}
"""

```

Aqui, `{{ ds }}` é a data de execução, e `macros.ds_add` adiciona dias a ela. Isso permite reutilização e dinamismo nas tarefas.

* * * * *

Documentação em DAGs e Tarefas
------------------------------

Você pode documentar DAGs e tarefas usando atributos como:

-   `doc_md` para markdown (renderizado na UI).

-   `doc`, `doc_rst`, `doc_json`, `doc_yaml` para outros formatos.

Exemplo de documentação em uma task:

```python
t1.doc_md = """
#### Task Documentation
You can document your task using markdown attributes, visible in the Airflow UI.
"""

```

E para o DAG:
```python
dag.doc_md = __doc__  # ou uma string com documentação

```

## Definindo Dependências
----------------------

Você pode definir dependências de execução entre tarefas para controlar a ordem.

Formas equivalentes:

```python
t1.set_downstream(t2)
t2.set_upstream(t1)
t1 >> t2
t2 << t1
t1 >> [t2, t3]

```
É importante evitar ciclos, que causam erro no DAG.

* * * * *

Trabalhando com Time Zones
--------------------------

Use datas com timezone-aware, preferencialmente com a biblioteca `pendulum`, para evitar problemas de horários, principalmente em ambientes distribuídos.

* * * * *

Testando seu Pipeline
---------------------

### Validando o Script Python

Execute:

```bash
python ~/airflow/dags/tutorial.py
```

## Comandos Úteis no Terminal

- Inicializar o banco: `airflow db migrate`
- Listar DAGs: `airflow dags list`
- Listar tarefas de uma DAG: `airflow tasks list tutorial`
- Mostrar o gráfico da DAG: `airflow dags show tutorial`

Testando Instâncias de Tarefas

Você pode executar tarefas isoladamente para uma data lógica específica (logical date), que representa a data do agendamento.

```bash
airflow tasks test tutorial print_date 2015-06-01
airflow tasks test tutorial sleep 2015-06-01
airflow tasks test tutorial templated 2015-06-01
```
Esse comando executa localmente, mostra logs no terminal, mas não altera o estado no banco de dados do Airflow, ideal para depuração rápida.

Recapitulação
-------------

-   Criar um DAG Python definindo tarefas e suas dependências.

-   Utilizar operadores para implementar unidades de trabalho.

-   Aplicar templating com Jinja para comandos dinâmicos.

-   Documentar DAGs e tarefas para melhor manutenção.

-   Controlar execução com dependências claras.

-   Testar tarefas e DAGs localmente via linha de comando.

* * * * *