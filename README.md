# MQTT com Node-RED

Sistema integrado de automação residencial usando **MQTT** e **Node-RED**, com monitoramento em tempo real de dispositivos, sensores e alertas de segurança.

---

## Sumário

- [Sobre o Projeto](#sobre-o-projeto)
- [Requisitos](#requisitos)
- [Instalação](#instalação)
- [Como Usar](#como-usar)
- [Estrutura MQTT](#estrutura-mqtt)
- [Configurações e QoS](#configurações-e-qos)
- [FAQ](#faq)

---

## Sobre o Projeto

Este projeto implementa uma solução completa de automação residencial baseada em:

- **MQTT**: Protocolo leve para comunicação entre dispositivos
- **Node-RED**: Plataforma visual para orquestração de fluxos
- **Mosquitto**: Broker MQTT de alta performance

**Funcionalidades:**
✅ Controle remoto de dispositivos (luzes, ar-condicionado, etc)  
✅ Monitoramento de sensores (temperatura, umidade, luminosidade, gás)  
✅ Sistema de alertas críticos em tempo real  
✅ Dashboard interativo  

---

## Requisitos

- **Sistema Operacional**: Linux/macOS/Windows (com WSL)
- **Node.js**: v16.0.0 ou superior
- **Mosquitto**: v2.0.0 ou superior
- **Docker** (opcional): Para containerização

### Verificar Instalações

```bash
# Verificar Node-RED
which node-red

# Verificar Mosquitto
mosquitto -v

# Verificar Node.js
node --version
```

---

## Instalação

### 1. Instalar Dependências

```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install mosquitto mosquitto-clients nodejs npm

# macOS
brew install mosquitto node-red node
```

### 2. Iniciar Mosquitto

```bash
# Verificar status
sudo systemctl status mosquitto

# Se não estiver rodando
sudo systemctl start mosquitto

# (Opcional) Habilitar na inicialização
sudo systemctl enable mosquitto
```

### 3. Instalar e Iniciar Node-RED

```bash
# Instalar globalmente
npm install -g node-red

# Iniciar
node-red
```

O Node-RED estará disponível em `http://localhost:1880`

### 4. Importar o Fluxo

1. Abra o Node-RED em seu navegador: **http://localhost:1880**
2. Clique em Menu (≡) → **Import**
3. Cole o conteúdo do arquivo `nodered.json`
4. Clique em **Deploy**

---

## Como Usar

### 1️⃣ Monitoramento Global (Modo Debug)

Escute **TUDO** que acontece na casa em tempo real:

```bash
mosquitto_sub -h localhost -p 1883 -t "casa/#" -v
```

**O que você verá:** Temperatura, umidade, luminosidade, alertas e comandos de todos os cômodos.

**Quando usar:** Verificar se o sistema inteiro está comunicando corretamente.

---

### 2️⃣ Estados dos Dispositivos

Monitore apenas o status dos aparelhos controláveis:

```bash
mosquitto_sub -h localhost -p 1883 -t "casa/+/+/estado" -v
```

**O que você verá:** Apenas respostas `ON` ou `OFF` de luzes, ar-condicionado, etc.

**Quando usar:** Confirmar que os comandos foram executados.

---

### 3️⃣ Canal de Alertas Críticos

Receba apenas notificações de emergência:

```bash
mosquitto_sub -h localhost -p 1883 -t "casa/alertas" -v
```

**O que você verá:** Apenas quando temperatura ultrapassa limite ou sensor de gás dispara.

**Quando usar:** Monitorar segurança de forma desatenta.

---

### 4️⃣ Teste de Comando Manual

Simule o envio de um comando:

```bash
mosquitto_pub -h localhost -p 1883 -t "casa/sala/luz/comando" \
  -m '{"comando":"LIGAR","dispositivo":"sala_luz_01","timestamp":"2026-04-28T20:00:00Z"}' \
  -q 1
```

---

## Estrutura MQTT

```
casa/
├── sala/
│   ├── temperatura          (QoS 0 - Telemetria)
│   ├── umidade              (QoS 0 - Telemetria)
│   ├── luminosidade         (QoS 0 - Telemetria)
│   └── luz/
│       ├── comando          (QoS 1 - Comando)
│       └── estado           (QoS 1 - Estado, Retain: true)
│
├── quarto/
│   ├── temperatura          (QoS 0 - Telemetria)
│   ├── umidade              (QoS 0 - Telemetria)
│   ├── luminosidade         (QoS 0 - Telemetria)
│   ├── presenca             (QoS 1 - Retain: true)
│   └── ar/
│       ├── comando          (QoS 1 - Comando)
│       └── estado           (QoS 1 - Estado, Retain: true)
│
├── cozinha/
│   ├── temperatura          (QoS 0 - Telemetria)
│   ├── umidade              (QoS 0 - Telemetria)
│   └── gas                  (QoS 2 - Crítico)
│
└── alertas                  (QoS 2 - Crítico, Retain: true)
```

---

## Configurações e QoS

| Recurso | Configuração | QoS | Motivo |
|---------|--------------|-----|--------|
| **Telemetria** | Temperatura, Umidade, etc. | 0 | Dados contínuos; se perder um pacote, o próximo chega em segundos |
| **Comandos** | Luzes, Ar-condicionado | 1 | Garante entrega pelo menos uma vez |
| **Estados** | ON/OFF de devices | 1 + Retain | Dispositivo recebe o estado ao conectar |
| **Alertas Críticos** | Temperatura > limite, Gás | 2 | Não pode ser perdido nem duplicado |
| **Retain** | Estados e Alertas | - | Dashboard carrega estado atual ao abrir |

---
