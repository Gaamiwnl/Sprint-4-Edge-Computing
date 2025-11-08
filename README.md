# ⚽ Sprint 4 - Edge Computing  
## Projeto: Monitoramento de gol via ESP32 com MQTT e Dashboard Web

### 👥 Integrantes do Grupo
- **Felipe de Andrade Godoi**  
- **Guilherme Augusto Ferreira**  
- **Raphael Naome Taketa**  
- **Victor Ribeiro Guimaraes Lira**

---

## 🧠 Descrição do Projeto

Este projeto tem como objetivo demonstrar uma aplicação prática de **Edge Computing** utilizando o **ESP32** para coleta de dados analógicos e integração com um **servidor MQTT**. Os dados analógicos são tirados de um potênciometro que simula um sensor de movimento que idealmente deverá ser implementado numa trave de gol de futebol.
A partir do **potenciômetro**, o ESP32 realiza a leitura do valor analógico e, com base nessa leitura, ativa **um LED e um buzzer** quando o valor ultrapassa **60%** da escala máxima.  

Os dados coletados são enviados a um **servidor MQTT**, que por sua vez transmite as informações a um **dashboard em HTML**, exibindo em tempo real o status dos sensores.

---

## ⚙️ Detalhes da Implementação

### 🧩 Hardware Utilizado
- ESP32 DevKit V1 (DOIT)
- Potenciômetro de 10kΩ
- Buzzer ativo
- LED vermelho + resistor de 220Ω
- Protoboard e jumpers

### 🔌 Ligações

| Componente | Pino ESP32 | Função |
|-------------|-------------|--------|
| Potenciômetro (saída do meio) | GPIO 34 | Entrada analógica |
| Potenciômetro (VCC) | 3.3V | Alimentação |
| Potenciômetro (GND) | GND | Terra |
| LED (via resistor) | GPIO 25 | Saída digital |
| Buzzer | GPIO 26 | Saída digital |
| GND comum | GND | Terra comum |

---

## 🧰 Funcionamento na plataforma Wokwi

A seguir estão os registros da simulação realizada na plataforma **Wokwi**, demonstrando o comportamento do circuito:

### 🖼️ 1. Montagem do circuito
![Imagem do WhatsApp de 2025-11-06 à(s) 15 06 16_ce391b17](https://github.com/user-attachments/assets/80d5bbe8-e10d-404e-8533-21170ea76195)

### 🖼️ 2. Funcionamento abaixo de 60%
![Imagem do WhatsApp de 2025-11-06 à(s) 15 04 25_77a8e400](https://github.com/user-attachments/assets/292e1f19-1c01-4930-97d1-2b5d244a1940)
### 🖼️ 3. Funcionamento acima de 60%

![Imagem do WhatsApp de 2025-11-06 à(s) 15 03 50_348549fe](https://github.com/user-attachments/assets/d6c31c2b-f4a2-46a1-b931-7015583cb4c8)

---
# 🖥️ Código fonte do ESP32 (CPP)
```cpp
#define POT_PIN 34
#define LED_PIN 25
#define BUZZER_PIN 26

void setup() {
  Serial.begin(115200);
  pinMode(LED_PIN, OUTPUT);
  pinMode(BUZZER_PIN, OUTPUT);
  pinMode(POT_PIN, INPUT);
}

void loop() {
  int valor = analogRead(POT_PIN);
  float percentual = (valor / 4095.0) * 100.0;
  
  Serial.print("Valor: ");
  Serial.print(valor);
  Serial.print(" (");
  Serial.print(percentual);
  Serial.println("%)");

  if (percentual > 60.0) {
    digitalWrite(LED_PIN, HIGH);
    digitalWrite(BUZZER_PIN, HIGH);
  } else {
    digitalWrite(LED_PIN, LOW);
    digitalWrite(BUZZER_PIN, LOW);
  }

  delay(200);
}
```
# ☁️ Servidor Flask + MQTT + Dashboard HTML

O servidor foi implementado em Python, utilizando Flask, Flask-SocketIO e Paho MQTT.
Ele recebe os dados do ESP32 via MQTT, processa as mensagens e atualiza dinamicamente um dashboard HTML com gráfico em tempo real.

```py
from flask import Flask, render_template_string
from flask_socketio import SocketIO
import paho.mqtt.client as mqtt
import json

# ==== MQTT ====
MQTT_BROKER = "54.172.140.81"        # seu broker
MQTT_PORT   = 1883
MQTT_KEEPALIVE = 60
MQTT_TOPIC  = "/TEF/device001/attrs/p"  # seu tópico

# ==== Flask / SocketIO ====
app = Flask(__name__)
socketio = SocketIO(app, async_mode="threading", cors_allowed_origins="*")

ultimo_valor = None

# ---- Callbacks MQTT (API V1) ----
def on_connect(client, userdata, flags, rc):
    print("[MQTT] Conectado. rc =", rc)
    client.subscribe(MQTT_TOPIC)
    print(f"[MQTT] Subscribed: {MQTT_TOPIC}")

def on_message(client, userdata, msg):
    global ultimo_valor
    raw = msg.payload
    valor = None

    try:
        decoded = raw.decode("utf-8", errors="replace").strip()
        try:
            obj = json.loads(decoded)
            if isinstance(obj, dict):
                for k in ("value", "l", "valor", "data", "v"):
                    if k in obj:
                        valor = obj[k]
                        break
                if valor is None:
                    valor = obj
            else:
                valor = obj
        except json.JSONDecodeError:
            try:
                valor = float(decoded)
            except ValueError:
                valor = decoded
    except Exception as e:
        print("[MQTT] Erro ao processar payload:", e)
        valor = None

    ultimo_valor = valor
    socketio.emit("novo_dado", {"valor": ultimo_valor})
    print(f"[MQTT] Mensagem em {msg.topic}: {ultimo_valor}")

# ---- Configura cliente MQTT ----
client = mqtt.Client(mqtt.CallbackAPIVersion.VERSION1)
client.on_connect = on_connect
client.on_message = on_message
client.connect(MQTT_BROKER, MQTT_PORT, MQTT_KEEPALIVE)
client.loop_start()

# ---- Página HTML com dashboard ----
@app.route("/")
def index():
    return render_template_string("""
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Potenciômetro</title>
  <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css">
  <script src="https://code.jquery.com/jquery-3.5.1.min.js"></script>
  <script src="https://cdn.socket.io/4.7.2/socket.io.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0"></script>
  <style>
    canvas { max-width: 400px; margin: auto; }
    #valor-potenciometro { font-size: 48px; font-weight: bold; text-align:center; }
    .container { max-width: 600px; }
  </style>
</head>
<body>
  <div class="container">
    <h1 class="mt-5 text-center">Potenciômetro</h1>
    <canvas id="gauge" width="400" height="400"></canvas>
    <div id="valor-potenciometro" class="mt-3">Aguardando dados...</div>
  </div>

  <script>
    const ctx = document.getElementById('gauge').getContext('2d');
    const gaugeChart = new Chart(ctx, {
      type: 'doughnut',
      data: {
        labels: ['Valor', 'Restante'],
        datasets: [{
          label: 'Valor do Potenciômetro',
          data: [0, 100],
          borderWidth: 1
        }]
      },
      options: {
        responsive: true,
        cutout: '70%',
        animation: { animateRotate: true }
      }
    });

    $(function() {
      const socket = io({ transports: ['websocket', 'polling'] });
      socket.on('novo_dado', (data) => {
        let v = data && data.valor;

        if (v && typeof v === 'object') {
          v = v.value ?? v.l ?? v.valor ?? NaN;
        }

        let num = Number(v);
        if (!Number.isFinite(num)) {
          $('#valor-potenciometro').text('Medição: ' + (data ? JSON.stringify(data.valor) : '—'));
          return;
        }

        num = Math.max(0, Math.min(100, num));

        $('#valor-potenciometro').text('Medição: ' + num);

        gaugeChart.data.datasets[0].data[0] = num;
        gaugeChart.data.datasets[0].data[1] = 100 - num;
        gaugeChart.update();
      });
    });
  </script>
</body>
</html>
    """)

if __name__ == "__main__":
    socketio.run(app, debug=True)
```
#🧑‍💻Imagens do Hands-On

![WhatsApp Image 2025-11-07 at 22 08 04 (1)](https://github.com/user-attachments/assets/510e76e9-b316-4309-afc4-b2ed8552f047)

 




