# Módulo 3.1: Armazenamento (Storage Drivers & Volumes)

Por padrão, os containers são **efêmeros**. Quando um container é removido, todos os dados escritos em seu sistema de arquivos desaparecem. Para aplicações que precisam persistir dados (como bancos de dados, logs ou uploads de usuários), o Docker oferece mecanismos robustos de armazenamento baseados em *Storage Drivers*.

## Tipos de Montagem

Existem três formas principais de montar dados em um container, cada uma com casos de uso específicos:

### **A. Volumes (Recomendado para Produção)**
*   **O que são:** Mecanismo de armazenamento gerenciado inteiramente pelo Docker. Os volumes são armazenados em uma parte específica do sistema de arquivos do host (geralmente `/var/lib/docker/volumes/` no Linux).
*   **Vantagens:**
    *   **Portabilidade:** Não dependem da estrutura de diretórios do host, facilitando a migração entre Linux, Windows e Mac.
    *   **Performance:** Geralmente oferecem melhor performance de I/O comparado a Bind Mounts, especialmente em macOS e Windows.
    *   **Gerenciamento:** Podem ser criados, listados e removidos via comandos Docker (`docker volume`).
    *   **Backup:** Mais fáceis de fazer backup usando ferramentas nativas ou containers auxiliares.
    *   **Segurança:** Isolados de outros processos do host.
*   **Cenário de Uso:** Bancos de dados (PostgreSQL, MySQL), dados persistentes de aplicações, caches que precisam sobreviver a reinícios.

**Exemplos de Comandos:**

```bash
# Criar um novo volume chamado 'meu-dados'
docker volume create meu-dados

# Listar todos os volumes
docker volume ls

# Inspecionar detalhes de um volume (localização no disco, drivers, containers associados)
docker volume inspect meu-dados

# Executar um container montando o volume na pasta /data dentro do container
docker run -d --name meu-app -v meu-dados:/data nginx

# Remover um volume (apenas se não estiver em uso por nenhum container)
docker volume rm meu-dados

# Limpeza agressiva: remove volumes órfãos (não utilizados por nenhum container)
docker system prune --volumes
```

### **B. Bind Mounts (Ideal para Desenvolvimento)**
*   **O que são:** Mapeiam um arquivo ou diretório específico do sistema de arquivos do host diretamente para dentro do container. O caminho deve ser absoluto.
*   **Vantagens:**
    *   **Desenvolvimento Ágil:** Permite editar código no host e ver alterações instantâneas no container sem reconstruir a imagem (Hot Reload).
    *   **Acesso Direto:** Útil para compartilhar arquivos de configuração específicos do host ou logs diretamente com o administrador do sistema.
*   **Desvantagens:** Acopla o container à estrutura de diretórios do host, reduzindo a portabilidade. Se o caminho não existir no host, o comportamento pode variar (erro ou criação automática dependendo da flag).
*   **Cenário de Uso:** Desenvolvimento de código fonte, montagem de arquivos de configuração locais (`nginx.conf`), compartilhamento de logs.

**Exemplos de Comandos:**

```bash
# Montar o diretório atual do host (/caminho/absoluto) na pasta /app do container
# Nota: Em Linux/Mac use $(pwd), no Windows PowerShell use ${PWD}
docker run -d --name dev-app -v $(pwd):/app node:alpine npm start

# Montar um arquivo específico de configuração
docker run -d --name webserver -v /etc/nginx/nginx.conf:/etc/nginx/nginx.conf:ro nginx
# :ro significa read-only (somente leitura), aumentando a segurança
```

### **C. tmpfs (Memória RAM)**
*   **O que são:** Armazenamento temporário mantido exclusivamente na memória RAM do host. Nada é escrito no disco rígido.
*   **Características:**
    *   **Velocidade Extrema:** Acesso direto à RAM.
    *   **Efemeridade Total:** Dados perdidos ao parar/remover o container.
    *   **Segurança:** Ideal para dados sensíveis que nunca devem tocar o disco (ex: chaves temporárias, tokens de sessão).
*   **Limitação:** Disponível nativamente apenas em containers Linux.
*   **Cenário de Uso:** Caches voláteis de alta performance, processamento de dados sensíveis temporários.

**Exemplos de Comandos:**

```bash
# Montar um tmpfs de 64MB na pasta /tmp/cache
docker run -d --name cache-app --tmpfs /tmp/cache:rw,noexec,nosuid,size=64m redis
```

## Estratégia de Backup de Volumes

Como os volumes residem no host, a estratégia mais segura e portátil é usar um container auxiliar para realizar o backup, evitando depender de ferramentas específicas do SO do host.

**Exemplo: Backup de um Volume para um arquivo `.tar.gz`**

```bash
# Sintaxe: docker run --rm --mount source=<volume>,target=/dados --mount type=bind,source=<pasta_host>,target=/backup alpine tar czf /backup/backup.tar.gz -C /dados .

docker run --rm \
  --mount source=meu-banco-dados,target=/data \
  --mount type=bind,source=$(pwd)/backups,target=/backup \
  alpine tar czf /backup/backup-$(date +%F).tar.gz -C /data .
```
_Este comando sobe um container Alpine temporário, monta o volume de dados e uma pasta local de backups, compacta tudo e encerra._

## Recursos Adicionais:
*   [Visão geral de storage](https://docs.docker.com/storage/)
*   [Usando volumes](https://docs.docker.com/storage/volumes/)
*   [Usando bind mounts](https://docs.docker.com/storage/bind-mounts/)
*   [Usando tmpfs](https://docs.docker.com/storage/tmpfs/)

## Próximos Passos

- Rede em single node