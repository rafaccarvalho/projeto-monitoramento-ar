# Monitoramento da Qualidade do Ar em Ambientes Escolares
### Contribuição ao ODS 3 – Saúde e Bem-Estar | Agenda 2030 da ONU
**Universidade Presbiteriana Mackenzie — Faculdade de Computação e Informática**

Integrantes: 

- Julia D'agrela Araújo
- Kauã Paixão
- Pedro Henrique Gonçalves
- Rafael Carvalho Cordeiro

---

## 🎯 Objetivo do Projeto

Este projeto propõe o desenvolvimento de um sistema IoT de baixo custo para monitoramento da qualidade do ar em salas de aula de escolas públicas brasileiras, alinhado ao Objetivo de Desenvolvimento Sustentável 3 (ODS 3 – Saúde e Bem-Estar) da Agenda 2030 da ONU, especificamente à meta 3.9, que prevê a redução de doenças causadas pela poluição do ar.

O sistema coleta dados de temperatura, umidade e concentração de CO2, processa essas informações em tempo real, aciona alertas automáticos e disponibiliza dashboards gerenciais para gestores escolares e profissionais de saúde pública.

---

## 📁 Estrutura do Repositório

```
/
├── sala101.ino      ← Código fonte do ESP32 da Sala 101
├── sala203.ino      ← Código fonte do ESP32 da Sala 203
├── flows.json       ← Exportação do Node-RED com toda a lógica de integração
└── README.md
```


---

## ⚙️ Descrição do Funcionamento

Dois dispositivos ESP32 simulados no Wokwi coletam continuamente dados de temperatura, umidade e qualidade do ar (CO2) através dos sensores DHT22 e MQ135. Esses dados são publicados via protocolo MQTT a um broker na nuvem (HiveMQ).

O Node-RED recebe as mensagens do broker, aplica as regras de negócio e executa simultaneamente três ações quando detecta condições críticas:

1. Publica um comando MQTT de volta ao dispositivo para acionar o buzzer e o LED RGB
2. Envia uma notificação automática via bot do Telegram ao responsável pela escola
3. Registra o evento no banco de dados InfluxDB para rastreabilidade histórica

O Grafana consome os dados do InfluxDB e gera três dashboards gerenciais com monitoramento em tempo real, histórico de alertas e análise temporal.

---

## 🏗️ Arquitetura da Solução

```
[ESP32 Sala 101]──┐
                  ├──→ [MQTT - HiveMQ Cloud] ──→ [Node-RED - Railway]
[ESP32 Sala 203]──┘         ↑                          │
        ↑                   │                    ┌─────┼──────────────┐
        │                   │                    ↓     ↓              ↓
   [Atuadores]         [Comando]           [InfluxDB] [Telegram]  [OpenWeatherMap]
   Buzzer/LED RGB      de volta                 │
                                                ↓
                                           [Grafana]
                                        Dashboards Gerenciais
```

### Topologia Lógica

| Elemento | Plataforma | FQDN | IP |
|---|---|---|---|
| Broker MQTT | HiveMQ Cloud | `clusternome-4c6b47d2.a01.euc1.aws.hivemq.cloud` | 3.125.109.117 |
| Node-RED | Railway | `node-red-production-93a1.up.railway.app` | 34.107.141.139 |
| Banco de dados | InfluxDB Cloud | `us-east-1-1.aws.cloud2.influxdata.com` | 3.224.23.140 |
| Dashboards | Grafana Cloud | `rafacarvalhocordeiro.grafana.net` | 104.18.12.97 |

---

## 🛠️ Tecnologias Utilizadas

| Camada | Tecnologia | Função |
|---|---|---|
| Hardware | ESP32 (Wokwi) | Microcontrolador com Wi-Fi integrado |
| Sensor | DHT22 | Temperatura e umidade relativa |
| Sensor | MQ135 | Concentração de CO2 e gases |
| Atuador | Buzzer + LED RGB | Alertas sonoros e visuais locais |
| Protocolo | MQTT (HiveMQ) | Comunicação entre dispositivos e nuvem |
| Plataforma | Node-RED | Orquestração e regras de negócio |
| Banco de dados | InfluxDB Cloud | Armazenamento de séries temporais |
| Dashboards | Grafana Cloud | Visualização gerencial |
| Notificações | Telegram Bot | Alertas automáticos em tempo real |
| API Externa | OpenWeatherMap | Dados climáticos externos |
| Simulação | Wokwi | Simulação dos dispositivos ESP32 |

---

## 🔄 Fluxo de Dados

```
1. ESP32 coleta dados (DHT22 + MQ135) a cada 5 segundos
         ↓
2. Publica JSON via MQTT:
   {"sala":"Sala 101","temp":25.3,"umidade":60,"co2":850}
         ↓
3. Node-RED recebe e aplica regras:
   - co2 < 1000 e temp < 24  →  NORMAL   →  LED Verde
   - co2 1000-2000 ou temp 24-28  →  ATENÇÃO  →  LED Amarelo
   - co2 > 2000 ou temp > 28  →  CRÍTICO  →  LED Vermelho + Buzzer
         ↓
4. Se ATENÇÃO ou CRÍTICO:
   - Aciona atuador via MQTT
   - Envia alerta no Telegram
   - Registra no InfluxDB
         ↓
5. Grafana lê InfluxDB e atualiza os dashboards em tempo real
```

---

## 🚀 Instruções de Instalação e Execução

### Pré-requisitos
- Conta no [Wokwi](https://wokwi.com)
- Conta no [HiveMQ Cloud](https://www.hivemq.com) (gratuito)
- Conta no [Railway](https://railway.app) (para Node-RED)
- Conta no [InfluxDB Cloud](https://cloud2.influxdata.com) (gratuito)
- Conta no [Grafana Cloud](https://grafana.com) (gratuito)
- Bot do Telegram criado via @BotFather

### 1. Configurar o Broker MQTT
1. Crie um cluster gratuito no HiveMQ Cloud
2. Anote: host, porta (8883), usuário e senha

### 2. Configurar o Node-RED
1. Faça deploy no Railway usando o template Node-RED
2. Instale as paletas: `node-red-contrib-influxdb` e `node-red-contrib-telegrambot`
3. Importe o arquivo `flows.json` via Menu → Import
4. Configure as credenciais do HiveMQ, InfluxDB e Telegram

### 3. Configurar o InfluxDB
1. Crie um bucket chamado `escola_ar`
2. Gere um API Token com permissão de escrita
3. Configure no Node-RED com URL, org, bucket e token

### 4. Configurar o Grafana
1. Adicione o InfluxDB como Data Source
2. Use linguagem de query: Flux
3. Importe os dashboards conforme demonstrado no vídeo

### 5. Executar os dispositivos no Wokwi
1. Abra `sala101.ino` no Wokwi e substitua as credenciais
2. Abra `sala203.ino` no Wokwi e substitua as credenciais
3. Clique em ▶ Play nos dois circuitos simultaneamente

---

## 📊 Dashboards

O sistema conta com três dashboards no Grafana:

**Dashboard 1 — Monitoramento em Tempo Real**
Gauges de temperatura, umidade e CO2 com faixas coloridas de alerta e painel de status por sala.

**Dashboard 2 — Alertas e Atuações**
Status dos atuadores por sala, registro cronológico de alertas e contador de eventos críticos por dia, semana e mês.

**Dashboard 3 — Análise Histórica e Gerencial**
Gráficos temporais (24h, 7 e 30 dias), heatmap de qualidade do ar por horário e correlação com dados climáticos externos via API OpenWeatherMap.

---

## 🔮 Melhorias Futuras

- Substituição do MQ135 por sensores de CO2 mais precisos (ex: SCD40)
- Implementação de algoritmos de análise preditiva com machine learning
- Integração com sistemas de automação predial para controle automático de ventilação e climatização
- Desenvolvimento de aplicativo móvel para gestores escolares
- Expansão para múltiplos ambientes escolares com centralização dos dados em um único dashboard
- Implementação de relatórios automáticos mensais por e-mail para secretarias de educação

---

## 🎥 Vídeo Demonstrativo



---


Universidade Presbiteriana Mackenzie
Faculdade de Computação e Informática
São Paulo – SP, Brasil
```
