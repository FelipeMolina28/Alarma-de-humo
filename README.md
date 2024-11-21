para este proyecto para el hogar vamos a usar una PCB, una ESP32, sensores MQ9, MQ135 Y FOTOELECTRICO. con los siguientes codigos.
Anexo imagenes de los sensores:
![image](https://github.com/user-attachments/assets/43edc0ab-c488-4c8a-b347-25390cbf8c1a)

![image](https://github.com/user-attachments/assets/875f86bc-925b-4b22-adf3-0ebd1c546b80)

![image](https://github.com/user-attachments/assets/c73744f1-916d-46b1-99bb-ba09f1ee417b)


#include <WiFi.h>
#include <WiFiClient.h>
#include "secrets.h"
#include "ThingSpeak.h"
#include <Arduino.h>

char ssid[] = SECRET_SSID;   // tu SSID de red (nombre) 
char pass[] = SECRET_PASS;   // tu contraseña de red
WiFiClient client;
unsigned long myChannelNumber = SECRET_CH_ID;
const char * myWriteAPIKey = SECRET_WRITE_APIKEY;

const int pinhumo = 32; // Pin donde está conectado el sensor de humo
int valorhumo = 0;      // Declara valorhumo como variable global
const int mq9Pin = 34; // Pin analógico donde se conecta el AO del MQ-9
const int photoPin = 33; // Pin donde está conectado el sensor fotoeléctrico

void setup() {
  Serial.begin(115200);  // Inicializa el puerto serie
  while (!Serial) {
    ; // espera a que el puerto serie se conecte
  }  
  WiFi.mode(WIFI_STA);
  ThingSpeak.begin(client);  // Inicializa ThingSpeak

  pinMode(pinhumo, INPUT); // Configura el pin como entrada
  pinMode(photoPin, INPUT); // Configurar el pin como entrada
}

void loop() {
  // Conéctate o reconéctate a WiFi
  if (WiFi.status() != WL_CONNECTED) {
    Serial.print("Intentando conectarse a SSID: ");
    Serial.println(SECRET_SSID);
    WiFi.begin(ssid, pass);
    
    while (WiFi.status() != WL_CONNECTED) {
      Serial.print(".");
      delay(5000); // Espera antes de volver a intentar
    }
    Serial.println("\nConectado.");
  }

  // Lee el valor del sensor de humo
  valorhumo = analogRead(pinhumo); // Lee el valor del sensor
  Serial.print("Valor humo: ");
  Serial.println(valorhumo); // Imprime el valor en el Monitor Serial

  int mq9Value = analogRead(mq9Pin); // Leer el valor del sensor
  Serial.print("Valor MQ-9: ");
  Serial.println(mq9Value); // Imprimir el valor en el Monitor Serial
  delay(1000); // Esperar 1 segundo antes de la siguiente lectura
  
  int photoValue = digitalRead(photoPin); // Leer el estado del sensor (0 o 1)
  
  Serial.print("Estado del sensor fotoeléctrico: ");
  Serial.println(photoValue == HIGH ? "Luz detectada" : "Sin luz");

  delay(1000); // Esperar 1 segundo antes de la siguiente lectura
  // Establece los campos con los valores
  ThingSpeak.setField(1, valorhumo);
  ThingSpeak.setField(2, mq9Value);
  ThingSpeak.setField(3, photoValue);
  
  // Escribe en el canal de ThingSpeak
  int x = ThingSpeak.writeFields(myChannelNumber, myWriteAPIKey);
  if (x == 200) {
    Serial.println("Actualización del canal exitosa.");
  } else {
    Serial.println("Problema al actualizar el canal. Código de error HTTP: " + String(x));
  }
  
  // Espera antes de la siguiente iteración
  delay(5000); // Espera 5 segundos antes de la siguiente lectura y actualización
}
