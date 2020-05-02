# Elasticsearch stack com Docker
Repositório do projeto de implantação de um Stack básica do Elasticsearch em cluster, utilizando docker.

## 1 - Considerações iniciais
Este projeto visa a implantação de uma stack básica do Elasticsearch, utilizando uma VM a ser provisionada pelo Vangrant no VirtualBox.
Essa VM tem a configuração de 4GB de RAM e 2 vCPUs. Favor observar se a máquina host que será implantada tem capacidade necessária.

O cluster é composto por 03 nodes Elasticsearch + 01 node do Kibana.

**Observação**: Em ambientes de produção não é recomendado ter outro elemento da stack, ou seja, o adequado era executar o Kibana em outro host. Portanto, o ideal, era o host subir apenas o cluster com o Elasticsearh em docker.

Essa configuração é básica, podendo ser aplicada uma configuração superior, principalmente em ambientes de produção.

### 1.1 - Pré-requisitos:
+ Softwares necessários:
    + [VirtualBox 6.0.x](https://www.virtualbox.org/wiki/Downloads)
    + [Vagrant 2.2.x](https://www.vagrantup.com/docs/installation/)
    + Necessário instalar o seguinte plugin do Vagrant, conforme comando abaixo:
    ```shell
    $ vagrant plugin install vagrant-vbguest
    ```
    + [Git](https://git-scm.com/downloads)

### 1.2 - Clonando esse projeto
Abra um terminal e execute:
```shell
$ git clone git@github.com:gutobrutus/elasticstack.git
```
Agora é só acessar o diretório raiz do projeto com:
```shell
$ cd elasticstack
```

## 2 - Subindo a VM es-cluster-vm
Dentro do diretório raiz do desse projeto, execute:
```shell
$ vagrant up
```
Ao executar o comando acima pela primeira vez, demorará um pouco, pois a VM será provisionada, com instalação de dependências, etc.

Para acessar a vm, após a sua inicialização, execute:
```shell
$ vagrant ssh
```
### 2.1 - Sobre o Vagrantfile
O Vagrantfile que está no diretório raiz, provisiona uma VM com as seguintes tarefas:
* Atualiza o Sistema Operacional
* Instala o git
* Instala o docker
* Instala o docker-compose

Os itens instalados são fundamentais para rodar a stack Elasticsearch via docker na VM.

### 2.2 - Observação importante para ambientes de produção
**a) Configurando o `vm.max_map_count`**

De acordo com a documentação oficial do Elasticsearch, em ambientes de produção é necessário realizar um ajuste em um parâmetro de kernel (vm.max_map_count). Esse parâmetro define o número máximo de áreas de mapas de memória que um processo pode ter. O Elastic necessita de um valor de 262144, conforme sua documentação.

Como estamos rodando a VM com um CentOS, devemos acessá-la e ajustar esse valor:

**Checando o valor atual**
```shell
$ sudo sysctl -a | grep vm.max_map_count
```
**Saída do comando acima:**
```shell
vm.max_map_count = 65530
```
**Ajustando o valor, conforme recomendado pela documentação do Elasticsearch**
```shell
$ sudo sysctl -w vm.max_map_count=262144
```
**Observação**: Com o último comando, a alteração não é de forma definitiva no CentOS 7, ou seja, se reiniciar a VM, terá novamente que executar o comando acima. Para que a alterção seja permanente, adicione um arquivo `sysctl-elk.conf` no diretório `/etc/sysctl.d/` com o conteúdo `vm.max_map_count=262144`. Se esta configuração não estiver efetiva, o cluster não sobe.

**b) Configurando o Heap Size**:

O Elasticsearch, por padrão, diz à JVM para usar um heap com um tamanho mínimo e máximo de 1 GB. Quando for implantar em ambiente de produção, é importante configurar o tamanho do heap para garantir que o Elasticsearch possua heap suficiente para trabalhar.

O Elasticsearch utilizará a configuração de heap especificado no arquivo jvm.options (Xms e Xmx). Deve-se definir essas duas configurações para serem iguais. Entretanto, ao executar o Elastic via docker, não é recomendado alterar esse arquivo, mas sim especificar via variáveis de ambiente, conforme próximo parágrafo.

O uso da variável de ambiente ES_JAVA_OPTS no arquivo docker-compose.yml, serve para definir o Xms e Xmx. Por exemplo, para usar 8GB, basta preencher da seguinte forma:
```shell
ES_JAVA_OPTS = "-Xms8g -Xmx8g"
```
Dessa forma, as configurações que estão no arquivo jvm.options, são sobrescritas.

**Atenção:** Deve-se configurar o Xmx e Xms de modo a **não ultrapassar 50% do total de memória RAM física disponível no computador** que irá executar os containers da stack. Isso deve-se ao fato do Elasticsearch utilizar memória para outros fins que não apenas o que está definido na heap. É necessário considerar nesse cálculo a quantidade de nós do Elastic no cluster.

## 3 - Subindo o cluster Elasticsearch + Kibana
Após a VM ter sido provisionada e estar UP, realizada as configurações anteriores, agora basta subir a stack configurada no docker-compose.yml.

Depois de ter acessado a VM com `vagrant ssh`, acesse o diretório mapeado por padrão entre a VM e o seu computador:
```shell
[vagrant@srvclusterelk-local ~]$ cd /vagrant
```
Execute um ls -l para listar o conteúdo do diretório, apenas para confirmar se conteúdo é o mesmo desse projeto do GitHub:
```shell
[vagrant@srvclusterelk-local vagrant]$ ls -l
total 48
-rw-rw-r--. 1 vagrant vagrant  2075 Abr 18 22:42 docker-compose.yml
-rw-rw-r--. 1 vagrant vagrant 35149 Abr 18 21:08 LICENSE
-rw-rw-r--. 1 vagrant vagrant  4037 Abr 18 22:58 README.md
-rw-rw-r--. 1 vagrant vagrant  1085 Abr 18 22:14 Vagrantfile

```
Para executar (subir) a stack configurada no docker-compose.yml, basta executar o comando abaixo:
```shell
[vagrant@srvclusterelk-local vagrant]$ sudo docker-compose up -d
```
Caso queira acompanhar os logs de execução da stack, execute:
```shell
[vagrant@srvclusterelk-local vagrant]$ sudo docker-compose logs -f
```
Para deixar de acompanhar os logs basta teclar **CTRL+C**.

### 3.1  - Testando para ver se o cluster Elasticsearh já está no ar
Abra o seu navegador preferido e acesse o seguinte endereço:
```
http://192.168.99.10:9200
```
Lembrando que esse IP é que foi configurado no Vagrantfile, é portanto, o IP da VM que em que está sendo executado a stack com docker.

Será exibido como resultado:
```json
{
  "name" : "es01",
  "cluster_name" : "es-docker-cluster",
  "cluster_uuid" : "EhFmwSQiQYa93MZE5mcwWg",
  "version" : {
    "number" : "7.6.1",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "aa751e09be0a5072e8570670309b1f12348f023b",
    "build_date" : "2020-02-29T00:15:25.529771Z",
    "build_snapshot" : false,
    "lucene_version" : "8.4.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```
Esse resultado indica que o cluster já subiu.

Para acessar o kibana, mude apenas a porta no endereço anterior:
```
http://192.168.99.10:5601
```
Desfrute o seu ambiente Elasticsearh em cluster!

## 4 - Pontos a serem esclarecidos/abordados em breve:
- Configuração do [loop-lvm](https://docs.docker.com/engine/userguide/storagedriver/device-mapper-driver/#configure-docker-with-devicemapper) mode do docker. Isso é importante para ambientes de produção que utilizam LVM;
- Mapeamento de volumes para configurações adicionais para os nós do Elasticsearch e Kibana;
- Configurar acesso com login e senha no Kibana;
- Adição de um nó de Logstash na mesma VM ou em outra.
