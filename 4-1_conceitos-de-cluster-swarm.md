# Conceitos-chave do modo Swarm

Este tópico introduz alguns dos conceitos exclusivos dos recursos de gerenciamento de cluster e orquestração do Docker Engine 1.12.

## O que é um swarm?

Os recursos de gerenciamento de cluster e orquestração incorporados ao Docker Engine são construídos usando o [swarmkit](https://github.com/docker/swarmkit/). O Swarmkit é um projeto separado que implementa a camada de orquestração do Docker e é usado diretamente dentro do Docker.

Um swarm consiste em múltiplos hosts Docker que executam no modo Swarm e atuam como **gerenciadores** (managers), para gerenciar a associação e delegação, e **trabalhadores** (workers), que executam [serviços do swarm](#serviços-e-tarefas). Um determinado host Docker pode ser um gerenciador, um trabalhador ou desempenhar ambos os papéis. Quando você cria um serviço, define seu estado ideal — número de réplicas, recursos de rede e armazenamento disponíveis para ele, portas que o serviço expõe ao mundo externo e muito mais. O Docker trabalha para manter esse estado desejado. Por exemplo, se um nó trabalhador se tornar indisponível, o Docker agenda as tarefas desse nó em outros nós. Uma **tarefa** é um container em execução que faz parte de um serviço do swarm e é gerenciado por um gerenciador do swarm, diferentemente de um container autônomo.

Uma das principais vantagens dos serviços do swarm sobre containers autônomos é que você pode modificar a configuração de um serviço, incluindo as redes e volumes aos quais está conectado, sem a necessidade de reiniciar manualmente o serviço. O Docker atualizará a configuração, interromperá as tarefas do serviço com configuração desatualizada e criará novas tarefas que correspondam à configuração desejada.

Quando o Docker está executando no modo Swarm, você ainda pode executar containers autônomos em qualquer um dos hosts Docker que participam do swarm, bem como serviços do swarm. Uma diferença fundamental entre containers autônomos e serviços do swarm é que apenas os gerenciadores do swarm podem gerenciar um swarm, enquanto containers autônomos podem ser iniciados em qualquer daemon. Daemons Docker podem participar de um swarm como gerenciadores, trabalhadores ou ambos.

Da mesma forma que você pode usar o [Docker Compose](2-4_o-que-e-docker-composer.md) para definir e executar containers, você pode definir e executar pilhas de [serviços do Swarm](https://docs.docker.com/engine/swarm/key-concepts/services/).

Continue lendo para obter detalhes sobre conceitos relacionados aos serviços do Docker swarm, incluindo nós, serviços, tarefas e balanceamento de carga.

## Nós (Nodes)

Um nó é uma instância do Docker engine participando do swarm. Você também pode pensar nisso como um nó Docker. Você pode executar um ou mais nós em um único computador físico ou servidor na nuvem, mas implantações de swarm em produção normalmente incluem nós Docker distribuídos em várias máquinas físicas e na nuvem.

Para implantar seu aplicativo em um swarm, você envia uma definição de serviço para um nó gerenciador. O nó gerenciador despacha unidades de trabalho chamadas [tarefas](#serviços-e-tarefas) para os nós trabalhadores.

Os nós gerenciadores também realizam as funções de orquestração e gerenciamento de cluster necessárias para manter o estado desejado do swarm. Os nós gerenciadores selecionam um único líder para conduzir as tarefas de orquestração.

Os nós trabalhadores recebem e executam tarefas despachadas pelos nós gerenciadores. Por padrão, os nós gerenciadores também executam serviços como nós trabalhadores, mas você pode configurá-los para executar exclusivamente tarefas de gerenciador e serem nós apenas de gerenciador (manager-only). Um agente é executado em cada nó trabalhador e relata as tarefas atribuídas a ele. O nó trabalhador notifica o nó gerenciador sobre o estado atual de suas tarefas atribuídas, para que o gerenciador possa manter o estado desejado de cada trabalhador.

## Serviços e Tarefas

Um serviço é a definição das tarefas a serem executadas nos nós gerenciadores ou trabalhadores. É a estrutura central do sistema swarm e a principal raiz da interação do usuário com o swarm.

Quando você cria um serviço, especifica qual imagem de container usar e quais comandos executar dentro dos containers em execução.

No modelo de **serviços replicados**, o gerenciador do swarm distribui um número específico de tarefas réplicas entre os nós com base na escala definida no estado desejado.

Para **serviços globais**, o swarm executa uma tarefa para o serviço em cada nó disponível no cluster.

Uma tarefa carrega um container Docker e os comandos a serem executados dentro do container. É a unidade atômica de agendamento do swarm. Os nós gerenciadores atribuem tarefas aos nós trabalhadores de acordo com o número de réplicas definido na escala do serviço. Uma vez que uma tarefa é atribuída a um nó, ela não pode se mover para outro nó. Ela só pode ser executada no nó atribuído ou falhar.

## Balanceamento de Carga

O gerenciador do swarm usa o balanceamento de carga de entrada (ingress) para expor os serviços que você deseja disponibilizar externamente ao swarm. O gerenciador do swarm pode atribuir automaticamente uma porta publicada ao serviço ou você pode configurar uma porta publicada para o serviço. Você pode especificar qualquer porta não utilizada. Se não especificar uma porta, o gerenciador do swarm atribuirá ao serviço uma porta no intervalo 30000-32767.

Componentes externos, como balanceadores de carga na nuvem, podem acessar o serviço na porta publicada de qualquer nó do cluster, independentemente de o nó estar executando atualmente a tarefa para o serviço. Todos os nós no swarm roteiam conexões de entrada para uma instância de tarefa em execução.

O modo Swarm possui um componente DNS interno que atribui automaticamente a cada serviço no swarm uma entrada DNS. O gerenciador do swarm usa o balanceamento de carga interno para distribuir solicitações entre os serviços dentro do cluster com base no nome DNS do serviço.

## Referências

* [docs.docker.com/engine/swarm/key-concepts](https://docs.docker.com/engine/swarm/key-concepts)

## Próximos passos

* Comece com o [tutorial do modo Swarm](https://docs.docker.com/engine/swarm/key-concepts/swarm-tutorial/).