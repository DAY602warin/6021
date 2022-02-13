#include <Adafruit_Fingerprint.h>
#include <TridentTD_LineNotify.h>
#include <LiquidCrystal_I2C.h>
LiquidCrystal_I2C lcd(0x27, 16, 2);
String NamesStrings[] =  {"ผบ.ร้อย", "จนท.คลัง", "สาม", "สี่", "ห้า",
                          "หก", "เจ็ด", "แปด", "เก้า", "สิบ",
                          " ", " ", " ", " ", " ",
                          " ", " ", " ", " ", " ",
                          " ", " ", " ", " ", " ",
                          " ", " ", " ", " ", " ",
                         };
boolean staruwork[] = {0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0};
#include <Keypad_I2C.h>
#include <Keypad.h>
#include <Wire.h>
 
#define I2CADDR 0x21 // กำหนด Address ของ I2C
// วิธีการหา Address ของ I2C สามารถดูได้จากลิงค์ข้างล่าง o
// https://www.myarduino.net/article/98
 
const byte ROWS = 4; // กำหนดจำนวนของ Rows
const byte COLS = 4; // กำหนดจำนวนของ Columns
const int DoorGpioPin = 14; // D5 of Node MCU
const int len_key = 6;
char master_key[len_key] = {'6','0','2','0','1','1'};
char attempt_key[len_key];
int z=0;
int doorState=0;
#define closed 0
#define opened 1
 
// กำหนด Key ที่ใช้งาน (4x4)
char keys[ROWS][COLS] = {
{'1','2','3','A'},
{'4','5','6','B'},
{'7','8','9','C'},
{'*','0','#','D'}
};
 
// กำหนด Pin ที่ใช้งาน (4x4)
byte rowPins[ROWS] = {0, 1, 2, 3}; // เชื่อมต่อกับ Pin แถวของปุ่มกด
byte colPins[COLS] = {4, 5, 6, 7}; // เชื่อมต่อกับ Pin คอลัมน์ของปุ่มกด
 
// makeKeymap(keys) : กำหนด Keymap
// rowPins : กำหนด Pin แถวของปุ่มกด
// colPins : กำหนด Pin คอลัมน์ของปุ่มกด
// ROWS : กำหนดจำนวนของ Rows
// COLS : กำหนดจำนวนของ Columns
// I2CADDR : กำหนด Address ขอ i2C
// PCF8574 : กำหนดเบอร์ IC
Keypad_I2C keypad( makeKeymap(keys), rowPins, colPins, ROWS, COLS, I2CADDR, PCF8574 );

void Buzzer_led() {
  digitalWrite(D8, 1);
  delay(50);
  digitalWrite(D8, 0);
  delay(50);
}
#define LINE_TOKEN  "eGkj1mJ3Z8XaQzygugwcoW8xMIqSq6FSkWVwiK2PkMf"   // TOKEN
#define SSID        "Rifle1"   // ชื่อ Wifi
#define PASSWORD    "60211111"   // รหัส Wifi

#ifdef ESP32
#define mySerial Serial2   //esp32
#define LED_GREEN 2
#define LED_RED 4

#elif defined(ESP8266)
SoftwareSerial mySerial(D6, D7);  //esp8266
#define LED_GREEN D4
#define LED_RED D3
#define LED_BLACK D8
#define LED_WHITE D0

#elif defined(__AVR__)
SoftwareSerial mySerial(2, 3);   //arduino uno ,nano ,promini
#define LED_GREEN 12
#define LED_RED 11

#endif

byte delayled = 100;
Adafruit_Fingerprint finger = Adafruit_Fingerprint(&mySerial);
String inputString = "";
String number = "";
uint8_t id;
String passin = "";
String idin = "";
String nameinstring = "";
byte stepin = 0;
boolean stringin = 0;

String password = "NZNZ";  //รหัส 4 ตัวอักษร


void setup(){
  lcd.begin();
  lcd.backlight();
  Wire.begin(); // เรียกการเชื่อมต่อ Wire
  keypad.begin( makeKeymap(keys) ); // เรียกการเชื่อมต่อ
  Serial.begin(9600);
  Serial.println("\n\nAdafruit finger detect test");
  finger.begin(57600);
  delay(5);
  if (finger.verifyPassword()) {
    Serial.println("Found fingerprint sensor!");
  } else {
    Serial.println("Did not find fingerprint sensor :(");
    while (1) {
    delay(1);
    }
  }
  Serial.println(F("Reading sensor parameters"));
  finger.getParameters();
  Serial.print(F("Status: 0x")); Serial.println(finger.status_reg, HEX);
  Serial.print(F("Sys ID: 0x")); Serial.println(finger.system_id, HEX);
  Serial.print(F("Capacity: ")); Serial.println(finger.capacity);
  Serial.print(F("Security level: ")); Serial.println(finger.security_level);
  Serial.print(F("Device address: ")); Serial.println(finger.device_addr, HEX);
  Serial.print(F("Packet len: ")); Serial.println(finger.packet_len);
  Serial.print(F("Baud rate: ")); Serial.println(finger.baud_rate);

  finger.getTemplateCount();

  if (finger.templateCount == 0) {
    Serial.print("Sensor doesn't contain any fingerprint data. Please run the 'enroll' example.");
  }
  else {
    Serial.println("Waiting for valid finger...");
    Serial.print("Sensor contains "); Serial.print(finger.templateCount); Serial.println(" templates");
  }
  pinMode(DoorGpioPin, INPUT);
  pinMode(LED_BLACK, OUTPUT);
  pinMode(LED_GREEN, OUTPUT);
  pinMode(LED_RED, OUTPUT);
  pinMode(LED_WHITE, OUTPUT);
  digitalWrite(LED_RED, LOW);
  digitalWrite(LED_GREEN, LOW);
  digitalWrite(LED_BLACK, LOW);
  digitalWrite(LED_WHITE, LOW);
  
  lcd.setCursor(0, 0); // กำหนดให้ เคอร์เซอร์ อยู่ตัวอักษรตำแหน่งที่0 แถวที่ 1 เตรียมพิมพ์ข้อความ
  lcd.print("INF 602 HELLO"); //พิมพ์ข้อความ 
  lcd.setCursor(0, 1); // กำหนดให้ เคอร์เซอร์ อยู่ตัวอักษรกำแหน่งที3 แถวที่ 2 เตรียมพิมพ์ข้อความ
  lcd.print("SEARCHING FOR WL"); //พิมพ์ข้อความ 
  
  Serial.println("");
  Serial.println(LINE.getVersion());
  WiFi.begin(SSID, PASSWORD);
  Serial.printf("WiFi connecting to %s\n",  SSID);
  while (WiFi.status() != WL_CONNECTED) {
  Serial.print(".");
  delay(500);
  }
  Serial.printf("\nWiFi connected\nIP : ");
  Serial.println(WiFi.localIP());
  LINE.setToken(LINE_TOKEN);
  LINE.notify("คลังอาวุธ ร้อย.1 เชื่อมต่อ WIFI");///แก้เป็นชื่อที่ต้องการ
  Serial.println("");
  lcd.clear();
  lcd.setCursor(0, 0); // กำหนดให้ เคอร์เซอร์ อยู่ตัวอักษรตำแหน่งที่0 แถวที่ 1 เตรียมพิมพ์ข้อความ
  lcd.print("RDY PWD / FINGER"); //พิมพ์ข้อความ 
}

void loop() {
  char key = keypad.getKey();
  lcd.setCursor(z-1,1);
  lcd.print("*");
  if (key){
    Buzzer_led();
     switch(key){
      case '*':
        z=0;
        break;
      case '#':
        delay(100); // added debounce
        checkKEY();
        break;
      default:
         attempt_key[z]=key;
         z++;
      }
  }
  if(digitalRead(DoorGpioPin)==LOW && doorState==closed){ 
       LINE.notify("คลังอาวุธ ร้อย.1 เปิด แล้ว ◀1▶ เสียงสัญญาณกำลังดัง");/////แก้เป็นข้อความที่ต้องการ แจ้งเตือน เมื่อเปิดคลัง
       Serial.println("Open");
       doorState=opened;
       digitalWrite(LED_WHITE, HIGH);
    }
 if(digitalRead(DoorGpioPin)==HIGH && doorState==opened){ // Print button pressed message.
       LINE.notify("คลังอาวุธ ร้อย.1 ปิด แล้ว ▶1◀ เสียงสัญญาณกำลังดัง");///แก้เป็นข้อความที่ต้องการ แจ้งเตือน เมื่อปิดคลัง
       Serial.println("Close");
       doorState=closed;
       digitalWrite(LED_WHITE, HIGH);
  }
  if(WiFi.status() == WL_IDLE_STATUS) {
  lcd.clear();
  lcd.setCursor(0, 0); // กำหนดให้ เคอร์เซอร์ อยู่ตัวอักษรตำแหน่งที่0 แถวที่ 1 เตรียมพิมพ์ข้อความ
  lcd.print("WL_IDLE_STATUS"); //พิมพ์ข้อความ 
 }
 if(WiFi.status() == WL_CONNECT_FAILED) {
  lcd.clear();
  lcd.setCursor(0, 0); // กำหนดให้ เคอร์เซอร์ อยู่ตัวอักษรตำแหน่งที่0 แถวที่ 1 เตรียมพิมพ์ข้อความ
  lcd.print("WL_CONNECT_FAILED"); //พิมพ์ข้อความ
 }
 if(WiFi.status() == WL_CONNECTION_LOST) {
  lcd.clear();
  lcd.setCursor(0, 0); // กำหนดให้ เคอร์เซอร์ อยู่ตัวอักษรตำแหน่งที่0 แถวที่ 1 เตรียมพิมพ์ข้อความ
  lcd.print("WL_CONNECTION_LOST"); //พิมพ์ข้อความ
 } 
 if(WiFi.status() == WL_DISCONNECTED) {
  lcd.clear();
  lcd.setCursor(0, 0); // กำหนดให้ เคอร์เซอร์ อยู่ตัวอักษรตำแหน่งที่0 แถวที่ 1 เตรียมพิมพ์ข้อความ
  lcd.print("WL_DISCONNECTED"); //พิมพ์ข้อความ
 }
 if(WiFi.status() == WL_CONNECTED) { //////รอป้อน ขา  แสดงไฟ สถานะไวไฟ
  lcd.setCursor(0, 0); // กำหนดให้ เคอร์เซอร์ อยู่ตัวอักษรตำแหน่งที่0 แถวที่ 1 เตรียมพิมพ์ข้อความ
  lcd.print("RDY PWD / FINGER"); //พิมพ์ข้อความ 
 }
  
    if (stringin == 1) {
    digitalWrite(LED_GREEN, HIGH);
    digitalWrite(LED_RED, HIGH);
    digitalWrite(LED_BLACK, HIGH); 
    digitalWrite(LED_WHITE, HIGH);
    Serial.print("Enrolling ID #");
    Serial.println(id);
    while (!  getFingerprintEnroll() );
    delay(3000);
    digitalWrite(LED_GREEN, LOW);
    digitalWrite(LED_RED, LOW);
    digitalWrite(LED_BLACK, LOW);
    digitalWrite(LED_WHITE, LOW);
    inputString = "";
    stringin = 0;
 }
  getFingerprintID();
  delay(100);
}
void checkKEY()
{
   int correct=0;
   int i;
   for (i=0; i<len_key; i++) {
    if (attempt_key[i]==master_key[i]) {
      correct++;
      }
    }
   if (correct==len_key && z==len_key){
    lcd.setCursor(0,1);
    lcd.print("CORRECT KEY");
    digitalWrite(LED_WHITE, LOW);
    LINE.notify("เสียงสัญญาณปิดเเล้ว ด้วยรหัสผ่าน");
    delay(2000);
    z=0;
    lcd.clear();
   }
   else
   {
    lcd.setCursor(0,1);
    lcd.print("INCORRECT KEY");
    delay(2000);
    z=0;
    lcd.clear();
   }
   for (int zz=0; zz<len_key; zz++) {
    attempt_key[zz]=0;
   }
}

  uint8_t getFingerprintID() {
  uint8_t p = finger.getImage();
  switch (p) {
    case FINGERPRINT_OK:
      Serial.println("Image taken");
      break;
    case FINGERPRINT_NOFINGER:
      Serial.println("No finger detected");
      return p;
    case FINGERPRINT_PACKETRECIEVEERR:
      Serial.println("Communication error");
      return p;
    case FINGERPRINT_IMAGEFAIL:
      Serial.println("Imaging error");
      return p;
    default:
      Serial.println("Unknown error");
      return p;
  }

  // OK success!

  p = finger.image2Tz();
  switch (p) {
    case FINGERPRINT_OK:
      Serial.println("Image converted");
      break;
    case FINGERPRINT_IMAGEMESS:
      Serial.println("Image too messy");
      return p;
    case FINGERPRINT_PACKETRECIEVEERR:
      Serial.println("Communication error");
      return p;
    case FINGERPRINT_FEATUREFAIL:
      Serial.println("Could not find fingerprint features");
      return p;
    case FINGERPRINT_INVALIDIMAGE:
      Serial.println("Could not find fingerprint features");
      return p;
    default:
      Serial.println("Unknown error");
      return p;
  }

  // OK converted!
  p = finger.fingerSearch();
  if (p == FINGERPRINT_OK){
    Serial.println("Found a print match!");
  } else if (p == FINGERPRINT_PACKETRECIEVEERR) {
    Serial.println("Communication error");
    return p;
  } else if (p == FINGERPRINT_NOTFOUND) {
    Serial.println("Did not find a match");
    digitalWrite(LED_RED, HIGH);
    //digitalWrite(LED_BLACK, HIGH);
    delay(1000);
    digitalWrite(LED_RED, LOW);
    //digitalWrite(LED_BLACK, LOW);
    
    lcd.setCursor(7, 1); // กำหนดให้ เคอร์เซอร์ อยู่ตัวอักษรตำแหน่งที่0 แถวที่ 1 เตรียมพิมพ์ข้อความ
    lcd.print("INCORRECT"); //พิมพ์ข้อความ 9 ตัว
    //lcd.setCursor(1, 1); // กำหนดให้ เคอร์เซอร์ อยู่ตัวอักษรกำแหน่งที3 แถวที่ 2 เตรียมพิมพ์ข้อความ
    //lcd.print("XXXXXXXXXXXXXXXX"); //พิมพ์ข้อความ 
    delay(3000);
    lcd.clear();
    return p;
  } else {
    Serial.println("Unknown error");
    return p;
  }

  // found a match!
  Serial.print("Found ID #"); Serial.print(finger.fingerID);
  for (int i = 1; i < 163; i++) {
       if (i == finger.fingerID){
      digitalWrite(LED_RED, LOW);
      digitalWrite(LED_GREEN, HIGH);
      //digitalWrite(LED_BLACK, HIGH);
      if (staruwork[i - 1]==0) {
        staruwork[i - 1]=1;
        String namein = NamesStrings[i - 1];
        namein += ">กำลังปิดเสียงสัญญาณ";
        LINE.notify(namein);
        ///digitalWrite(LED_WHITE, LOW);       
        lcd.setCursor(8, 1); // กำหนดให้ เคอร์เซอร์ อยู่ตัวอักษรตำแหน่งที่0 แถวที่ 1 เตรียมพิมพ์ข้อความ
        lcd.print("CORRECT"); //พิมพ์ข้อความ 
        delay(2000);
        lcd.clear();
      }
        else{
        staruwork[i - 1]=0;
        String namein = NamesStrings[i - 1];
        namein += ">กำลังปิดเสียงสัญญาณ";
        LINE.notify(namein);
        lcd.setCursor(8, 1); // กำหนดให้ เคอร์เซอร์ อยู่ตัวอักษรตำแหน่งที่0 แถวที่ 1 เตรียมพิมพ์ข้อความ
        lcd.print("CORRECT"); //พิมพ์ข้อความ 
        delay(2000);
        lcd.clear();
     }
      //delay(2000);
      digitalWrite(LED_GREEN, LOW);
      digitalWrite(LED_WHITE, LOW);
      break;
    }
  }
  digitalWrite(LED_RED, LOW);
  Serial.print(" with confidence of "); Serial.println(finger.confidence);
  return finger.fingerID;
}

// returns -1 if failed, otherwise returns ID #
int getFingerprintIDez() {
  uint8_t p = finger.getImage();
  if (p != FINGERPRINT_OK)  return -1;

  p = finger.image2Tz();
  if (p != FINGERPRINT_OK)  return -1;

  p = finger.fingerFastSearch();
  if (p != FINGERPRINT_OK)  return -1;

  // found a match!
  Serial.print("Found ID #"); Serial.print(finger.fingerID);
  Serial.print(" with confidence of "); Serial.println(finger.confidence);
  return finger.fingerID;
}

void serialEvent() {

  while (Serial.available()) {
    char inChar = (char)Serial.read();
    if (inChar == '\n') {
      passin = "";
      idin = "";
      nameinstring = "";
      stepin = 0;
      for (int j = 0; j < inputString.length(); j++) {
        if (inputString[j] == ',') {
          stepin++;
        }
        else if (stepin == 0) {
          passin += inputString[j];
        }
        else if (stepin == 1) {
          idin += inputString[j];
        }
      }
      id = idin.toInt();
      Serial.println(passin);
      Serial.println(id);
      Serial.println(nameinstring);
      delay(5000);
      if (passin == password && id > 0 ) {
        stringin = 1;
      }
      else {
        Serial.println(">>>ERROR");
        delay(5000);
      }
    }
    else {
      inputString += inChar;
    }
  }
}

uint8_t readnumber(void) {
  uint8_t num = 0;

  while (num == 0) {
    while (! Serial.available());
    num = Serial.parseInt();
  }
  return num;
}

uint8_t getFingerprintEnroll() {

  int p = -1;
  Serial.print("Waiting for valid finger to enroll as #"); Serial.println(id);
  while (p != FINGERPRINT_OK) {
    p = finger.getImage();
    switch (p) {
      case FINGERPRINT_OK:
        Serial.println("Image taken");
        break;
      case FINGERPRINT_NOFINGER:
        Serial.println(".");
        break;
      case FINGERPRINT_PACKETRECIEVEERR:
        Serial.println("Communication error");
        break;
      case FINGERPRINT_IMAGEFAIL:
        Serial.println("Imaging error");
        break;
      default:
        Serial.println("Unknown error");
        break;
    }
  }

  // OK success!

  p = finger.image2Tz(1);
  switch (p) {
    case FINGERPRINT_OK:
      Serial.println("Image converted");
      break;
    case FINGERPRINT_IMAGEMESS:
      Serial.println("Image too messy");
      return p;
    case FINGERPRINT_PACKETRECIEVEERR:
      Serial.println("Communication error");
      return p;
    case FINGERPRINT_FEATUREFAIL:
      Serial.println("Could not find fingerprint features");
      return p;
    case FINGERPRINT_INVALIDIMAGE:
      Serial.println("Could not find fingerprint features");
      return p;
    default:
      Serial.println("Unknown error");
      return p;
  }

  Serial.println("Remove finger");
  delay(1000);
  p = 0;
  while (p != FINGERPRINT_NOFINGER) {
    p = finger.getImage();
    delay(300);
  }
  Serial.print("ID "); Serial.println(id);
  p = -1;
  Serial.println("Place same finger again");

  while (p != FINGERPRINT_OK) {
    p = finger.getImage();
    switch (p) {
      case FINGERPRINT_OK:
        Serial.println("Image taken");
        break;
      case FINGERPRINT_NOFINGER:
        Serial.print(".");
        break;
      case FINGERPRINT_PACKETRECIEVEERR:
        Serial.println("Communication error");
        break;
      case FINGERPRINT_IMAGEFAIL:
        Serial.println("Imaging error");
        break;
      default:
        Serial.println("Unknown error");
        break;
    }
  }

  // OK success!

  p = finger.image2Tz(2);
  switch (p) {
    case FINGERPRINT_OK:
      Serial.println("Image converted");
      break;
    case FINGERPRINT_IMAGEMESS:
      Serial.println("Image too messy");
      return p;
    case FINGERPRINT_PACKETRECIEVEERR:
      Serial.println("Communication error");
      return p;
    case FINGERPRINT_FEATUREFAIL:
      Serial.println("Could not find fingerprint features");
      return p;
    case FINGERPRINT_INVALIDIMAGE:
      Serial.println("Could not find fingerprint features");
      return p;
    default:
      Serial.println("Unknown error");
      return p;
  }

  // OK converted!
  Serial.print("Creating model for #");  Serial.println(id);

  p = finger.createModel();
  if (p == FINGERPRINT_OK) {
    Serial.println("Prints matched!");
  } else if (p == FINGERPRINT_PACKETRECIEVEERR) {
    Serial.println("Communication error");
    return p;
  } else if (p == FINGERPRINT_ENROLLMISMATCH) {
    Serial.println("Fingerprints did not match");
    return p;
  } else {
    Serial.println("Unknown error");
    return p;
  }

  Serial.print("ID "); Serial.println(id);
  p = finger.storeModel(id);
  if (p == FINGERPRINT_OK) {
    Serial.println("Stored!");

  } else if (p == FINGERPRINT_PACKETRECIEVEERR) {
    Serial.println("Communication error");
    return p;
  } else if (p == FINGERPRINT_BADLOCATION) {
    Serial.println("Could not store in that location");
    return p;
  } else if (p == FINGERPRINT_FLASHERR) {
    Serial.println("Error writing to flash");
    return p;
  } else {
    Serial.println("Unknown error");
    return p;
  }

  return true;
  
}

