Sexta-feira, 5 de setembro de 2025, 22:03. A noite em São Caetano do Sul continua.

Entendido. Aqui está a versão reescrita do arquivo README.md, com o mesmo conteúdo e estrutura, mas sem o uso de emojis.

Projeto MQTT Seguro com Docker
Este projeto oferece um ambiente MQTT completo e seguro, totalmente containerizado com Docker Compose. A arquitetura inclui um broker Mosquitto protegido, um sensor de temperatura em Python e um subscriber para testes, todos comunicando-se através de um canal criptografado com TLS.

O foco principal é demonstrar as melhores práticas de segurança em IoT: Autenticação por usuário/senha, Autorização por tópico (ACLs) e Criptografia de ponta a ponta (TLS).

Principais Funcionalidades
Segurança Robusta: Comunicação forçada via TLS na porta 8883.

Controle de Acesso: Autenticação obrigatória e permissões de leitura/escrita por usuário definidas em um arquivo de ACL.

Containerizado e Portátil: Todos os serviços são definidos no docker-compose.yml, garantindo um setup rápido e consistente em qualquer máquina com Docker.

Fluxo de Desenvolvimento Eficiente: O código-fonte do sensor é mapeado diretamente para o contêiner, permitindo edições em tempo real sem a necessidade de reconstruir a imagem.

Configuração Centralizada: As credenciais e configurações sensíveis são gerenciadas através de um arquivo .env para maior segurança.

Arquitetura
O ambiente opera em uma rede Docker interna, onde os clientes se conectam ao broker através de seu nome de serviço (mqtt-broker). As portas são expostas no host para interação externa e testes.

 MÁQUINA HOST
+---------------------------------------------------------------------------------+
|                                                                                 |
| [MQTT Explorer] ---- TLS/8883 ----> |           REDE DOCKER INTERNA           | |
| [mosquitto_sub] ---- TLS/8883 ----> |                                           | |
|                                     |  +------------------+       +-----------+ | |
|                                     |  | temperature-     |       | mqtt-     | | |
|                                     |  | sensor-1 (py)    |------>| broker    | | |
|                                     |  +------------------+ <------| Mosquitto | | |
|                                     |        ^ |                  +-----------+ | |
|                                     +--------|-|--------------------------------+ |
|                                              | |                                  |
| (Live Reload)                                | +--- (Arquivos de Configuração)      |
|                                              |                                  |
| [src/sensor.py] <----------------------------+      [mosquitto.conf, acl.conf,    |
|                                                      password.txt, certs/]        |
+---------------------------------------------------------------------------------+
Pré-requisitos
Docker

Docker Compose

OpenSSL (geralmente já instalado em Linux/macOS)

Como Executar o Projeto
Siga estes passos a partir de um terminal na pasta raiz do projeto.

1. Clonar o Repositório
Bash

git clone https://github.com/csai4cps/mqtt-project.git mqtt-project-seguro
cd mqtt-project-seguro
2. Criar e Configurar o Arquivo de Ambiente
As senhas dos usuários são gerenciadas em um arquivo .env para não ficarem expostas no docker-compose.yml.

Crie o arquivo a partir do exemplo:

Bash

# (No Linux/macOS)
cp .env.example .env

# (No Windows)
copy .env.example .env
Abra o arquivo .env e altere as senhas se desejar.

3. Gerar os Certificados TLS
Estes comandos criarão os certificados autoassinados necessários para a comunicação segura.

Bash

# Cria o diretório para os certificados
mkdir -p mosquitto/certs

# Gera o certificado do servidor (server.crt) e a chave (server.key)
docker run --rm -it -v $(pwd)/mosquitto/certs:/certs alpine/openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout /certs/server.key -out /certs/server.crt -subj "/CN=mqtt-broker"

# Gera o certificado da Autoridade Certificadora (ca.crt)
docker run --rm -it -v $(pwd)/mosquitto/certs:/certs alpine/openssl req -x509 -newkey rsa:2048 -keyout /certs/ca.key -out /certs/ca.crt -days 3650 -nodes -subj "/CN=MyProjectCA"
4. Criar os Usuários do Broker
Estes comandos usam o utilitário do Mosquitto para criar os usuários e senhas no arquivo password.txt.

Bash

# Cria o usuário 'sensor_user'
docker run --rm -it -v $(pwd)/mosquitto:/mosquitto eclipse-mosquitto:2.0.20 mosquitto_passwd -c -b /mosquitto/password.txt sensor_user ${SENSOR_PASSWORD}

# Adiciona o usuário 'subscriber_user'
docker run --rm -it -v $(pwd)/mosquitto:/mosquitto eclipse-mosquitto:2.0.20 mosquitto_passwd -b /mosquitto/password.txt subscriber_user ${SUBSCRIBER_PASSWORD}
5. Iniciar os Serviços
Este comando irá construir a imagem do sensor, baixar la imagem do Mosquitto e iniciar todos os contêineres em segundo plano.

Bash

docker compose up -d --build
O ambiente está no ar!

Testando a Aplicação
Verificando os Logs
Para ver o sensor publicando e o subscriber recebendo as mensagens, use o comando:

Bash

docker compose logs -f mqtt-subscriber
Você deverá ver as mensagens de temperatura chegando a cada 5 segundos.

Conectando um Cliente Externo (via Terminal)
Use o mosquitto_sub da sua máquina local para se conectar ao broker, usando as credenciais e o certificado.

Bash

mosquitto_sub \
  -h localhost \
  -p 8883 \
  -t 'sensor/#' \
  -v \
  -u "subscriber_user" \
  -P "SENHA_DO_SUBSCRIBER_AQUI" \
  --cafile ./mosquitto/certs/ca.crt
Conectando com o MQTT Explorer
Host: mqtts://localhost (ou o IP da sua máquina)

Port: 8883

Username: subscriber_user

Password: A senha que você definiu para o subscriber.

TLS: Ativado.

Certificate > CA File: Selecione o arquivo ca.crt na pasta mosquitto/certs.

Estrutura do Projeto (Recomendada)
.
├── .env                    # Credenciais (não versionar)
├── .env.example            # Arquivo de exemplo para as credenciais
├── docker-compose.yml      # Orquestração dos serviços
├── Dockerfile.temperature-sensor-1 # Definição da imagem do sensor
├── mosquitto/
│   ├── acl.conf            # Regras de acesso (ACL)
│   ├── certs/              # Certificados TLS
│   ├── mosquitto.conf      # Configuração do broker
│   └── password.txt        # Arquivo de senhas (não versionar)
└── src/
    └── temperature-sensor-1.py # Código Python do sensor

