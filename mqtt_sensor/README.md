# Projeto MQTT - Sensor Python com TLS e Autenticação

Este projeto demonstra um ambiente MQTT seguro, composto por um **broker Mosquitto**, um **sensor de temperatura em Python** e um **subscriber TLS**.  
O objetivo é garantir **autenticação, segregação de portas e criptografia** nas conexões externas, mantendo o sensor funcional na rede interna.

---

## Estrutura do Projeto

- `docker-compose.yml`: Define os serviços do broker, do sensor e do subscriber.
- `src/temperature-sensor-1.py`: Código Python do sensor de temperatura.
- `mosquitto/config/mosquitto.conf`: Arquivo de configuração do broker.
- `mosquitto/config/mosquitto.passwd`: Senhas de autenticação.
- `mosquitto/config/mosquitto.acl`: Regras de controle de acesso por tópico.
- `mosquitto/certs/`: Certificados TLS (`ca.crt`, `server.crt`, `server.key`).

---

## 🔧 Alterações e Correções

### 1. Sensor Python
O sensor Python foi corrigido para funcionar de forma confiável:

- **Protocolo:** Alterado de `MQTTv5` para `MQTTv3.1.1` (`MQTTv311`) garantindo compatibilidade com o Mosquitto 2.0.20.
- **Host e Porta:** Definidos como `mqtt-broker:1883` para comunicação interna via TCP.

Resultado: o sensor agora publica leituras de forma contínua e estável.

---

### 2. Broker MQTT
O broker foi configurado para oferecer comunicação segura:

- **Porta 1883:** Comunicação interna de sensores, sem TLS.
- **Porta 8883:** Conexões externas com **TLS** e **autenticação obrigatória**.
- **Segurança:**  
  - `allow_anonymous false` na porta externa.  
  - `password_file` configurado.  
  - **ACLs (Access Control Lists)** para controle de acesso.  
- **Criptografia:** Certificados TLS gerados para garantir integridade e autenticidade das mensagens.

**Exemplo de ACL:**
```text
user <USUÁRIO_CRIADO>
topic readwrite sensor/#
