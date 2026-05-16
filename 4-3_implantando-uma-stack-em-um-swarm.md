# Módulo 4.3: Implantantando uma _stack_ em um Swarm

Ao executar o Docker Engine no modo swarm, você pode usar `docker stack deploy` para implantar uma _stack_ de aplicação completa no swarm. O comando `deploy` aceita uma descrição da _stack_ na forma de um [arquivo Compose](https://docs.docker.com/reference/compose-file/legacy-versions/).

> [!NOTA]
>
> O comando `docker stack deploy` usa o formato legado do [arquivo Compose versão 3](https://docs.docker.com/reference/compose-file/legacy-versions/), utilizado pelo Compose V1. O formato mais recente, definido pela [especificação Compose](https://docs.docker.com/reference/compose-file/), não é compatível com o comando `docker stack deploy`.
>
> Para mais informações sobre a evolução do Compose, consulte [Histórico do Compose](https://docs.docker.com/compose/history/).

Para seguir este tutorial, você precisa de:

1.  Um Docker Engine executando no [modo Swarm](https://docs.docker.com/engine/swarm/stack-deploy/swarm-mode/). Se você não estiver familiarizado com o modo Swarm, talvez queira ler [Conceitos-chave do modo Swarm](https://docs.docker.com/engine/swarm/stack-deploy/key-concepts/) e [Como os serviços funcionam](https://docs.docker.com/engine/swarm/stack-deploy/how-swarm-mode-works/services/).

    > [!NOTA]
    >
    > Se você estiver testando em um ambiente de desenvolvimento local, pode colocar seu engine no modo Swarm com `docker swarm init`.
    >
    > Se você já tiver um swarm multi-nó em execução, lembre-se de que todos os comandos `docker stack` e `docker service` devem ser executados a partir de um nó gerenciador (manager).

2.  Uma versão atual do [Docker Compose](https://docs.docker.com/compose/install/).

## Configurar um registro Docker

Como um swarm consiste em múltiplos Docker Engines, um registro é necessário para distribuir imagens para todos eles. Você pode usar o [Docker Hub](https://hub.docker.com) ou manter o seu próprio. Veja como criar um registro descartável, que você pode remover afterward.

1.  Inicie o registro como um serviço no seu swarm:

    ```console
    $ docker service create --name registry --publish published=5000,target=5000 registry:2
    ```

2.  Verifique o status com `docker service ls`:

    ```console
    $ docker service ls

    ID            NAME      REPLICAS  IMAGE                                                                               COMMAND
    l7791tpuwkco  registry  1/1       registry:2@sha256:1152291c7f93a4ea2ddc95e46d142c31e743b6dd70e194af9e6ebe530f782c17
    ```

    Quando mostrar `1/1` sob `REPLICAS`, ele estará em execução. Se mostrar `0/1`, provavelmente ainda está baixando a imagem.

3.  Verifique se está funcionando com `curl`:

    ```console
    $ curl http://127.0.0.1:5000/v2/

    {}
    ```

## Criar a aplicação de exemplo

A aplicação usada neste guia é baseada na aplicação de contador de acessos do guia [Começando com Docker Compose](/compose/gettingstarted/). Ela consiste em uma aplicação Python que mantém um contador em uma instância Redis e incrementa o contador sempre que você a acessa.

1.  Crie um diretório para o projeto:

    ```console
    $ mkdir stackdemo
    $ cd stackdemo
    ```

2.  Crie um arquivo chamado `app.py` no diretório do projeto e cole o seguinte conteúdo:

    ```python
    from flask import Flask
    from redis import Redis

    app = Flask(__name__)
    redis = Redis(host='redis', port=6379)

    @app.route('/')
    def hello():
        count = redis.incr('hits')
        return 'Hello World! I have been seen {} times.\n'.format(count)

    if __name__ == "__main__":
        app.run(host="0.0.0.0", port=8000, debug=True)
    ```

3.  Crie um arquivo chamado `requirements.txt` e cole estas duas linhas:

    ```text
    flask
    redis
    ```

4.  Crie um arquivo chamado `Dockerfile` e cole o seguinte conteúdo:

    ```dockerfile
    # syntax=docker/dockerfile:1
    FROM python:3.4-alpine
    ADD . /code
    WORKDIR /code
    RUN pip install -r requirements.txt
    CMD ["python", "app.py"]
    ```

5.  Crie um arquivo chamado `compose.yaml` e cole o seguinte conteúdo:

    ```yaml
      services:
        web:
          image: 127.0.0.1:5000/stackdemo
          build: .
          ports:
            - "8000:8000"
        redis:
          image: redis:alpine
    ```

    A imagem para a aplicação web é construída usando o Dockerfile definido acima. Ela também é marcada com `127.0.0.1:5000` - o endereço do registro criado anteriormente. Isso é importante ao distribuir a aplicação para o swarm.

## Testar a aplicação com Compose

1.  Inicie a aplicação com `docker compose up`. Isso constrói a imagem da aplicação web, baixa a imagem do Redis se você ainda não a tiver e cria dois containers.

    Você verá um aviso sobre o Engine estar no modo swarm. Isso ocorre porque o Compose não aproveita o modo swarm e implanta tudo em um único nó. Você pode ignorar isso com segurança.

    ```console
    $ docker compose up -d

    WARNING: The Docker Engine you're using is running in swarm mode.

    Compose does not use swarm mode to deploy services to multiple nodes in
    a swarm. All containers are scheduled on the current node.

    To deploy your application across the swarm, use `docker stack deploy`.

    Creating network "stackdemo_default" with the default driver
    Building web
    ...(saída da construção)...
    Creating stackdemo_redis_1
    Creating stackdemo_web_1
    ```

2.  Verifique se a aplicação está em execução com `docker compose ps`:

    ```console
    $ docker compose ps

          Name                     Command               State           Ports
    -----------------------------------------------------------------------------------
    stackdemo_redis_1   docker-entrypoint.sh redis ...   Up      6379/tcp
    stackdemo_web_1     python app.py                    Up      0.0.0.0:8000->8000/tcp
    ```

    Você pode testar a aplicação com `curl`:

    ```console
    $ curl http://localhost:8000
    Hello World! I have been seen 1 times.

    $ curl http://localhost:8000
    Hello World! I have been seen 2 times.

    $ curl http://localhost:8000
    Hello World! I have been seen 3 times.
    ```

3.  Pare a aplicação:

    ```console
    $ docker compose down --volumes

    Stopping stackdemo_web_1 ... done
    Stopping stackdemo_redis_1 ... done
    Removing stackdemo_web_1 ... done
    Removing stackdemo_redis_1 ... done
    Removing network stackdemo_default
    ```

## Enviar a imagem gerada para o registro

Para distribuir a imagem da aplicação web por todo o swarm, ela precisa ser enviada (push) para o registro que você configurou anteriormente. Com o Compose, isso é muito simples:

```console
$ docker compose push

Pushing web (127.0.0.1:5000/stackdemo:latest)...
The push refers to a repository [127.0.0.1:5000/stackdemo]
5b5a49501a76: Pushed
be44185ce609: Pushed
bd7330a79bcf: Pushed
c9fc143a069a: Pushed
011b303988d2: Pushed
latest: digest: sha256:a81840ebf5ac24b42c1c676cbda3b2cb144580ee347c07e1bc80e35e5ca76507 size: 1372
```

A _stack_ agora está pronta para ser implantada.

## Implantar a _stack_ no swarm

1.  Crie a _stack_ com `docker stack deploy`:

    ```console
    $ docker stack deploy --compose-file compose.yaml stackdemo

    Ignoring unsupported options: build

    Creating network stackdemo_default
    Creating service stackdemo_web
    Creating service stackdemo_redis
    ```

    O último argumento é um nome para a _stack_. Cada rede, volume e nome de serviço será prefixado com o nome da _stack_.

2.  Verifique se está em execução com `docker stack services stackdemo`:

    ```console
    $ docker stack services stackdemo

    ID            NAME             MODE        REPLICAS  IMAGE
    orvjk2263y1p  stackdemo_redis  replicated  1/1       redis:3.2-alpine@sha256:f1ed3708f538b537eb9c2a7dd50dc90a706f7debd7e1196c9264edeea521a86d
    s1nf0xy8t1un  stackdemo_web    replicated  1/1       127.0.0.1:5000/stackdemo@sha256:adb070e0805d04ba2f92c724298370b7a4eb19860222120d43e0f6351ddbc26f
    ```

    Uma vez em execução, você deve ver `1/1` sob `REPLICAS` para ambos os serviços. Isso pode levar algum tempo se você tiver um swarm multi-nó, pois as imagens precisam ser baixadas.

    Como antes, você pode testar a aplicação com `curl`:

    ```console
    $ curl http://localhost:8000
    Hello World! I have been seen 1 times.

    $ curl http://localhost:8000
    Hello World! I have been seen 2 times.

    $ curl http://localhost:8000
    Hello World! I have been seen 3 times.
    ```

    Com a malha de roteamento integrada do Docker, você pode acessar qualquer nó no swarm na porta `8000` e ser direcionado para a aplicação:

    ```console
    $ curl http://endereco-do-outro-no:8000
    Hello World! I have been seen 4 times.
    ```

3.  Remova a _stack_ com `docker stack rm`:

    ```console
    $ docker stack rm stackdemo

    Removing service stackdemo_web
    Removing service stackdemo_redis
    Removing network stackdemo_default
    ```

4.  Remova o registro com `docker service rm`:

    ```console
    $ docker service rm registry
    ```

5.  Se você estiver apenas testando em uma máquina local e quiser tirar seu Docker Engine do modo Swarm, use `docker swarm leave`:

    ```console
    $ docker swarm leave --force

    Node left the swarm.
    ```