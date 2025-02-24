## Projeto ETL Inbev

### Run da aplicação - resumo

* Para rodar o ETL é necessário clicar em Debug do Data Factory. Deste modo, a coleta é feita na API, o arquivo JSON é salvo na camada bronze,e é acionado o tratamento é feito conforme o código no notebook do databrick, salvando os arquivos parquet no lake, nas camadas silver e gold.

Pipeline no Data Factory:

![ETL_adb_ls](https://github.com/user-attachments/assets/86b9140c-57e2-4538-80e2-3daca47da457)

Camada bronze:

![image](https://github.com/user-attachments/assets/271d3d4a-6850-4241-b4cf-473402366c5c)

Camada silver:
![adls_silver_content](https://github.com/user-attachments/assets/670a9dba-2bf2-4617-9f9f-164f78400ecc)

Tabela Silver no Azure Data Studio (para conferência):
![sql_silver](https://github.com/user-attachments/assets/d91bd3d4-7dab-48ce-b98f-72f9c3da527d)

Tabela final:

![sql_gold](https://github.com/user-attachments/assets/c1fa4def-e213-465b-bb7e-bf9561a6fe83)

## Configuração e detalhamento

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

* Criado um Resource Group para agrupar todos os recursos do projeto.
  ![rg_azure](https://github.com/user-attachments/assets/fa716c6e-2de9-49af-891d-adfcc721f1ee)


#### 2. Storage Account (Azure Data Lake Storage Gen2)

* Foi criado uma Storage Account com o namespace hierárquico habilitado para utilizar o Data Lake Storage Gen2.
  ![adls_containers](https://github.com/user-attachments/assets/f5f7c767-acac-4bdc-8c27-3e248084ec99)

* Foram criados os containers para as camadas Medalhão: `bronze`, `silver` e `gold`.
  ![adls_bronze_content](https://github.com/user-attachments/assets/7f2b3be3-5c87-4374-a172-9a9bbe79cfc4)
  ![adls_silver_content](https://github.com/user-attachments/assets/a19c849d-fe8a-488b-9c7c-385836033db9)

#### 3. Azure Data Factory

* Setar o Data Factory antes do Key Vault, pois o Data Factory precisa de um aplicativo para acessar o Key Vault.

#### 4. Azure Key Vault

* Criado um Key Vault para armazenar segredos e credenciais de forma segura.
* Configurado a política de acesso para permitir que o Data Factory se conecte ao Key Vault.

  ![keyvault](https://github.com/user-attachments/assets/ca4afe17-f11f-46f1-a511-683bee8baff6)


#### 5. Azure SQL Database

* Criado um Azure SQL Database (nível Basic para este projeto).
* Criado um SQL Server com autenticação de usuário e senha.
* Liberar o IP da máquina para acesso ao SQL Server.
* Armazenar a string de conexão do SQL Server no Key Vault.
* Testar a conexão com o Azure Data Studio para verificar o acesso às tabelas.

![sql_silver](https://github.com/user-attachments/assets/ffc3fa70-c9b3-4c3d-b379-6de9619bb470)

![sql_net_roles](https://github.com/user-attachments/assets/22fedea8-98ee-462c-9066-c531894b087d)

![sql_gold](https://github.com/user-attachments/assets/cce02cd2-2e68-4dc9-892d-dfa71689024f)

#### 6. GitHub Repository

* Criar um repositório no GitHub para controle de versão do código e configuração do pipeline.

  ![git_configuration](https://github.com/user-attachments/assets/2c36ea5b-4997-47fc-ad2f-e6c0a8b5e9a9)


#### 7. Azure Databricks

* Criado um Workspace Databricks (nível Premium - 14 dias).
* Vinculado o Databricks ao Key Vault e criado um Secret Scope para armazenar os segredos.
    * Utilizado a URI e o Resource ID do Key Vault na configuração do Secret Scope.
    * Verificado a permissão do Databricks nas políticas de acesso do Key Vault.
* Criado um token de acesso pessoal no Databricks para integração com o Data Factory.
* Criado um token de acesso pessoal no GitHub para integração com o Databricks.
* Adicionado o repositório GitHub ao Databricks.
* Criado um cluster Databricks para executar os notebooks.
  ![db_cluster_config](https://github.com/user-attachments/assets/3bc1f1c5-ef42-4031-8bde-f4c7274d8827)

* Criado um notebook Databricks para realizar as transformações de dados.
  ![adb_silver_conf](https://github.com/user-attachments/assets/97fee473-a59d-4d6a-a4c1-06e2a1e5fb39)

* Elaborado o mount dos containers do Data Lake no Databricks.

### Configuração do Pipeline no Azure Data Factory

#### 1. Secrets

* Criadas as Secrets no Data Factory para armazenar credenciais e tokens.

  ![keyvault](https://github.com/user-attachments/assets/efa585c8-ce54-47f6-a714-9b0b9f0fe45f)


#### 2. Linked Services

* Criar Linked Services para conectar o Data Factory aos seguintes recursos:
    * Azure Blob Storage (Data Lake)
    * Azure SQL Database
    * Azure Key Vault
    * Azure Databricks
    * REST API (para coleta de dados)

![linked_services](https://github.com/user-attachments/assets/ca5f263b-baf6-49f4-9861-be12596a6ace)


#### 3. GitHub Integration

* Configurar a integração do Data Factory com o repositório GitHub.

#### 4. Pipeline

* Criar um pipeline no Data Factory com as seguintes atividades:
    * **Copy Data:** Coletar dados da API REST e armazená-los no container `bronze` do Data Lake.
      ![adf_raw_collect_sink](https://github.com/user-attachments/assets/098a48a2-3376-4825-a4c1-615363bcb49f)

    * **Notebook:** Executar o notebook Databricks para realizar as transformações de dados e salvar os arquivos Parquet nas camadas `silver` e `gold` do Data Lake.
      ![adf_silvertogold_ls](https://github.com/user-attachments/assets/03b7aaef-911c-49d1-a9c1-88930ca0c0d8)

