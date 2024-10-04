Integrantes do Grupo:
Gustavo Alves - rm557876
Gabriel Galerani - rm557421
Gabriel Dias - rm556830
Pedro Paulo - rm554880
Pedro Pawlik - rm556968

Documentação do Projeto FIWARE com Postman e Wokwi         

Este README fornece um guia detalhado para a configuração e operação de um dispositivo IoT utilizando a plataforma FIWARE, o Postman e o ambiente Wokwi. O objetivo é assegurar que o usuário consiga configurar corretamente seu dispositivo e interagir com o broker MQTT.

Índice

Pré-requisitos

Passo a Passo:

  Passo 1: Download do Postman

  Passo 2: Provisionamento do Dispositivo

  Passo 3: Registro do Dispositivo

  Passo 4: Verificação e Deleção

Configuração do Código no Wokwi

Exibição dos Dados no MyMQTT

Considerações Finais

Pré-requisitos

Conta no GitHub.

Postman instalado.

Acesso à internet.

Ambiente de desenvolvimento Wokwi configurado.

Aplicativo MyMQTT instalado em um dispositivo móvel.

Passo a Passo

Passo 1: Download do Postman
Acesse o repositório do GitHub: FIWARE no GitHub.
Baixe a pasta do Postman.
Abra o Postman e verifique os "health checks" para garantir que o sistema está respondendo ao servidor. Não esqueça de verificar o IP correto do servidor.

Passo 2: Provisionamento do Dispositivo
No Postman, vá para a opção de POST e envie as informações para o servidor nas portas 40 e 41.
Configure o nome do dispositivo e o respectivo ID. Lembre-se de trocar a entidade com o nome que foi definido e o tipo.
Verifique se o protocolo e o método de transporte estão corretos.
A lista de comandos não precisa ser alterada, e a lista de atributos pode permanecer a mesma.
Clique em "Send" para enviar o comando. Se a resposta for {}, significa que o provisionamento está correto.

Passo 3: Registro do Dispositivo
Vá para a opção 4 chamada "registrar".
Insira o ID que você configurou anteriormente.
Como os demais parâmetros estão corretos, clique em "Send" novamente. Se tudo estiver certo, a resposta será 1.

Passo 4: Verificação e Deleção
Para verificar se o dispositivo foi configurado corretamente, vá para a opção 5 e clique em "Send" novamente.
Você verá os 4 dispositivos configurados anteriormente. Confira os atributos e outros parâmetros para garantir que tudo está correto.
Se desejar deletar tudo, vá para a opção 9 e altere a aba de "HTTP" para hosp200 e envie o comando.

Configuração do Código no Wokwi
Aqui está o código padrão para a configuração do dispositivo no Wokwi:

#include <WiFi.h>
#include <PubSubClient.h>
#include "DHT.h"

// Defines para MQTT e DHT
#define TOPICO_SUBSCRIBE    "/TEF/hosp200/cmd"
#define TOPICO_PUBLISH      "/TEF/hosp200/attrs"
#define TOPICO_PUBLISH_2    "/TEF/hosp200/attrs/l"
#define TOPICO_PUBLISH_3    "/TEF/hosp200/attrs/h"
#define TOPICO_PUBLISH_4    "/TEF/hosp200/attrs/t"
#define ID_MQTT  "fiware_200"
#define DHTTYPE DHT22
#define DHTPIN 4

DHT dht(DHTPIN, DHTTYPE);

// WiFi e MQTT
const char* SSID = "Wokwi-GUEST";
const char* PASSWORD = "";
const char* BROKER_MQTT = "46.17.108.113";
int BROKER_PORT = 1883;

// Saídas
int D4 = 2;
WiFiClient espClient;
PubSubClient MQTT(espClient);
char EstadoSaida = '0';

// Prototypes
void initSerial();
void initWiFi();
void initMQTT();
void reconnectWiFi();
void mqtt_callback(char* topic, byte* payload, unsigned int length);
void VerificaConexoesWiFIEMQTT();
void InitOutput();

void setup() {
    InitOutput();
    initSerial();
    initWiFi();
    initMQTT();
    dht.begin();
    MQTT.publish(TOPICO_PUBLISH, "s|off");
}

// (Demais funções omitidas para brevidade)

// Loop principal
void loop() {
    const int potPin = 34;
    char msgBuffer[4];
    VerificaConexoesWiFIEMQTT();

    int sensorValue = analogRead(potPin);
    float voltage = sensorValue * (3.3 / 4096.0);
    float luminosity = map(sensorValue, 0, 4095, 0, 100);
    
    Serial.print("Voltage: ");
    Serial.print(voltage);
    Serial.print("V - ");
    Serial.print("Luminosidade: ");
    Serial.print(luminosity);
    Serial.println("%");

    dtostrf(luminosity, 4, 1, msgBuffer);
    MQTT.publish(TOPICO_PUBLISH_2, msgBuffer);

    float humidity = dht.readHumidity();
    float temperature = dht.readTemperature();
    if (isnan(humidity) || isnan(temperature)) {
        Serial.println(F("Falha leitura do sensor DHT-22!"));
    }

    Serial.print(F("Umidade: "));
    Serial.print(humidity);
    Serial.print(F("% Temperatura: "));
    Serial.print(temperature);
    Serial.println(F("°C "));

    dtostrf(humidity, 4, 1, msgBuffer);
    MQTT.publish(TOPICO_PUBLISH_3, msgBuffer);
    
    dtostrf(temperature, 4, 1, msgBuffer);
    MQTT.publish(TOPICO_PUBLISH_4, msgBuffer);
    
    MQTT.loop();
}

Observações
Verifique os tópicos que estão relacionados ao objeto criado no IoT do Postman.

Assegure-se de que o ID_MQTT esteja configurado corretamente.

Conecte-se ao IP 46.17.108.113.

Dependendo da mensagem recebida nos tópicos, ações correspondentes serão acionadas.

Faça leituras dos parâmetros do DHT22 e publique nos tópicos correspondentes.

Exibição dos Dados no MyMQTT
Para visualizar os dados coletados pelo seu dispositivo no aplicativo MyMQTT, siga estas etapas:

Abra o MyMQTT em seu dispositivo móvel.
Adicione um novo broker com as seguintes configurações:
Endereço do Broker: 46.17.108.113
Porta: 1883
Conecte-se ao broker.
Navegue até a seção de Subscriptions (Inscrições).
Inscreva-se nos tópicos relevantes para receber atualizações:
/TEF/hosp200/cmd (para comandos)
/TEF/hosp200/attrs (para atributos)
/TEF/hosp200/attrs/l (luminosidade)
/TEF/hosp200/attrs/h (umidade)
/TEF/hosp200/attrs/t (temperatura)
Agora, você poderá ver os dados de luminosidade, umidade e temperatura em tempo real no MyMQTT, conforme seu dispositivo publica as informações.

Considerações Finais
Após realizar todas as configurações e verificar que tudo está funcionando corretamente, inicie a simulação. O LED deve acender continuamente, indicando que a conexão com a rede Wokwi foi estabelecida com sucesso e que o dispositivo está pronto para operar.

Se tiver dúvidas ou enfrentar problemas, consulte a documentação oficial do FIWARE ou do Postman.


![image](https://github.com/user-attachments/assets/ea1a8591-a8dd-4dbf-a49f-6e4bf011a795)
