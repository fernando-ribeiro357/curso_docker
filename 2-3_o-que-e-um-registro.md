# Módulo 2.3: Registro de Imagens

## O que é um registro (registry)?

Agora que você sabe o que é uma imagem de container e como ela funciona, você pode se perguntar: onde você armazena essas imagens?

Bem, você pode armazenar suas imagens de container no seu sistema de computador, mas e se quiser compartilhá-las com seus amigos ou usá-las em outra máquina? É aí que entra o registro de imagens.

Um registro de imagens é um local centralizado para armazenar e compartilhar suas imagens de container. Ele pode ser público ou privado. O [Docker Hub](https://hub.docker.com) é um registro público que qualquer pessoa pode usar e é o registro padrão.

Embora o Docker Hub seja uma opção popular, existem muitos outros registros de containers disponíveis hoje, incluindo o [Amazon Elastic Container Registry (ECR)](https://aws.amazon.com/ecr/), o [Azure Container Registry (ACR)](https://azure.microsoft.com/en-in/products/container-registry) e o [Google Container Registry (GCR)](https://cloud.google.com/artifact-registry). Você também pode executar seu próprio registro privado em seu sistema local ou dentro de sua organização. Exemplos incluem Harbor, JFrog Artifactory, GitLab Container Registry, entre outros.

### Registro vs. Repositório

Enquanto trabalha com registros, você pode ouvir os termos _registro_ e _repositório_ como se fossem intercambiáveis. Embora estejam relacionados, não são exatamente a mesma coisa.

Um **registro** é um local centralizado que armazena e gerencia imagens de container, enquanto um **repositório** é uma coleção de imagens de container relacionadas dentro de um registro. Pense nisso como uma pasta onde você organiza suas imagens com base em projetos. Cada repositório contém uma ou mais imagens de container.

O diagrama a seguir mostra a relação entre um registro, repositórios e imagens.

```goat {class="text-sm"}
+---------------------------------------+
|               Registro                |
|---------------------------------------|
|                                       |
|    +-----------------------------+    |
|    |        Repositório A        |    |
|    |-----------------------------|    |
|    |   Imagem: project-a:v1.0    |    |
|    |   Imagem: project-a:v2.0    |    |
|    +-----------------------------+    |
|                                       |
|    +-----------------------------+    |
|    |        Repositório B        |    |
|    |-----------------------------|    |
|    |   Imagem: project-b:v1.0    |    |
|    |   Imagem: project-b:v1.1    |    |
|    |   Imagem: project-b:v2.0    |    |
|    +-----------------------------+    |
|                                       |
+---------------------------------------+
```


## Experimente

Nesta prática, você aprenderá como construir e enviar (push) uma imagem Docker para um repositório do Docker Hub.

### Inscreva-se para uma conta Docker gratuita

1. Se você ainda não criou uma, vá até a página do [Docker Hub](https://hub.docker.com) para se inscrever em uma nova conta Docker. Certifique-se de concluir as etapas de verificação enviadas para seu e-mail.

    ![Captura de tela da página oficial do Docker Hub mostrando a página de inscrição](img/dockerhub-signup.webp?border)

    Você pode usar sua conta do Google ou GitHub para autenticar.

### Crie seu primeiro repositório

1. Faça login no [Docker Hub](https://hub.docker.com).
2. Selecione o botão **Create repository** (Criar repositório) no canto superior direito.
3. Selecione seu namespace (provavelmente seu nome de usuário) e digite `docker-quickstart` como nome do repositório.

    ![Captura de tela da página do Docker Hub que mostra como criar um repositório público](img/create-hub-repository.webp?border)

4. Defina a visibilidade como **Public** (Público).
5. Selecione o botão **Create** (Criar) para criar o repositório.

É isso. Você criou com sucesso seu primeiro repositório. 🎉

Este repositório está vazio no momento. Agora vamos resolver isso enviando uma imagem para ele.

### Faça login com o Docker CLI

- ### Autenticar no Docker Hub com login baseado na web

Por padrão, o comando `docker login` autentica no Docker Hub usando um fluxo de código de dispositivo. Esse fluxo permite que você se autentique no Docker Hub sem digitar sua senha. Em vez disso, você acessa uma URL no seu navegador da web, insere um código e realiza a autenticação.

```console
$ docker login

USANDO LOGIN BASEADO NA WEB
Para fazer login com credenciais na linha de comando, use 'docker login -u <nome_de_usuario>'

Seu código de confirmação de dispositivo único é: LNFR-PGCJ
Pressione ENTER para abrir seu navegador ou envie seu código de dispositivo aqui: https://login.docker.com/activate

Aguardando autenticação no navegador…
```

Após inserir o código no seu navegador, você será autenticado no Docker Hub usando a conta com a qual está atualmente conectado no site do Docker Hub ou no Docker Desktop. Se você não estiver conectado, será solicitado que faça o login após inserir o código do dispositivo.

- ### Autenticar com nome de usuário e senha {#username}

Para autenticar em um registro com nome de usuário e senha, você pode usar a flag `--username` ou `-u`. O exemplo a seguir autentica no Docker Hub com o nome de usuário `moby`. A senha é inserida interativamente.

```console
$ docker login -u moby
```

- ### Fornecer uma senha usando STDIN (--password-stdin) {#password-stdin}

Para executar o comando `docker login` de forma não interativa, você pode definir a flag `--password-stdin` para fornecer uma senha através do `STDIN`. O uso de `STDIN` evita que a senha acabe no histórico do shell ou em arquivos de log.

O exemplo a seguir lê uma senha de um arquivo e a passa para o comando `docker login` usando `STDIN`:

```console
$ cat ~/minha_senha.txt | docker login --username foo --password-stdin
```

- ### Autenticar em um registro auto-hospedado

Se desejar autenticar em um registro auto-hospedado, você pode especificá-lo adicionando o nome do servidor.

```console
$ docker login registry.exemplo.com
```

Por padrão, o comando `docker login` assume que o registro está escutando nas portas 443 ou 80. Se o registro estiver escutando em uma porta diferente, você pode especificá-la adicionando o número da porta ao nome do servidor.

```console
$ docker login registry.exemplo.com:1337
```

> [!NOTA]
> Os endereços de registro não devem incluir componentes de caminho de URL, apenas o nome do host e (opcionalmente) a porta. Endereços de registro com componentes de caminho de URL podem resultar em erro. Por exemplo, `docker login registry.exemplo.com/foo/` está incorreto, enquanto `docker login registry.exemplo.com` está correto.
>
> A exceção a essa regra é o registro do Docker Hub, que pode usar o componente de caminho `/v1/` no endereço por razões históricas.


### Clone o código de exemplo Node.js

Para criar uma imagem, primeiro você precisa de um projeto. Para começar rapidamente, você usará um projeto de exemplo em Node.js encontrado em [github.com/dockersamples/helloworld-demo-node](https://github.com/dockersamples/helloworld-demo-node). Este repositório contém um Dockerfile pré-construído necessário para criar uma imagem Docker.

Não se preocupe com os detalhes específicos do Dockerfile, pois você aprenderá sobre isso em seções posteriores.

1. Clone o repositório do GitHub usando o seguinte comando:

    ```console
    git clone https://github.com/dockersamples/helloworld-demo-node
    ```

2. Navegue até o diretório recém-criado.

    ```console
    cd helloworld-demo-node
    ```

3. Execute o seguinte comando para construir uma imagem Docker, substituindo `<SEU_NOME_DE_USUARIO_DOCKER>` pelo seu nome de usuário.

    ```console
    docker build -t <SEU_NOME_DE_USUARIO_DOCKER>/docker-quickstart .
    ```

    > [!NOTA]
    >
    > Certifique-se de incluir o ponto (.) no final do comando `docker build`. Isso diz ao Docker onde encontrar o Dockerfile.

4. Execute o seguinte comando para listar a imagem Docker recém-criada:

    ```console
    docker images
    ```

    Você verá uma saída semelhante à seguinte:

    ```console
    REPOSITORY                                 TAG       IMAGE ID       CREATED         SIZE
    <SEU_NOME_DE_USUARIO_DOCKER>/docker-quickstart   latest    476de364f70e   2 minutes ago   170MB
    ```

5. Inicie um container para testar a imagem executando o seguinte comando (substitua o nome de usuário pelo seu próprio):

    ```console
    docker run -d -p 8080:8080 <SEU_NOME_DE_USUARIO_DOCKER>/docker-quickstart 
    ```

    Você pode verificar se o container está funcionando acessando [http://localhost:8080](http://localhost:8080) com seu navegador.

6. Use o comando [`docker tag`](https://docs.docker.com/reference/cli/docker/image/tag/) para marcar (taggear) a imagem Docker. As tags do Docker permitem rotular e versionar suas imagens.

    ```console 
    docker tag <SEU_NOME_DE_USUARIO_DOCKER>/docker-quickstart <SEU_NOME_DE_USUARIO_DOCKER>/docker-quickstart:1.0 
    ```

7. Finalmente, é hora de enviar a imagem recém-construída para o seu repositório do Docker Hub usando o comando [`docker push`](https://docs.docker.com/reference/cli/docker/image/push/):

    ```console 
    docker push <SEU_NOME_DE_USUARIO_DOCKER>/docker-quickstart:1.0
    ```

8. Abra o [Docker Hub](https://hub.docker.com) e navegue até o seu repositório. Vá para a seção **Tags** e veja sua imagem recém-enviada.

    ![Captura de tela da página do Docker Hub que exibe a tag da imagem recém-adicionada](img/dockerhub-tags.webp?border=true) 

Neste passo a passo, você se inscreveu para uma conta Docker, criou seu primeiro repositório no Docker Hub e construiu, marcou e enviou uma imagem de container para o seu repositório no Docker Hub.

## Recursos adicionais

- [Início rápido do Docker Hub](https://docs.docker.com/docker-hub/quickstart/)
- [Visão geral do Docker Build](https://docs.docker.com/build/concepts/overview/)
- [Gerenciar repositórios do Docker Hub](https://docs.docker.com/docker-hub/repos/)
- [Lab: Construindo Imagens de Container](https://docs.docker.com/guides/lab-building-images/)

## Referências

[docs.docker.com/get-started/docker-concepts/the-basics/what-is-a-registry](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-a-registry/)

## Próximos passos

Agora que você entende o básico de containers e imagens, está pronto para aprender sobre o Docker Compose.

[O que é Docker Compose?](2-4_o-que-e-docker-composer.md)
