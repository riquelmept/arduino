//Bibliotecas Necessárias para utilização do Sensor DS18B20 e LCD
#include <OneWire.h>
#include <DallasTemperature.h>
#include <LiquidCrystal.h>

float sensorValue;
float  outputValue;
int telas = 0;
int buttonState = 0;
const int TMP = 13; //Pino do Sensor
OneWire oneWire(TMP);
DallasTemperature sensors(&oneWire);
DeviceAddress sensor1;
float const TSin = A0;
int const Botao = 1;
int const Refrig = 6; // Relé Sistema de Refrigeração
int const Aquecedor = 7; //Relé Sistema de Aquecimento
int const LAquec = 8; //Led indicador Aquecimento
int const LNormal = 9; //Led indicador Temperatura Estável
int const LRefrig= 10; //Led indicador Refrigeração
float temp; //Variável para receber temperatura lida

LiquidCrystal lcd(12, 11, 5, 4, 3, 2); //Pinos LCD

void setup()
{
//Declaração de variáveis de saída e entrada
  lcd.begin(16, 2); //Indicando número de linhas e colunas do LCD
  pinMode(TSin, INPUT);
  pinMode(Aquecedor, OUTPUT);
  pinMode(Refrig, OUTPUT);
  pinMode(LRefrig, OUTPUT);
  pinMode(LAquec, OUTPUT);
  pinMode(LNormal, OUTPUT);
  pinMode(Botao, INPUT_PULLUP);
  Serial.begin(9600);
  sensors.begin(); //Inicialização do sensor
  sensors.getAddress(sensor1, 0); //Endereçamento do Sensor
  delay(1000);
}

void loop(){
  if(telas == 0) {  
  sensorValue = analogRead(TSin);
  outputValue = map(sensorValue, 0.0, 1023.0, 0.0, 52.0);
  Serial.println(outputValue);
  lcd.setCursor(0, 0);
  lcd.print("Temp desejada");
  lcd.setCursor(0, 1);
  lcd.print(outputValue);
  delay(300);
}
  buttonState = digitalRead(Botao);
  if(buttonState == LOW){
    telas = 1;
  } 
  
  if(telas == 1){
    ControleTemperatura();
  }
  delay(500);
}

void ControleTemperatura()
{
    sensors.requestTemperatures(); //Requerimento de temperatura ao sensor
    if (!sensors.getAddress(sensor1,0)) { // Encontra o endereco do sensor no barramento
    Serial.println("SENSOR NAO CONECTADO"); // Sensor conectado, imprime mensagem de erro
  } else {
    Serial.print("Temperatura = "); // Imprime a temperatura no monitor serial
    Serial.println(sensors.getTempC(sensor1), 1); // Busca temperatura para dispositivo
  
  temp = sensors.getTempCByIndex(0);
  Serial.print("Temperatura: ");
  Serial.print(temp);
  lcd.setCursor(0, 1);

  int tempinf = outputValue - 2.0;
  int tempsup = outputValue + 2.0;
  if(temp<tempinf) //Aciona o sistema de aquecimento caso a temperatura esteja menor que 26ºC
  {

    digitalWrite(Aquecedor, HIGH);
    digitalWrite(Refrig, LOW);
    digitalWrite(LAquec, HIGH);
    digitalWrite(LNormal, LOW);
    digitalWrite(LRefrig, LOW);
    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print("TL:");
    lcd.setCursor(4,0);
    lcd.print(temp);
    lcd.setCursor(10,0);
    lcd.print("AQUEC");
    lcd.setCursor(0,1);
    lcd.print("TS:");
    lcd.setCursor(4,1);
    lcd.print(outputValue);
    lcd.setCursor(11,1);
    lcd.print("ON");
    Serial.println("\t Aquecedor ligado");
    delay(100);
  }

  if(temp == outputValue || tempinf<temp && temp<tempsup) //Estabiliza o sistema caso a temperatura esteja dentro do range
  {

    digitalWrite(Aquecedor, LOW);
    digitalWrite(Refrig, LOW);
    digitalWrite(LAquec, LOW);
    digitalWrite(LNormal, HIGH);
    digitalWrite(LRefrig, LOW);
    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print("TL:");
    lcd.setCursor(4,0);
    lcd.print(temp);
    lcd.setCursor(10,0);
    lcd.print("TEMP");
    lcd.setCursor(0,1);
    lcd.print("TS:");
    lcd.setCursor(4,1);
    lcd.print(outputValue);
    lcd.setCursor(11,1);
    lcd.print("OK!");
    Serial.println("\t Temperatura Estável");
    delay(100);

  }

  if(temp > tempsup)//Aciona o sistema de refrigeração caso a temperatura esteja maior que 30ºC
  {

    digitalWrite(Aquecedor, LOW);
    digitalWrite(Refrig, HIGH);
    digitalWrite(LAquec, LOW);
    digitalWrite(LNormal, LOW);
    digitalWrite(LRefrig, HIGH);
    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print("TL:");
    lcd.setCursor(4,0);
    lcd.print(temp);
    lcd.setCursor(10,0);
    lcd.print("REFRIG");
    lcd.setCursor(0,1);
    lcd.print("TS:");
    lcd.setCursor(4,1);
    lcd.print(outputValue);
    lcd.setCursor(11,1);
    lcd.print("ON");
    Serial.println("\t Refrigeração ligada");
    delay(100);

  }
    delay(100);
  }
}
