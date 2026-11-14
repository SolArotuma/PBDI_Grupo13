# Código SENSOR CUENTA GOTAS: Sensor Encoder Infrarrojo FC-03:

const int pinDO = 2;
const int pinAO = A0;

volatile unsigned long interrupciones = 0;
volatile unsigned long ultimoTiempoISR = 0;
unsigned long conteoAnterior = 0;
unsigned long tiempoPrevio = 0;

void contar() {
  unsigned long ahora = micros();
  // Filtro anti-rebote: ignora pulsos separados por menos de 1000 us (1 ms)
  if (ahora - ultimoTiempoISR > 1000) {
    interrupciones++;
    ultimoTiempoISR = ahora;
  }
}

void setup() {
  Serial.begin(115200); // Cambia el Monitor Serie a 115200 baudios
  pinMode(pinDO, INPUT);
  attachInterrupt(digitalPinToInterrupt(pinDO), contar, RISING);
}

void loop() {
  unsigned long actual = micros();

  // Muestreo cada 5000 us (5 ms = 200 muestras/segundo)
  if (actual - tiempoPrevio >= 5000) {
    tiempoPrevio = actual;

    // Lectura segura de la variable de interrupción
    noInterrupts();
    unsigned long conteoActual = interrupciones;
    interrupts();

    int nivelADC = analogRead(pinAO);
    bool huboCambio = (conteoActual != conteoAnterior);
    conteoAnterior = conteoActual;

    Serial.print("Int:");
    Serial.print(conteoActual);
    Serial.print(" | ADC:");
    Serial.print(nivelADC);
    Serial.print(" | Cambio:");
    Serial.println(huboCambio ? "SI" : "NO");
  }
}

# Código HC-SR04 +OLED:

#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define ANCHO 128
#define ALTO 64
#define DIRECCION_OLED 0x3C      // si no funciona, prueba 0x3D

Adafruit_SSD1306 oled(ANCHO, ALTO, &Wire, -1);

const int Trigger = 2;   // Pin para el Trigger del sensor
const int Echo = 3;      // Pin para el Echo del sensor

long caudalTeo = 0;      
long caudalReal = 0;     

void setup() {
  Serial.begin(9600);
  pinMode(Trigger, OUTPUT);
  pinMode(Echo, INPUT);
  digitalWrite(Trigger, LOW);

  if (!oled.begin(SSD1306_SWITCHCAPVCC, DIRECCION_OLED)) {
    Serial.println("No se encontro la OLED");
    while (true);
  }
  oled.setTextColor(SSD1306_WHITE);
  oled.setTextSize(1);
  oled.clearDisplay();
  oled.display();
}

void loop() {
  long t;   // tiempo que demora en llegar el eco
  long d;   // distancia en centímetros

  digitalWrite(Trigger, LOW);
  delayMicroseconds(2);
  digitalWrite(Trigger, HIGH);
  delayMicroseconds(10);
  digitalWrite(Trigger, LOW);

  t = pulseIn(Echo, HIGH, 20000);   
  d = t / 59;

  caudalTeo = 14;          // se pondrá el valor del caudal después de los cálculos
  caudalReal = d / 4;      // se pondrá el valor del caudal real

  oled.clearDisplay();

  oled.setCursor(0, 8);
  oled.print("Distancia: ");
  oled.print(d);
  oled.print(" cm");

  oled.setCursor(0, 28);
  oled.print("Caudal Teo: ");
  oled.print(caudalTeo);
  oled.print(" g/s");

  oled.setCursor(0, 48);
  oled.print("Caudal Real: ");
  oled.print(caudalReal);
  oled.print(" g/s");

  oled.display();

  delay(200);
}

