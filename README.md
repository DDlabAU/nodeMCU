# NodeMCU og Arduino
NodeMCU er et arduino-kompatibelt board med indygget wifi. Dette er en kort guide til hvordan du hurtigt får boardet op og køre i dit arduino IDE.

### Kendte problemer

#### Suspekte fejlbeskeder

Får du mærkelige fejlmeldinger fra boardet så som

```
ets Jan 8 2013,rst cause:2, boot mode:(1,6)

ets Jan 8 2013,rst cause:4, boot mode:(1,6)
```


kan en mulig løsning være bare at genstarte wifi-forbindelsen så den ikke forsøger at forbinde til den tidligere forbindelse. Dette gør du ved at indsætte ``` WiFi.persistent(false);``` helt i starten af din setup(). Det er vores erfaring at denne linje og kan være løsningen på fejlmeldinger/problemer, der ikke virker til at have noget at gøre med wifi-forbindelsen. Får du mærkelige problemer med boardet er det altså et forsøg værd, bare for at være på den sikre side.


#### Seriel modul

NodeMCU modulerne bruger en alternativt seriel modul, som kræver særlige drivere både på [MAC](http://blog.sengotta.net/signed-mac-os-driver-for-winchiphead-ch340-serial-bridge/) og [Windows](http://www.wch.cn/download/CH341SER_EXE.html).


~~Kontakt en lab-medarbejder for adgang til den driver der skal bruges til OSX .. medmindre du har opdateret til ElCapitan ... Det er endnu ikke lykkedes nogen fra labbet at få hul igennem til et NodeMCU fra ElCapitan på MAC, men vi arbejder på det. :/~~

Det er nu lykkedes at få hul igennem til ElCapitan!


### Installation Windows
#### og måske tidligere versioner af osx (ikke færdig testet).
Sørg for at du har den nyeste version af [arduino](https://www.arduino.cc/en/Main/Software) installeret før du går i gang.
For at anvende dette board skal du have installeret en pakke af custom boards i dit arduino IDE. Det gør du ved at åbne arduino og navigere til dine indstillinger:

1. "File" -> "Preferences".
* I tekstboksen ud for "Additional Boards Manager URLs:" indsætter du dette link: http://arduino.esp8266.com/stable/package_esp8266com_index.json og trykker på "OK" knappen nederst i vinduet.
* Efter det skal du ind i din "Board Manager" og installere pakken.
  * Det gør du ved at navigere til "Tools" -> "Board: " -> "Boards Manager...".
  * Søg på "esp8266" og du kan vælge: "esp8266 by ESP8266 Community".
  * Du trykker på denne boks og trykker så på "Install".

* Når installationen er færdig, vil du se en mængde nye boards at vælge imellem, når du bevæger dig til "Tools" -> "Board: ".

* Vælg "NodeMCU 0.9 (ESP-12 Module)" og du er i gang :)

### Installation ElCapitan
Vi har indtil videre kun haft held med at bruge terminalen og git til installation af NodeMCU til ElCapitan og selvom det ser svært ud er det ganske ufarligt og du er velkommen til at spørge en ansat hvis du har problemer.Vi laver en guide om hvordan man bruger terminalen og git ganske snart.
På [denne hjemmeside](http://esp8266.github.io/Arduino/versions/2.1.0/doc/installing.html#boards-manager), kan du finde hvordan man henter de rigtige biblioteker og som sagt har vi haft mest held med git, men man kan også forsøge sig med første del af siden der forklarer hvordan man forklare det igennem Arduino IDE(/programmet).
Efter man har fuldt instruktionerne, så installer denne [driver](http://blog.sengotta.net/signed-mac-os-driver-for-winchiphead-ch340-serial-bridge/)(som er den vi har linket til i ovenstående).
Nu skulle det være muligt at NodeMCU 0.9 under boards.


### GPIO
Pinout for NodeMCU boardet:
![Pinout for NodeMCU V0.9](http://ddlab.dk/Node-MCU-Pin-Out-Diagram1.png)
Det er GPIOx numrene som man bruger når man koder det i Arduino IDE'en.

I det hele taget er GPIO lidt underligt på NodeMCU boardet, det er heldigvis ret godt beskrevet [her](https://github.com/esp8266/Arduino/blob/master/variants/nodemcu/pins_arduino.h#L37-L59)

Især linierne:

```
static const uint8_t SDA = 4;
static const uint8_t SCL = 5;

static const uint8_t SS    = 15;
static const uint8_t MOSI  = 13;
static const uint8_t MISO  = 12;
static const uint8_t SCK   = 14;

static const uint8_t BUILTIN_LED = 16;

static const uint8_t A0 = 17;

static const uint8_t D0   = 16;
static const uint8_t D1   = 5;
static const uint8_t D2   = 4;
static const uint8_t D3   = 0;
static const uint8_t D4   = 2;
static const uint8_t D5   = 14;
static const uint8_t D6   = 12;
static const uint8_t D7   = 13;
static const uint8_t D8   = 15;
static const uint8_t D9   = 3;
static const uint8_t D10  = 1;
```
Er vigtige.. .. Der står nemlig at de numre som er printet på modulet (D#) svarer til de ovenstående i koden.

### Eksempler
Har du adgang til et trådløst netværk kan du bruge dette eksempel, der forbinder til netværket og printer den tildelte ip-adresse. Husk selv at indsætte netværksnavn og kode. Dette kan også gøres med et hotspot som du opretter med din smartphone.

```
#include <ESP8266WiFi.h>

// Insert network id and password here

const char* ssid     = "******";
const char* password = "******";


void setup() {
  Serial.begin(115200);
  delay(10);

  // We start by connecting to a WiFi network

  Serial.println();
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}


void loop() {
  Serial.println();
  if (WiFi.status() != WL_CONNECTED) {
    Serial.println("WiFi not connected");
  } else {
    Serial.println("WiFi connected");
    Serial.println("IP address: ");
    Serial.println(WiFi.localIP());
  }
  delay(5000);
}
```

Har du ikke adgang til et trådløst netværk, kan du i stedet bruge denne kode, der periodisk scanner efter og printer omkringværende netværk.

```
#include "ESP8266WiFi.h"

void setup() {
  Serial.begin(115200);

  // Set WiFi to station mode and disconnect from an AP if it was previously connected
  WiFi.mode(WIFI_STA);
  WiFi.disconnect();
  delay(100);

  Serial.println("Setup done");
}

void loop() {
  Serial.println("scan start");

  // WiFi.scanNetworks will return the number of networks found
  int n = WiFi.scanNetworks();
  Serial.println("scan done");
  if (n == 0)
    Serial.println("no networks found");
  else
  {
    Serial.print(n);
    Serial.println(" networks found");
    for (int i = 0; i < n; ++i)
    {
      // Print SSID and RSSI for each network found
      Serial.print(i + 1);
      Serial.print(": ");
      Serial.print(WiFi.SSID(i));
      Serial.print(" (");
      Serial.print(WiFi.RSSI(i));
      Serial.print(")");
      Serial.println((WiFi.encryptionType(i) == ENC_TYPE_NONE)?" ":"*");
      delay(10);
    }
  }
  Serial.println("");

  // Wait a bit before scanning again
  delay(5000);
}
```

Vær i begge tilfælde opmærksom på at boardet ikke resetter når du åbner din "Serial Monitor", så hvis du vil være sikker på at alle dens print med, skal du åbne denne inden du uploader din kode. Husk også at indstille baudraten til "115200".


Som altid kan du finde flere eksempler under "File" -> "Examples". Størstedelen af alle ESP8266 eksemplerne burde kunne anvendes med boardet.
