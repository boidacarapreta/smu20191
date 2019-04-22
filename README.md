# Sobre o projeto
Este é um projeto em desenvolvimento na disciplina de Sistemas Multimídia do Curso de Engenharia de Telecomunicações do IFSC câmpus São José - primeiro semestre de 2019.

Até o momento, foi criada uma máquina virtual na [Google Compute Engine](https://cloud.google.com/compute/) com servidor SIP [OpenSIPS](https://opensips.org). O processo foi realizado com a ajuda do [Terraform](https://terraform.io).

# Como usar
O arquivo `Makefile` pode ser usado para criar, visualizar e destruir o cenário criado.

## Instalar
Além do comando `make`, é preciso [obter o Terraform](https://www.terraform.io/downloads.html).

## Obter acesso
Antes de mais nada, é preciso [obter uma chave API](https://console.cloud.google.com/apis/credentials/serviceaccountkey) para controlar os recursos via Terraform. O nome do arquivo será usado na sequência, na variável `gcp_sakey` do arquivo de [configuração](#Configurar).

## Configurar
Há apenas um arquivo a configurar, `gcp.tfvars`
Sobre regiões e zonas da Google Cloud Platform (GCP), ver a [geografia da nuvem](https://cloud.google.com/docs/geography-and-regions).
```ini
gcp_sakey = "<arquivo com chave API em formato JSON>"
gce_project = "<nome do projeto no Google Cloud Platform>"
gce_region = "<região a escolher>"
gce_zone = "<zona específica da região a escolher>"
gce_ssh_user = "<usuário para login na máquina virtual>"
gce_ssh_pub_key_file = "<arquivo com a chave pública SSH>"
```

Exemplo:
```ini
gcp_sakey = "/home/usuario/gcp.json"
gce_project = "projeto-123"
gce_region = "southamerica-east1"
gce_zone = "southamerica-east1-a"
gce_ssh_user = "joao"
gce_ssh_pub_key_file = "/home/usuario/.ssh/id_rsa.pub"
```

## Criar o ambiente
Apenas `make`. É suficiente :)

Se quiser, `make show` apresenta o estado do Terraform.

## Destruir o ambiente
Somente `make clean`. Nada mais.
