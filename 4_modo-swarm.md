# Módulo 4: Modo Swarm

As versões atuais do Docker incluem o modo Swarm para gerenciar nativamente um cluster de Engines Docker chamado de *swarm*. Use a CLI do Docker para criar um swarm, fazer deploy de serviços de aplicação no swarm e gerenciar o comportamento do swarm.

## Destaques dos recursos

### Gerenciamento de cluster integrado ao Docker Engine

Use a CLI do Docker Engine para criar um swarm de Engines Docker onde você pode fazer deploy de serviços de aplicação. Você não precisa de software de orquestração adicional para criar ou gerenciar um swarm.

### Design descentralizado

Em vez de lidar com a diferenciação entre funções de nó (node roles) no momento do deployment, o Docker Engine lida com qualquer especialização em tempo de execução. Você pode fazer deploy de ambos os tipos de nós, gerenciadores (managers) e trabalhadores (workers), usando o Docker Engine. Isso significa que você pode construir um swarm inteiro a partir de uma única imagem de disco.

### Modelo de serviço declarativo

O Docker Engine usa uma abordagem declarativa para permitir que você defina o estado desejado dos vários serviços na sua pilha de aplicações. Por exemplo, você pode descrever uma aplicação composta por um serviço de front-end web com serviços de fila de mensagens e um back-end de banco de dados.

### Escalabilidade

Para cada serviço, você pode declarar o número de tarefas que deseja executar. Quando você escala para cima ou para baixo, o gerenciador do swarm se adapta automaticamente adicionando ou removendo tarefas para manter o estado desejado.

### Reconciliação do estado desejado

O nó gerenciador do swarm monitora constantemente o estado do cluster e reconcilia quaisquer diferenças entre o estado real e o estado desejado expresso por você. Por exemplo, se você configurar um serviço para executar 10 réplicas de um container e uma máquina worker que hospeda duas dessas réplicas falhar, o gerenciador cria duas novas réplicas para substituir as que falharam. O gerenciador do swarm atribui as novas réplicas aos workers que estão em execução e disponíveis.

### Rede multi-host

Você pode especificar uma rede overlay para seus serviços. O gerenciador do swarm atribui automaticamente endereços aos containers na rede overlay quando inicializa ou atualiza a aplicação.

### Descoberta de serviços (Service Discovery)

Os nós gerenciadores do swarm atribuem a cada serviço no swarm um nome DNS único e fazem balanceamento de carga dos containers em execução. Você pode consultar cada container em execução no swarm através de um servidor DNS embutido no swarm.

### Balanceamento de carga

Você pode expor as portas dos serviços para um balanceador de carga externo. Internamente, o swarm permite que você especifique como distribuir os containers de serviço entre os nós.

### Seguro por padrão

Cada nó no swarm impõe autenticação mútua TLS e criptografia para proteger as comunicações entre si e todos os outros nós. Você tem a opção de usar certificados raiz autoassinados ou certificados de uma Autoridade Certificadora (CA) raiz personalizada.

### Atualizações contínuas (Rolling Updates)

No momento do rollout, você pode aplicar atualizações de serviço aos nós incrementalmente. O gerenciador do swarm permite que você controle o atraso entre o deployment do serviço em diferentes conjuntos de nós. Se algo der errado, você pode reverter (rollback) para uma versão anterior do serviço.

## Referências

[docs.docker.com/engine/swarm/](https://docs.docker.com/engine/swarm/)

* Explore os comandos da CLI do modo Swarm
  * [swarm init](https://docs.docker.com/reference/cli/docker/swarm/init/)
  * [swarm join](https://docs.docker.com/reference/cli/docker/swarm/join/)
  * [service create](https://docs.docker.com/reference/cli/docker/service/create/)
  * [service inspect](https://docs.docker.com/reference/cli/docker/service/inspect/)
  * [service ls](https://docs.docker.com/reference/cli/docker/service/ls/)
  * [service rm](https://docs.docker.com/reference/cli/docker/service/rm/)
  * [service scale](https://docs.docker.com/reference/cli/docker/service/scale/)
  * [service ps](https://docs.docker.com/reference/cli/docker/service/ps/)
  * [service update](https://docs.docker.com/reference/cli/docker/service/update/)

## Próximos passos?

* Aprenda os [conceitos-chave](4-1_conceitos-de-cluster-swarm.md) do modo Swarm.
* Comece com o [tutorial do modo Swarm](https://docs.docker.com/engine/swarm/swarm-tutorial/).