# Proyectos-ESP8266
proyectos creados con la IDE de arduino con un modulo wifi ESP 8266 

# HOLA, las librerias utilizadas se ppueden descargar directamente desde arduino.


Codigo : 


#ifdef ESP32
#include <WiFi.h>
#else
#include <ESP8266WiFi.h>
#endif
#include <WiFiClientSecure.h>
#include <UniversalTelegramBot.h>
#include <ArduinoJson.h>

// Coloca Aqui los credenciales del Wifi
const char* ssid = "MICHELLIN_";
const char* password = "alryht1986";

// inicializamos el Bot de Telegram
#define BOTtoken "5976248010:AAHL7gDiC-ePf-vDP5-LE0BA_19Wuq9afOU"  // Coloca el Token de Telegram

// Coloc aqui la ID del Chat Telegram
#define CHAT_ID "5505114819"

#ifdef ESP8266
X509List cert(TELEGRAM_CERTIFICATE_ROOT);
#endif

WiFiClientSecure client;
UniversalTelegramBot bot(BOTtoken, client);

// Verifica si hay un nuevo mensaje cada segundo.
int botRequestDelay = 1000;
unsigned long lastTimeBotRan;

const int PinRele = D6; //Pin conectado al IN del Rele
bool ledState = LOW;

// Maneja lo que Sucede Cuando se recibe un nuevo mensaje
void handleNewMessages(int numNewMessages) {
  Serial.println("New Messages");
  Serial.println(String(numNewMessages));

  for (int i = 0; i < numNewMessages; i++) {
    // Chat id of the requester
    String chat_id = String(bot.messages[i].chat_id);
    if (chat_id != CHAT_ID) {
      bot.sendMessage(chat_id, "usuario no autorizado", "");
      continue;
    }

    // Imprime el mensaje recibido
    String text = bot.messages[i].text;
    Serial.println(text);

    String from_name = bot.messages[i].from_name;

    if (text == "/start") {
      String welcome = "Welcome, " + from_name + ".\n";
      welcome += "Use the following commands to control your outputs.\n\n";
      welcome += "/led_1 to turn GPIO ON \n";
      welcome += "/led_0 to turn GPIO OFF \n";
      welcome += "/state to request current GPIO state \n";
      bot.sendMessage(chat_id, welcome, "");
    }

    if (text == "/led_1") {
      bot.sendMessage(chat_id, "El Bombillo esta Encendido", "");
      ledState = HIGH;
      digitalWrite(PinRele, ledState);
    }

    if (text == "/led_0") {
      bot.sendMessage(chat_id, "El Bombillo esta Apagado", "");
      ledState = LOW;
      digitalWrite(PinRele, ledState);
    }

    if (text == "/state") {
      if (digitalRead(PinRele)) {
        bot.sendMessage(chat_id, "Luz Encendidas", "");
      }
      else {
        bot.sendMessage(chat_id, "Luz Apagadas", "");
      }
    }
  }
}

void setup() {
  Serial.begin(115200);

#ifdef ESP8266
  configTime(0, 0, "pool.ntp.org");      // obtenemos el tiempo a traves del protocolo NTP
  client.setTrustAnchors(&cert); // Agregamos el certificado Raiz api.telegram.org
#endif

  pinMode(PinRele, OUTPUT);
  digitalWrite(PinRele, ledState);

  // Connect to Wi-Fi
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);
#ifdef ESP32
  client.setCACert(TELEGRAM_CERTIFICATE_ROOT); // Agregamos el certificado raiz para api.telegram.org
#endif
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi..");
  }
  // Imprimimos la direccion IP local
  Serial.println("wifi conectec.");
}

void loop() {
  if (millis() > lastTimeBotRan + botRequestDelay)  {
    int numNewMessages = bot.getUpdates(bot.last_message_received + 1);

    while (numNewMessages) {
      Serial.println("procesando respuesta");
      handleNewMessages(numNewMessages);
      numNewMessages = bot.getUpdates(bot.last_message_received + 1);
    }
    lastTimeBotRan = millis();
  }
}
