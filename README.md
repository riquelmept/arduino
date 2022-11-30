//Bibliotecas Necessárias para utilização do Sensor DS18B20 e LCD
#include <OneWire.h>
#include <DallasTemperature.h>
#include <LiquidCrystal.h>

const int TMP = 13; //Pino do Sensor
OneWire oneWire(TMP);
DallasTemperature sensors(&oneWire);
DeviceAddress sensor1;
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
  pinMode(Aquecedor, OUTPUT);
  pinMode(Refrig, OUTPUT);
  pinMode(LRefrig, OUTPUT);
  pinMode(LAquec, OUTPUT);
  pinMode(LNormal, OUTPUT);
  Serial.begin(9600);
  sensors.begin(); //Inicialização do sensor
  sensors.getAddress(sensor1, 0); //Endereçamento do Sensor
  delay(1000);
}

void loop()
{
  sensors.requestTemperatures(); //Requerimento de temperatura ao sensor
    if (!sensors.getAddress(sensor1,0)) { // Encontra o endereco do sensor no barramento
    Serial.println("SENSOR NAO CONECTADO"); // Sensor conectado, imprime mensagem de erro
  } else {
    Serial.print("Temperatura = "); // Imprime a temperatura no monitor serial
    Serial.println(sensors.getTempC(sensor1), 1); // Busca temperatura para dispositivo
  }
  temp = sensors.getTempCByIndex(0);
  Serial.print("Temperatura: ");
  Serial.print(temp);
  lcd.setCursor(0, 1);
  delay(250);
  
  if(temp<28.0) //Aciona o sistema de aquecimento caso a temperatura esteja menor que 26ºC
  {

    digitalWrite(Aquecedor, HIGH);
    digitalWrite(Refrig, LOW);
    digitalWrite(LAquec, HIGH);
    digitalWrite(LNormal, LOW);
    digitalWrite(LRefrig, LOW);
    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print("Aquecedor ligado");
    lcd.setCursor(0,1);
    lcd.print("Temp Agua: ");
    lcd.print(temp);
    Serial.println("\t Aquecedor ligado");

  }

  if(temp == 28.0 || temp>28.0 && temp<30 || temp == 30.0) //Estabiliza o sistema caso a temperatura esteja entre 28ºC e 30ºC
  {

    digitalWrite(Aquecedor, LOW);
    digitalWrite(Refrig, LOW);
    digitalWrite(LAquec, LOW);
    digitalWrite(LNormal, HIGH);
    digitalWrite(LRefrig, LOW);
    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print("Temp Normal");
    lcd.setCursor(0,1);
    lcd.print("Temp Agua: ");
    lcd.print(temp);
    Serial.println("\t Temperatura Estável");

  }

  if(temp>30)//Aciona o sistema de refrigeração caso a temperatura esteja maior que 30ºC
  {

    digitalWrite(Aquecedor, LOW);
    digitalWrite(Refrig, HIGH);
    digitalWrite(LAquec, LOW);
    digitalWrite(LNormal, LOW);
    digitalWrite(LRefrig, HIGH);
    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print("Refrig Ligada");
    lcd.setCursor(0,1);
    lcd.print("Temp Agua: ");
    lcd.print(temp);
    Serial.println("\t Refrig ligado");

  }
    delay(500);
  }
