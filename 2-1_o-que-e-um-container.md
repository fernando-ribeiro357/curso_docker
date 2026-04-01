# **Módulo 2.1: Gerenciamento de Containers**

## *O que é um container?*

Imagine que você está desenvolvendo um aplicativo web revolucionário que possui três componentes principais: um frontend em React, uma API em Python e um banco de dados PostgreSQL. Se você quisesse trabalhar neste projeto, teria que instalar Node, Python e PostgreSQL.

Como garantir que você tenha as mesmas versões que os outros desenvolvedores da sua equipe? Ou o seu sistema de CI/CD? Ou o que é usado em produção?

Como garantir que a versão do Python (ou Node ou do banco de dados) que seu aplicativo precisa não seja afetada pelo que já está instalado na sua máquina? Como gerenciar conflitos potenciais?

É aí que entram os containers!

O que é um container? Simplificando, containers são processos isolados para cada um dos componentes do seu aplicativo. Cada componente — o aplicativo frontend React, o motor da API Python e o banco de dados — roda em seu próprio ambiente isolado, completamente separado de tudo o mais na sua máquina.

Aqui está o que os torna incríveis. Os containers são:

- **Autocontidos:** Cada container tem tudo o que precisa para funcionar, sem depender de nenhuma dependência pré-instalada na máquina hospedeira.
- **Isolados:** Como os containers rodam em isolamento, eles têm influência mínima no host e em outros containers, aumentando a segurança de suas aplicações.
- **Independentes:** Cada container é gerenciado de forma independente. Excluir um container não afetará nenhum outro.
- **Portáteis:** Os containers podem rodar em qualquer lugar! O container que roda na sua máquina de desenvolvimento funcionará da mesma maneira em um data center ou em qualquer lugar na nuvem!

### Containers versus Máquinas Virtuais (VMs)

Sem entrar em muitos detalhes técnicos, uma VM é um sistema operacional inteiro com seu próprio kernel, drivers de hardware, programas e aplicativos. Criar uma VM apenas para isolar um único aplicativo gera muita sobrecarga.

Um container é simplesmente um processo isolado com todos os arquivos necessários para rodar. Se você executar múltiplos containers, todos eles compartilham o mesmo kernel, permitindo que você execute mais aplicações com menos infraestrutura.

> **Usando VMs e containers juntos**
>
> Frequentemente, você verá containers e VMs sendo usados juntos. Por exemplo, em um ambiente de nuvem, as máquinas provisionadas são tipicamente VMs. No entanto, em vez de provisionar uma máquina para rodar um aplicativo, uma VM com um runtime de container pode executar múltiplas aplicações conteinerizadas, aumentando a utilização de recursos e reduzindo custos.

## Experimente

**Usando a CLI (Linha de Comando)**

Siga as instruções para executar um container usando a CLI:

1. Abra seu terminal CLI e inicie um container usando o comando [`docker run`](https://docs.docker.com/reference/cli/docker/container/run/):

    ```console
    $ docker run -d -p 8080:80 docker/welcome-to-docker
    ```

    A saída deste comando é o ID completo do container.


> **Aqui está a explicação detalhada de cada parte do comando:**
> * `docker run`: É o comando principal que cria e inicia um novo container a partir de uma imagem.
> * `-d` (ou `--detach`): Executa o container em **modo desacoplado** (background). Isso significa que o terminal não ficará "preso" aguardando o processo do container terminar; ele retorna imediatamente ao prompt de comando, deixando o container rodando silenciosamente.
> * `-p 8080:80`: Define o **mapeamento de portas** (Port Mapping).
>   * O primeiro número (**8080**) é a porta no seu **computador host** (sua máquina).
>   * O segundo número (**80**) é a porta dentro do **container**.
>   * **O que isso faz:** Qualquer tráfego que chegar na porta `8080` do seu computador será redirecionado automaticamente para a porta `80` dentro do container. Como a aplicação dentro da imagem `welcome-to-docker` (que é um servidor web Nginx) escuta na porta 80, você consegue acessá-la digitando `http://localhost:8080` no seu navegador.
>
> *  `docker/welcome-to-docker`: É o nome da **imagem** que será usada para criar o container. Se essa imagem não estiver baixada localmente, o Docker a buscará automaticamente no Docker Hub antes de iniciar o container.

Parabéns! Você acabou de iniciar seu primeiro container! 🎉

### Visualize seus containers em execução

Você pode verificar se o container está ativo e rodando usando o comando [`docker ps`](https://docs.docker.com/reference/cli/docker/container/ls/):

```console
docker ps
```

Você verá uma saída semelhante à seguinte:

```console
 CONTAINER ID   IMAGE                      COMMAND                  CREATED          STATUS          PORTS                      NAMES
 a1f7a4bb3a27   docker/welcome-to-docker   "/docker-entrypoint.…"   11 seconds ago   Up 11 seconds   0.0.0.0:8080->80/tcp       gracious_keldysh
```

Este container executa um servidor web que exibe um site simples. Ao trabalhar com projetos mais complexos, você executará partes diferentes em containers diferentes. Por exemplo, um container diferente para o `frontend`, `backend` e `database`.

> [!DICA]
>
> O comando `docker ps` mostrará _apenas_ os containers em execução. Para visualizar containers parados, adicione a flag `-a` para listar todos os containers: `docker ps -a`

### Acesse o frontend

Quando você lançou o container, expôs uma das portas do container na sua máquina. Pense nisso como criar uma configuração que permite conectar-se através do ambiente isolado do container.

Para este container, o frontend é acessível na porta `8080`. Para abrir o site, visite [http://localhost:8080](http://localhost:8080) no seu navegador.

![Captura de tela da página inicial do servidor web Nginx, vinda do container em execução](img/access-the-frontend.webp?border)

### Pare seu container

O container `docker/welcome-to-docker` continua rodando até que você o pare. Você pode parar um container usando o comando `docker stop`.

1. Execute `docker ps` para obter o ID do container.
2. Forneça o ID ou nome do container ao comando [`docker stop`](https://docs.docker.com/reference/cli/docker/container/stop/):

    ```console
    docker stop <o-id-do-container>
    ```

> [!DICA]
>
> Ao referenciar containers por ID, você não precisa fornecer o ID completo. Você só precisa fornecer parte suficiente do ID para torná-lo único. Por exemplo, o container anterior poderia ser parado executando o seguinte comando:
>
> ```console
> docker stop a1f
> ```

## Recursos adicionais

Os links a seguir fornecem orientações adicionais sobre containers:

- [Executando um container](https://docs.docker.com/engine/containers/run/)
- [Visão geral de container](https://www.docker.com/resources/what-container/)
- [Por que Docker?](https://www.docker.com/why-docker/)

## Referências

[docs.docker.com/get-started/docker-concepts/the-basics/what-is-a-container](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-a-container/)

## Próximos passos

Agora que você aprendeu o básico de um container Docker, é hora de aprender sobre imagens Docker.

- [O que é uma imagem?](2-2_o-que-e-uma-imagem.md)
