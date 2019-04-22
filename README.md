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

# OpenSIPS
Para controlar o OpenSIPS, uma sugestão é que seja feito via systemd. Primeiro, é preciso ativar o suporte a esse, alterando a seguinte linha no arquivo `/etc/default/opensips`:
```ini
RUN_OPENSIPS=yes
```
Se quiser ativar o serviço por demanda, pode-se deixar o `opensips` desativado por padrão:
```bash
systemctl disable opensips
```
Para ativá-lo por demanda:
```bash
systemctl start opensips
```
E desativá-lo quando terminar os experimentos:
```bash
systemctl stop opensips
```
 Combinado a isso, pode-se também usar a faixa do IFSC câmpus São José, `191.36.8.0/21`, na regra de _firewall_ do GCP.

# Configurar
As seguintes linhas foram adicionadas no arquivo `/etc/opensips/opensips.cfg`:
1. Suporte a NAT do servidor SIP (onde `<IP externo>` é o valor obtido na saída do `make`/Terraform). Na seção global:
```
listen=udp:0.0.0.0:5060
alias="<IP externo>:5060"
alias="<IP externo>"
advertised_address=<IP externo>
```

2. Suporte a NAT dos agentes remotos (UACs e UASs). Na seção de módulos:
```
loadmodule "nathelper.so"
modparam("nathelper", "received_avp", "$avp(42)")
```
