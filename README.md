# Projeto 2 - Data Warehouse na Nuvem com Amazon Redshift e Terraform

Este projeto tem como objetivo a modelagem e implementação de um Data Warehouse (DW) na nuvem utilizando Amazon Redshift e Terraform para infraestrutura como código (IaC). Também utilizamos Docker para ambiente de desenvolvimento e Metabase para visualização dos dados.

---

## 🧰 Pré-requisitos

- Conta AWS ativa
- Docker instalado
- AWS CLI instalado
- Terraform instalado
- PostgreSQL Client (`psql`) instalado

---

## 🚀 Etapas do Projeto

### 1. Criar Bucket no S3

- Acesse sua conta AWS
- Crie um bucket na região **Ohio (us-east-2)** com o nome:

dsa-fonte-p2-<ID-AWS>

yaml
Copy
Edit

- Dentro do bucket, crie uma pasta chamada `dados`.

- Faça o upload dos 5 arquivos `.csv` para essa pasta.

---

### 2. Preparar o Ambiente com Docker

No terminal, navegue até a pasta do projeto e execute:

```bash
docker build -t dsa-imagem-terraform:p2 .
Em seguida:

bash
Copy
Edit
docker run -dit --name dsa-p2 -v ./IaC:/iac dsa-imagem-terraform:p2 /bin/bash
ℹ️ Nota para Windows: Substitua ./IaC pelo caminho completo, ex: C:\DSA\Cap15\IaC

3. Configurar Credenciais AWS
Acesse o container:

bash
Copy
Edit
docker exec -it dsa-p2 /bin/bash
Configure suas credenciais:

bash
Copy
Edit
aws configure
Verifique as versões:

bash
Copy
Edit
terraform version
aws --version
4. Criar Infraestrutura com Terraform
No container, navegue até a pasta etapa1:

bash
Copy
Edit
cd /iac/etapa1
terraform init
terraform validate
terraform plan
terraform apply
5. Validar Recursos Criados
Acesse o console da AWS

Confirme a criação do cluster Redshift

Verifique a role RedshiftS3AccessRole no IAM

Copie o ARN da role e edite o arquivo modelo_fisico.sql com esse valor

6. Executar Script SQL
Na pasta etapa2, edite o arquivo modelo_fisico.sql:

Substitua o ACCOUNT ID pela sua conta AWS

Substitua o nome do bucket

Execute o script:

bash
Copy
Edit
psql -h <ENDPOINT_REDSHIFT> -U adminuser -d dsadb -p 5439 -f modelo_fisico.sql
Senha: dsaS9curePassw2rd

7. Atualizar Cluster com Role
Edite o arquivo redshift.tf:

hcl
Copy
Edit
iam_roles = [aws_iam_role.redshift_role.arn]
Volte para a pasta etapa1 e execute:

bash
Copy
Edit
terraform apply
Repita o comando psql do passo anterior.

8. Verificar Dados no Redshift
Acesse o editor de consultas do Redshift

Verifique se os dados foram carregados corretamente

9. Visualizar Dados com Metabase
Execute o container do Metabase:

bash
Copy
Edit
docker run -d -p 3000:3000 --name metabase metabase/metabase
Acesse via navegador: http://localhost:3000

Configure a conexão com o Redshift e explore os dados.

10. Exemplo de Query no Metabase
sql
Copy
Edit
SELECT 
    p.nome_produto,
    p.categoria,
    p.subcategoria,
    SUM(fv.receita_vendas) AS receita_total
FROM 
    dsaschema.fato_vendas fv
JOIN 
    dsaschema.dim_produto p ON fv.sk_produto = p.sk_produto
GROUP BY 
    p.nome_produto, p.categoria, p.subcategoria
ORDER BY 
    receita_total DESC;
11. Finalizar Projeto
Para destruir a infraestrutura criada:

bash
Copy
Edit
terraform destroy