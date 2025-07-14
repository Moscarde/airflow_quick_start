# 🚀 Etapa 1 – Quick Start com Apache Airflow

> Baseado no guia oficial: https://airflow.apache.org/docs/apache-airflow/stable/start.html

## 🎯 Objetivo

Iniciar um ambiente Airflow local e executar DAGs de exemplo usando a CLI, com foco em:

- Estrutura básica de projeto
- Conceitos fundamentais (DAG, Task, Scheduler, Executor, DagRun)
- Execução manual via `backfill` e observação via UI

---

## 📁 Estrutura Inicial

```bash
airflow_quick_start/
├── dags/                  # Diretório padrão de DAGs
├── docs/                  # Documentação das etapas
├── .venv/                 # Ambiente virtual (não versionado)
└── README.md              # Visão geral do projeto
