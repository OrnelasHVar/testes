
# Projeto de API com FastAPI, Google Cloud, Terraform e Skaffold

Este projeto configura e implementa uma API utilizando FastAPI na Google Cloud Platform (GCP), com gerenciamento de infraestrutura por Terraform e Skaffold.

## Introdução

### Objetivo
O objetivo deste projeto é criar um mecanismo de pesquisa utilizando o Vertex AI. A API realiza consultas no vector store do Vertex AI e alimenta a ferramenta Freshworks com as informações retornadas. Isso permite que os usuários da Copel façam consultas precisas na base de conhecimento da empresa.

## Estrutura de Arquivos

### Configurações
- `config.yaml`: Contém as configurações e credenciais dos serviços Artifact Registry, BigQuery e Vertex AI.

### Skaffold
- `skaffold.yaml`: Configuração do Skaffold.
- `cloud-run-service.yaml`: Configuração do Cloud Run.

### API
- `main.py`: Código principal da API.
- `requirements.txt`: Dependências Python.

### Docker
- `Dockerfile`: Define a imagem Docker para a API.

### Terraform
- `main.tf`: Configura o Artifact Registry.
- `bigquery.tf`: Configura o BigQuery (executar apenas uma vez, caso seja necessário destruir o ambiente atual, lembre-se de retirar este arquivo do diretório, pois ao utilizar o comando `terraform destroy` você pode apagar a tabela de logs).


## Passo a Passo para Deploy

### 0. Autorização no Google Cloud
Certifique-se de que você possui as permissões necessárias no Google Cloud e esteja autenticado na sua conta GCP.

### 1. Preencher os Arquivos config, skaffold e cloud-run-service
- `config.yaml`: Configure os parâmetros necessários:
  - `project_id`: ID do projeto.
  - `Regions`: Região do projeto (os serviços devem estar na mesma região para evitar alta latência).
  - `index_endpoint_name`, `deployed_index_id`, `api_endpoint`, `model`: Configurações para os índices do Vertex AI.
  - `dataset`, `table`: Nome do dataset e da tabela no BigQuery para armazenar logs.
  - `repository_id`, `description`, `format`: Configurações do Artifact Registry.

  Observações: Certifique-se de que os índices do Vector Search já estejam configurados. Se não desejar criar novas tabelas com o Terraform, apenas conecte a um dataset existente no BigQuery e remova o arquivo `bigquery.tf`.

- `cloud-run-service.yaml`: Configure o Cloud Run:
  - `name`: Nome do serviço.
  - `autoscaling.knative.dev/maxScale`, `autoscaling.knative.dev/minScale`: Número máximo e mínimo de instâncias.
  - `autoscaling.knative.dev/target`: Carga mínima para iniciar uma nova instância.
  - `run.googleapis.com/revision-timeout-sec`: Tempo máximo de espera por nova requisição.
  - `image`: Imagem do Artifact Registry usada pelo Cloud Run.
  - `memory`, `cpu`: Memória e CPUs por instância.
  - `timeoutSeconds`: Tempo que o sistema aguardará a resposta de uma verificação antes de considerar que a verificação falhou.
  - `periodSeconds`: Especifica a frequência com que a verificação será realizada.
  - `failureThreshold`: Especifica o número de tentativas de verificação consecutivas que devem falhar antes que o sistema considere que o contêiner não está funcionando corretamente e tome uma ação.

- `skaffold.yaml`: Configure o Skaffold:
  - `image`: Imagem no Artifact Registry.
  - `projectid`: ID do projeto no Cloud Run.
  - `region`: Região do serviço do Cloud Run.
  - `resourceType`, `resourceName`: Tipo e nome do recurso.

### 2. Configurar o Terraform
Certifique-se de que você esta no diretório dos arquivos de configuração. Caso seja realizado alguma modificação no código do terraform ou da API é necessário executar este passo novamente.
1. Inicialize o Terraform:
   ```sh
   terraform init
   ```
2. Aplique as configurações do Terraform:
   ```sh
   terraform apply
   ```
   **Nota:** Execute `bigquery.tf` apenas uma vez para criar o dataset e a tabela no BigQuery. Remova este arquivo para evitar exclusão acidental de dados existentes.

### 3. Build e Deploy com Skaffold
Caso seja realizado alguma modificação no código da API, no código do Terraform ou no código do Cloud Run, é necessário executar este passo novamente.
1. Execute o Skaffold para criar/atualizar o serviço da API no Cloud Run:
   ```sh
   skaffold run
   ```

### 4. Testar a API
Para testar a API, use `curl` ou qualquer ferramenta de API (como Postman). Exemplo de requisição POST:

```sh
curl -X POST "https://<your-api-url>/search" -H "accept: application/json" -H "Content-Type: application/json" -H "Authorization: Bearer $ACCESS_TOKEN" -d '{
  "text_search": "Reembolso",
  "id": "123",
  "similarity": 0,
  "databases": ["2"],
  "category": ["23000081305"]
}'
```

## Considerações Finais
Certifique-se de ajustar ou remover o arquivo `bigquery.tf` após a primeira execução para evitar a exclusão acidental dos dados no BigQuery. Para atualizações futuras, siga os passos descritos na seção de Deploy.

Para dúvidas ou problemas, sinta-se à vontade para abrir uma issue ou entrar em contato.
