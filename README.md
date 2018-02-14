
#include <SoftwareSerial.h>
#include "Adafruit_FONA.h"

#define FONA_RX 7
#define FONA_TX 8
#define FONA_RST 9

char replybuffer[100];

SoftwareSerial fonaSS = SoftwareSerial(FONA_TX, FONA_RX);
Adafruit_FONA fona = Adafruit_FONA(FONA_RST);

uint8_t readline(char *buff, uint8_t maxbuff, uint16_t timeout = 0);


#include <Adafruit_GPS.h>
#include <SoftwareSerial.h>

double lng[6];
double lat[6];
int count=6;
double X;
double Y;
int xnum=0;
int ynum=0;
double m;
double Xtemp;
double Ytemp;
int buttonState = 0;

bool repeatg=true;
 
SoftwareSerial mySerial(16, 10);
Adafruit_GPS GPS(&mySerial);
boolean usingInterrupt = false;
void useInterrupt(boolean);
 
void setup() {
 
  Serial.begin(115200);
  lng[0]=-88.14140793270522;
  lat[0]=41.87074587535906;
  lng[1]=-88.1411955332527;
  lat[1]=41.86966556595091;
  lng[2]=-88.1389411660823;
  lat[2]=41.86989395831537;
  lng[3]=-88.1394993533701;
  lat[3]=41.87128823261915;
  lng[4]=-88.14085297220645;
  lat[4]=41.87155553543375;
  lng[5]=-88.14140793270522;
  lat[5]=41.87074587535906;
 GPS.begin(9600);
 GPS.sendCommand(PMTK_SET_NMEA_OUTPUT_RMCGGA);
 GPS.sendCommand(PMTK_SET_NMEA_UPDATE_1HZ);
 useInterrupt(true);

 pinMode(13, OUTPUT);
 pinMode(12, INPUT);
// pinMode(4, INPUT);
 delay(1000);
   fonaSS.begin(4800); // if you're using software serial

fona.begin(fonaSS);

//digitalWrite(13,HIGH);



fona.deleteSMS(1);
if(fona.sendSMS("16305448893", "Connected")){
delay(2000);
Serial.println("good");
}


}
 
SIGNAL(TIMER0_COMPA_vect) {
  char c = GPS.read();
}
void useInterrupt(boolean v) {
  if (v) {
    OCR0A = 0xAF;
    TIMSK0 |= _BV(OCIE0A);
    usingInterrupt = true;
  } else {
    TIMSK0 &= ~_BV(OCIE0A);
    usingInterrupt = false;
  }
}
 
void loop() {

  uint16_t smslen;
  if(fona.readSMS(1, replybuffer, 100 , &smslen)){
    Serial.println(replybuffer);
    if(replybuffer[0]=='1'){
      if(replybuffer[1] == '2'){
        if(replybuffer[2] == '3'){
          if(replybuffer[3] == '4'){
            if(replybuffer[4] == '5'){
              repeatg=true;
            } 
          }
        }
      }
    }
    fona.deleteSMS(1);
  }


//  

//flushSerial();

  
  if (! usingInterrupt) {
    char c = GPS.read();
  }
 
  if (GPS.newNMEAreceived()) {
    if (!GPS.parse(GPS.lastNMEA()))
      return;
  }
  if (GPS.fix) {
    X=GPS.longitudeDegrees;
    Y=GPS.latitudeDegrees;
    Serial.print("lat:");
    Serial.print(GPS.latitudeDegrees,7);
    Serial.print("    lon:");
    Serial.println(GPS.longitudeDegrees,7);
    for(int i=0;i!=(count-1);i++){
      m=(lat[i+1]-lat[i])/(lng[i+1]-lng[i]);
 
      if((lat[i]<Y && lat[i+1]>Y) || (lat[i]>Y && lat[i+1]<Y)){
        Xtemp=((Y-lat[i])/m)+lng[i];
        if(Xtemp<=X){xnum++;}
      }
      if((lng[i]<X && lng[i+1]>X) || (lng[i]>X && lng[i+1]<X)){
        Ytemp=m*(X-lng[i])+lat[i];
        if(Ytemp<=Y){ynum++;}
      }
    }
 
    if(xnum%2!=0 && ynum%2!=0){
      
      while(repeatg){
        String sing = "https://maps.google.com/maps?q=";
        sing = sing + GPS.latitudeDegrees + "," + GPS.longitudeDegrees;
        char charBuff[100];
        sing.toCharArray(charBuff, 100);
        repeatg=fona.sendSMS("16305448893", charBuff);
        Serial.println("IN");
        repeatg=!repeatg;
      }
      
      delay(5000);
    }
    else{
      if(repeatg){
        repeatg=false;
      }
      else{
        repeatg=true;
      }
      while(repeatg){
        String sing = "https://maps.google.com/maps?q=";
        sing = sing + GPS.latitudeDegrees + "," + GPS.longitudeDegrees;
        char charBuff[100];
        sing.toCharArray(charBuff, 100);
        repeatg=fona.sendSMS("16305448893", charBuff);
       // Serial.println(repeat);
        repeatg=!repeatg;
        Serial.println("OUT");
      }
      repeatg=false;
      
      
      delay(1000);
      
    }
  xnum=0;
  ynum=0;
  }
  else{
    digitalWrite(10, HIGH);
    delay(1000);
    digitalWrite(10, LOW);
    delay(1000);
    Serial.print("lat:");
    Serial.print(GPS.latitudeDegrees,7);
    Serial.print("    lon:");
    Serial.println(GPS.longitudeDegrees,7);
  }

}

void flushSerial() {
    while (Serial.available()) 
    Serial.read();
}

