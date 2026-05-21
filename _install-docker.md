# Instalar o Docker Engine no Debian

Para começar a usar o Docker Engine no Debian, certifique-se de que você [atende aos pré-requisitos](#pré-requisitos) e, em seguida, siga as [etapas de instalação](#métodos-de-instalação).

## Pré-requisitos

### Limitações de Firewall

> \[!ATENÇÃO\]

> Antes de instalar o Docker, certifique-se de considerar as seguintes implicações de segurança e incompatibilidades de firewall.

- Se você usa `ufw` ou `firewalld` para gerenciar configurações de firewall, esteja ciente de que, ao expor portas de container usando o Docker, essas portas contornam suas regras de firewall. Para mais informações, consulte [Docker e ufw](file:///engine/network/packet-filtering-firewalls/#docker-and-ufw).

- O Docker é compatível apenas com `iptables-nft` e `iptables-legacy`. Regras de firewall criadas com `nft` não são suportadas em um sistema com Docker instalado. Certifique-se de que quaisquer conjuntos de regras de firewall que você use sejam criados com `iptables` ou `ip6tables`, e que você os adicione à cadeia `DOCKER-USER`; veja [Filtragem de pacotes e firewalls](file:///engine/network/packet-filtering-firewalls/).

### Requisitos do Sistema Operacional

Para instalar o Docker Engine, você precisa de uma destas versões do Debian:

- Debian Trixie 13 (estável)

- Debian Bookworm 12 (antiga estável)

- Debian Bullseye 11 (antiga antiga estável)

O Docker Engine para Debian é compatível com as arquiteturas x86\_64 (ou amd64), armhf (arm/v7), arm64 e ppc64le (ppc64el).

### Desinstalar versões antigas

Antes de poder instalar o Docker Engine, você precisa desinstalar quaisquer pacotes conflitantes.

Sua distribuição Linux pode fornecer pacotes não oficiais do Docker, que podem conflitar com os pacotes oficiais fornecidos pelo Docker. Você deve desinstalar esses pacotes antes de instalar a versão oficial do Docker Engine.

Os pacotes não oficiais a serem desinstalados são:

- `docker.io`

- `docker-compose`

- `docker-doc`

- `podman-docker`

Além disso, o Docker Engine depende de `containerd` e `runc`. O Docker Engine empacota essas dependências como um único pacote: `containerd.io`. Se você instalou o `containerd` ou `runc` anteriormente, desinstale-os para evitar conflitos com as versões empacotadas com o Docker Engine.

Execute o seguinte comando para desinstalar todos os pacotes conflitantes:

```console
$ sudo apt remove $(dpkg --get-selections docker.io docker-compose docker-doc podman-docker containerd runc | cut -f1)
```

O `apt` pode informar que nenhum desses pacotes está instalado.

Imagens, containers, volumes e redes armazenados em `/var/lib/docker/` não são removidos automaticamente quando você desinstala o Docker. Se você deseja começar com uma instalação limpa e prefere limpar quaisquer dados existentes, leia a seção [desinstalar o Docker Engine](#desinstalar-o-docker-engine).

## Métodos de Instalação

Você pode instalar o Docker Engine de diferentes maneiras, dependendo de suas necessidades:

- Configure e instale o Docker Engine a partir do [repositório `apt` do Docker](#instalar-usando-o-repositório).

- [Instale manualmente](#instalar-a-partir-de-um-pacote) e gerencie as atualizações manualmente.

- Use um [script de conveniência](#instalar-usando-o-script-de-conveniência). Recomendado apenas para ambientes de teste e desenvolvimento.

*Apache License, Version 2.0. Consulte [LICENSE](https://github.com/moby/moby/blob/master/LICENSE) para ver a licença completa.*

### Instalar usando o repositório `apt` \{\#instalar-usando-o-repositório\}

Antes de instalar o Docker Engine pela primeira vez em uma nova máquina host, você precisa configurar o repositório `apt` do Docker. Depois disso, você pode instalar e atualizar o Docker a partir do repositório.

1. Configure o repositório `apt` do Docker.

```
\# Adicione a chave GPG oficial do Docker:  
sudo apt update  
sudo apt install ca-certificates curl  
sudo install -m 0755 -d /etc/apt/keyrings  
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc  
sudo chmod a+r /etc/apt/keyrings/docker.asc  
  
\# Adicione o repositório às fontes do Apt:  
sudo tee /etc/apt/sources.list.d/docker.sources \<\<EOF  
Types: deb  
URIs: https://download.docker.com/linux/debian  
Suites: $(. /etc/os-release && echo "$VERSION\_CODENAME")  
Components: stable  
Architectures: $(dpkg --print-architecture)  
Signed-By: /etc/apt/keyrings/docker.asc  
EOF  
  
sudo apt update
```

> \[!NOTA\]

> Se você usar uma distribuição derivada, como o Kali Linux, talvez precise substituir a parte deste comando que deve imprimir o codinome da versão:

```
$(. /etc/os-release && echo "$VERSION\_CODENAME")
```

Substitua esta parte pelo codinome da versão correspondente do Debian, como `bookworm`.

2. Instale os pacotes do Docker.

**Versão Mais Recente**

Para instalar a versão mais recente, execute:

```
$ sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

**Versão Específica**

Para instalar uma versão específica do Docker Engine, comece listando as versões disponíveis no repositório:

```
$ apt list --all-versions docker-ce  
  
docker-ce/bookworm 5:29.3.0-1~debian.12~bookworm \<arch\>  
docker-ce/bookworm 5:29.2.1-1~debian.12~bookworm \<arch\>  
...
```

Selecione a versão desejada e instale:

```
$ VERSION\_STRING=5:29.3.0-1~debian.12~bookworm  
$ sudo apt install docker-ce=$VERSION\_STRING docker-ce-cli=$VERSION\_STRING containerd.io docker-buildx-plugin docker-compose-plugin
```

> \[!NOTA\]

> O serviço Docker inicia automaticamente após a instalação. Para verificar se o Docker está em execução, use:

```
$ sudo systemctl status docker
```

Alguns sistemas podem ter esse comportamento desativado e exigirão um início manual:

```
$ sudo systemctl start docker
```

3. Verifique se a instalação foi bem-sucedida executando a imagem `hello-world`:

```
$ sudo docker run hello-world
```

Este comando baixa uma imagem de teste e a executa em um container. Quando o container é executado, ele imprime uma mensagem de confirmação e sai.

Agora você instalou e iniciou o Docker Engine com sucesso.

> \[!DICA\]

> Recebendo erros ao tentar executar sem root?

> O grupo de usuários `docker` existe, mas não contém usuários, razão pela qual você é obrigado a usar `sudo` para executar comandos do Docker. Continue para [pós-instalação no Linux](file:///engine/install/linux-postinstall) para permitir que usuários não privilegiados executem comandos do Docker e para outras etapas de configuração opcionais.

#### Atualizar o Docker Engine

Para atualizar o Docker Engine, siga a etapa 2 das [instruções de instalação](#instalar-usando-o-repositório), escolhendo a nova versão que deseja instalar.

### Instalar a partir de um pacote

Se você não puder usar o repositório `apt` do Docker para instalar o Docker Engine, poderá baixar o arquivo `.deb` para sua versão e instalá-lo manualmente. Você precisará baixar um novo arquivo sempre que quiser atualizar o Docker Engine.

1. Acesse [`https://download.docker.com/linux/debian/dists/`](https://download.docker.com/linux/debian/dists/).

2. Selecione sua versão do Debian na lista.

3. Vá para `pool/stable/` e selecione a arquitetura aplicável (`amd64`, `armhf`, `arm64` ou `s390x`).

4. Baixe os seguintes arquivos `.deb` para os pacotes Docker Engine, CLI, containerd e Docker Compose:

   - `containerd.io\_\<versão\>\_\<arquitetura\>.deb`

   - `docker-ce\_\<versão\>\_\<arquitetura\>.deb`

   - `docker-ce-cli\_\<versão\>\_\<arquitetura\>.deb`

   - `docker-buildx-plugin\_\<versão\>\_\<arquitetura\>.deb`

   - `docker-compose-plugin\_\<versão\>\_\<arquitetura\>.deb`

5. Instale os pacotes `.deb`. Atualize os caminhos no exemplo abaixo para onde você baixou os pacotes do Docker.

```
$ sudo dpkg -i ./containerd.io\_\<versão\>\_\<arquitetura\>.deb \\  
  ./docker-ce\_\<versão\>\_\<arquitetura\>.deb \\  
  ./docker-ce-cli\_\<versão\>\_\<arquitetura\>.deb \\  
  ./docker-buildx-plugin\_\<versão\>\_\<arquitetura\>.deb \\  
  ./docker-compose-plugin\_\<versão\>\_\<arquitetura\>.deb
```

> \[!NOTA\]
>
> O serviço Docker inicia automaticamente após a instalação. Para verificar se o Docker está em execução, use:

```
$ sudo systemctl status docker
```

Alguns sistemas podem ter esse comportamento desativado e exigirão um início manual:

```
$ sudo systemctl start docker
```

6. Verifique se a instalação foi bem-sucedida executando a imagem `hello-world`:

```
$ sudo docker run hello-world
```

Este comando baixa uma imagem de teste e a executa em um container. Quando o container é executado, ele imprime uma mensagem de confirmação e sai.

Agora você instalou e iniciou o Docker Engine com sucesso.

> \[!DICA\]
>
> Recebendo erros ao tentar executar sem root?
>
> O grupo de usuários `docker` existe, mas não contém usuários, razão pela qual você é obrigado a usar `sudo` para executar comandos do Docker. Continue para [pós-instalação no Linux](file:///engine/install/linux-postinstall) para permitir que usuários não privilegiados executem comandos do Docker e para outras etapas de configuração opcionais.

#### Atualizar o Docker Engine

Para atualizar o Docker Engine, baixe os arquivos de pacote mais recentes e repita o [procedimento de instalação](#instalar-a-partir-de-um-pacote), apontando para os novos arquivos.

### Instalar usando o script de conveniência

O Docker fornece um script de conveniência em [https://get.docker.com/](https://get.docker.com/) para instalar o Docker em ambientes de desenvolvimento de forma não interativa. O script de conveniência não é recomendado para ambientes de produção, mas é útil para criar um script de provisionamento adaptado às suas necessidades. Consulte também as etapas de [instalar usando o repositório](#instalar-usando-o-repositório) para aprender sobre as etapas de instalação usando o repositório de pacotes. O código-fonte do script é open source e você pode encontrá-lo no [repositório `docker-install` no GitHub](https://github.com/docker/docker-install).

<!-- prettier-ignore -->
Sempre examine scripts baixados da internet antes de executá-los localmente. Antes de instalar, familiarize-se com os riscos e limitações potenciais do script de conveniência:

- O script requer privilégios de `root` ou `sudo` para ser executado.
- O script tenta detectar sua distribuição Linux e versão e configurar seu sistema de gerenciamento de pacotes para você.
- O script não permite personalizar a maioria dos parâmetros de instalação.
- O script instala dependências e recomendações sem pedir confirmação. Isso pode instalar um grande número de pacotes, dependendo da configuração atual da sua máquina host.
- Por padrão, o script instala a versão estável mais recente do Docker, containerd e runc. Ao usar este script para provisionar uma máquina, isso pode resultar em atualizações de versão principais inesperadas do Docker. Sempre teste atualizações em um ambiente de teste antes de implantar em seus sistemas de produção.
- O script não foi projetado para atualizar uma instalação existente do Docker. Ao usar o script para atualizar uma instalação existente, as dependências podem não ser atualizadas para a versão esperada, resultando em versões desatualizadas.

> [!DICA]
>
> Visualize as etapas do script antes de executar. Você pode executar o script com a opção `--dry-run` para saber quais etapas o script executará quando invocado:
>
> ```console
> $ curl -fsSL https://get.docker.com -o get-docker.sh
> $ sudo sh ./get-docker.sh --dry-run
> ```

Este exemplo baixa o script de [https://get.docker.com/](https://get.docker.com/) e o executa para instalar a versão estável mais recente do Docker no Linux:

```console
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
Executing docker install script, commit: 7cae5f8b0decc17d6571f9f52eb840fbc13b2737
<...>
```

Agora você instalou e iniciou o Docker Engine com sucesso. O serviço `docker` inicia automaticamente em distribuições baseadas em Debian. Em distribuições baseadas em `RPM`, como CentOS, Fedora ou RHEL, você precisa iniciá-lo manualmente usando o comando apropriado `systemctl` ou `service`. Como a mensagem indica, usuários não-root não podem executar comandos do Docker por padrão.

> **Usar o Docker como um usuário não privilegiado ou instalar no modo rootless?**
>
> O script de instalação requer privilégios de `root` ou `sudo` para instalar e usar o Docker. Se você deseja conceder acesso ao Docker a usuários não-root, consulte as [etapas de pós-instalação para Linux](file:///engine/install/linux-postinstall/#manage-docker-as-a-non-root-user). Você também pode instalar o Docker sem privilégios de `root` ou configurá-lo para rodar no modo rootless. Para instruções sobre como executar o Docker no modo rootless, consulte [executar o daemon Docker como um usuário não-root (modo rootless)](file:///engine/security/rootless/).

#### Instalar pré-lançamentos

O Docker também fornece um script de conveniência em [https://test.docker.com/](https://test.docker.com/) para instalar pré-lançamentos do Docker no Linux. Este script é igual ao script em `get.docker.com`, mas configura seu gerenciador de pacotes para usar o canal de teste do repositório de pacotes do Docker. O canal de teste inclui versões estáveis e pré-lançamentos (versões beta, candidatos a lançamento) do Docker. Use este script para obter acesso antecipado a novos lançamentos e avaliá-los em um ambiente de teste antes de serem lançados como estáveis.

Para instalar a versão mais recente do Docker no Linux a partir do canal de teste, execute:

```
$ curl -fsSL https://test.docker.com -o test-docker.sh  
$ sudo sh test-docker.sh
```

#### Atualizar o Docker após usar o script de conveniência

Se você instalou o Docker usando o script de conveniência, deve atualizar o Docker diretamente usando seu gerenciador de pacotes. Não há vantagem em executar novamente o script de conveniência. Executá-lo novamente pode causar problemas se ele tentar reinstalar repositórios que já existem na máquina host.

## Desinstalar o Docker Engine

1. Desinstale os pacotes Docker Engine, CLI, containerd e Docker Compose:

```
$ sudo apt purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
```

2. Imagens, containers, volumes ou arquivos de configuração personalizados em seu host não são removidos automaticamente. Para excluir todas as imagens, containers e volumes:

```
$ sudo rm -rf /var/lib/docker  
$ sudo rm -rf /var/lib/containerd
```

3. Remova a lista de fontes e os chaveiros:

```
$ sudo rm /etc/apt/sources.list.d/docker.sources  
$ sudo rm /etc/apt/keyrings/docker.asc
```

Você deve excluir manualmente quaisquer arquivos de configuração editados.

