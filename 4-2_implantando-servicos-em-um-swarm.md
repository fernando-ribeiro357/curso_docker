# Módulo 4.2: Implantando serviços em um Swarm

Os serviços do Swarm usam um modelo declarativo, o que significa que você define o estado desejado do serviço e confia no Docker para manter esse estado. O estado inclui informações como (mas não se limitando a):

- O nome e a tag da imagem que os containers do serviço devem executar
- Quantos containers participam do serviço
- Se alguma porta é exposta para clientes fora do swarm
- Se o serviço deve iniciar automaticamente quando o Docker inicia
- O comportamento específico que ocorre quando o serviço é reiniciado (como se uma reinicialização contínua/rolling restart é usada)
- Características dos nós onde o serviço pode ser executado (como restrições de recursos e preferências de posicionamento)

Para uma visão geral do modo Swarm, consulte [Conceitos-chave do modo Swarm](4-1_conceitos-de-cluster-swarm.md).
Para uma visão geral de como os serviços funcionam, consulte
[Como os serviços funcionam](https://docs.docker.com/engine/swarm/how-swarm-mode-works/services/).

## Criar um serviço

Para criar um serviço com uma única réplica sem configuração extra, você só precisa fornecer o nome da imagem. Este comando inicia um serviço Nginx com um nome gerado aleatoriamente e sem portas publicadas. Este é um exemplo ingênuo, já que você não pode interagir com o serviço Nginx.

```console
$ docker service create nginx
```

O serviço é agendado em um nó disponível. Para confirmar que o serviço foi criado e iniciado com sucesso, use o comando `docker service ls`:

```console
$ docker service ls

ID                  NAME                MODE                REPLICAS            IMAGE                                                                                             PORTS
a3iixnklxuem        quizzical_lamarr    replicated          1/1                 docker.io/library/nginx@sha256:41ad9967ea448d7c2b203c699b429abe1ed5af331cd92533900c6d77490e0268
```

Serviços criados nem sempre são executados imediatamente. Um serviço pode estar em estado pendente se sua imagem não estiver disponível, se nenhum nó atender aos requisitos configurados para o serviço ou por outras razões. Consulte
[Serviços pendentes](https://docs.docker.com/engine/swarm/how-swarm-mode-works/services/#pending-services) para mais informações.

Para fornecer um nome ao seu serviço, use a flag `--name`:

```console
$ docker service create --name my_web nginx
```

Assim como com containers autônomos, você pode especificar um comando que os containers do serviço devem executar, adicionando-o após o nome da imagem. Este exemplo inicia um serviço chamado `helloworld` que usa uma imagem `alpine` e executa o comando `ping docker.com`:

```console
$ docker service create --name helloworld alpine ping docker.com
```

Você também pode especificar uma tag de imagem para o serviço usar. Este exemplo modifica o anterior para usar a tag `alpine:3.6`:

```console
$ docker service create --name helloworld alpine:3.6 ping docker.com
```

Para mais detalhes sobre a resolução de tags de imagem, consulte
[Especificar a versão da imagem que o serviço deve usar](#especificar-a-versão-da-imagem-que-um-serviço-deve-usar).

### Criar um serviço usando uma imagem em um registro privado

Se sua imagem estiver disponível em um registro privado que requer login, use a flag `--with-registry-auth` com `docker service create`, após fazer o login. Se sua imagem estiver armazenada em `registry.example.com`, que é um registro privado, use um comando como o seguinte:

```console
$ docker login registry.example.com

$ docker service  create \
  --with-registry-auth \
  --name my_service \
  registry.example.com/acme/my_image:latest
```

Isso passa o token de login do seu cliente local para os nós do swarm onde o serviço é implantado, usando os logs WAL criptografados. Com essas informações, os nós podem fazer login no registro e baixar a imagem.

## Atualizar um serviço

Você pode alterar quase tudo sobre um serviço existente usando o comando `docker service update`. Quando você atualiza um serviço, o Docker para seus containers e os reinicia com a nova configuração.

Como o Nginx é um serviço web, ele funciona muito melhor se você publicar a porta 80 para clientes fora do swarm. Você pode especificar isso ao criar o serviço, usando a flag `-p` ou `--publish`. Ao atualizar um serviço existente, a flag é `--publish-add`. Há também uma flag `--publish-rm` para remover uma porta que foi publicada anteriormente.

Supondo que o serviço `my_web` da seção anterior ainda exista, use o seguinte comando para atualizá-lo para publicar a porta 80.

```console
$ docker service update --publish-add 80 my_web
```

Para verificar se funcionou, use `docker service ls`:

```console
$ docker service ls

ID                  NAME                MODE                REPLICAS            IMAGE                                                                                             PORTS
4nhxl7oxw5vz        my_web              replicated          1/1                 docker.io/library/nginx@sha256:41ad9967ea448d7c2b203c699b429abe1ed5af331cd92533900c6d77490e0268   *:0->80/tcp
```

Para mais informações sobre como a publicação de portas funciona, consulte
[publicar portas](#publicar-portas).

Você pode atualizar quase todos os detalhes de configuração de um serviço existente, incluindo o nome e a tag da imagem que ele executa. Consulte
[Atualizar a imagem de um serviço após a criação](#atualizar-a-imagem-de-um-serviço-após-a-criação).

## Remover um serviço

Para remover um serviço, use o comando `docker service remove`. Você pode remover um serviço pelo seu ID ou nome, conforme mostrado na saída do comando `docker service ls`. O comando a seguir remove o serviço `my_web`.

```console
$ docker service remove my_web
```

## Detalhes de configuração do serviço

As seções a seguir fornecem detalhes sobre a configuração do serviço. Este tópico não cobre todas as flags ou cenários. Em quase todos os casos em que você pode definir uma configuração na criação do serviço, você também pode atualizar a configuração de um serviço existente de maneira semelhante.

Consulte as referências de linha de comando para
[`docker service create`](https://docs.docker.com/reference/cli/docker/service/create/) e
[`docker service update`](https://docs.docker.com/reference/cli/docker/service/update/), ou execute um desses comandos com a flag `--help`.

### Configurar o ambiente de execução

Você pode configurar as seguintes opções para o ambiente de execução no container:

* Variáveis de ambiente usando a flag `--env`
* O diretório de trabalho dentro do container usando a flag `--workdir`
* O nome de usuário ou UID usando a flag `--user`

Os containers do serviço a seguir têm uma variável de ambiente `$MYVAR` definida como `myvalue`, executam a partir do diretório `/tmp/` e rodam como o usuário `my_user`.

```console
$ docker service create --name helloworld \
  --env MYVAR=myvalue \
  --workdir /tmp \
  --user my_user \
  alpine ping docker.com
```

### Atualizar o comando que um serviço existente executa

Para atualizar o comando que um serviço existente executa, você pode usar a flag `--args`. O exemplo a seguir atualiza um serviço existente chamado `helloworld` para que ele execute o comando `ping docker.com` em vez de qualquer comando que estivesse executando antes:

```console
$ docker service update --args "ping docker.com" helloworld
```

### Especificar a versão da imagem que um serviço deve usar

Quando você cria um serviço sem especificar detalhes sobre a versão da imagem a ser usada, o serviço usa a versão marcada com a tag `latest`. Você pode forçar o serviço a usar uma versão específica da imagem de algumas maneiras diferentes, dependendo do resultado desejado.

Uma versão de imagem pode ser expressa de várias maneiras:

- Se você especificar uma tag, o gerenciador (ou o cliente Docker, se você usar
  [confiança de conteúdo](https://docs.docker.com/engine/security/trust/)) resolve essa tag para um digest (hash).
  Quando a solicitação para criar uma tarefa de container é recebida em um nó worker, o nó worker vê apenas o digest, não a tag.

  ```console
  $ docker service create --name="myservice" ubuntu:16.04
  ```

  Algumas tags representam lançamentos discretos, como `ubuntu:16.04`. Tags como esta quase sempre resolvem para um digest estável ao longo do tempo. É recomendado que você use este tipo de tag sempre que possível.

  Outros tipos de tags, como `latest` ou `nightly`, podem resolver para um novo digest frequentemente, dependendo de quão frequentemente o autor da imagem atualiza a tag. Não é recomendado executar serviços usando uma tag que é atualizada frequentemente, para evitar que diferentes tarefas de réplica do serviço usem versões diferentes da imagem.

- Se você não especificar uma versão, por convenção, a tag `latest` da imagem é resolvida para um digest. Os workers usam a imagem neste digest ao criar a tarefa do serviço.

  Assim, os dois comandos a seguir são equivalentes:

  ```console
  $ docker service create --name="myservice" ubuntu

  $ docker service create --name="myservice" ubuntu:latest
  ```

- Se você especificar um digest diretamente, essa versão exata da imagem será sempre usada ao criar tarefas de serviço.

  ```console
  $ docker service create \
      --name="myservice" \
      ubuntu:16.04@sha256:35bc48a1ca97c3971611dc4662d08d131869daa692acb281c7e9e052924e38b1
  ```

Quando você cria um serviço, a tag da imagem é resolvida para o digest específico para o qual a tag aponta **no momento da criação do serviço**. Os nós workers para esse serviço usam esse digest específico para sempre, a menos que o serviço seja explicitamente atualizado. Este recurso é particularmente importante se você usar tags que mudam frequentemente, como `latest`, porque garante que todas as tarefas do serviço usem a mesma versão da imagem.

> [!NOTE]
>
> Se a [confiança de conteúdo](https://docs.docker.com/engine/security/trust/) estiver habilitada, o cliente realmente resolve a tag da imagem para um digest antes de contatar o gerenciador do swarm, para verificar se a imagem está assinada.
> Assim, se você usar confiança de conteúdo, o gerenciador do swarm recebe a solicitação já resolvida. Neste caso, se o cliente não puder resolver a imagem para um digest, a solicitação falhará.

Se o gerenciador não puder resolver a tag para um digest, cada nó worker é responsável por resolver a tag para um digest, e nós diferentes podem usar versões diferentes da imagem. Se isso acontecer, um aviso como o seguinte será registrado, substituindo os espaços reservados por informações reais.

```text
unable to pin image <NOME-DA-IMAGEM> to digest: <MOTIVO>
```

Para ver o digest atual de uma imagem, execute o comando `docker inspect <IMAGEM>:<TAG>` e procure pela linha `RepoDigests`. A seguir está o digest atual para `ubuntu:latest` no momento em que este conteúdo foi escrito. A saída foi truncada para clareza.

```console
$ docker inspect ubuntu:latest
```

```json
"RepoDigests": [
    "ubuntu@sha256:35bc48a1ca97c3971611dc4662d08d131869daa692acb281c7e9e052924e38b1"
],
```

Depois de criar um serviço, sua imagem nunca é atualizada, a menos que você execute explicitamente `docker service update` com a flag `--image`, conforme descrito abaixo. Outras operações de atualização, como escalonar o serviço, adicionar ou remover redes ou volumes, renomear o serviço ou qualquer outro tipo de operação de atualização, não atualizam a imagem do serviço.

### Atualizar a imagem de um serviço após a criação

Cada tag representa um digest, semelhante a um hash do Git. Algumas tags, como `latest`, são atualizadas frequentemente para apontar para um novo digest. Outras, como `ubuntu:16.04`, representam uma versão de software lançada e não devem ser atualizadas para apontar para um novo digest frequentemente, se é que o fazem. Quando você cria um serviço, ele fica restrito a criar tarefas usando um digest específico de uma imagem até que você atualize o serviço usando `service update` com a flag `--image`.

Quando você executa `service update` com a flag `--image`, o gerenciador do swarm consulta o Docker Hub ou seu registro Docker privado para obter o digest para o qual a tag aponta atualmente e atualiza as tarefas do serviço para usar esse digest.

> [!NOTE]
>
> Se você usar [confiança de conteúdo](https://docs.docker.com/engine/security/trust/), o cliente Docker resolve a imagem e o gerenciador do swarm recebe a imagem e o digest, em vez de uma tag.

Geralmente, o gerenciador pode resolver a tag para um novo digest e o serviço é atualizado, reimplantando cada tarefa para usar a nova imagem. Se o gerenciador não conseguir resolver a tag ou algum outro problema ocorrer, as próximas duas seções descrevem o que esperar.

#### Se o gerenciador resolver a tag

Se o gerenciador do swarm puder resolver a tag da imagem para um digest, ele instrui os nós workers a reimplantar as tarefas e usar a imagem nesse digest.

- Se um worker tiver a imagem em cache nesse digest, ele a usará.

- Caso contrário, ele tentará baixar a imagem do Docker Hub ou do registro privado.

  - Se tiver sucesso, a tarefa será implantada usando a nova imagem.

  - Se o worker falhar ao baixar a imagem, o serviço falhará ao implantar nesse nó worker. O Docker tenta novamente implantar a tarefa, possivelmente em um nó worker diferente.

#### Se o gerenciador não puder resolver a tag

Se o gerenciador do swarm não puder resolver a imagem para um digest, nem tudo está perdido:

- O gerenciador instrui os nós workers a reimplantar as tarefas usando a imagem naquela tag.

- Se o worker tiver uma imagem em cache localmente que resolva para aquela tag, ele usará essa imagem.

- Se o worker não tiver uma imagem em cache localmente que resolva para a tag, o worker tenta se conectar ao Docker Hub ou ao registro privado para baixar a imagem naquela tag.

  - Se isso tiver sucesso, o worker usará essa imagem.

  - Se isso falhar, a tarefa falhará ao implantar e o gerenciador tentará novamente implantar a tarefa, possivelmente em um nó worker diferente.

### Publicar portas

Ao criar um serviço swarm, você pode publicar as portas desse serviço para hosts fora do swarm de duas maneiras:

- [Você pode confiar na malha de roteamento (routing mesh)](#publicar-as-portas-de-um-serviço-usando-a-malha-de-roteamento).
  Quando você publica uma porta de serviço, o swarm torna o serviço acessível na porta de destino em cada nó, independentemente de haver uma tarefa para o serviço sendo executada nesse nó ou não. Isso é menos complexo e é a escolha certa para muitos tipos de serviços.

- [Você pode publicar a porta de uma tarefa de serviço diretamente no nó do swarm](#publicar-as-portas-de-um-serviço-diretamente-no-nó-do-swarm)
  onde esse serviço está sendo executado. Isso ignora a malha de roteamento e fornece máxima flexibilidade, incluindo a capacidade de desenvolver sua própria estrutura de roteamento. No entanto, você é responsável por rastrear onde cada tarefa está sendo executada e rotear solicitações para as tarefas, além de fazer balanceamento de carga entre os nós.

Continue lendo para mais informações e casos de uso para cada um desses métodos.

#### Publicar as portas de um serviço usando a malha de roteamento

Para publicar as portas de um serviço externamente ao swarm, use a flag `--publish <PORTA-PUBLICADA>:<PORTA-DO-SERVIÇO>`. O swarm torna o serviço acessível na porta publicada em cada nó do swarm. Se um host externo se conectar a essa porta em qualquer nó do swarm, a malha de roteamento o encaminhará para uma tarefa. O host externo não precisa conhecer os endereços IP ou as portas usadas internamente pelas tarefas do serviço para interagir com o serviço. Quando um usuário ou processo se conecta a um serviço, qualquer nó worker executando uma tarefa do serviço pode responder. Para mais detalhes sobre o networking de serviços swarm, consulte
[Gerenciar redes de serviços swarm](https://docs.docker.com/engine/swarm/services/networking/).

##### Exemplo: Executar um serviço Nginx de três tarefas em um swarm de 10 nós

Imagine que você tenha um swarm de 10 nós e implante um serviço Nginx executando três tarefas em um swarm de 10 nós:

```console
$ docker service create --name my_web \
                        --replicas 3 \
                        --publish published=8080,target=80 \
                        nginx
```

Três tarefas são executadas em até três nós. Você não precisa saber quais nós estão executando as tarefas; conectar-se à porta 8080 em qualquer um dos 10 nós conecta você a uma das três tarefas `nginx`. Você pode testar isso usando `curl`. O exemplo a seguir assume que `localhost` é um dos nós do swarm. Se este não for o caso, ou `localhost` não resolver para um endereço IP no seu host, substitua pelo endereço IP do host ou nome de host resolvido.

A saída HTML está truncada:

```console
$ curl localhost:8080

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...truncado...
</html>
```

Conexões subsequentes podem ser roteadas para o mesmo nó do swarm ou para um diferente.

#### Publicar as portas de um serviço diretamente no nó do swarm

Usar a malha de roteamento pode não ser a escolha certa para sua aplicação se você precisar tomar decisões de roteamento com base no estado da aplicação ou precisar de controle total do processo de roteamento de solicitações para as tarefas do seu serviço. Para publicar a porta de um serviço diretamente no nó onde ele está sendo executado, use a opção `mode=host` na flag `--publish`.

> [!NOTE]
>
> Se você publicar as portas de um serviço diretamente no nó do swarm usando `mode=host` e também definir `published=<PORTA>`, isso cria uma limitação implícita de que você só pode executar uma tarefa para esse serviço em um determinado nó do swarm. Você pode contornar isso especificando `published` sem uma definição de porta, o que faz com que o Docker atribua uma porta aleatória para cada tarefa.
>
> Além disso, se você usar `mode=host` e não usar a flag `--mode=global` em `docker service create`, é difícil saber quais nós estão executando o serviço para rotear o trabalho para eles.

##### Exemplo: Executar um serviço de servidor web `nginx` em cada nó do swarm

[nginx](https://hub.docker.com/_/nginx/) é um proxy reverso de código aberto, balanceador de carga, cache HTTP e servidor web. Se você executar o nginx como um serviço usando a malha de roteamento, conectar-se à porta nginx em qualquer nó do swarm mostrará a página web de (efetivamente) um nó do swarm aleatório executando o serviço.

O exemplo a seguir executa o nginx como um serviço em cada nó do seu swarm e expõe a porta nginx localmente em cada nó do swarm.

```console
$ docker service create \
  --mode global \
  --publish mode=host,target=80,published=8080 \
  --name=nginx \
  nginx:latest
```

Você pode acessar o servidor nginx na porta 8080 de cada nó do swarm. Se você adicionar um nó ao swarm, uma tarefa nginx será iniciada nele. Você não pode iniciar outro serviço ou container em qualquer nó do swarm que se vincule à porta 8080.

> [!NOTE]
>
> Este é um exemplo puramente ilustrativo. Criar uma estrutura de roteamento em nível de aplicação para um serviço em várias camadas é complexo e está fora do escopo deste tópico.

### Conectar o serviço a uma rede overlay

Você pode usar redes overlay para conectar um ou mais serviços dentro do swarm.

Primeiro, crie uma rede overlay em um nó manager usando o comando `docker network create` com a flag `--driver overlay`.

```console
$ docker network create --driver overlay my-network
```

Depois de criar uma rede overlay no modo swarm, todos os nós managers têm acesso à rede.

Você pode criar um novo serviço e passar a flag `--network` para anexar o serviço à rede overlay:

```console
$ docker service create \
  --replicas 3 \
  --network my-network \
  --name my-web \
  nginx
```

O swarm estende `my-network` para cada nó executando o serviço.

Você também pode conectar um serviço existente a uma rede overlay usando a flag `--network-add`.

```console
$ docker service update --network-add my-network my-web
```

Para desconectar um serviço em execução de uma rede, use a flag `--network-rm`.

```console
$ docker service update --network-rm my-network my-web
```

Para mais informações sobre redes overlay e descoberta de serviços, consulte
[Anexar serviços a uma rede overlay](https://docs.docker.com/engine/swarm/services/networking/) e
[Modelo de segurança de rede overlay do modo swarm Docker](/engine/network/drivers/overlay/).

### Conceder acesso a segredos a um serviço

Para criar um serviço com acesso a segredos gerenciados pelo Docker, use a flag `--secret`. Para mais informações, consulte
[Gerenciar strings sensíveis (segredos) para serviços Docker](https://docs.docker.com/engine/swarm/services/secrets/)

### Controlar o posicionamento do serviço

Os serviços Swarm oferecem algumas maneiras diferentes de controlar a escala e o posicionamento de serviços em nós diferentes.

- Você pode especificar se o serviço precisa executar um número específico de réplicas ou se deve ser executado globalmente em cada nó worker. Consulte
  [Serviços replicados ou globais](#serviços-replicados-ou-globais).

- Você pode configurar os
  [requisitos de CPU ou memória](#reservar-memória-ou-cpus-para-um-serviço) do serviço, e o serviço só será executado em nós que possam atender a esses requisitos.

- [Restrições de posicionamento](#restrições-de-posicionamento) permitem configurar o serviço para ser executado apenas em nós com metadados específicos (arbitrários) definidos e fazem a implantação falhar se nós apropriados não existirem. Por exemplo, você pode especificar que seu serviço deve ser executado apenas em nós onde um rótulo arbitrário `pci_compliant` está definido como `true`.

- [Preferências de posicionamento](#preferências-de-posicionamento) permitem aplicar um rótulo arbitrário com uma faixa de valores a cada nó e distribuir as tarefas do seu serviço entre esses nós usando um algoritmo. Atualmente, o único algoritmo suportado é `spread`, que tenta colocá-los uniformemente. Por exemplo, se você rotular cada nó com um rótulo `rack` que tem um valor de 1 a 10 e especificar uma preferência de posicionamento baseada em `rack`, as tarefas do serviço serão colocadas o mais uniformemente possível em todos os nós com o rótulo `rack`, após considerar outras restrições de posicionamento, preferências de posicionamento e outras limitações específicas do nó.

  Ao contrário das restrições, as preferências de posicionamento são de "melhor esforço", e um serviço não falha ao implantar se nenhum nó puder satisfazer a preferência. Se você especificar uma preferência de posicionamento para um serviço, os nós que correspondem a essa preferência são classificados mais alto quando os gerenciadores do swarm decidem quais nós devem executar as tarefas do serviço. Outros fatores, como alta disponibilidade do serviço, também influenciam quais nós são agendados para executar as tarefas do serviço. Por exemplo, se você tiver N nós com o rótulo rack (e alguns outros), e seu serviço estiver configurado para executar N+1 réplicas, a +1 será agendada em um nó que ainda não tenha o serviço, se houver um, independentemente de esse nó ter o rótulo `rack` ou não.

#### Serviços replicados ou globais

O modo Swarm tem dois tipos de serviços: replicados e globais. Para serviços replicados, você especifica o número de tarefas de réplica para o gerenciador do swarm agendar em nós disponíveis. Para serviços globais, o agendador coloca uma tarefa em cada nó disponível que atenda às
[restrições de posicionamento](#restrições-de-posicionamento) e
[requisitos de recursos](#reservar-memória-ou-cpus-para-um-serviço) do serviço.

Você controla o tipo de serviço usando a flag `--mode`. Se você não especificar um modo, o serviço padrão será `replicated`. Para serviços replicados, você especifica o número de tarefas de réplica que deseja iniciar usando a flag `--replicas`. Por exemplo, para iniciar um serviço nginx replicado com 3 tarefas de réplica:

```console
$ docker service create \
  --name my_web \
  --replicas 3 \
  nginx
```

Para iniciar um serviço global em cada nó disponível, passe `--mode global` para `docker service create`. Cada vez que um novo nó se tornar disponível, o agendador coloca uma tarefa para o serviço global no novo nó. Por exemplo, para iniciar um serviço que executa alpine em cada nó do swarm:

```console
$ docker service create \
  --name myservice \
  --mode global \
  alpine top
```

As restrições de serviço permitem definir critérios para um nó atender antes que o agendador implante um serviço no nó. Você pode aplicar restrições ao serviço com base em atributos e metadados do nó ou metadados do engine. Para mais informações sobre restrições, consulte a
[referência CLI](https://docs.docker.com/reference/cli/docker/service/create/) do `docker service create`.

#### Reservar memória ou CPUs para um serviço

Para reservar uma determinada quantidade de memória ou número de CPUs para um serviço, use as flags `--reserve-memory` ou `--reserve-cpu`. Se nenhum nó disponível puder satisfazer o requisito (por exemplo, se você solicitar 4 CPUs e nenhum nó no swarm tiver 4 CPUs), o serviço permanecerá em estado pendente até que um nó apropriado esteja disponível para executar suas tarefas.

##### Exceções de Falta de Memória (OOME)

Se seu serviço tentar usar mais memória do que o nó do swarm tem disponível, você pode experimentar uma Exceção de Falta de Memória (OOME) e um container, ou o daemon Docker, pode ser morto pelo OOM killer do kernel. Para evitar que isso aconteça, certifique-se de que sua aplicação seja executada em hosts com memória adequada e consulte
[Entender os riscos de ficar sem memória](https://docs.docker.com/engine/containers/resource_constraints/#understand-the-risks-of-running-out-of-memory).

Os serviços Swarm permitem usar restrições de recursos, preferências de posicionamento e rótulos para garantir que seu serviço seja implantado nos nós do swarm apropriados.

#### Restrições de posicionamento

Use restrições de posicionamento para controlar os nós aos quais um serviço pode ser atribuído. No exemplo a seguir, o serviço é executado apenas em nós com o
[rótulo](https://docs.docker.com/engine/swarm/services/manage-nodes/#add-or-remove-label-metadata) `region` definido como `east`. Se nenhum nó com o rótulo apropriado estiver disponível, as tarefas aguardarão em `Pending` até que se tornem disponíveis. A flag `--constraint` usa um operador de igualdade (`==` ou `!=`). Para serviços replicados, é possível que todos os serviços sejam executados no mesmo nó, ou que cada nó execute apenas uma réplica, ou que alguns nós não executem nenhuma réplica. Para serviços globais, o serviço é executado em cada nó que atenda à restrição de posicionamento e a quaisquer [requisitos de recursos](#reservar-memória-ou-cpus-para-um-serviço).

```console
$ docker service create \
  --name my-nginx \
  --replicas 5 \
  --constraint node.labels.region==east \
  nginx
```

Você também pode usar a chave `constraint` em nível de serviço em um arquivo `compose.yaml`.

Se você especificar múltiplas restrições de posicionamento, o serviço só será implantado em nós onde todas elas forem atendidas. O exemplo a seguir limita o serviço a ser executado em todos os nós onde `region` está definido como `east` e `type` não está definido como `devel`:

```console
$ docker service create \
  --name my-nginx \
  --mode global \
  --constraint node.labels.region==east \
  --constraint node.labels.type!=devel \
  nginx
```

Você também pode usar restrições de posicionamento em conjunto com preferências de posicionamento e restrições de CPU/memória. Tenha cuidado para não usar configurações que não possam ser cumpridas.

Para mais informações sobre restrições, consulte a
[referência CLI](https://docs.docker.com/reference/cli/docker/service/create/) do `docker service create`.

#### Preferências de posicionamento

Enquanto as [restrições de posicionamento](#restrições-de-posicionamento) limitam os nós em que um serviço pode ser executado, as _preferências de posicionamento_ tentam colocar tarefas em nós apropriados de maneira algorítmica (atualmente, apenas distribuição uniforme). Por exemplo, se você atribuir a cada nó um rótulo `rack`, poderá definir uma preferência de posicionamento para distribuir o serviço uniformemente entre os nós com o rótulo `rack`, por valor. Dessa forma, se você perder um rack, o serviço ainda estará sendo executado em nós em outros racks.

As preferências de posicionamento não são estritamente aplicadas. Se nenhum nó tiver o rótulo especificado na sua preferência, o serviço será implantado como se a preferência não estivesse definida.

> [!NOTE]
>
> As preferências de posicionamento são ignoradas para serviços globais.

O exemplo a seguir define uma preferência para distribuir a implantação entre os nós com base no valor do rótulo `datacenter`. Se alguns nós tiverem `datacenter=us-east` e outros tiverem `datacenter=us-west`, o serviço será implantado o mais uniformemente possível entre os dois conjuntos de nós.

```console
$ docker service create \
  --replicas 9 \
  --name redis_2 \
  --placement-pref 'spread=node.labels.datacenter' \
  redis:7.4.0
```

> [!NOTE]
>
> Nós que não possuem o rótulo usado para a distribuição ainda recebem atribuições de tarefas. Como grupo, esses nós recebem tarefas em proporção igual a qualquer um dos outros grupos identificados por um valor de rótulo específico. Em certo sentido, um rótulo ausente é o mesmo que ter o rótulo com um valor nulo anexado a ele. Se o serviço deve ser executado apenas em nós com o rótulo sendo usado para a preferência de distribuição, a preferência deve ser combinada com uma restrição.

Você pode especificar múltiplas preferências de posicionamento, e elas são processadas na ordem em que são encontradas. O exemplo a seguir configura um serviço com múltiplas preferências de posicionamento. As tarefas são distribuídas primeiro pelos vários datacenters e depois pelos racks (conforme indicado pelos respectivos rótulos):

```console
$ docker service create \
  --replicas 9 \
  --name redis_2 \
  --placement-pref 'spread=node.labels.datacenter' \
  --placement-pref 'spread=node.labels.rack' \
  redis:7.4.0
```

Você também pode usar preferências de posicionamento em conjunto com restrições de posicionamento ou restrições de CPU/memória. Tenha cuidado para não usar configurações que não possam ser cumpridas.

Este diagrama ilustra como as preferências de posicionamento funcionam:

![Como as preferências de posicionamento funcionam](https://docs.docker.com/engine/swarm/services/images/placement_prefs.png)

Ao atualizar um serviço com `docker service update`, `--placement-pref-add` anexa uma nova preferência de posicionamento após todas as preferências de posicionamento existentes. `--placement-pref-rm` remove uma preferência de posicionamento existente que corresponda ao argumento.

### Configurar o comportamento de atualização de um serviço

Ao criar um serviço, você pode especificar um comportamento de atualização contínua (rolling update) para como o swarm deve aplicar alterações ao serviço quando você executa `docker service update`. Você também pode especificar essas flags como parte da atualização, como argumentos para `docker service update`.

A flag `--update-delay` configura o atraso de tempo entre as atualizações de uma tarefa de serviço ou conjuntos de tarefas. Você pode descrever o tempo `T` como uma combinação do número de segundos `Ts`, minutos `Tm` ou horas `Th`. Portanto, `10m30s` indica um atraso de 10 minutos e 30 segundos.

Por padrão, o agendador atualiza 1 tarefa por vez. Você pode passar a flag `--update-parallelism` para configurar o número máximo de tarefas de serviço que o agendador atualiza simultaneamente.

Quando uma atualização de uma tarefa individual retorna um estado de `RUNNING`, o agendador continua a atualização prosseguindo para outra tarefa até que todas as tarefas sejam atualizadas. Se, a qualquer momento durante uma atualização, uma tarefa retornar `FAILED`, o agendador pausa a atualização. Você pode controlar o comportamento usando a flag `--update-failure-action` para `docker service create` ou `docker service update`.

No exemplo de serviço abaixo, o agendador aplica atualizações a no máximo 2 réplicas por vez. Quando uma tarefa atualizada retorna `RUNNING` ou `FAILED`, o agendador aguarda 10 segundos antes de parar a próxima tarefa para atualizar:

```console
$ docker service create \
  --replicas 10 \
  --name my_web \
  --update-delay 10s \
  --update-parallelism 2 \
  --update-failure-action continue \
  alpine
```

A flag `--update-max-failure-ratio` controla qual fração de tarefas pode falhar durante uma atualização antes que a atualização como um todo seja considerada falha. Por exemplo, com `--update-max-failure-ratio 0.1 --update-failure-action pause`, após 10% das tarefas sendo atualizadas falharem, a atualização é pausada.

Uma atualização de tarefa individual é considerada falha se a tarefa não iniciar ou se parar de ser executada dentro do período de monitoramento especificado com a flag `--update-monitor`. O valor padrão para `--update-monitor` é 30 segundos, o que significa que uma tarefa falhando nos primeiros 30 segundos após ser iniciada conta para o limite de falha da atualização do serviço, e uma falha após esse período não é contada.

### Reverter para a versão anterior de um serviço

Caso a versão atualizada de um serviço não funcione conforme o esperado, é possível reverter manualmente para a versão anterior do serviço usando a flag `--rollback` do `docker service update`. Isso reverte o serviço para a configuração que estava vigente antes do comando `docker service update` mais recente.

Outras opções podem ser combinadas com `--rollback`; por exemplo, `--update-delay 0s`, para executar a reversão sem atraso entre as tarefas:

```console
$ docker service update \
  --rollback \
  --update-delay 0s
  my_web
```

Você pode configurar um serviço para reverter automaticamente se uma atualização do serviço falhar ao implantar. Consulte [Reverter automaticamente se uma atualização falhar](#reverter-automaticamente-se-uma-atualização-falhar).

A reversão manual é tratada no lado do servidor, o que permite que reversões iniciadas manualmente respeitem os novos parâmetros de reversão. Observe que `--rollback` não pode ser usado em conjunto com outras flags para `docker service update`.

### Reverter automaticamente se uma atualização falhar

Você pode configurar um serviço de tal forma que, se uma atualização do serviço causar falha na reimplantação, o serviço possa reverter automaticamente para a configuração anterior. Isso ajuda a proteger a disponibilidade do serviço. Você pode definir uma ou mais das seguintes flags na criação ou atualização do serviço. Se você não definir um valor, o padrão será usado.

| Flag                           | Padrão | Descrição                                                                                                                                                                                                                                                                                                             |
|:-------------------------------|:--------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `--rollback-delay`             | `0s`    | Quantidade de tempo para aguardar após reverter uma tarefa antes de reverter a próxima. Um valor de `0` significa reverter a segunda tarefa imediatamente após a primeira tarefa revertida ser implantada.                                                                                                                              |
| `--rollback-failure-action`    | `pause` | Quando uma tarefa falha ao reverter, se deve `pausar` ou `continuar` tentando reverter outras tarefas.                                                                                                                                                                                                                       |
| `--rollback-max-failure-ratio` | `0`     | A taxa de falha a ser tolerada durante uma reversão, especificada como um número de ponto flutuante entre 0 e 1. Por exemplo, dadas 5 tarefas, uma taxa de falha de `.2` toleraria uma tarefa falhando ao reverter. Um valor de `0` significa que nenhuma falha é tolerada, enquanto um valor de `1` significa que qualquer número de falhas é tolerado. |
| `--rollback-monitor`           | `5s`    | Duração após cada reversão de tarefa para monitorar falhas. Se uma tarefa parar antes que esse período tenha decorrido, a reversão é considerada falha.                                                                                                                                                               |
| `--rollback-parallelism`       | `1`     | O número máximo de tarefas a serem revertidas em paralelo. Por padrão, uma tarefa é revertida por vez. Um valor de `0` faz com que todas as tarefas sejam revertidas em paralelo.                                                                                                                                                     |

O exemplo a seguir configura um serviço `redis` para reverter automaticamente se um `docker service update` falhar ao implantar. Duas tarefas podem ser revertidas em paralelo. As tarefas são monitoradas por 20 segundos após a reversão para garantir que não saiam, e uma taxa máxima de falha de 20% é tolerada. Valores padrão são usados para `--rollback-delay` e `--rollback-failure-action`.

```console
$ docker service create --name=my_redis \
                        --replicas=5 \
                        --rollback-parallelism=2 \
                        --rollback-monitor=20s \
                        --rollback-max-failure-ratio=.2 \
                        redis:latest
```

### Dar acesso a volumes ou bind mounts a um serviço

Para melhor desempenho e portabilidade, você deve evitar escrever dados importantes diretamente na camada gravável de um container. Em vez disso, você deve usar volumes de dados ou bind mounts. Este princípio também se aplica a serviços.

Você pode criar dois tipos de montagens para serviços em um swarm, montagens de `volume` ou montagens de `bind` (vinculação). Independentemente do tipo de montagem que você usar, configure-a usando a flag `--mount` ao criar um serviço, ou as flags `--mount-add` ou `--mount-rm` ao atualizar um serviço existente. O padrão é um volume de dados se você não especificar um tipo.

#### Volumes de dados

Volumes de dados são armazenamento que existem independentemente de um container. O ciclo de vida dos volumes de dados sob serviços swarm é semelhante ao de containers. Os volumes sobrevivem às tarefas e serviços, portanto, sua remoção deve ser gerenciada separadamente. Os volumes podem ser criados antes de implantar um serviço ou, se não existirem em um host específico quando uma tarefa for agendada lá, eles serão criados automaticamente de acordo com a especificação de volume no serviço.

Para usar volumes de dados existentes com um serviço, use a flag `--mount`:

```console
$ docker service create \
  --mount src=<NOME-DO-VOLUME>,dst=<CAMINHO-NO-CONTAINER> \
  --name myservice \
  <IMAGEM>
```

Se um volume com o nome `<NOME-DO-VOLUME>` não existir quando uma tarefa for agendada para um host específico, então um será criado. O driver de volume padrão é `local`. Para usar um driver de volume diferente com este padrão de criação sob demanda, especifique o driver e suas opções com a flag `--mount`:

```console
$ docker service create \
  --mount type=volume,src=<NOME-DO-VOLUME>,dst=<CAMINHO-NO-CONTAINER>,volume-driver=<DRIVER>,volume-opt=<CHAVE0>=<VALOR0>,volume-opt=<CHAVE1>=<VALOR1>
  --name myservice \
  <IMAGEM>
```

Para mais informações sobre como criar volumes de dados e o uso de drivers de volume, consulte [Usar volumes](https://docs.docker.com/engine/storage/volumes/).

#### Bind mounts

Bind mounts são caminhos do sistema de arquivos do host onde o agendador implanta o container para a tarefa. O Docker monta o caminho dentro do container. O caminho do sistema de arquivos deve existir antes que o swarm inicialize o container para a tarefa.

Os exemplos a seguir mostram a sintaxe de bind mount:

- Para montar um bind de leitura e escrita:

  ```console
  $ docker service create \
    --mount type=bind,src=<CAMINHO-DO-HOST>,dst=<CAMINHO-NO-CONTAINER> \
    --name myservice \
    <IMAGEM>
  ```

- Para montar um bind somente leitura:

  ```console
  $ docker service create \
    --mount type=bind,src=<CAMINHO-DO-HOST>,dst=<CAMINHO-NO-CONTAINER>,readonly \
    --name myservice \
    <IMAGEM>
  ```

> [!IMPORTANT]
>
> Bind mounts podem ser úteis, mas também podem causar problemas. Na maioria dos casos, é recomendado que você arquitete sua aplicação de forma que a montagem de caminhos do host seja desnecessária. Os principais riscos incluem o seguinte:
>
> - Se você montar um caminho do host nos containers do seu serviço, o caminho deve existir em cada nó do swarm. O agendador do modo Docker swarm pode agendar containers em qualquer máquina que atenda aos requisitos de disponibilidade de recursos e satisfaça todas as restrições e preferências de posicionamento que você especificar.
>
> - O agendador do modo Docker swarm pode reagendar seus containers de serviço em execução a qualquer momento se eles se tornarem não íntegros ou inacessíveis.
>
> - Bind mounts do host não são portáteis. Quando você usa bind mounts, não há garantia de que sua aplicação funcione da mesma forma no desenvolvimento quanto na produção.

### Criar serviços usando templates

Você pode usar templates para algumas flags de `service create`, usando a sintaxe fornecida pelo pacote [text/template](https://golang.org/pkg/text/template/) do Go.

As seguintes flags são suportadas:

- `--hostname`
- `--mount`
- `--env`

Os placeholders válidos para o template Go são:

| Placeholder       | Descrição    |
|:------------------|:---------------|
| `.Service.ID`     | ID do Serviço     |
| `.Service.Name`   | Nome do Serviço   |
| `.Service.Labels` | Rótulos do Serviço |
| `.Node.ID`        | ID do Nó        |
| `.Node.Hostname`  | Hostname do Nó  |
| `.Task.Name`      | Nome da Tarefa      |
| `.Task.Slot`      | Slot da Tarefa      |

#### Exemplo de template

Este exemplo define o template dos containers criados com base no nome do serviço e no ID do nó onde o container está sendo executado:

```console
$ docker service create --name hosttempl \
                        --hostname="{{.Node.ID}}-{{.Service.Name}}"\
                         busybox top
```

Para ver o resultado do uso do template, use os comandos `docker service ps` e `docker inspect`.

```console
$ docker service ps va8ew30grofhjoychbr6iot8c

ID            NAME         IMAGE                                                                                   NODE          DESIRED STATE  CURRENT STATE               ERROR  PORTS
wo41w8hg8qan  hosttempl.1  busybox:latest@sha256:29f5d56d12684887bdfa50dcd29fc31eea4aaf4ad3bec43daf19026a7ce69912  2e7a8a9c4da2  Running        Running about a minute ago
```

```console
$ docker inspect --format="{{.Config.Hostname}}" hosttempl.1.wo41w8hg8qanxwjwsg4kxpprj
```

## Saiba Mais

* [Guia de administração do Swarm](https://docs.docker.com/engine/swarm/services/admin_guide/)
* [Referência de linha de comando do Docker Engine](https://docs.docker.com/reference/cli/docker/)
* [Tutorial do modo Swarm](https://docs.docker.com/engine/swarm/services/swarm-tutorial/)