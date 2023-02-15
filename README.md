# Nginx Proxy Manager Deployer

[NGINX](https://docs.nginx.com) é um servidor HTTP gratuito, de código aberto e de alto desempenho e proxy reverso, bem como um servidor proxy IMAP/POP3 e balanceador de carga. O NGINX é conhecido por seu alto desempenho, estabilidade, rico conjunto de recursos, configuração simples e baixo consumo de recursos.

O [Nginx Proxy Manager](https://nginxproxymanager.com) é um projeto que fornece uma maneira fácil de realizar proxy de hosts. Ele oferece um interface de administração bonita e segura; permite criar facilmente encaminhamentos e redirecionamentos de fluxos; fornece gerenciamento de certificados SSL/TLS; entre outros.  

## Sumário
- [Nginx Proxy Manager Deployer](#nginx-proxy-manager-deployer)
  - [Sumário](#sumário)
  - [Requisitos e Dependências](#requisitos-e-dependências)
  - [Instalação](#instalação)
    - [Estrutura de Diretórios](#estrutura-de-diretórios)
    - [Docker-Compose](#docker-compose)
      - [Portas](#portas)
      - [Volumes](#volumes)
      - [Variáveis de Ambiente (Environment)](#variáveis-de-ambiente-environment)
    - [Docker-Compose - Rede](#docker-compose---rede)
  - [Adicionando Containers na Rede - Nginx](#adicionando-containers-na-rede---nginx)
    - [Método 1 - Docker-Compose](#método-1---docker-compose)
    - [Método 2 - Terminal](#método-2---terminal)

## Requisitos e Dependências
- [Docker e Docker-Compose](https://docs.docker.com/)
  
## Instalação

### Estrutura de Diretórios

```bash
# Crie os diretórios.

# Dir. Config/Data
$ mkdir $(pwd)/lib-data

# Dir. Certificados letsencrypt
$ mkdir $(pwd)/etc-letsencrypt
```

Sugestão (no Linux):
- Dir. Config/Data: */var/lib/nginx-proxy-manager*
- Dir. Certificados letsencrypt: */etc/letsencrypt*
  
### Docker-Compose

#### Portas

```yml
# docker-compose.yml (Em services.app)

# Descomente (e/ou altere) as portas/serviços que você deseja oferecer.

ports:
# Porta de serviço HTTP
  - '80:80'
# Porta para a interface Web de administração.
  - '81:81'
# Porta de serviço HTTPS
  - '443:443'
```

#### Volumes

```yml
# docker-compose.yml (Em services.app)
# Aponte para as pastas criadas anteriormente.

# Antes
volumes:
  - '$(pwd)/var_lib_data:/data'
  - '$(pwd)/etc_letsencrypt:/etc/letsencrypt'

# Depois (exemplo)
volumes:
  - '/var/lib/nginx-proxy-manager:/data'
  - '/etc/letsencrypt:/etc/letsencrypt'
```

#### Variáveis de Ambiente (Environment)    

```yml
# docker-compose.yml (Em services.pihole-app)
# Altere o valores caso necessário. 

environment:
# Desabilita família IPV6.
  - DISABLE_IPV6=true
# Defini o caminho do DB, arquivos *.sqlite.
  - DB_SQLITE_FILE=/data/database.sqlite
# Configura diretiva HTTP.
  - X_FRAME_OPTIONS=sameorigin
```

### Docker-Compose - Rede

```yml
# docker-compose.yml (Em networks.nginx-proxymng-net.ipam)
# Altere o valores caso necessário. 

config:
# Endereço da rede
  - subnet: '172.18.0.0/28'
```

## Adicionando Containers na Rede - Nginx

A rede Nginx foi pensada para que matenha o isolamento completo de outros containers presentes na máquina host, por isso, para que o container Nginx alcance outros containers/hosts é necessário adicioná-los a rede. 

### Método 1 - Docker-Compose

```yml
# No [..]docker-compose.yml do seu serviço alvo.

# (Em networks) adicione:
nginx-proxymng-net:
  name: 'nginx-proxymng-net'
  external: true

# (Em services.SERVICENAME.networks) adicione:
- nginx-proxymng-net
```

### Método 2 - Terminal

```bash
# Execute

docker network connect nginx-proxymng-net CONTAINER_NAME --alias CONTAINER_ALIAS
```

Dica.: você poderá localizar os containers na rede através de seus IPs, para inspecionar isso use o comando "***docker inspect CONTAINER_NAME***". Ou simplesmene use o "***alias***" do container como se fosse um **Hostname/DNS**.