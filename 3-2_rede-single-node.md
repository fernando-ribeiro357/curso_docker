# Módullo 3.2: Rede em Single Node

O Docker fornece redes isoladas para que containers possam se comunicar de forma segura e eficiente, simulando máquinas virtuais conectadas em uma rede privada virtual.

## Drivers de Rede Padrão

Ao instalar o Docker, três redes são criadas automaticamente:

1.  **`bridge`**: A rede padrão.
    *   Containers nesta rede recebem IPs privados internos.
    *   Comunicação externa requer mapeamento de portas (`-p`).
    *   *Limitação:* Resolução de DNS por nome do container **não** funciona automaticamente aqui (precisa usar IP ou criar rede customizada).
2.  **`host`**: Remove o isolamento de rede.
    *   O container compartilha a interface de rede e IP diretamente com o host.
    *   Não há mapeamento de portas (`-p` não funciona).
    *   *Uso:* Máxima performance de rede ou serviços que exigem muitas portas dinâmicas.
3.  **`none`**: Desabilita completamente a rede.
    *   Apenas loopback local (`lo`).
    *   *Uso:* Cargas de trabalho totalmente isoladas ou que gerenciam sua própria rede internamente.

## Criação de Redes Customizadas (Melhor Prática)

Para aproveitar todo o potencial do Docker, recomenda-se criar redes personalizadas para cada aplicação ou conjunto de serviços relacionados.

**Por que usar redes customizadas?**
1.  **DNS Interno Automático:** Containers na mesma rede customizada conseguem se resolver mutuamente pelo **nome do container** (ex: `ping meu-banco`) sem precisar saber o IP.
2.  **Isolamento:** Segmenta serviços (ex: rede `frontend`, rede `backend`, rede `database`). Um container na rede `frontend` não consegue acessar a `database` diretamente se não estiver conectado a ela.
3.  **Drivers Avançados:** Permite usar drivers como `overlay` (para Swarm) ou `macvlan`.

**Exemplos de Comandos:**

```bash
# Criar uma nova rede bridge chamada 'minha-rede-app'
docker network create minha-rede-app

# Listar redes
docker network ls

# Inspecionar detalhes da rede (IPs dos containers, gateway, etc.)
docker network inspect minha-rede-app

# Conectar um container existente a uma rede
docker network connect minha-rede-app nome-do-container

# Executar um container já conectado à rede customizada
docker run -d --name meu-backend --network minha-rede-app node:alpine
docker run -d --name meu-db --network minha-rede-app postgres

# Agora, dentro do 'meu-backend', você pode conectar no banco usando o hostname 'meu-db':
# psql -h meu-db -U postgres
```

## Isolamento e Exposição Seletiva

*   **Isolamento:** Por padrão, containers em redes diferentes não se comunicam. Isso é fundamental para segurança (Defense in Depth).
*   **Exposição Seletiva:** Para permitir acesso externo, use a flag `-p` (publish). Sem isso, o serviço roda apenas internamente.
```bash
# Expor porta 8080 do host para a porta 80 do container
docker run -d --name web -p 8080:80 --network minha-rede-app nginx
```
*   **Firewall:** Lembre-se que regras de firewall do host (`ufw`, `firewalld`, `iptables`) podem afetar o tráfego nas portas publicadas. O Docker gerencia suas próprias regras na cadeia `DOCKER-USER`, mas é recomendável adicionar regras personalizadas lá para evitar conflitos.

## Referências:
*   [Visão geral de networking](https://docs.docker.com/network/)
*   [Comando docker network](https://docs.docker.com/engine/reference/commandline/network/)
*   [Configuração de firewalls e packet filtering](https://docs.docker.com/engine/network/packet-filtering-firewalls/)

## Próximos Passos
*   [Configs e Secrets](3-3_configs-e-secrets.md)