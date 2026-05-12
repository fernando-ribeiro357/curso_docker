# Módulo 3.1: Persistindo dados de container

## Explicação

Quando um container é iniciado, ele utiliza os arquivos e a configuração fornecidos pela imagem. Cada container é capaz de criar, modificar e excluir arquivos, fazendo isso sem afetar nenhum outro container. Quando o container é excluído, essas alterações de arquivo também são apagadas.

Embora essa natureza efêmera dos containers seja excelente, ela representa um desafio quando você deseja persistir os dados. Por exemplo, se você reiniciar um container de banco de dados, provavelmente não desejará começar com um banco de dados vazio. Então, como persistir arquivos?

## Volumes de container

Volumes são um mecanismo de armazenamento que fornece a capacidade de persistir dados além do ciclo de vida de um container individual. Pense nisso como fornecer um atalho ou link simbólico (symlink) de dentro do container para fora dele.

Como exemplo, imagine que você cria um volume chamado `log-data`.

```console
$ docker volume create log-data
```

Ao iniciar um container com o seguinte comando, o volume será montado (ou anexado) dentro do container em `/logs`:

```console
$ docker run -d -p 80:80 -v log-data:/logs docker/welcome-to-docker
```

Se o volume `log-data` não existir, o Docker o criará automaticamente para você.

Quando o container estiver em execução, todos os arquivos que ele gravar na pasta `/logs` serão salvos neste volume, fora do container. Se você excluir o container e iniciar um novo container usando o mesmo volume, os arquivos ainda estarão lá.

> **Compartilhando arquivos usando volumes**
>
> Você pode anexar o mesmo volume a múltiplos containers para compartilhar arquivos entre eles. Isso pode ser útil em cenários como agregação de logs, pipelines de dados ou outras aplicações orientadas a eventos.

### Gerenciando volumes

Os volumes têm seu próprio ciclo de vida, independente dos containers, e podem crescer bastante dependendo do tipo de dados e das aplicações que você está usando. Os seguintes comandos serão úteis para gerenciar volumes:

- `docker volume ls` - lista todos os volumes
- `docker volume rm <nome-do-volume-ou-id>` - remove um volume (funciona apenas quando o volume não está anexado a nenhum container)
- `docker volume prune` - remove todos os volumes não utilizados (desanexados)

## Experimente

Neste guia, você praticará a criação e o uso de volumes para persistir dados criados por um container Postgres. Quando o banco de dados é executado, ele armazena arquivos no diretório `/var/lib/postgresql`. Ao anexar o volume aqui, você poderá reiniciar o container várias vezes mantendo os dados.

### Usando volumes

1. [Baixe e instale](_install-docker.md) o Docker.

2. Inicie um container usando a [imagem Postgres](https://hub.docker.com/_/postgres) com o seguinte comando:

    ```console
    $ docker run --name=db -e POSTGRES_PASSWORD=secret -d -v postgres_data:/var/lib/postgresql postgres:18
    ```

    Isso iniciará o banco de dados em segundo plano, configurará uma senha e anexará um volume ao diretório onde o PostgreSQL persistirá os arquivos do banco de dados.

3. Conecte-se ao banco de dados usando o seguinte comando:

    ```console
    $ docker exec -ti db psql -U postgres
    ```

4. Na linha de comando do PostgreSQL, execute o seguinte para criar uma tabela de banco de dados e inserir dois registros:

    ```text
    CREATE TABLE tasks (
        id SERIAL PRIMARY KEY,
        description VARCHAR(100)
    );
    INSERT INTO tasks (description) VALUES ('Finish work'), ('Have fun');
    ```

5. Verifique se os dados estão no banco de dados executando o seguinte na linha de comando do PostgreSQL:

    ```text
    SELECT * FROM tasks;
    ```

    Você deve obter uma saída semelhante à seguinte:

    ```text
     id | description
    ----+-------------
      1 | Finish work
      2 | Have fun
    (2 rows)
    ```

6. Saia do shell do PostgreSQL executando o seguinte comando:

    ```console
    \q
    ```

7. Pare e remova o container do banco de dados. Lembre-se de que, embora o container tenha sido excluído, os dados são persistidos no volume `postgres_data`.

    ```console
    $ docker stop db
    $ docker rm db
    ```

8. Inicie um novo container executando o seguinte comando, anexando o mesmo volume com os dados persistidos:

    ```console
    $ docker run --name=new-db -d -v postgres_data:/var/lib/postgresql postgres:18
    ```

    Você pode ter notado que a variável de ambiente `POSTGRES_PASSWORD` foi omitida. Isso ocorre porque essa variável é usada apenas ao inicializar (bootstrap) um novo banco de dados.

9. Verifique se o banco de dados ainda possui os registros executando o seguinte comando:

    ```console
    $ docker exec -ti new-db psql -U postgres -c "SELECT * FROM tasks"
    ```

### Visualizando o conteúdo do volume



### Removendo volumes

Antes de remover um volume, ele não deve estar anexado a nenhum container. Se você não removeu o container anterior, faça-o com o seguinte comando (a flag `-f` parará o container primeiro e depois o removerá):

```console
$ docker rm -f new-db
```

Existem alguns métodos para remover volumes, incluindo os seguintes:

- Selecione a opção **Delete Volume** (Excluir Volume) em um volume no Painel do Docker Desktop.
- Use o comando `docker volume rm`:

    ```console
    $ docker volume rm postgres_data
    ```
- Use o comando `docker volume prune` para remover todos os volumes não utilizados:

    ```console
    $ docker volume prune
    ```

## Recursos adicionais

Os seguintes recursos ajudarão você a aprender mais sobre volumes:

- [Armazenamento (storage drivers e volumes)](_armazenamento.md)
- [Gerenciar dados no Docker](https://docs.docker.com/engine/storage)
- [Volumes](https://docs.docker.com/engine/storage/volumes)
- [Montagens de volume](https://docs.docker.com/engine/containers/run/#volume-mounts)

## Próximos passos

- [Rede em single node](3-2_rede-single-node.md)
