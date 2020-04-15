# Elasticsearch stack com Docker
Repositório do projeto de implantação de um Stack básica do Elasticsearch em cluster, utilizando docker.

## Considerações iniciais
Este projeto visa a implantação de uma stack básica do Elasticsearch, utilizando um VM a ser provisionada pelo Vangrant no VirtualBox. 
Essa VM tem a configuração de 4GB de RAM e 2 vCPUs. Favor observar se a máquina host que será implantada tem capacidade necessária.

## Pré-requisitos:
+ Softwares necessários:
    + VirtualBox 6.0.x
    + Vagrant 2.2.x

## Subindo a VM elk-cluster-vm
Dentro do diretório raiz do desse projeto, execute:
```console
    cd elk-vm && vagrant up
```
Para acessar a vm, após a sua inicilização, execute:
```console
    vagrant ssh
```
### O Vagrantfile
O Vagrantfile que está no diretório elk-vm, provisiona uma VM com as seguintes tarefas:
* Atualiza o Sistema Operacional
* Instala o git
* Instala o docker
* Instala o docker-compose

Os itens instalados são fundamentais para rodar a stack Elasticsearch via docker na VM.