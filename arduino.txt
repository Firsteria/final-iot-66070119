#include <WiFiS3.h>
#include <PubSubClient.h>
#include <Adafruit_SHT31.h> // Library สำหรับ SHT31


const char* ssid = "First"; 
const char* password = "38110455";   



const char* mqtt_server = "broker.hivemq.com"; 
const int   mqtt_port = 1883; 

// ตั้งชื่อ Topic 
const char* topic_sensor = "final-66070119/sensor"; 
// ⬆️⬆️⬆️ --------------------- ⬆️⬆️⬆️

// --- ตัวแปร Global ---
WiFiClient wifiClient;
PubSubClient mqttClient(wifiClient);
Adafruit_SHT31 sht31 = Adafruit_SHT31();

void setup() {
  Serial.begin(115200);
  while (!Serial); 
  
  // 1. เริ่มต้น SHT31
  if (!sht31.begin(0x44)) { 
    Serial.println("Couldn't find SHT31 sensor!");
    while (1) delay(1);
  }

  // 2. เชื่อมต่อ WiFi
  Serial.print("Connecting to WiFi: ");
  Serial.println(ssid);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi connected!");
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());

  // 3. ตั้งค่า MQTT Broker
  mqttClient.setServer(mqtt_server, mqtt_port);
}


void reconnect_mqtt() {
  while (!mqttClient.connected()) {
    Serial.print("Attempting MQTT connection...");
    
    
    String clientId = "Arduino-66070119-";
    clientId += String(random(0xffff), HEX); // สุ่มเลขต่อท้าย
    
    if (mqttClient.connect(clientId.c_str())) { 
    // ⬆️⬆️⬆️ --------------------- ⬆️⬆️⬆️
      
      Serial.println("connected");
    } else {
      Serial.print("failed, rc=");
      Serial.print(mqttClient.state());
      Serial.println(" try again in 5 seconds");
      delay(5000); 
    }
  }
}

void loop() {
  
  if (!mqttClient.connected()) {
    reconnect_mqtt();
  }
  mqttClient.loop(); 

  float temp = sht31.readTemperature();
  float humi = sht31.readHumidity();

  if (isnan(temp) || isnan(humi)) {
    Serial.println("Failed to read from SHT31 sensor!");
    return;
  }
  
  // แปลงค่า float เป็น string
  char tempString[8];
  char humiString[8];
  dtostrf(temp, 4, 2, tempString);
  dtostrf(humi, 4, 2, humiString);

  // สร้าง payload
  String payload = String(tempString) + "," + String(humiString);

  // ส่งข้อมูล
  Serial.print("Publishing to topic '");
  Serial.print(topic_sensor);
  Serial.print("': ");
  Serial.println(payload);
  
  mqttClient.publish(topic_sensor, payload.c_str()); 

  delay(5000); 
}
