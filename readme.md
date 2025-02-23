## Projeto ETL Inbev

### Visão Geral

Este projeto tem como objetivo criar um pipeline de ETL robusto na nuvem Azure, utilizando a arquitetura Medalhão (Bronze, Silver, Gold) para organizar e processar dados. A orquestração será realizada com o Azure Data Factory, e as transformações de dados serão executadas no Azure Databricks.

### Recursos Azure Utilizados

* **Resource Group:** Organização lógica dos recursos Azure.
* **Storage Account (Azure Data Lake Storage Gen2):** Armazenamento de dados em camadas (Bronze, Silver, Gold).
* **Azure Data Factory:** Orquestração do pipeline de ETL.
* **Azure Key Vault:** Gerenciamento seguro de segredos e credenciais.
* **Azure SQL Database:** Armazenamento de dados transformados e agregados.
* **Azure Databricks:** Processamento e transformação de dados.
* **GitHub:** Controle de versão do código e configuração do pipeline.

### Convenções de Nomenclatura

Serão utilizadas as convenções de nomenclatura da Microsoft Azure para garantir a consistência e organização dos recursos:

[https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-abbreviations?source=recommendations](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-abbreviations?source=recommendations)

### Configuração dos Recursos

#### 1. Resource Group

* Criar um Resource Group para agrupar todos os recursos do projeto.

#### 2. Storage Account (Azure Data Lake Storage Gen2)

* Criar uma Storage Account com o namespace hierárquico habilitado para utilizar o Data Lake Storage Gen2.
* Criar os containers para as camadas Medalhão: `bronze`, `silver` e `gold`.

#### 3. Azure Data Factory

* Criar um Data Factory antes do Key Vault, pois o Data Factory precisa de um aplicativo para acessar o Key Vault.

#### 4. Azure Key Vault

* Criar um Key Vault para armazenar segredos e credenciais de forma segura.
* Configurar a política de acesso para permitir que o Data Factory se conecte ao Key Vault.

#### 5. Azure SQL Database

* Criar um Azure SQL Database (nível Basic para este projeto).
* Criar um SQL Server com autenticação de usuário e senha.
* Liberar o IP da máquina para acesso ao SQL Server.
* Armazenar a string de conexão do SQL Server no Key Vault.
* Testar a conexão com o Azure Data Studio para verificar o acesso às tabelas.

#### 6. GitHub Repository

* Criar um repositório no GitHub para controle de versão do código e configuração do pipeline.

#### 7. Azure Databricks

* Criar um Workspace Databricks (nível Premium - 14 dias).
* Vincular o Databricks ao Key Vault e criar um Secret Scope para armazenar os segredos.
    * Utilizar a URI e o Resource ID do Key Vault na configuração do Secret Scope.
    * Verificar a permissão do Databricks nas políticas de acesso do Key Vault.
* Criar um token de acesso pessoal no Databricks para integração com o Data Factory.
* Criar um token de acesso pessoal no GitHub para integração com o Databricks.
* Adicionar o repositório GitHub ao Databricks.
* Criar um cluster Databricks para executar os notebooks.
* Criar um notebook Databricks para realizar as transformações de dados.
* Realizar o mount dos containers do Data Lake no Databricks.

### Configuração do Pipeline no Azure Data Factory

#### 1. Secrets

* Criar Secrets no Data Factory para armazenar credenciais e tokens.

#### 2. Linked Services

* Criar Linked Services para conectar o Data Factory aos seguintes recursos:
    * Azure Blob Storage (Data Lake)
    * Azure SQL Database
    * Azure Key Vault
    * Azure Databricks
    * REST API (para coleta de dados)

#### 3. GitHub Integration

* Configurar a integração do Data Factory com o repositório GitHub.

#### 4. Pipeline

* Criar um pipeline no Data Factory com as seguintes atividades:
    * **Copy Data:** Coletar dados da API REST e armazená-los no container `bronze` do Data Lake.
    * **Notebook:** Executar o notebook Databricks para realizar as transformações de dados e salvar os arquivos Parquet nas camadas `silver` e `gold` do Data Lake.

### Próximos Passos

* Implementar a lógica de transformação de dados no notebook Databricks.
* Configurar o monitoramento e alertas do pipeline no Data Factory.
* Implementar a lógica de carga dos dados transformados no Azure SQL Database.
* Documentar o esquema das tabelas no Azure SQL Database.
* Implementar testes de qualidade de dados.
* Configurar a integração contínua e entrega contínua (CI/CD) do pipeline.

#### Observações

* Este documento fornece uma visão geral do projeto e pode ser atualizado à medida que o projeto avança.
* É importante seguir as melhores práticas de segurança ao lidar com segredos e credenciais.
* A escolha dos recursos e configurações pode variar dependendo dos requisitos específicos do projeto.
* Sempre teste o pipeline em um ambiente de desenvolvimento antes de implantar em produção.
