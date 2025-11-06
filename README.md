# 🌐 Sprint 4 - Edge Computing  
## Projeto: Monitoramento via ESP32 com MQTT e Dashboard Web

### 👥 Integrantes do Grupo
- **Felipe de Andrade Godoi**  
- **Guilherme Augusto Ferreira**  
- **Raphael Naome Taketa**  
- **Victor Ribeiro Guimaraes Lira**

---

## 🧠 Descrição do Projeto

Este projeto tem como objetivo demonstrar uma aplicação prática de **Edge Computing** utilizando o **ESP32** para coleta de dados analógicos e integração com um **servidor MQTT**.  
A partir de um **potenciômetro**, o ESP32 realiza a leitura do valor analógico e, com base nessa leitura, ativa **um LED e um buzzer** quando o valor ultrapassa **60%** da escala máxima.  

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

