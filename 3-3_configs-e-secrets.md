# Módulo 3.3: Configs e Secrets (Introdução)

Gerenciar configurações e dados sensíveis corretamente é vital para a segurança e flexibilidade da aplicação. Evite "hardcodar" senhas ou caminhos de configuração dentro das imagens Docker.

## Por que não usar Variáveis de Ambiente (`ENV`) para tudo?

Embora variáveis de ambiente sejam convenientes, elas têm limitações críticas de segurança e usabilidade:
1.  **Visibilidade:** São visíveis em texto plano no comando `docker inspect`, no histórico do shell e no processo (`/proc/<pid>/environ`), expondo senhas acidentalmente.
2.  **Tamanho Limitado:** Não são ideais para arquivos grandes de configuração ou certificados SSL completos.
3.  **Imutabilidade:** Alterar uma variável de ambiente geralmente exige recriar o container.
4.  **Formato de Arquivo:** Muitas aplicações esperam ler configurações de arquivos (ex: `config.json`), não de variáveis. Converter ENVs para arquivos exige scripts complexos de entrada (`entrypoint`).

## Conceito de Docker Configs

**Configs** são projetadas para arquivos de configuração **não sensíveis** que precisam ser injetados nos containers.
*   **Funcionamento:** Você cria um config a partir de um arquivo no host. O Docker armazena esse conteúdo e o monta como um arquivo de leitura apenas dentro do container (geralmente em `/configs/`).
*   **Vantagem:** Permite atualizar a configuração sem reconstruir a imagem. Mantém a imagem genérica e limpa.

## Conceito de Docker Secrets

**Secrets** são projetados especificamente para dados **sensíveis** (senhas, chaves SSH, tokens API, certificados).
*   **Segurança:** Nunca aparecem em texto plano no `docker inspect` ou logs. São transmitidos de forma criptografada (no Swarm) e montados como arquivos temporários em RAM (`/run/secrets/`), com permissões restritas (apenas o usuário do processo lê).
*   **Rotação:** Facilita a rotação de credenciais sem downtime.

> **Nota Importante sobre Modo Standalone vs. Swarm:**
> Os recursos nativos `docker config` e `docker secret` foram desenhados primariamente para o **Docker Swarm**.
> *   **No Swarm:** Funcionalidade completa, gerenciamento centralizado, distribuição criptografada e rotação segura.
> *   **No Modo Standalone (Single Node):** O Docker permite criar e usar configs/secrets localmente para fins de teste e compatibilidade, mas eles funcionam de maneira limitada (simulando a montagem de arquivos locais). Para ambientes de produção robustos que exigem gestão segura de segredos em escala, a migração para Swarm ou o uso de soluções externas (como HashiCorp Vault) é recomendada. No entanto, entender esses conceitos é fundamental, pois a filosofia de "injetar segredos como arquivos" é uma prática padrão na indústria.

**Exemplos de Comandos (Contexto Swarm/Avançado):**

```bash
# Criar um Secret (ex: senha do banco)
echo "minha_senha_super_secreta" | docker secret create db_password -

# Criar um Config (ex: arquivo de configuração nginx)
docker config create nginx_conf ./nginx.conf

# Usar no serviço (Swarm)
docker service create \
  --name meu-servico \
  --secret db_password \
  --config source=nginx_conf,target=/etc/nginx/nginx.conf \
  nginx:latest

# Dentro do container, os dados estarão disponíveis como arquivos:
# Senha em: /run/secrets/db_password
# Config em: /configs/nginx_conf
```

**Alternativa para Single Node (Simulação Segura):**
Em desenvolvimento local sem Swarm, uma prática comum é usar volumes secretos ou ferramentas como `docker-compose` com arquivos `.env` (que devem estar no `.gitignore`) para gerenciar variáveis, embora não ofereça o mesmo nível de isolamento de memória dos Secrets nativos.

## Referências:
*   [Gerenciando dados sensíveis com Secrets](https://docs.docker.com/engine/swarm/secrets/)
*   [Gerenciando configurações com Configs](https://docs.docker.com/engine/swarm/configs/)
*   [Melhores práticas de segurança](https://docs.docker.com/engine/security/)

## Próximos Passos:
*   Módulo 4: Orquestração com Docker Swarm