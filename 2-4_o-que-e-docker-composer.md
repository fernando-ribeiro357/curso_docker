# O que é Docker Compose?

## Explicação

Se você tem seguido os guias até agora, tem trabalhado com aplicações de container único. Mas, agora você quer fazer algo mais complicado — executar bancos de dados, filas de mensagens, caches ou uma variedade de outros serviços. Você instala tudo em um único container? Executa múltiplos containers? Se executar múltiplos, como conecta todos eles?

Uma melhor prática para containers é que cada container deve fazer uma coisa e fazê-la bem. Embora existam exceções a essa regra, evite a tendência de fazer um container realizar múltiplas tarefas.

Você pode usar múltiplos comandos `docker run` para iniciar vários containers. No entanto, logo perceberá que precisará gerenciar redes, todas as flags necessárias para conectar containers a essas redes e muito mais. E quando terminar, a limpeza será um pouco mais complicada.

Com o Docker Compose, você pode definir todos os seus containers e suas configurações em um único arquivo YAML. Se você incluir este arquivo no seu repositório de código, qualquer pessoa que clonar seu repositório poderá colocar tudo em funcionamento com um único comando.

É importante entender que o Compose é uma ferramenta declarativa — você simplesmente define e executa. Nem sempre é necessário recriar tudo do zero. Se você fizer uma alteração, execute `docker compose up` novamente e o Compose reconciliará as mudanças no seu arquivo e as aplicará de forma inteligente.

> **Dockerfile versus arquivo Compose**
>
> Um Dockerfile fornece instruções para construir uma imagem de container, enquanto um arquivo Compose define seus containers em execução. Frequentemente, um arquivo Compose referencia um Dockerfile para construir uma imagem a ser usada para um serviço específico.

## Experimente

Nesta prática, você aprenderá como usar o Docker Compose para executar uma aplicação multi-container. Você usará uma aplicação simples de lista de tarefas (to-do list) construída com Node.js e MySQL como servidor de banco de dados.

### Inicie a aplicação

Siga as instruções para executar a aplicação de lista de tarefas no seu sistema.

1. [Baixe e instale](_install-docker.md) o Docker.
2. Abra um terminal e [clone esta aplicação de exemplo](https://github.com/dockersamples/todo-list-app).

    ```console
    git clone https://github.com/dockersamples/todo-list-app 
    ```

3. Navegue até o diretório `todo-list-app`:

    ```console
    cd todo-list-app
    ```

    Dentro deste diretório, você encontrará um arquivo chamado `compose.yaml`. Este arquivo YAML é onde toda a mágica acontece! Ele define todos os serviços que compõem sua aplicação, juntamente com suas configurações. Cada serviço especifica sua imagem, portas, volumes, redes e quaisquer outras configurações necessárias para sua funcionalidade. Reserve um tempo para explorar o arquivo YAML e familiarizar-se com sua estrutura. 

4. Use o comando [`docker compose up`](https://docs.docker.com/reference/cli/docker/compose/up/) para iniciar a aplicação:

    ```console
    docker compose up -d --build
    ```

    Quando você executar este comando, deverá ver uma saída semelhante a esta:

    ```console
    [+] Running 5/5
    ✔ app 3 layers [⣿⣿]      0B/0B            Pulled          7.1s
      ✔ e6f4e57cc59e Download complete                          0.9s
      ✔ df998480d81d Download complete                          1.0s
      ✔ 31e174fedd23 Download complete                          2.5s
      ✔ 43c47a581c29 Download complete                          2.0s
    [+] Running 4/4
      ⠸ Network todo-list-app_default           Created         0.3s
      ⠸ Volume "todo-list-app_todo-mysql-data"  Created         0.3s
      ✔ Container todo-list-app-app-1           Started         0.3s
      ✔ Container todo-list-app-mysql-1         Started         0.3s
    ```

    Muita coisa aconteceu aqui! Alguns pontos importantes a destacar:

    - Duas imagens de container foram baixadas do Docker Hub — node e MySQL
    - Uma rede foi criada para sua aplicação
    - Um volume foi criado para persistir os arquivos do banco de dados entre reinicializações dos containers
    - Dois containers foram iniciados com todas as configurações necessárias

    Se isso parecer avassalador, não se preocupe! Você chegará lá!

5. Com tudo agora instalado e em execução, você pode abrir [http://localhost:3000](http://localhost:3000) no seu navegador para ver o site. Note que a aplicação pode levar de 10 a 15 segundos para iniciar completamente. Se a página não carregar imediatamente, espere um momento e atualize. Sinta-se à vontade para adicionar itens à lista, marcá-los como concluídos e removê-los.

    ![Uma captura de tela de uma página da web mostrando a aplicação todo-list rodando na porta 3000](img/todo-list-app.webp?border=true&w=950&h=400)

6. Se você olhar para a GUI do Docker Desktop, poderá ver os containers e mergulhar mais fundo em suas configurações.

    ![Uma captura de tela do painel do Docker Desktop mostrando a lista de containers executando a aplicação todo-list](img/todo-list-containers.webp?border=true&w=950&h=400)

### Desmonte a aplicação

Como esta aplicação foi iniciada usando o Docker Compose, é fácil desmontar tudo quando você terminar.

1. Na CLI, use o comando [`docker compose down`](https://docs.docker.com/reference/cli/docker/compose/down/) para remover tudo:

    ```console
    docker compose down
    ```

    Você verá uma saída semelhante à seguinte:

    ```console
    [+] Running 3/3
    ✔ Container todo-list-app-mysql-1  Removed        2.9s
    ✔ Container todo-list-app-app-1    Removed        0.1s
    ✔ Network todo-list-app_default    Removed        0.1s
    ```

    > **Persistência de Volume**
    >
    > Por padrão, os volumes **_não_** são removidos automaticamente quando você desmonta uma stack do Compose. A ideia é que você pode querer os dados de volta se iniciar a stack novamente.
    >
    > Se você quiser remover os volumes, adicione a flag `--volumes` ao executar o comando `docker compose down`:
    >
    > ```console
    > docker compose down --volumes
    > [+] Running 1/0
    > ✔ Volume todo-list-app_todo-mysql-data  Removed
    > ```

Neste passo a passo, você aprendeu como usar o Docker Compose para iniciar e parar uma aplicação multi-container.

## Recursos adicionais

Esta página foi uma breve introdução ao Compose. Nos recursos a seguir, você pode se aprofundar no Compose e em como escrever arquivos Compose.

* [Visão geral do Docker Compose](https://docs.docker.com/compose/)
* [Visão geral da CLI do Docker Compose](https://docs.docker.com/reference/cli/docker/compose/)
* [Como o Compose funciona](https://docs.docker.com/compose/intro/compose-application-model/)

## Referências

[docs.docker.com/get-started/docker-concepts/the-basics/what-is-docker-compose/](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-docker-compose/)


## Próximos Passos

- Módulo 3: Persistência, Configuração e Segurança Básica