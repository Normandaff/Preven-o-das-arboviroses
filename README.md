# Preven-o-das-arboviroses
#include <WiFi.h> // Biblioteca para conectar no Wi-Fi
#include <PubSubClient.h> // Biblioteca para comunicação via MQTT
#include <DHT.h> // Biblioteca para o sensor DHT
// Definições do sensor DHT
#define DHTPIN 4 // Pino onde o DHT está conectado (GPIO2 - D4)
#define DHTTYPE DHT11 // Tipo do sensor (DHT11 ou DHT22)
DHT dht(DHTPIN, DHTTYPE);
// Configurações da rede Wi-Fi
const char* ssid = "Seu_SSID";
const char* password = "Sua_Senha";
// Configurações do MQTT
const char* mqtt_server = "broker.mqtt-dashboard.com"; // Use o seu broker MQTT (ou Azure IoT
Hub)
const int mqtt_port = 1883; // Porta MQTT padrão
const char* mqtt_user = "mqtt_user"; // Usuário MQTT (deixe vazio se não houver)
const char* mqtt_password = "mqtt_password"; // Senha MQTT (deixe vazio se não houver)
// Cliente Wi-Fi e MQTT
WiFiClient espClient;
PubSubClient client(espClient);
void setup() {
Serial.begin(115200); // Inicializa comunicação serial
dht.begin(); // Inicializa o sensor DHTsetup_wifi(); // Conecta ao Wi-Fi
client.setServer(mqtt_server, mqtt_port); // Configura o servidor MQTT
}
void setup_wifi() {
delay(10);
Serial.println();
Serial.print("Conectando-se à rede Wi-Fi: ");
Serial.println(ssid);
WiFi.begin(ssid, password);
while (WiFi.status() != WL_CONNECTED) {
delay(1000);
Serial.print(".");
}
Serial.println("");
Serial.println("Wi-Fi conectado!");
}
void reconnect() {
// Loop até conseguir conectar ao servidor MQTT
while (!client.connected()) {
Serial.print("Tentando conectar ao MQTT...");
if (client.connect("ESP8266Client", mqtt_user, mqtt_password)) {
Serial.println("Conectado!");
} else {
Serial.print("Falha ao conectar. Estado = ");
Serial.print(client.state());
delay(5000); // Aguarda 5 segundos antes de tentar novamente}
}
}
void loop() {
if (!client.connected()) {
reconnect(); // Tenta reconectar ao servidor MQTT
}
client.loop();
// Coleta dados do sensor DHT
float humidity = dht.readHumidity();
float temperature = dht.readTemperature();
// Verifica se a leitura é válida
if (isnan(humidity) || isnan(temperature)) {
Serial.println("Falha ao ler o sensor DHT!");
return;
}
// Cria a string com os dados de temperatura e umidade
String payload = "{";
payload += "\"temperatura\":";
payload += String(temperature);
payload += ", \"umidade\":";
payload += String(humidity);
payload += "}";
// Converte a string para o formato adequado e publica via MQTT
char payload_cstr[100];
payload.toCharArray(payload_cstr, 100);Serial.print("Publicando dados: ");
Serial.println(payload_cstr);
client.publish("casa/quarto/dht", payload_cstr); // Publica os dados no tópico MQTT
}
