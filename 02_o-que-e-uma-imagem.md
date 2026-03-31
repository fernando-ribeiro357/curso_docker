# O que é uma imagem?

## Explicação

Visto que um [container](01_o-que-e-um-container.md) é um processo isolado, de onde ele obtém seus arquivos e configurações? Como você compartilha esses ambientes?

É aí que entram as imagens de container. Uma imagem de container é um pacote padronizado que inclui todos os arquivos, binários, bibliotecas e configurações necessários para executar um container.

Para uma imagem do [PostgreSQL](https://hub.docker.com/_/postgres), essa imagem empacotará os binários do banco de dados, arquivos de configuração e outras dependências. Para um aplicativo web em Python, ela incluirá o runtime do Python, o código do seu aplicativo e todas as suas dependências.

Existem dois princípios importantes das imagens:

1.  **As imagens são imutáveis.** Uma vez criada, uma imagem não pode ser modificada. Você só pode criar uma nova imagem ou adicionar alterações sobre ela.
2.  **As imagens de container são compostas por camadas (layers).** Cada camada representa um conjunto de alterações no sistema de arquivos que adicionam, removem ou modificam arquivos.

Esses dois princípios permitem que você estenda ou adicione funcionalidades a imagens existentes. Por exemplo, se você estiver construindo um aplicativo em Python, pode começar a partir da [imagem Python](https://hub.docker.com/_/python) e adicionar camadas adicionais para instalar as dependências do seu aplicativo e incluir seu código. Isso permite que você se concentre no seu aplicativo, em vez de no próprio Python.

### Encontrando imagens

O [Docker Hub](https://hub.docker.com) é o mercado global padrão para armazenar e distribuir imagens. Ele possui mais de 100.000 imagens criadas por desenvolvedores que você pode executar localmente. Você pode pesquisar imagens no Docker Hub e executá-las diretamente pelo Docker Desktop.

O Docker Hub oferece uma variedade de imagens suportadas e endossadas pelo Docker, conhecidas como **Conteúdo Confiável do Docker (Docker Trusted Content)**. Elas fornecem serviços totalmente gerenciados ou excelentes pontos de partida para suas próprias imagens. Estas incluem:

*   [**Imagens Oficiais do Docker**](https://hub.docker.com/search?badges=official) – um conjunto curado de repositórios Docker, servem como ponto de partida para a maioria dos usuários e estão entre as mais seguras no Docker Hub.
*   [**Imagens Endurecidas do Docker (Hardened Images)**](https://hub.docker.com/hardened-images/catalog) – imagens mínimas, seguras e prontas para produção com quase zero CVEs (vulnerabilidades comuns e expostas), projetadas para reduzir a superfície de ataque e simplificar a conformidade. Gratuitas e de código aberto sob a licença Apache 2.0.
*   [**Publicadores Verificados do Docker**](https://hub.docker.com/search?badges=verified_publisher) – imagens de alta qualidade de publicadores comerciais verificados pelo Docker.
*   [**Código Aberto Patrocinado pelo Docker**](https://hub.docker.com/search?badges=open_source) – imagens publicadas e mantidas por projetos de código aberto patrocinados pelo Docker através do programa de código aberto da empresa.

Por exemplo, [Redis](https://hub.docker.com/_/redis) e [Memcached](https://hub.docker.com/_/memcached) são algumas Imagens Oficiais do Docker populares e prontas para uso. Você pode baixar essas imagens e ter esses serviços instalados e rodando em questão de segundos. Existem também imagens base, como a imagem Docker do [Node.js](https://hub.docker.com/_/node), que você pode usar como ponto de partida e adicionar seus próprios arquivos e configurações. Para cargas de trabalho de produção que exigem segurança aprimorada, as Imagens Endurecidas do Docker oferecem variantes mínimas de imagens populares como Node.js, Python e Go.

## Experimente

**Usando a CLI (Linha de Comando)**

Siga as instruções para pesquisar e baixar uma imagem Docker usando a CLI para visualizar suas camadas.

### Pesquise e baixe uma imagem

1.  Abra um terminal e pesquise por imagens usando o comando [`docker search`](/reference/cli/docker/search/):

    ```console
    docker search docker/welcome-to-docker
    ```

    Você verá uma saída semelhante à seguinte:

    ```console
    NAME                       DESCRIPTION                                     STARS     OFFICIAL
    docker/welcome-to-docker   Docker image for new users getting started w…   20
    ```

    Esta saída mostra informações sobre imagens relevantes disponíveis no Docker Hub.

2.  Baixe a imagem usando o comando [`docker pull`](/reference/cli/docker/image/pull/):

    ```console
    docker pull docker/welcome-to-docker
    ```

    Você verá uma saída semelhante à seguinte:

    ```console
    Using default tag: latest
    latest: Pulling from docker/welcome-to-docker
    579b34f0a95b: Download complete
    d11a451e6399: Download complete
    1c2214f9937c: Download complete
    b42a2f288f4d: Download complete
    54b19e12c655: Download complete
    1fb28e078240: Download complete
    94be7e780731: Download complete
    89578ce72c35: Download complete
    Digest: sha256:eedaff45e3c78538087bdd9dc7afafac7e110061bbdd836af4104b10f10ab693
    Status: Downloaded newer image for docker/welcome-to-docker:latest
    docker.io/docker/welcome-to-docker:latest
    ```

    Cada linha representa uma camada diferente baixada da imagem. Lembre-se de que cada camada é um conjunto de alterações no sistema de arquivos e fornece funcionalidade à imagem.

### Aprenda sobre a imagem

1.  Liste suas imagens baixadas usando o comando [`docker image ls`](/reference/cli/docker/image/ls/):

    ```console
    docker image ls
    ```

    Você verá uma saída semelhante à seguinte:

    ```console
    REPOSITORY                 TAG       IMAGE ID       CREATED        SIZE
    docker/welcome-to-docker   latest    eedaff45e3c7   4 months ago   29.7MB
    ```

    O comando mostra uma lista de imagens Docker atualmente disponíveis no seu sistema. A `docker/welcome-to-docker` tem um tamanho total de aproximadamente 29,7 MB.

    > **Tamanho da imagem**
    >
    > O tamanho da imagem representado aqui reflete o tamanho descompactado da imagem, não o tamanho do download das camadas.

2.  Liste as camadas da imagem usando o comando [`docker image history`](https://docs.docker.com/reference/cli/docker/image/history/):

    ```console
    docker image history docker/welcome-to-docker
    ```

    Você verá uma saída semelhante à seguinte:

    ```console
    IMAGE          CREATED        CREATED BY                                      SIZE      COMMENT
    648f93a1ba7d   4 months ago   COPY /app/build /usr/share/nginx/html # buil…   1.6MB     buildkit.dockerfile.v0
    <missing>      5 months ago   /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B
    <missing>      5 months ago   /bin/sh -c #(nop)  STOPSIGNAL SIGQUIT           0B
    <missing>      5 months ago   /bin/sh -c #(nop)  EXPOSE 80                    0B
    <missing>      5 months ago   /bin/sh -c #(nop)  ENTRYPOINT ["/docker-entr…   0B
    <missing>      5 months ago   /bin/sh -c #(nop) COPY file:9e3b2b63db9f8fc7…   4.62kB
    <missing>      5 months ago   /bin/sh -c #(nop) COPY file:57846632accc8975…   3.02kB
    <missing>      5 months ago   /bin/sh -c #(nop) COPY file:3b1b9915b7dd898a…   298B
    <missing>      5 months ago   /bin/sh -c #(nop) COPY file:caec368f5a54f70a…   2.12kB
    <missing>      5 months ago   /bin/sh -c #(nop) COPY file:01e75c6dd0ce317d…   1.62kB
    <missing>      5 months ago   /bin/sh -c set -x     && addgroup -g 101 -S …   9.7MB
    <missing>      5 months ago   /bin/sh -c #(nop)  ENV PKG_RELEASE=1            0B
    <missing>      5 months ago   /bin/sh -c #(nop)  ENV NGINX_VERSION=1.25.3     0B
    <missing>      5 months ago   /bin/sh -c #(nop)  LABEL maintainer=NGINX Do…   0B
    <missing>      5 months ago   /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B
    <missing>      5 months ago   /bin/sh -c #(nop) ADD file:ff3112828967e8004…   7.66MB
    ```

    Esta saída mostra todas as camadas, seus tamanhos e o comando usado para criar a camada.

    > **Visualizando o comando completo**
    >
    > Se você adicionar a flag `--no-trunc` ao comando, verá o comando completo. Observe que, como a saída está em formato de tabela, comandos mais longos podem tornar a navegação pela saída muito difícil.

Neste passo a passo, você pesquisou e baixou uma imagem Docker. Além de baixar uma imagem Docker, você também aprendeu sobre as camadas de uma imagem Docker.

## Recursos adicionais

Os seguintes recursos ajudarão você a aprender mais sobre como explorar, encontrar e construir imagens:

*   [Conteúdo confiável do Docker](https://docs.docker.com/docker-hub/image-library/trusted-content/)
*   [Visão geral do Docker Build](https://docs.docker.com/build/concepts/overview/)
*   [Docker Hub](https://hub.docker.com)

## Referências

[docs.docker.com/get-started/docker-concepts/the-basics/what-is-an-image](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-an-image/)

## Próximos passos

Agora que você aprendeu o básico sobre imagens, é hora de aprender sobre a distribuição de imagens através de registros (registries).

[O que é um registro?](03_o-que-e-um-registro.md)
