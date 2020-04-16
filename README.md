# Elasticsearch stack com Docker
Repositório do projeto de implantação de um Stack básica do Elasticsearch em cluster, utilizando docker.

## Considerações iniciais
Este projeto visa a implantação de uma stack básica do Elasticsearch, utilizando um VM a ser provisionada pelo Vangrant no VirtualBox. 
Essa VM tem a configuração de 4GB de RAM e 2 vCPUs. Favor observar se a máquina host que será implantada tem capacidade necessária.

## Pré-requisitos:
+ Softwares necessários:
    + VirtualBox 6.0.x
    + Vagrant 2.2.x
    + Necessário instalar o seguinte plugin do Vagrant, conforme comando abaixo:
    ```console
        vagrant plugin install vagrant-vbguest
    ```

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

## Observação importante para ambientes de produção
1 - De acordo com a documentação oficial do Elastic search, em ambientes de produção é necessário realizar um ajuste em um parâmetro de kernel (vm.max_map_count). Esse parâmetro define o número máximo de áreas de mapas de memória que um processo pode ter. O Elastic necessita de um valor de 262144, conforme sua documentação.

Como estamos rodando a VM com um CentOS, devemos acessá-la e ajustar esse valor:

**Checando o valor atual**
```console
    sudo sysctl -a | grep vm.max_map_count
```
Saída do comando acima:
```console
    vm.max_map_count = 65530
```
**Ajustando o valor, conforme recomendado pela documetação**
```console
    sudo sysctl -w vm.max_map_count=262144
```
2 - Congigurando o Heap Size
O Elasticsearch, por padrão, diz à JVM para usar um heap com um tamanho mínimo e máximo de 1 GB. Quando for implantar em ambiente de produção, é importante configurar o tamanho do heap para garantir que o Elasticsearch possua heap suficiente para trabalhar.

O Elasticsearch utilizará a configuração de heap especificado no arquivo jvm.options (Xms e Xmx). Deve-se definir essas duas configurações para serem iguais umas às outras. Entretanto, ao executar o Elastic via docker, não é recomendado alterar esse arquivo, mas sim especificar via variáveis de ambiente, conforme próximo parágrafo.

O uso da variável de ambiente ES_JAVA_OPTS no arquivo docker-compose.yml, serve para definir o Xms e Xmx. Por exemplo, para usar 16 GB, basta preencher da seguinte forma:
```console
    ES_JAVA_OPTS = "- Xms16g -Xmx16g"
```
Dessa forma, as configurações que estão no arquivo jvm.options, são sobrescritas.

**Atenção:** Deve-se configurar o Xmx e Xms de modo a não ultrapassar 50% do total de memória RAM física disponível no computador que irá executar o container da stack. Isso deve-se ao fato do Elasticsearch utilizar memória para outros fins que não apenas o que está definido na heap. É necessário considerar nesse cálculo a quantidade de nós do Elastic no cluster.

# Pontos a serem esclarecidos em breve:
- Configuração do [loop-lvm](https://docs.docker.com/engine/userguide/storagedriver/device-mapper-driver/#configure-docker-with-devicemapper) mode do docker. Isso é importante para ambientes de produção.