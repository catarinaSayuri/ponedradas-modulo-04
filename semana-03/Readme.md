## 🚦 Projeto Semáforo de Pedestres Offline

Este repositório contém os arquivos de mídia, e o código-fonte do projeto de Semáforo Offline. Este projeto simula um semáforo de pedestres, controlado por um microcontrolador (ESP32), utilizando LEDs para os semáforos de carros e pedestres, um botão de acionamento e um **Buzzer** para acessibilidade, que toca quando a travessia é permitida.

---

## ⭐️ Avaliação dos Pares

O projeto foi avaliado por Nicole e João, que analisaram a funcionalidade, a implementação e a documentação.

| Métrica | Peso (%) | Nota de Nicole | Nota de João |
| :--- | :---: | :---: | :---: |
| **Funcionalidade e Lógica** (Transições e Tempos) | 40% | 9,5 | 10,0 |
| **Implementação e Componentes** (Uso do Buzzer) | 30% | 8,5 | 9,0 |
| **Organização do Código** (Clareza e Comentários) | 20% | 9,0 | 9,5 |
| **Documentação (README)** (Clareza e Detalhe) | 10% | 8,0 | 8,0 |
| **Média Final** | **100%** | **9,0** | **9,5** |

---

## 📹 Demonstração (vdopond1.mp4)

O vídeo demonstra o ciclo de funcionamento completo do semáforo.

**Lógica Principal:**

1.  **Estado Padrão (Carro Verde / Pedestre Vermelho):** O sinal está aberto para os carros (LED Verde - Pino 12 aceso). O sistema aguarda o acionamento do botão.
2.  **Acionamento do Botão:** Ao ser acionado (Pino 17), o sistema inicia o ciclo de transição após um tempo de segurança (30 segundos).
3.  **Transição 1 (Pisca Pedestre Vermelho):** O LED Vermelho do Pedestre (Pino 21) pisca antes da mudança, alertando o pedestre.
4.  **Transição 2 (Carro Amarelo):** O LED Verde do carro é desligado e o LED Amarelo (Pino 22) é acionado por 7 segundos.
5.  **Transição 3 (Pisca Carro Amarelo/Verde):** O LED Amarelo e o LED Verde do carro piscam alternadamente.
6.  **Travessia Permitida (Pedestre Verde / Carro Vermelho):** O LED Vermelho do Carro (Pino 14) e o LED Verde do Pedestre (Pino 23) são ligados por 7 segundos.
7.  **Alerta de Fechamento (Pisca Pedestre Verde e BUZZER):** O LED Verde do Pedestre (Pino 23) pisca e o **Buzzer (Pino 27) é acionado de forma intermitente** para alertar o pedestre de que o tempo de travessia está finalizando.
8.  **Retorno ao Padrão:** O sistema volta ao estado inicial, liberando o tráfego de carros.

---

## 🖼️ Arquivos de Média

Abaixo estão os arquivos de imagem e vídeo relacionados ao projeto:

**Imagem 1: Vista Geral do Circuito**
![Imagem do projeto](assets/fotopond2.jpg)

**Imagem 2: Detalhe da Montagem**
![Imagem do projeto](assets/fotopond1.jpg)

**Vídeo da Simulação:**
[Link para o vídeo da simulação](assets/vdopond1.mp4)

---

## 💻 Código-Fonte (Sketch C/Arduino)

O código a seguir implementa a lógica de transição do semáforo, a contagem de tempo para os carros e o acionamento para a travessia de pedestres, incluindo o **Buzzer** (Pino 27).

```c
#define but 17
unsigned long changetime;
int cont;

void setup() {
  Serial.begin(115200);
  pinMode(12, OUTPUT); // LED Verde Carro
  pinMode(14, OUTPUT); // LED Vermelho Carro
  pinMode(but, INPUT_PULLUP); // Botão do Pedestre
  pinMode(21, OUTPUT); // LED Vermelho Pedestre
  pinMode(22, OUTPUT); // LED Amarelo Carro
  pinMode(23, OUTPUT); // LED Verde Pedestre
  pinMode(27, OUTPUT); // Buzzer
}

void loop() {
  // Configuração Padrão: Carros Liberados / Pedestres Fechados
  digitalWrite(14, 0); // Vermelho Carro OFF
  digitalWrite(23, 0); // Verde Pedestre OFF
  digitalWrite(21, 1); // Vermelho Pedestre ON
  digitalWrite(12, 1); // Verde Carro ON
  Serial.println("Nao atravesse!!");
  delay(5000); // 5s iniciais de espera

  // Contagem regressiva (simulada via Serial)
  cont = 31;
  for(int j = 0; j < 30; j++){
    cont--;
    Serial.println(cont);
    delay(800);
  }
  delay(500);
  Serial.println("Aperte o botão para atravessar");

  // Espera o acionamento do botão do pedestre
  while(true){
    int leitura = digitalRead(but);
    // Verifica se o botão foi pressionado (LOW) e se o tempo de espera (30s) passou
    if (leitura == LOW && (millis() - changetime) > 30000){
      time(); // Inicia o ciclo de travessia
    }
  }
  delay(2000);
}

void time() {
  // 1. Transição: Pisca o LED Verde do Carro
  for(int i = 0; i < 6; i++){
    digitalWrite(21, 0); // Pisca Vermelho Pedestre (o Vermelho Pedestre pisca antes do Amarelo Carro)
    delay(500);
    digitalWrite(21, 1);
    delay(500);
  }
  digitalWrite(21, 0); // Vermelho Pedestre OFF

  // 2. Carro Amarelo
  digitalWrite(22, 1); // Amarelo Carro ON
  delay(7000); // 7s de Amarelo

  // 3. Transição: Pisca o LED Amarelo do Carro
  for(int i = 0; i < 6; i++){
    digitalWrite(22, 0); // Amarelo Carro OFF/ON
    digitalWrite(12, 0); // Verde Carro OFF/ON
    delay(500);
    digitalWrite(22, 1);
    digitalWrite(12, 1);
    delay(500);
  }
  digitalWrite(22, 0); // Amarelo Carro OFF
  digitalWrite(12, 0); // Verde Carro OFF

  // 4. Travessia Permitida: Pedestre Verde / Carro Vermelho
  digitalWrite(23, 1); // Verde Pedestre ON
  digitalWrite(14, 1); // Vermelho Carro ON
  delay(7000); // 7s de Travessia
  // (Neste ponto, o buzzer deveria ser ligado se fosse uma lógica mais detalhada)

  // 5. Alerta de Fechamento: Pisca o LED Verde do Pedestre E O BUZZER
  for(int i = 0; i < 6; i++){
    digitalWrite(23, 0); // Verde Pedestre OFF
    digitalWrite(14, 0); // Vermelho Carro OFF
    digitalWrite(27, 1); // Buzzer ON (Alerta de fechamento)
    delay(500);
    digitalWrite(23, 1); // Verde Pedestre ON
    digitalWrite(14, 1); // Vermelho Carro ON
    digitalWrite(27, 0); // Buzzer OFF
    delay(500);
  }
  
  loop(); // Retorna ao estado inicial (Carro Verde)
}