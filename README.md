# UniversalRunner

**A Web-based, extensible platform for running independent Python engines through a common interface.**

> Think of it like a DAW in music: one common platform, with new capabilities added as independent plugins.

[English](#english) | [Português](#português)

---

# English

## Overview

UniversalRunner started from a simple question:

> **Instead of building a new application for every process, why not build a common platform where new capabilities can be added as independent engines?**

UniversalRunner provides the common execution environment and Web interface.

Each engine contains the knowledge and dependencies required for a specific process.

An engine may:

- analyse documents;
- process Excel files;
- validate information;
- generate reports;
- process PDFs;
- integrate external systems;
- perform data analysis;
- incorporate AI capabilities;
- or implement any other Python-based process.

The core platform does not need to understand the business logic behind those engines.

---

## Architecture

```text
UniversalRunner
│
├── Web Interface
│
├── Plugin Discovery
│
├── Schema Renderer
│
├── Environment Manager
│
├── Executor
│
├── Inspection Engine
│
│
└── Engines
    │
    ├── Engine A
    │   ├── plugin.json
    │   ├── schema_input.json
    │   ├── schema_output.json
    │   ├── requirements.txt
    │   ├── engine.py
    │   └── .venv/
    │
    ├── Engine B
    │   └── ...
    │
    └── Engine C
        └── ...
```

The objective is to keep the **Host stable** while allowing engines to evolve independently.

---

## Core principles

### 1. Engines are external to the executable

Engines are stored outside the UniversalRunner executable:

```text
UniversalRunner/
├── UniversalRunner.exe
├── engines/
├── runtime_data/
└── .streamlit/
```

A new engine can therefore be installed by adding its directory to:

```text
engines/
```

The UniversalRunner executable does not need to be rebuilt simply because a new engine is added.

---

### 2. Each engine owns its dependencies

A plugin can declare its Python dependencies through:

```text
requirements.txt
```

Example:

```text
pandas
openpyxl
matplotlib
```

When an engine is used for the first time, UniversalRunner can automatically create:

```text
engine/
└── .venv/
```

and install the required libraries.

Different engines can therefore use different dependency sets without forcing those dependencies into the UniversalRunner Host.

---

### 3. Dependency changes are detected automatically

UniversalRunner calculates a hash of:

```text
requirements.txt
```

and stores the environment state in:

```text
.environment.json
```

If the requirements change, the engine environment can be updated automatically.

This allows an engine to evolve without requiring the entire platform to be rebuilt.

---

## Plugin structure

A typical engine contains:

```text
my_engine/
├── plugin.json
├── schema_input.json
├── schema_output.json
├── requirements.txt
└── engine.py
```

After its first execution, it may also contain:

```text
my_engine/
├── .venv/
└── .environment.json
```

---

## Plugin manifest

`plugin.json` describes the engine.

Example:

```text
{
    "id": "pdf_inspector",
    "name": "PDF Inspector",
    "version": "1.0.0",
    "author": "José Lopes",
    "category": "Documents",

    "description": "Analyses PDF metadata and form fields.",

    "entrypoint": "engine.py",

    "inputSchema": "schema_input.json",
    "outputSchema": "schema_output.json",

    "requirements": "requirements.txt",

    "timeoutSeconds": 300,

    "permissions": []
}
```

---

## Dynamic input schemas

The user interface is generated from:

```text
schema_input.json
```

Supported field types currently include:

- string;
- text;
- integer;
- number;
- boolean;
- date;
- select;
- file;
- dynamic select;
- dynamic multiselect.

Fields may also define optional properties such as:

- required;
- default;
- help;
- placeholder;
- minimum;
- maximum;
- options;
- extensions;
- multiple.

Example:

```text
{
    "name": "document",
    "label": "Document",
    "type": "file",
    "required": true,
    "extensions": ["pdf"],
    "help": "Select the PDF document to analyse."
}
```

---

## Output schemas

Engines return structured outputs instead of interacting directly with the Web interface.

Example:

```text
{
    "success": true,

    "message": "Processing completed successfully.",

    "outputs": [
        {
            "type": "json",
            "label": "Summary",
            "content": {
                "records": 120,
                "errors": 2
            }
        },

        {
            "type": "file",
            "label": "Download report",
            "file": "report.xlsx"
        }
    ]
}
```

Supported output types can include:

- text;
- JSON;
- tables;
- files;
- logs.

The engine produces results.

The Host decides how they are presented.

---

## The `execute()` contract

Every engine exposes:

```text
execute(payload)
```

Example:

```text
def execute(payload):

    parameters = payload.get(
        "parameters",
        {}
    )

    return {
        "success": True,
        "message": "Processing completed.",
        "outputs": []
    }
```

The payload may contain:

```text
executionId
pluginId
parameters
inputFiles
outputDirectory
executionDirectory
```

This keeps the engine independent from the Web interface.

An engine does not need to know anything about Streamlit.

---

## The `inspect()` contract

Some interfaces require information that is not known until another input has been provided.

For example:

```text
Upload Excel
        ↓
Discover columns
        ↓
Populate dropdowns
```

For these scenarios, an engine may optionally expose:

```text
inspect(payload)
```

Example:

```text
def inspect(payload):

    return {
        "success": True,

        "options": {
            "excel_columns": [
                "Name",
                "NIF",
                "Group",
                "Region"
            ]
        },

        "data": {}
    }
```

The interface can then use those options dynamically.

---

## Field dependencies

`inspect()` is not limited to Excel.

It provides a generic foundation for dependent fields such as:

```text
Country
   ↓
Region
   ↓
Municipality
```

or:

```text
Database
   ↓
Table
   ↓
Column
```

or:

```text
Process Type
   ↓
Applicable Options
```

The engine owns the domain knowledge.

UniversalRunner only provides the mechanism used to expose it.

---

## Isolated executions

Each execution receives a unique identifier and its own workspace:

```text
runtime_data/
└── executions/
    └── <execution-id>/
        ├── payload.json
        ├── response.json
        ├── input/
        └── output/
```

Uploaded files are stored under:

```text
input/
```

Generated files are placed under:

```text
output/
```

This keeps individual executions separated.

---

## Process isolation

Engines do not execute directly inside the Web interface process.

The flow is:

```text
Web Interface
      ↓
Executor
      ↓
Engine virtual environment
      ↓
Plugin Runner
      ↓
engine.py
      ↓
execute(payload)
```

This allows the Host to apply timeouts and prevents normal engine execution from being tightly coupled to the Web process.

---

## Web interface

UniversalRunner provides a Web-based user interface.

The current implementation can be used locally while still providing the user with a browser-based experience.

This separates the user experience from the deployment model.

---

## Deployment

The current implementation has been packaged for Windows and can be distributed through an installer.

However, Windows is a deployment choice rather than an architectural requirement.

The same architecture can be adapted to other environments, including Linux.

Because the user experience is Web-based, UniversalRunner can also evolve towards a centrally hosted model:

```text
Users
   ↓
HTTPS
   ↓
Reverse Proxy
   ↓
UniversalRunner
   ↓
Independent Engines
```

This makes both local and centrally managed deployments possible.

---

## Creating a new engine

A new engine normally requires only:

```text
engines/
└── my_engine/
    ├── plugin.json
    ├── schema_input.json
    ├── schema_output.json
    ├── requirements.txt
    └── engine.py
```

The typical development cycle is:

1. Define the plugin manifest.
2. Define the input schema.
3. Define the output schema.
4. Add the required Python dependencies.
5. Implement `execute(payload)`.
6. Optionally implement `inspect(payload)`.
7. Copy the engine into `engines/`.
8. Run it through UniversalRunner.

---

## Project philosophy

The goal of UniversalRunner is not simply to execute Python scripts.

The broader objective is:

> **to create a reusable execution platform where solving the next problem requires adding a capability rather than rebuilding the application.**

Automating one process creates value.

Building reusable capabilities that reduce the effort required to automate the next one can create even more.

---

## Project status

UniversalRunner is currently an evolving project.

The architecture and core execution model are implemented, but several areas can still be expanded, particularly for larger multi-user or enterprise deployments.

Potential future areas include:

- authentication and authorization;
- plugin permissions;
- execution quotas;
- centralized audit logs;
- retention policies;
- plugin version management;
- execution queues;
- additional dynamic field dependencies;
- centralized server deployment;
- containerized deployment.

The foundation is in place. Further work is focused on increasing robustness, security and operational maturity.

---

# Português

## Visão geral

O UniversalRunner nasceu de uma pergunta simples:

> **Em vez de desenvolver uma nova aplicação para cada processo, porque não criar uma plataforma comum à qual podemos acrescentar novas capacidades através de motores independentes?**

O UniversalRunner disponibiliza o ambiente comum de execução e a interface Web.

Cada motor contém o conhecimento e as dependências necessárias para executar um processo específico.

Um motor pode:

- analisar documentos;
- processar ficheiros Excel;
- validar informação;
- produzir relatórios;
- processar PDFs;
- integrar diferentes sistemas;
- analisar dados;
- incorporar Inteligência Artificial;
- ou implementar qualquer outro processo baseado em Python.

A plataforma central não precisa de conhecer a lógica de negócio desses motores.

---

## Arquitetura

```text
UniversalRunner
│
├── Interface Web
├── Descoberta de Plugins
├── Renderização de Schemas
├── Environment Manager
├── Executor
├── Mecanismo de Inspeção
│
└── Engines
    │
    ├── Motor A
    │   ├── plugin.json
    │   ├── schema_input.json
    │   ├── schema_output.json
    │   ├── requirements.txt
    │   ├── engine.py
    │   └── .venv/
    │
    ├── Motor B
    │   └── ...
    │
    └── Motor C
        └── ...
```

O objetivo é manter o **Host estável**, permitindo que os motores evoluam de forma independente.

---

## Princípios fundamentais

### 1. Os motores são externos ao executável

Os motores são armazenados fora do executável do UniversalRunner:

```text
UniversalRunner/
├── UniversalRunner.exe
├── engines/
├── runtime_data/
└── .streamlit/
```

Um novo motor pode ser instalado adicionando simplesmente a sua diretoria a:

```text
engines/
```

Adicionar um novo motor não obriga, por princípio, a recompilar o UniversalRunner.

---

### 2. Cada motor gere as suas dependências

Cada plugin pode declarar as suas dependências Python através de:

```text
requirements.txt
```

Por exemplo:

```text
pandas
openpyxl
matplotlib
```

Na primeira utilização do motor, o UniversalRunner pode criar automaticamente:

```text
motor/
└── .venv/
```

e instalar as bibliotecas necessárias.

Assim, motores diferentes podem utilizar conjuntos de dependências diferentes sem obrigar a incluir todas essas bibliotecas no Host.

---

### 3. Alterações às dependências são detetadas

O UniversalRunner calcula um hash de:

```text
requirements.txt
```

e guarda o estado do ambiente em:

```text
.environment.json
```

Quando as dependências são alteradas, o ambiente do motor pode ser atualizado automaticamente.

---

## Estrutura de um plugin

Um motor típico contém:

```text
meu_motor/
├── plugin.json
├── schema_input.json
├── schema_output.json
├── requirements.txt
└── engine.py
```

Depois da primeira execução poderá também conter:

```text
meu_motor/
├── .venv/
└── .environment.json
```

---

## Manifesto do plugin

O `plugin.json` identifica e configura o motor.

Exemplo:

```text
{
    "id": "pdf_inspector",
    "name": "PDF Inspector",
    "version": "1.0.0",
    "author": "José Lopes",
    "category": "Documentos",

    "entrypoint": "engine.py",

    "inputSchema": "schema_input.json",
    "outputSchema": "schema_output.json",

    "requirements": "requirements.txt",

    "timeoutSeconds": 300,

    "permissions": []
}
```

---

## Schemas de entrada

A interface é construída dinamicamente a partir de:

```text
schema_input.json
```

Atualmente podem ser utilizados tipos como:

- string;
- texto;
- inteiro;
- número;
- booleano;
- data;
- seleção;
- ficheiro;
- seleção dinâmica;
- seleção múltipla dinâmica.

Os campos podem ainda declarar propriedades opcionais como:

- obrigatório;
- valor por defeito;
- ajuda;
- placeholder;
- mínimo;
- máximo;
- opções;
- extensões;
- seleção de vários ficheiros.

---

## Outputs dos motores

Os motores não interagem diretamente com a interface Web.

Devolvem resultados estruturados:

```text
execute(payload)
        ↓
outputs
        ↓
UniversalRunner
        ↓
Interface
```

Um motor pode devolver:

- texto;
- JSON;
- tabelas;
- ficheiros;
- logs.

Isto mantém a lógica do motor independente da tecnologia utilizada pela interface.

---

## Contrato `execute()`

Todos os motores implementam:

```text
execute(payload)
```

O payload pode disponibilizar:

```text
executionId
pluginId
parameters
inputFiles
outputDirectory
executionDirectory
```

O motor recebe dados, executa a sua lógica e devolve resultados.

Não necessita de conhecer Streamlit.

---

## Contrato `inspect()`

Alguns campos da interface dependem da informação fornecida noutros campos.

Para estes cenários, um motor pode opcionalmente implementar:

```text
inspect(payload)
```

Por exemplo:

```text
Upload Excel
       ↓
inspect()
       ↓
identificar colunas
       ↓
preencher dropdowns
```

Este mecanismo permite manter o conhecimento no próprio motor.

A plataforma não precisa de saber o que significa uma coluna Excel.

---

## Dependências entre campos

O mesmo mecanismo poderá suportar relações como:

```text
País
 ↓
Distrito
 ↓
Concelho
```

ou:

```text
Base de dados
 ↓
Tabela
 ↓
Coluna
```

ou:

```text
Tipo de processo
 ↓
Opções aplicáveis
```

O conhecimento pertence ao motor.

O UniversalRunner fornece o mecanismo genérico para o utilizar.

---

## Execuções isoladas

Cada execução possui um identificador próprio:

```text
runtime_data/
└── executions/
    └── <execution-id>/
        ├── payload.json
        ├── response.json
        ├── input/
        └── output/
```

Isto permite separar ficheiros de entrada, resultados e informação de diferentes execuções.

---

## Isolamento dos processos

Os motores não são executados diretamente dentro do processo da interface Web.

O fluxo é:

```text
Interface Web
      ↓
Executor
      ↓
Ambiente virtual do motor
      ↓
Plugin Runner
      ↓
engine.py
      ↓
execute(payload)
```

Isto mantém a execução do motor desacoplada da interface.

---

## Interface Web e deployment

O UniversalRunner disponibiliza uma experiência baseada em Web.

A implementação atual foi preparada e empacotada para Windows, incluindo um instalador.

No entanto, Windows é uma opção de distribuição e não um princípio da arquitetura.

A mesma abordagem pode ser adaptada para Linux.

A natureza Web da interface permite também evoluir para um modelo centralizado:

```text
Utilizadores
     ↓
HTTPS
     ↓
Reverse Proxy
     ↓
UniversalRunner
     ↓
Motores independentes
```

Assim, a mesma arquitetura pode suportar utilização local ou uma plataforma central acessível através do browser.

---

## Criar um novo motor

O desenvolvimento de um novo motor normalmente exige apenas:

```text
engines/
└── meu_motor/
    ├── plugin.json
    ├── schema_input.json
    ├── schema_output.json
    ├── requirements.txt
    └── engine.py
```

O ciclo típico é:

1. Definir o manifesto.
2. Definir os inputs.
3. Definir os outputs.
4. Declarar as dependências.
5. Implementar `execute(payload)`.
6. Opcionalmente implementar `inspect(payload)`.
7. Copiar o motor para `engines/`.
8. Executar através do UniversalRunner.

---

## Filosofia do projeto

O objetivo do UniversalRunner não é simplesmente executar scripts Python.

O objetivo é:

> **criar uma plataforma reutilizável onde resolver o próximo problema signifique acrescentar uma capacidade, em vez de voltar a construir a aplicação.**

Automatizar um processo cria valor.

Criar capacidade reutilizável para reduzir o esforço necessário à próxima automatização pode criar ainda mais.

---

## Estado do projeto

O UniversalRunner é atualmente um projeto em evolução.

A arquitetura base e o modelo principal de execução estão implementados, mas existem áreas que poderão ser desenvolvidas para cenários multiutilizador ou empresariais, nomeadamente:

- autenticação e autorização;
- permissões por motor;
- quotas de execução;
- auditoria centralizada;
- políticas de retenção;
- gestão de versões dos plugins;
- filas de execução;
- dependências dinâmicas mais complexas;
- deployment centralizado;
- containers.

**A base está construída.**

A evolução seguinte passa sobretudo por aumentar a robustez, segurança e maturidade operacional do modelo.
