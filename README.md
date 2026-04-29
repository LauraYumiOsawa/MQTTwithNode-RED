# MQTTwithNode-RED

1. Monitoramento Global (Modo Debug)
Bash
mosquitto_sub -h localhost -p 1883 -t "casa/#" -v
O que você vai ouvir: TUDO o que acontece na casa.

Pra que serve: O símbolo # é o super-coringa, significa "tudo o que vier daqui pra frente". Você verá uma enxurrada de dados: temperatura, umidade, luminosidade, alertas e comandos de todos os cômodos. Perfeito para verificar se o sistema inteiro está comunicando.

2. Estados dos Dispositivos
Bash
mosquitto_sub -h localhost -p 1883 -t "casa/+/+/estado" -v
O que você vai ouvir: A confirmação de estado de TODOS os aparelhos controláveis (Luz, Ar-condicionado, etc).

Pra que serve: O símbolo + funciona como um coringa para um único nível. Este comando filtra por: casa / qualquer_comodo / qualquer_aparelho / estado. Se você ligar a luz da sala no Dashboard, a confirmação ("ON" ou "OFF") aparecerá aqui.

3. Canal de Alertas Críticos
Bash
mosquitto_sub -h localhost -p 1883 -t "casa/alertas" -v
O que você vai ouvir: Apenas os avisos de emergência.

Pra que serve: A tela ficará vazia a maior parte do tempo. Ele só imprimirá algo se a temperatura passar do limite ou se o sensor de gás disparar. Ideal para rodar em um monitor separado focado em segurança.

## Estrutura MQTT

casa/
├── sala/
│   ├── temperatura, umidade, luminosidade (QoS 0)
│   └── luz/ (comando: QoS 1 | estado: QoS 1, Retain true)
├── quarto/
│   ├── temperatura, umidade, luminosidade (QoS 0)
│   ├── presenca (QoS 1, Retain true)
│   └── ar/ (comando: QoS 1 | estado: QoS 1, Retain true)
├── cozinha/
│   ├── temperatura, umidade (QoS 0)
│   └── gas (QoS 2)
└── alertas (QoS 2)

Como Instalar e Testar
1. Pré-requisitos
Certifique-se de que o Mosquitto e o Node-RED estão em execução:

Bash
# Verificar Mosquitto
sudo systemctl status mosquitto

# Iniciar Node-RED
node-red
2. Importar o Fluxo
Abra o Node-RED em seu navegador.

Vá em Menu (≡) -> Import.

Cole o conteúdo do arquivo nodered-flow.json.

Clique em Deploy.

3. Teste de Comando Manual
Você pode simular o envio de um comando pelo terminal para ver o sistema respondendo:

Bash
mosquitto_pub -h localhost -p 1883 -t "casa/sala/luz/comando" \
  -m '{"comando":"LIGAR","dispositivo":"sala_luz_01","timestamp":"2026-04-28T20:00:00Z"}' \
  -q 1



Recurso,Configuração,Motivo
QoS 0,Telemetria Simples,"Dados contínuos; se perder um pacote, o próximo chega em segundos."
QoS 1,Comandos e Estados,Garante que a lâmpada receba a ordem de ligar ao menos uma vez.
QoS 2,Alertas e Gás,Crítico; o alerta não pode ser perdido nem duplicado.
Retain: True,Estados (ON/OFF),"Garante que, ao abrir o Dashboard, ele saiba o estado atual do aparelho imediatamente."
