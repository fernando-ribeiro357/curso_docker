###  Módulo 1: Conhecendo o Docker
*Objetivo: Entender o problema que o Docker resolve e as tecnologias subjacentes.*

1.  [**Fundamentos e Contexto Histórico**](1-1_fundamentos-e-contexto-historico-do-docker.md)
    * O clássico "funciona na minha máquina".
    * Diferenças entre Desenvolvimento, Teste e Produção.
    * A Evolução da Virtualização
    * Limitações das VMs: Overhead de recursos, lentidão no boot, consumo de disco.

2.  [**O que é o Docker**](1-2_o-que-e-o-docker.md)
    * A plataforma Docker
    * Para que posso usar o Docker?
    * Arquitetura do Docker
    * Tecnologias que Permitiram o Docker (Linux Kernel)
    

---

### Módulo 2: Operações Básicas e Ciclo de Vida
*Objetivo: Construir, executar e gerenciar containers individuais.*

1.  [**Gerenciamento de Containers**](2-1_o-que-e-um-container.md)
    *   Ciclo de vida: `run`, `start`, `stop`, `restart`, `rm`.
    *   Inspeção: `docker ps`, `docker inspect`, `docker logs`, `docker exec`.
    *   Limpeza: `docker system prune`.
2.  [**Imagens e Dockerfile**](2-2_o-que-e-uma-imagem.md)
    * O que é uma imagem?
    * Pesquise e baixe uma imagem: `docker search`, `docker pull`.
    * Aprenda sobre a imagem: `docker image ls`, `docker image history`.
3.  [**Registro de Imagens (Registry)**](2-3_o-que-e-um-registro.md)
    * O que é um Registry?
    * Tipos de Registries: públicos (Docker Hub, GitHub Container Registry, Quay.io) ou privados (ex: usando a imagem oficial registry:2, Harbor, Nexus, GitLab Registry).
    * Nomenclatura e Tags: Estrutura do nome da imagem (`[registry-host]/[namespace]/[repository]:[tag]`).
    * Sintaxe básica do `Dockerfile` (`FROM`, `RUN`, `COPY`, `CMD`, `ENTRYPOINT`, `ENV`, `EXPOSE`).
    * Melhores práticas: Camadas (Layers), cache de build, redução de tamanho de imagem (Multi-stage builds).
    * Comandos: `docker build`, `docker tag`, `docker push/pull`.
4.  [**Docker Compose**](2-4_o-que-e-docker-composer.md)
    * O que é Docker Compose: explicação; Dockerfile versus arquivo Compose.
    * Experimente: inicie uma aplicação; desmonte a aplicação.

---

### Módulo 3: Persistência, Configuração e Segurança Básica
*Objetivo: Gerenciar dados stateful e segredos fora do código da aplicação.*

1.  [**Persistindo dados de container**](3-1_persistindo-dados-de-container.md)
    *   Volumes de container:
        *   Gerenciando volumes
        *   Usando volumes
        *   Visualizando o conteúdo do volume
        *   Removendo volumes
    *   [Armazenamento (Storage Drivers & Volumes)](_armazenamento.md)
        *   Tipos de montagem:volumes, bind mount e tmpfs.
        *   Comandos: `docker volume create`, `ls`, `inspect`, `rm`.
        *   Estratégia de backup de volumes.
2.  [**Rede em Single Node**](3-2_rede-single-node.md)
    *   Drivers de rede padrão: `bridge`, `host`, `none`.
    *   Criação de redes customizadas (`docker network create`).
    *   DNS interno do Docker (resolução de nomes entre containers na mesma rede).
    *   Isolamento de rede e exposição seletiva.
3.  [**Configs e Secrets (Introdução)**](3-3_configs-e-secrets.md)
    *   Por que não usar variáveis de ambiente para tudo?
    *   Conceito de **Docker Configs** (arquivos de configuração não sensíveis).
    *   Conceito de **Docker Secrets** (senhas, chaves API, certificados).
    *   *Nota:* Embora funcionem em modo standalone com limitações, seu uso pleno será explorado no Swarm.

---

###  Módulo 4: Orquestração com Docker Swarm
*Objetivo: Transformar múltiplos hosts em um único cluster virtualizado.*

1.  **Conceitos de Cluster Swarm**
    *   Arquitetura: Manager Nodes vs. Worker Nodes.
    *   Raft Consensus Algorithm (tolerância a falhas dos managers).
    *   Inicialização: `docker swarm init` e `docker swarm join`.
2.  **Serviços (Services) vs. Containers**
    *   Modelo declarativo: Definir o estado desejado (`replicas`, `image`, `ports`).
    *   Comandos: `docker service create`, `ls`, `ps`, `scale`, `update`, `rollback`.
    *   Estratégias de atualização: `rolling update` (atualização contínua sem downtime) e `global` (um container por nó).
3.  **Rede no Swarm (Overlay Network)**
    *   Criando redes Overlay para comunicação entre nós diferentes.
    *   Balanceamento de carga interno (Routing Mesh): Como o tráfego chega a qualquer nó e é roteado para o container correto, mesmo que ele esteja em outro nó.
    *   Publicação de portas no modo `host` vs. modo `ingress`.
4.  **Gestão Avançada de Configs e Secrets no Swarm**
    *   Criando e atualizando secrets/configs no nível do cluster.
    *   Montagem segura em serviços (arquivos temporários em `/run/secrets`).
    *   Rotação de segredos sem reiniciar o serviço manualmente (atualização rolling).
5.  **Escalonamento e Restrições**
    *   Labels e Constraints: Rodar serviços apenas em nós específicos (ex: `node.role == manager` ou `disktype == ssd`).
    *   Resource Limits no Swarm: Definir CPU/Memória máxima e garantida por serviço.
    *   Placement Preferences: Preferências de espalhamento (spread) ou concentração (binpack).

---

### Módulo 5: Administração de Cluster em Escala (Dezenas de Serviços)
*Objetivo: Operar, manter e otimizar um ambiente produtivo complexo.*

1.  **Governança de Imagens em Cluster**
    *   Implementação de Registry Privado (ex: Harbor ou Registry local com TLS).
    *   Políticas de retenção de imagens antigas nos nós workers.
2.  **Segurança do Cluster**
    *   Gerenciamento de certificados TLS mútuos entre nós.
    *   Token de junção (rotation de tokens).
    *   Hardening do Daemon Docker (`daemon.json`).
    *   User Namespaces e Seccomp profiles.
3.  **Manutenção e Atualização do Cluster**
    *   Atualização do Docker Engine em nós vivos (Drain node -> Update -> Uncordon).
    *   Adição e remoção dinâmica de nós workers.
    *   Recuperação de desastre: O que acontece se perdermos a maioria dos Managers? (Backup do diretório `/var/lib/docker/swarm`).
4.  **Logs Centralizados**
    *   O problema dos logs distribuídos em dezenas de nós.
    *   Configuração de drivers de log (`json-file`, `syslog`, `fluentd`, `gelf`).
    *   Encaminhamento de logs para agregadores externos.

---

### Módulo 6: Monitoramento e Observabilidade
*Objetivo: Visibilidade total sobre a saúde dos serviços e do cluster.*

1.  **Métricas Nativas e Exportação**
    *   Uso do `docker stats` (limitado para produção).
    *   Habilitação da API de métricas do Docker (`--metrics-addr`).
2.  **Stack de Monitoramento (Prometheus + Grafana)**
    *   Deploy do **Prometheus** como um serviço no Swarm.
    *   Configuração do **cAdvisor** ou **Docker Exporter** para coletar métricas de containers.
    *   Criação de Dashboards no **Grafana**:
        *   Uso de CPU/Memória por serviço.
        *   Taxa de restarts de containers.
        *   Latência de rede entre serviços.
3.  **Alertas e Healthchecks**
    *   Configuração de `HEALTHCHECK` no Dockerfile ou no Service definition.
    *   Integração do Prometheus Alertmanager (enviar alertas para Slack/Email/PagerDuty se um serviço cair ou consumir muitos recursos).
4.  **Logging Stack (ELK ou Loki)**
    *   Implementação de **Loki** (leve) ou **ELK Stack** (Elasticsearch, Logstash, Kibana) rodando no Swarm.
    *   Correlação de logs entre múltiplos serviços de uma mesma transação (Trace ID).

---

###  Projeto Prático Sugerido para Fixação

Para consolidar todo esse conhecimento, sugiro construir o seguinte cenário:

1.  Suba 3 VMs (pode ser VirtualBox, AWS EC2 ou DigitalOcean).
2.  Instale Docker e forme um **Swarm Cluster** (1 Manager, 2 Workers).
3.  Crie uma aplicação composta por:
    *   Frontend (Nginx/React).
    *   Backend (Node.js/Python/Go).
    *   Banco de Dados (Postgres com Volume persistente).
    *   Cache (Redis).
4.  Configure uma **Rede Overlay**.
5.  Use **Secrets** para a senha do banco de dados e **Configs** para arquivos de configuração do app.
6.  Defina restrições para que o Banco de Dados rode apenas em um nó específico com label `db-node=true`.
7.  Simule falha: Derrube o nó worker onde o backend está rodando e observe o Swarm recriando o serviço automaticamente em outro nó.
8.  Implante **Prometheus/Grafana** para monitorar o cluster e crie um dashboard mostrando o uso de recursos de cada serviço.

### Recursos Recomendados
*   **Documentação Oficial**: [docs.docker.com](https://docs.docker.com/) (sempre a fonte mais atualizada).

