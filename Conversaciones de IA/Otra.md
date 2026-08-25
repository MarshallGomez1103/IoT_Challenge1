# Revisa código Arduino de sensores

> #include <Wire.h>
> #include <LiquidCrystal_I2C.h>
> #include <Adafruit_BMP085.h>
> #include <DHT.h>
>
> // HC-SR04
> const int PIN_TRIG = 18;
> const int PIN_ECHO = 19;
>
> // DHT11
> const int PIN_DHT = 27;
> #define TIPO_DHT DHT11
>
> // LCD y sensores
> LiquidCrystal_I2C lcd(0x27, 16, 2);
> Adafruit_BMP085 bmp;
> DHT dht(PIN_DHT, TIPO_DHT);
>
> // Variables ambientales
> float humedad = NAN;
> float temperatura = NAN;
> unsigned long ultimaLecturaDHT = 0;
>
> float medirDistancia() {
>   digitalWrite(PIN_TRIG, LOW);
>   delayMicroseconds(2);
>
>   digitalWrite(PIN_TRIG, HIGH);
>   delayMicroseconds(10);
>   digitalWrite(PIN_TRIG, LOW);
>
>   long duracion = pulseIn(PIN_ECHO, HIGH, 30000);
>
>   if (duracion == 0) {
>     return -1;
>   }
>
>   return duracion * 0.0343 / 2.0;
> }
>
> void setup() {
>   Serial.begin(115200);
>
>   pinMode(PIN_TRIG, OUTPUT);
>   pinMode(PIN_ECHO, INPUT);
>
>   // I2C: LCD y BMP180
>   Wire.begin(21, 22);
>
>   lcd.init();
>   lcd.backlight();
>
>   dht.begin();
>
>   lcd.setCursor(0, 0);
>   lcd.print("Iniciando...");
>
>   if (!bmp.begin(BMP085_STANDARD, &Wire)) {
>     lcd.setCursor(0, 1);
>     lcd.print("BMP180 no OK");
>
>     while (true) {
>       delay(10);
>     }
>   }
>
>   lcd.clear();
>   lcd.print("Sensores listos");
>   delay(1500);
> }
>
> void loop() {
>   float distancia = medirDistancia();
>   float presion = bmp.readPressure() / 100.0F; // Pa a hPa
>
>   // El DHT11 debe leerse máximo una vez por segundo.
>   // Aquí se lee cada dos segundos.
>   if (millis() - ultimaLecturaDHT >= 2000) {
>     ultimaLecturaDHT = millis();
>
>     humedad = dht.readHumidity();
>     temperatura = dht.readTemperature();
>
>     Serial.print("Temperatura: ");
>     Serial.print(temperatura);
>     Serial.print(" C | Humedad: ");
>     Serial.print(humedad);
>     Serial.println(" %");
>   }
>
>   // Cambia de pantalla cada dos segundos.
>   bool pantallaAmbiental = (millis() / 2000) % 2;
>
>   lcd.clear();
>
>   if (!pantallaAmbiental) {
>     // Pantalla 1: distancia y presión
>     lcd.setCursor(0, 0);
>
>     if (distancia < 0) {
>       lcd.print("D: Fuera rango");
>     } else {
>       lcd.print("D:");
>       lcd.print(distancia, 1);
>       lcd.print(" cm");
>     }
>
>     lcd.setCursor(0, 1);
>     lcd.print("P:");
>     lcd.print(presion, 1);
>     lcd.print(" hPa");
>
>   } else {
>     // Pantalla 2: temperatura y humedad
>     lcd.setCursor(0, 0);
>
>     if (isnan(temperatura) || isnan(humedad)) {
>       lcd.print("Error DHT11");
>     } else {
>       lcd.print("T:");
>       lcd.print(temperatura, 1);
>       lcd.print(" C");
>
>       lcd.setCursor(0, 1);
>       lcd.print("H:");
>       lcd.print(humedad, 1);
>       lcd.print(" %");
>     }
>   }
>
>   delay(200);
> }
>
> Muchachos, dígame, el código, qué patas van con qué patas, porque, o sea, me estoy a volver a armar. ¿Cuáles van con cuáles? pues para volver a armar todo el código. La pata tal del CP, va con la pata tal del Bitcoin y eso eso. Mencióname todo eso y pues te, con este código que ya está funcionando.

Con ese código, el ESP32 queda conectado así:

| Componente | Pata del componente | Va al ESP32 |
|---|---|---|
| HC-SR04 | VCC | 5V / VIN |
| HC-SR04 | GND | GND |
| HC-SR04 | TRIG | GPIO 18 |
| HC-SR04 | ECHO | GPIO 19, **con divisor de voltaje** |
| DHT11 | VCC | 3.3V |
| DHT11 | GND | GND |
| DHT11 | DATA / OUT | GPIO 27 |
| BMP180 | VCC | 3.3V |
| BMP180 | GND | GND |
| BMP180 | SDA | GPIO 21 |
| BMP180 | SCL | GPIO 22 |
| LCD I2C 16x2 | GND | GND |
| LCD I2C 16x2 | VCC | normalmente 5V / VIN |
| LCD I2C 16x2 | SDA | GPIO 21 |
| LCD I2C 16x2 | SCL | GPIO 22 |

El LCD y el BMP180 comparten los cables I2C:

- GPIO 21: SDA de LCD + SDA de BMP180.
- GPIO 22: SCL de LCD + SCL de BMP180.
- Todos los GND deben estar unidos.

Ojo importante con el HC-SR04: su pin `ECHO` puede mandar 5V, pero el ESP32 solo acepta 3.3V. Entre `ECHO` y GPIO 19 coloca un divisor resistivo, por ejemplo:

```text
ECHO HC-SR04 ── resistencia 1 kΩ ──┬── GPIO 19 ESP32
                                  |
                              resistencia 2 kΩ
                                  |
                                 GND
```

Para el DHT11, si es el sensor suelto de cuatro patas, necesita una resistencia de 10 kΩ entre `VCC` y `DATA`. Si es un módulo de tres patas, normalmente ya la trae integrada.

Tu código usa:

```cpp
Wire.begin(21, 22);
```

Así que no cambies SDA/SCL: 21 es SDA y 22 es SCL.

> Listo, ahí tienes el código. Tengo el DHT11. En el código, pero en realidad, tengo el DHT22 en realidad. Entonces necesito que cambies el DHT11 por el DHT22. Cambiar el código, pero sin cambiar las patas. Vuelveme a pasar el código de nuevo.

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <Adafruit_BMP085.h>
#include <DHT.h>

// HC-SR04
const int PIN_TRIG = 18;
const int PIN_ECHO = 19;

// DHT22
const int PIN_DHT = 27;
#define TIPO_DHT DHT22

// LCD y sensores
LiquidCrystal_I2C lcd(0x27, 16, 2);
Adafruit_BMP085 bmp;
DHT dht(PIN_DHT, TIPO_DHT);

// Variables ambientales
float humedad = NAN;
float temperatura = NAN;
unsigned long ultimaLecturaDHT = 0;

float medirDistancia() {
  digitalWrite(PIN_TRIG, LOW);
  delayMicroseconds(2);

  digitalWrite(PIN_TRIG, HIGH);
  delayMicroseconds(10);
  digitalWrite(PIN_TRIG, LOW);

  long duracion = pulseIn(PIN_ECHO, HIGH, 30000);

  if (duracion == 0) {
    return -1;
  }

  return duracion * 0.0343 / 2.0;
}

void setup() {
  Serial.begin(115200);

  pinMode(PIN_TRIG, OUTPUT);
  pinMode(PIN_ECHO, INPUT);

  // I2C: LCD y BMP180
  Wire.begin(21, 22);

  lcd.init();
  lcd.backlight();

  dht.begin();

  lcd.setCursor(0, 0);
  lcd.print("Iniciando...");

  if (!bmp.begin(BMP085_STANDARD, &Wire)) {
    lcd.setCursor(0, 1);
    lcd.print("BMP180 no OK");

    while (true) {
      delay(10);
    }
  }

  lcd.clear();
  lcd.print("Sensores listos");
  delay(1500);
}

void loop() {
  float distancia = medirDistancia();
  float presion = bmp.readPressure() / 100.0F; // Pa a hPa

  // El DHT22 debe leerse máximo una vez cada dos segundos.
  if (millis() - ultimaLecturaDHT >= 2000) {
    ultimaLecturaDHT = millis();

    humedad = dht.readHumidity();
    temperatura = dht.readTemperature();

    Serial.print("Temperatura: ");
    Serial.print(temperatura);
    Serial.print(" C | Humedad: ");
    Serial.print(humedad);
    Serial.println(" %");
  }

  // Cambia de pantalla cada dos segundos.
  bool pantallaAmbiental = (millis() / 2000) % 2;

  lcd.clear();

  if (!pantallaAmbiental) {
    // Pantalla 1: distancia y presión
    lcd.setCursor(0, 0);

    if (distancia < 0) {
      lcd.print("D: Fuera rango");
    } else {
      lcd.print("D:");
      lcd.print(distancia, 1);
      lcd.print(" cm");
    }

    lcd.setCursor(0, 1);
    lcd.print("P:");
    lcd.print(presion, 1);
    lcd.print(" hPa");
  } else {
    // Pantalla 2: temperatura y humedad
    lcd.setCursor(0, 0);

    if (isnan(temperatura) || isnan(humedad)) {
      lcd.print("Error DHT22");
    } else {
      lcd.print("T:");
      lcd.print(temperatura, 1);
      lcd.print(" C");

      lcd.setCursor(0, 1);
      lcd.print("H:");
      lcd.print(humedad, 1);
      lcd.print(" %");
    }
  }

  delay(200);
}
```

No se cambia ninguna pata: el pin de datos del DHT22 sigue conectado a GPIO 27. Solo cambió `DHT11` por `DHT22`.

> Ahora te envío, escribo una transcripción que se llama un futuro register, de Sound Founder Team Education. y ya, dice que es la versión 1.0. Entonces, dame el código, pero para agregar ese lineamiento de lo que.

No alcancé a identificar el componente por la transcripción. ¿Puedes escribir el nombre tal como aparece en la pieza o mandarme una foto? Con eso te agrego el código sin cambiar las conexiones actuales.

> fotoresistor v1 sunfounder

Conecta el fotoresistor SunFounder así, sin tocar las patas que ya tienes:

| Fotoresistor | ESP32 |
|---|---|
| `S` / `AO` / señal | GPIO 34 |
| `+` / `VCC` | 3.3V |
| `-` / `GND` | GND |

Este código agrega una tercera pantalla con el nivel de luz. El valor va de 0 a 4095: no son “lux”, sino una lectura relativa de claridad.

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <Adafruit_BMP085.h>
#include <DHT.h>

// HC-SR04
const int PIN_TRIG = 18;
const int PIN_ECHO = 19;

// DHT22
const int PIN_DHT = 27;
#define TIPO_DHT DHT22

// Fotoresistor SunFounder
const int PIN_FOTORESISTOR = 34;

// LCD y sensores
LiquidCrystal_I2C lcd(0x27, 16, 2);
Adafruit_BMP085 bmp;
DHT dht(PIN_DHT, TIPO_DHT);

// Variables ambientales
float humedad = NAN;
float temperatura = NAN;
unsigned long ultimaLecturaDHT = 0;

float medirDistancia() {
  digitalWrite(PIN_TRIG, LOW);
  delayMicroseconds(2);

  digitalWrite(PIN_TRIG, HIGH);
  delayMicroseconds(10);
  digitalWrite(PIN_TRIG, LOW);

  long duracion = pulseIn(PIN_ECHO, HIGH, 30000);

  if (duracion == 0) {
    return -1;
  }

  return duracion * 0.0343 / 2.0;
}

void setup() {
  Serial.begin(115200);

  pinMode(PIN_TRIG, OUTPUT);
  pinMode(PIN_ECHO, INPUT);
  pinMode(PIN_FOTORESISTOR, INPUT);

  // I2C: LCD y BMP180
  Wire.begin(21, 22);

  lcd.init();
  lcd.backlight();
  dht.begin();

  lcd.setCursor(0, 0);
  lcd.print("Iniciando...");

  if (!bmp.begin(BMP085_STANDARD, &Wire)) {
    lcd.setCursor(0, 1);
    lcd.print("BMP180 no OK");

    while (true) {
      delay(10);
    }
  }

  lcd.clear();
  lcd.print("Sensores listos");
  delay(1500);
}

void loop() {
  float distancia = medirDistancia();
  float presion = bmp.readPressure() / 100.0F;
  int luz = analogRead(PIN_FOTORESISTOR);

  // El DHT22 se lee cada dos segundos.
  if (millis() - ultimaLecturaDHT >= 2000) {
    ultimaLecturaDHT = millis();

    humedad = dht.readHumidity();
    temperatura = dht.readTemperature();

    Serial.print("Temperatura: ");
    Serial.print(temperatura);
    Serial.print(" C | Humedad: ");
    Serial.print(humedad);
    Serial.print(" % | Luz: ");
    Serial.println(luz);
  }

  // Tres pantallas, cada una dura dos segundos.
  int pantalla = (millis() / 2000) % 3;

  lcd.clear();

  if (pantalla == 0) {
    lcd.setCursor(0, 0);

    if (distancia < 0) {
      lcd.print("D: Fuera rango");
    } else {
      lcd.print("D:");
      lcd.print(distancia, 1);
      lcd.print(" cm");
    }

    lcd.setCursor(0, 1);
    lcd.print("P:");
    lcd.print(presion, 1);
    lcd.print(" hPa");

  } else if (pantalla == 1) {
    lcd.setCursor(0, 0);

    if (isnan(temperatura) || isnan(humedad)) {
      lcd.print("Error DHT22");
    } else {
      lcd.print("T:");
      lcd.print(temperatura, 1);
      lcd.print(" C");

      lcd.setCursor(0, 1);
      lcd.print("H:");
      lcd.print(humedad, 1);
      lcd.print(" %");
    }

  } else {
    lcd.setCursor(0, 0);
    lcd.print("Luz: ");
    lcd.print(luz);

    lcd.setCursor(0, 1);

    if (luz < 1200) {
      lcd.print("Ambiente oscuro");
    } else if (luz < 2800) {
      lcd.print("Luz media");
    } else {
      lcd.print("Ambiente claro");
    }
  }

  delay(200);
}
```

Si al tapar el fotoresistor el número sube en vez de bajar, está bien: depende de cómo venga armado el módulo. Lo importante es mirar los valores reales por el Monitor Serie y, si quieres, ajustamos los rangos de “oscuro”, “medio” y “claro”.

> Bueno, estoy acá y lo que pasa es que cuando le pongo el cable resistencia al fotorresistor, me dice min 0 y me dice 28. Pero, o sea, siempre uno. y si lo dejo, lo que pasa es que, mi diagrama, 1,690, entonces, calculemos eso.

Si ahora te está dando `1690`, en el ESP32 eso equivale a:

```text
1690 / 4095 × 100 = 41.3 %
```

O sea: el fotoresistor está detectando aproximadamente **41 % de luz relativa** dentro de su rango de lectura. No son lux reales.

Para que en la pantalla muestre ese porcentaje, cambia estas líneas:

```cpp
int luz = analogRead(PIN_FOTORESISTOR);
```

por:

```cpp
int luz = analogRead(PIN_FOTORESISTOR);
int porcentajeLuz = (luz * 100) / 4095;
```

Y, en la tercera pantalla, cambia:

```cpp
lcd.print("Luz: ");
lcd.print(luz);
```

por:

```cpp
lcd.print("Luz: ");
lcd.print(porcentajeLuz);
lcd.print("%");
```

Entonces con el valor `1690` verás aproximadamente:

```text
Luz: 41%
```

Si te aparece siempre `0`, `1`, `28` o valores muy pequeños, revisa que el cable de señal del fotoresistor esté conectado a `GPIO 34`, no a una pata digital, y que el módulo esté alimentado con `3.3V` y `GND`.

> que?, dame el codigo completo

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <Adafruit_BMP085.h>
#include <DHT.h>

// HC-SR04
const int PIN_TRIG = 18;
const int PIN_ECHO = 19;

// DHT22
const int PIN_DHT = 27;
#define TIPO_DHT DHT22

// Fotoresistor SunFounder
const int PIN_FOTORESISTOR = 34;

// LCD y sensores
LiquidCrystal_I2C lcd(0x27, 16, 2);
Adafruit_BMP085 bmp;
DHT dht(PIN_DHT, TIPO_DHT);

// Variables ambientales
float humedad = NAN;
float temperatura = NAN;
unsigned long ultimaLecturaDHT = 0;

float medirDistancia() {
  digitalWrite(PIN_TRIG, LOW);
  delayMicroseconds(2);

  digitalWrite(PIN_TRIG, HIGH);
  delayMicroseconds(10);
  digitalWrite(PIN_TRIG, LOW);

  long duracion = pulseIn(PIN_ECHO, HIGH, 30000);

  if (duracion == 0) {
    return -1;
  }

  return duracion * 0.0343 / 2.0;
}

void setup() {
  Serial.begin(115200);

  pinMode(PIN_TRIG, OUTPUT);
  pinMode(PIN_ECHO, INPUT);
  pinMode(PIN_FOTORESISTOR, INPUT);

  // I2C: LCD y BMP180
  Wire.begin(21, 22);

  lcd.init();
  lcd.backlight();

  dht.begin();

  lcd.setCursor(0, 0);
  lcd.print("Iniciando...");

  if (!bmp.begin(BMP085_STANDARD, &Wire)) {
    lcd.setCursor(0, 1);
    lcd.print("BMP180 no OK");

    while (true) {
      delay(10);
    }
  }

  lcd.clear();
  lcd.print("Sensores listos");
  delay(1500);
}

void loop() {
  float distancia = medirDistancia();
  float presion = bmp.readPressure() / 100.0F;

  // Lectura del fotoresistor: 0 a 4095
  int luz = analogRead(PIN_FOTORESISTOR);
  int porcentajeLuz = (luz * 100) / 4095;

  // El DHT22 se lee cada dos segundos.
  if (millis() - ultimaLecturaDHT >= 2000) {
    ultimaLecturaDHT = millis();

    humedad = dht.readHumidity();
    temperatura = dht.readTemperature();

    Serial.print("Temperatura: ");
    Serial.print(temperatura);
    Serial.print(" C | Humedad: ");
    Serial.print(humedad);
    Serial.print(" % | Luz: ");
    Serial.print(luz);
    Serial.print(" (");
    Serial.print(porcentajeLuz);
    Serial.println("%)");
  }

  // Tres pantallas; cada una cambia cada dos segundos.
  int pantalla = (millis() / 2000) % 3;

  lcd.clear();

  if (pantalla == 0) {
    // Pantalla 1: distancia y presión
    lcd.setCursor(0, 0);

    if (distancia < 0) {
      lcd.print("D: Fuera rango");
    } else {
      lcd.print("D:");
      lcd.print(distancia, 1);
      lcd.print(" cm");
    }

    lcd.setCursor(0, 1);
    lcd.print("P:");
    lcd.print(presion, 1);
    lcd.print(" hPa");

  } else if (pantalla == 1) {
    // Pantalla 2: temperatura y humedad
    lcd.setCursor(0, 0);

    if (isnan(temperatura) || isnan(humedad)) {
      lcd.print("Error DHT22");
    } else {
      lcd.print("T:");
      lcd.print(temperatura, 1);
      lcd.print(" C");

      lcd.setCursor(0, 1);
      lcd.print("H:");
      lcd.print(humedad, 1);
      lcd.print(" %");
    }

  } else {
    // Pantalla 3: luz
    lcd.setCursor(0, 0);
    lcd.print("Luz: ");
    lcd.print(porcentajeLuz);
    lcd.print("%");

    lcd.setCursor(0, 1);

    if (porcentajeLuz < 30) {
      lcd.print("Ambiente oscuro");
    } else if (porcentajeLuz < 70) {
      lcd.print("Luz media");
    } else {
      lcd.print("Ambiente claro");
    }
  }

  delay(200);
}
```

> Entonces, muchacho, ahora necesito conectar un, espérame, tenemos un RGB. Un RGB, le tengo las propiedades. Muchacho, tengo, te iba a decir que tengo un LED rojo y quiero conectarlo a una pata del LCD. Entonces, cuando la distancia, con el sensor de distancia, que es menor a 5 centímetros, o sea, entre 0 y 5, que se prenda. Y también hay un paso paser de la misma marca, ¿no? SunFounder, que sería bueno que hiciera eso. Pero no otra vez encontrar el paso paser, salió un RGB LED de la misma marca SunFounder. Entonces quiero que el RGB LED y el paso paser, me regreses el código para que cuando la distancia sea menor a 5 centímetros, se prenda y haga luces, pero haz una luz de policía. Entonces que sea como alerta, ¿no? Rojo, azul, rojo, azul, quiero que sea la luz de policía. Y pues que el paso paser haga una sirena de policía. Solo, por favor, lo.

No conectes el RGB a una pata del LCD: conéctalo directamente al ESP32.

| Componente | Pata | ESP32 |
|---|---|---|
| RGB SunFounder | `R` | GPIO 25 |
| RGB SunFounder | `G` | GPIO 26 |
| RGB SunFounder | `B` | GPIO 32 |
| RGB SunFounder | `-` / `GND` común | GND |
| Buzzer pasivo SunFounder | `+` / `S` | GPIO 33 |
| Buzzer pasivo SunFounder | `-` / `GND` | GND |

Si tu RGB es un LED suelto de cuatro patas, pon una resistencia de 220–330 Ω en cada cable `R`, `G` y `B`.

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <Adafruit_BMP085.h>
#include <DHT.h>

// HC-SR04
const int PIN_TRIG = 18;
const int PIN_ECHO = 19;

// DHT22
const int PIN_DHT = 27;
#define TIPO_DHT DHT22

// Fotoresistor SunFounder
const int PIN_FOTORESISTOR = 34;

// RGB SunFounder
const int PIN_RGB_ROJO = 25;
const int PIN_RGB_VERDE = 26;
const int PIN_RGB_AZUL = 32;

// Buzzer pasivo SunFounder
const int PIN_BUZZER = 33;

// LCD y sensores
LiquidCrystal_I2C lcd(0x27, 16, 2);
Adafruit_BMP085 bmp;
DHT dht(PIN_DHT, TIPO_DHT);

// Variables ambientales
float humedad = NAN;
float temperatura = NAN;
unsigned long ultimaLecturaDHT = 0;

// Variables de alerta policial
unsigned long ultimoCambioAlerta = 0;
bool luzRoja = true;

float medirDistancia() {
  digitalWrite(PIN_TRIG, LOW);
  delayMicroseconds(2);

  digitalWrite(PIN_TRIG, HIGH);
  delayMicroseconds(10);
  digitalWrite(PIN_TRIG, LOW);

  long duracion = pulseIn(PIN_ECHO, HIGH, 30000);

  if (duracion == 0) {
    return -1;
  }

  return duracion * 0.0343 / 2.0;
}

void apagarAlerta() {
  digitalWrite(PIN_RGB_ROJO, LOW);
  digitalWrite(PIN_RGB_VERDE, LOW);
  digitalWrite(PIN_RGB_AZUL, LOW);
  noTone(PIN_BUZZER);
}

void actualizarAlerta(float distancia) {
  // Se activa solamente desde 0 cm hasta 5 cm.
  bool objetoCerca = (distancia >= 0 && distancia <= 5.0);

  if (!objetoCerca) {
    apagarAlerta();
    return;
  }

  // Alterna rojo y azul cada 250 milisegundos.
  if (millis() - ultimoCambioAlerta >= 250) {
    ultimoCambioAlerta = millis();
    luzRoja = !luzRoja;
  }

  digitalWrite(PIN_RGB_VERDE, LOW);

  if (luzRoja) {
    digitalWrite(PIN_RGB_ROJO, HIGH);
    digitalWrite(PIN_RGB_AZUL, LOW);
    tone(PIN_BUZZER, 900);   // Tono bajo de sirena
  } else {
    digitalWrite(PIN_RGB_ROJO, LOW);
    digitalWrite(PIN_RGB_AZUL, HIGH);
    tone(PIN_BUZZER, 1300);  // Tono alto de sirena
  }
}

void setup() {
  Serial.begin(115200);

  pinMode(PIN_TRIG, OUTPUT);
  pinMode(PIN_ECHO, INPUT);
  pinMode(PIN_FOTORESISTOR, INPUT);

  pinMode(PIN_RGB_ROJO, OUTPUT);
  pinMode(PIN_RGB_VERDE, OUTPUT);
  pinMode(PIN_RGB_AZUL, OUTPUT);
  pinMode(PIN_BUZZER, OUTPUT);

  apagarAlerta();

  // I2C: LCD y BMP180
  Wire.begin(21, 22);

  lcd.init();
  lcd.backlight();
  dht.begin();

  lcd.setCursor(0, 0);
  lcd.print("Iniciando...");

  if (!bmp.begin(BMP085_STANDARD, &Wire)) {
    lcd.setCursor(0, 1);
    lcd.print("BMP180 no OK");

    while (true) {
      delay(10);
    }
  }

  lcd.clear();
  lcd.print("Sensores listos");
  delay(1500);
}

void loop() {
  float distancia = medirDistancia();
  float presion = bmp.readPressure() / 100.0F;

  int luz = analogRead(PIN_FOTORESISTOR);
  int porcentajeLuz = (luz * 100) / 4095;

  // Activa RGB y sirena si hay algo a 5 cm o menos.
  actualizarAlerta(distancia);

  // El DHT22 se lee cada dos segundos.
  if (millis() - ultimaLecturaDHT >= 2000) {
    ultimaLecturaDHT = millis();

    humedad = dht.readHumidity();
    temperatura = dht.readTemperature();

    Serial.print("Temperatura: ");
    Serial.print(temperatura);
    Serial.print(" C | Humedad: ");
    Serial.print(humedad);
    Serial.print(" % | Luz: ");
    Serial.print(porcentajeLuz);
    Serial.println("%");
  }

  // Tres pantallas; cambian cada dos segundos.
  int pantalla = (millis() / 2000) % 3;

  lcd.clear();

  if (pantalla == 0) {
    lcd.setCursor(0, 0);

    if (distancia < 0) {
      lcd.print("D: Fuera rango");
    } else {
      lcd.print("D:");
      lcd.print(distancia, 1);
      lcd.print(" cm");
    }

    lcd.setCursor(0, 1);

    if (distancia >= 0 && distancia <= 5.0) {
      lcd.print("!! ALERTA !!");
    } else {
      lcd.print("P:");
      lcd.print(presion, 1);
      lcd.print(" hPa");
    }

  } else if (pantalla == 1) {
    lcd.setCursor(0, 0);

    if (isnan(temperatura) || isnan(humedad)) {
      lcd.print("Error DHT22");
    } else {
      lcd.print("T:");
      lcd.print(temperatura, 1);
      lcd.print(" C");

      lcd.setCursor(0, 1);
      lcd.print("H:");
      lcd.print(humedad, 1);
      lcd.print(" %");
    }

  } else {
    lcd.setCursor(0, 0);
    lcd.print("Luz: ");
    lcd.print(porcentajeLuz);
    lcd.print("%");

    lcd.setCursor(0, 1);

    if (porcentajeLuz < 30) {
      lcd.print("Ambiente oscuro");
    } else if (porcentajeLuz < 70) {
      lcd.print("Luz media");
    } else {
      lcd.print("Ambiente claro");
    }
  }

  delay(100);
}
```

Si el RGB queda con los colores invertidos o permanece prendido cuando debería apagarse, tu RGB es de ánodo común. Dime si el pin común está marcado `+`, y te paso la versión exacta para ese tipo.

> El RGB es un mismo componente de Sunfounder, se llama RGB LED y tiene cuatro patas, RGB y VCC, no tiene más. No sé si se llama RGB LED. Y el passive buzzer tiene tres paticas: signal, VCC y ground. Entonces se llama passive buzzer version 1.0. Entonces recalibra todo, código y patas.

Correcto: tu RGB es de **ánodo común** porque trae `VCC`, `R`, `G` y `B`. Por eso cambia la lógica: `LOW` enciende un color y `HIGH` lo apaga.

Conexiones completas:

| Componente | Pata | Conectar al ESP32 |
|---|---|---|
| RGB LED SunFounder | `VCC` | 3.3V |
| RGB LED SunFounder | `R` | GPIO 25 |
| RGB LED SunFounder | `G` | GPIO 26 |
| RGB LED SunFounder | `B` | GPIO 32 |
| Passive Buzzer V1.0 | `Signal` / `S` | GPIO 33 |
| Passive Buzzer V1.0 | `VCC` | 3.3V |
| Passive Buzzer V1.0 | `GND` | GND |

El RGB y el buzzer comparten el GND mediante el ESP32. No cambies las conexiones anteriores.

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <Adafruit_BMP085.h>
#include <DHT.h>

// HC-SR04
const int PIN_TRIG = 18;
const int PIN_ECHO = 19;

// DHT22
const int PIN_DHT = 27;
#define TIPO_DHT DHT22

// Fotoresistor SunFounder
const int PIN_FOTORESISTOR = 34;

// RGB LED SunFounder: R, G, B y VCC
const int PIN_RGB_ROJO = 25;
const int PIN_RGB_VERDE = 26;
const int PIN_RGB_AZUL = 32;

// Passive Buzzer SunFounder V1.0
const int PIN_BUZZER = 33;

// LCD y sensores
LiquidCrystal_I2C lcd(0x27, 16, 2);
Adafruit_BMP085 bmp;
DHT dht(PIN_DHT, TIPO_DHT);

// Variables ambientales
float humedad = NAN;
float temperatura = NAN;
unsigned long ultimaLecturaDHT = 0;

// Variables de alerta
unsigned long ultimoCambioAlerta = 0;
bool luzRoja = true;

float medirDistancia() {
  digitalWrite(PIN_TRIG, LOW);
  delayMicroseconds(2);

  digitalWrite(PIN_TRIG, HIGH);
  delayMicroseconds(10);
  digitalWrite(PIN_TRIG, LOW);

  long duracion = pulseIn(PIN_ECHO, HIGH, 30000);

  if (duracion == 0) {
    return -1;
  }

  return duracion * 0.0343 / 2.0;
}

// RGB de ánodo común: HIGH apaga, LOW enciende.
void apagarRGB() {
  digitalWrite(PIN_RGB_ROJO, HIGH);
  digitalWrite(PIN_RGB_VERDE, HIGH);
  digitalWrite(PIN_RGB_AZUL, HIGH);
}

void apagarAlerta() {
  apagarRGB();
  noTone(PIN_BUZZER);
}

void actualizarAlerta(float distancia) {
  // Alerta desde 0 cm hasta 5 cm.
  bool objetoCerca = (distancia >= 0 && distancia <= 5.0);

  if (!objetoCerca) {
    apagarAlerta();
    return;
  }

  // Alterna rojo y azul cada 250 ms.
  if (millis() - ultimoCambioAlerta >= 250) {
    ultimoCambioAlerta = millis();
    luzRoja = !luzRoja;
  }

  if (luzRoja) {
    // Rojo encendido
    digitalWrite(PIN_RGB_ROJO, LOW);
    digitalWrite(PIN_RGB_VERDE, HIGH);
    digitalWrite(PIN_RGB_AZUL, HIGH);

    tone(PIN_BUZZER, 900);
  } else {
    // Azul encendido
    digitalWrite(PIN_RGB_ROJO, HIGH);
    digitalWrite(PIN_RGB_VERDE, HIGH);
    digitalWrite(PIN_RGB_AZUL, LOW);

    tone(PIN_BUZZER, 1300);
  }
}

void setup() {
  Serial.begin(115200);

  pinMode(PIN_TRIG, OUTPUT);
  pinMode(PIN_ECHO, INPUT);
  pinMode(PIN_FOTORESISTOR, INPUT);

  pinMode(PIN_RGB_ROJO, OUTPUT);
  pinMode(PIN_RGB_VERDE, OUTPUT);
  pinMode(PIN_RGB_AZUL, OUTPUT);
  pinMode(PIN_BUZZER, OUTPUT);

  apagarAlerta();

  // I2C: LCD y BMP180
  Wire.begin(21, 22);

  lcd.init();
  lcd.backlight();
  dht.begin();

  lcd.setCursor(0, 0);
  lcd.print("Iniciando...");

  if (!bmp.begin(BMP085_STANDARD, &Wire)) {
    lcd.setCursor(0, 1);
    lcd.print("BMP180 no OK");

    while (true) {
      delay(10);
    }
  }

  lcd.clear();
  lcd.print("Sensores listos");
  delay(1500);
}

void loop() {
  float distancia = medirDistancia();
  float presion = bmp.readPressure() / 100.0F;

  int luz = analogRead(PIN_FOTORESISTOR);
  int porcentajeLuz = (luz * 100) / 4095;

  // Luz policial y sirena si la distancia es menor o igual a 5 cm.
  actualizarAlerta(distancia);

  // Lectura DHT22 cada dos segundos.
  if (millis() - ultimaLecturaDHT >= 2000) {
    ultimaLecturaDHT = millis();

    humedad = dht.readHumidity();
    temperatura = dht.readTemperature();

    Serial.print("Temperatura: ");
    Serial.print(temperatura);
    Serial.print(" C | Humedad: ");
    Serial.print(humedad);
    Serial.print(" % | Luz: ");
    Serial.print(porcentajeLuz);
    Serial.println("%");
  }

  // Cambia pantalla cada dos segundos.
  int pantalla = (millis() / 2000) % 3;

  lcd.clear();

  if (pantalla == 0) {
    lcd.setCursor(0, 0);

    if (distancia < 0) {
      lcd.print("D: Fuera rango");
    } else {
      lcd.print("D:");
      lcd.print(distancia, 1);
      lcd.print(" cm");
    }

    lcd.setCursor(0, 1);

    if (distancia >= 0 && distancia <= 5.0) {
      lcd.print("!! ALERTA !!");
    } else {
      lcd.print("P:");
      lcd.print(presion, 1);
      lcd.print(" hPa");
    }

  } else if (pantalla == 1) {
    lcd.setCursor(0, 0);

    if (isnan(temperatura) || isnan(humedad)) {
      lcd.print("Error DHT22");
    } else {
      lcd.print("T:");
      lcd.print(temperatura, 1);
      lcd.print(" C");

      lcd.setCursor(0, 1);
      lcd.print("H:");
      lcd.print(humedad, 1);
      lcd.print(" %");
    }

  } else {
    lcd.setCursor(0, 0);
    lcd.print("Luz: ");
    lcd.print(porcentajeLuz);
    lcd.print("%");

    lcd.setCursor(0, 1);

    if (porcentajeLuz < 30) {
      lcd.print("Ambiente oscuro");
    } else if (porcentajeLuz < 70) {
      lcd.print("Luz media");
    } else {
      lcd.print("Ambiente claro");
    }
  }

  delay(100);
}
```

> Pasame el código, pero cambiarle, voy a decir, hacer, por un Active Fuzzer, el cual requiere un GPIO para prender, entonces de pronto hay que cambiar las patas, pero no creo. O sea, está en la misma pata que la 33.

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <Adafruit_BMP085.h>
#include <DHT.h>

// HC-SR04
const int PIN_TRIG = 18;
const int PIN_ECHO = 19;

// DHT22
const int PIN_DHT = 27;
#define TIPO_DHT DHT22

// Fotoresistor SunFounder
const int PIN_FOTORESISTOR = 34;

// RGB LED SunFounder: R, G, B y VCC
const int PIN_RGB_ROJO = 25;
const int PIN_RGB_VERDE = 26;
const int PIN_RGB_AZUL = 32;

// Active Buzzer SunFounder V1.0
const int PIN_BUZZER = 33;

// LCD y sensores
LiquidCrystal_I2C lcd(0x27, 16, 2);
Adafruit_BMP085 bmp;
DHT dht(PIN_DHT, TIPO_DHT);

// Variables ambientales
float humedad = NAN;
float temperatura = NAN;
unsigned long ultimaLecturaDHT = 0;

// Variables de alerta
unsigned long ultimoCambioAlerta = 0;
bool luzRoja = true;

float medirDistancia() {
  digitalWrite(PIN_TRIG, LOW);
  delayMicroseconds(2);

  digitalWrite(PIN_TRIG, HIGH);
  delayMicroseconds(10);
  digitalWrite(PIN_TRIG, LOW);

  long duracion = pulseIn(PIN_ECHO, HIGH, 30000);

  if (duracion == 0) {
    return -1;
  }

  return duracion * 0.0343 / 2.0;
}

// RGB de ánodo común: HIGH apaga, LOW enciende.
void apagarRGB() {
  digitalWrite(PIN_RGB_ROJO, HIGH);
  digitalWrite(PIN_RGB_VERDE, HIGH);
  digitalWrite(PIN_RGB_AZUL, HIGH);
}

void apagarAlerta() {
  apagarRGB();
  digitalWrite(PIN_BUZZER, LOW);
}

void actualizarAlerta(float distancia) {
  // Activa alerta desde 0 cm hasta 5 cm.
  bool objetoCerca = (distancia >= 0 && distancia <= 5.0);

  if (!objetoCerca) {
    apagarAlerta();
    return;
  }

  // Alterna rojo y azul cada 250 ms.
  if (millis() - ultimoCambioAlerta >= 250) {
    ultimoCambioAlerta = millis();
    luzRoja = !luzRoja;
  }

  // El active buzzer no usa tone(): solo ON u OFF.
  digitalWrite(PIN_BUZZER, HIGH);

  if (luzRoja) {
    // Rojo encendido
    digitalWrite(PIN_RGB_ROJO, LOW);
    digitalWrite(PIN_RGB_VERDE, HIGH);
    digitalWrite(PIN_RGB_AZUL, HIGH);
  } else {
    // Azul encendido
    digitalWrite(PIN_RGB_ROJO, HIGH);
    digitalWrite(PIN_RGB_VERDE, HIGH);
    digitalWrite(PIN_RGB_AZUL, LOW);
  }
}

void setup() {
  Serial.begin(115200);

  pinMode(PIN_TRIG, OUTPUT);
  pinMode(PIN_ECHO, INPUT);
  pinMode(PIN_FOTORESISTOR, INPUT);

  pinMode(PIN_RGB_ROJO, OUTPUT);
  pinMode(PIN_RGB_VERDE, OUTPUT);
  pinMode(PIN_RGB_AZUL, OUTPUT);
  pinMode(PIN_BUZZER, OUTPUT);

  apagarAlerta();

  // I2C: LCD y BMP180
  Wire.begin(21, 22);

  lcd.init();
  lcd.backlight();
  dht.begin();

  lcd.setCursor(0, 0);
  lcd.print("Iniciando...");

  if (!bmp.begin(BMP085_STANDARD, &Wire)) {
    lcd.setCursor(0, 1);
    lcd.print("BMP180 no OK");

    while (true) {
      delay(10);
    }
  }

  lcd.clear();
  lcd.print("Sensores listos");
  delay(1500);
}

void loop() {
  float distancia = medirDistancia();
  float presion = bmp.readPressure() / 100.0F;

  int luz = analogRead(PIN_FOTORESISTOR);
  int porcentajeLuz = (luz * 100) / 4095;

  // Activa RGB y buzzer si la distancia es menor o igual a 5 cm.
  actualizarAlerta(distancia);

  // Lectura DHT22 cada dos segundos.
  if (millis() - ultimaLecturaDHT >= 2000) {
    ultimaLecturaDHT = millis();

    humedad = dht.readHumidity();
    temperatura = dht.readTemperature();

    Serial.print("Temperatura: ");
    Serial.print(temperatura);
    Serial.print(" C | Humedad: ");
    Serial.print(humedad);
    Serial.print(" % | Luz: ");
    Serial.print(porcentajeLuz);
    Serial.println("%");
  }

  // Cambia pantalla cada dos segundos.
  int pantalla = (millis() / 2000) % 3;

  lcd.clear();

  if (pantalla == 0) {
    lcd.setCursor(0, 0);

    if (distancia < 0) {
      lcd.print("D: Fuera rango");
    } else {
      lcd.print("D:");
      lcd.print(distancia, 1);
      lcd.print(" cm");
    }

    lcd.setCursor(0, 1);

    if (distancia >= 0 && distancia <= 5.0) {
      lcd.print("!! ALERTA !!");
    } else {
      lcd.print("P:");
      lcd.print(presion, 1);
      lcd.print(" hPa");
    }

  } else if (pantalla == 1) {
    lcd.setCursor(0, 0);

    if (isnan(temperatura) || isnan(humedad)) {
      lcd.print("Error DHT22");
    } else {
      lcd.print("T:");
      lcd.print(temperatura, 1);
      lcd.print(temperatura, 1);
      lcd.print(" C");

      lcd.setCursor(0, 1);
      lcd.print("H:");
      lcd.print(humedad, 1);
      lcd.print(" %");
    }

  } else {
    lcd.setCursor(0, 0);
    lcd.print("Luz: ");
    lcd.print(porcentajeLuz);
    lcd.print("%");

    lcd.setCursor(0, 1);

    if (porcentajeLuz < 30) {
      lcd.print("Ambiente oscuro");
    } else if (porcentajeLuz < 70) {
      lcd.print("Luz media");
    } else {
      lcd.print("Ambiente claro");
    }
  }

  delay(100);
}
```

El Active Buzzer queda exactamente en la misma conexión:

- `Signal` → GPIO 33
- `VCC` → 3.3V
- `GND` → GND

Nota: en el bloque de temperatura había una impresión duplicada; en el código de arriba revisa y elimina una de estas dos líneas idénticas si aparecen:

```cpp
lcd.print(temperatura, 1);
```

> \ Y el sensor pasivo, no, el sensor de sonido activo quedó mal porque la lógica está invertida. Entonces, cuando está a menos de 5 centímetros, no suena, pero cuando está a más de 5 centímetros, sí suena. Entonces quedó mal. Entonces, queda al revés. Solo eso, la luz sí quedó bien. No toques la luz, por favor.

El Active Buzzer de tu módulo funciona con lógica invertida: `LOW` suena y `HIGH` se apaga.

No toques nada del RGB. Solo cambia estas dos líneas:

```cpp
// En apagarAlerta():
digitalWrite(PIN_BUZZER, HIGH);
```

```cpp
// En actualizarAlerta(), cuando el objeto está a 5 cm o menos:
digitalWrite(PIN_BUZZER, LOW);
```

Quedan así las funciones:

```cpp
void apagarAlerta() {
  apagarRGB();
  digitalWrite(PIN_BUZZER, HIGH); // Buzzer apagado
}

void actualizarAlerta(float distancia) {
  bool objetoCerca = (distancia >= 0 && distancia <= 5.0);

  if (!objetoCerca) {
    apagarAlerta();
    return;
  }

  if (millis() - ultimoCambioAlerta >= 250) {
    ultimoCambioAlerta = millis();
    luzRoja = !luzRoja;
  }

  digitalWrite(PIN_BUZZER, LOW); // Buzzer encendido

  // El código del RGB queda exactamente como ya lo tienes.
}
```

> ahora quiero que cuando se active la condicion por debajo de los 5cm quiero que la alarma no haga solo un beep sino multiples como beeep beep beep asi porfa

Agrega estas dos variables junto a las variables de alerta:

```cpp
unsigned long ultimoCambioBuzzer = 0;
bool buzzerSonando = false;
```

Luego reemplaza por completo tu función `actualizarAlerta()` por esta. El RGB queda igual; solo el buzzer hará `beep beep beep` mientras haya algo a 5 cm o menos:

```cpp
void actualizarAlerta(float distancia) {
  bool objetoCerca = (distancia >= 0 && distancia <= 5.0);

  if (!objetoCerca) {
    apagarAlerta();
    buzzerSonando = false;
    return;
  }

  // Luces policiales: rojo y azul.
  if (millis() - ultimoCambioAlerta >= 250) {
    ultimoCambioAlerta = millis();
    luzRoja = !luzRoja;
  }

  if (luzRoja) {
    digitalWrite(PIN_RGB_ROJO, LOW);
    digitalWrite(PIN_RGB_VERDE, HIGH);
    digitalWrite(PIN_RGB_AZUL, HIGH);
  } else {
    digitalWrite(PIN_RGB_ROJO, HIGH);
    digitalWrite(PIN_RGB_VERDE, HIGH);
    digitalWrite(PIN_RGB_AZUL, LOW);
  }

  // Beep beep beep: cambia entre sonar y callar cada 140 ms.
  if (millis() - ultimoCambioBuzzer >= 140) {
    ultimoCambioBuzzer = millis();
    buzzerSonando = !buzzerSonando;
  }

  // El buzzer es activo con lógica invertida:
  // LOW = suena, HIGH = silencio.
  if (buzzerSonando) {
    digitalWrite(PIN_BUZZER, LOW);
  } else {
    digitalWrite(PIN_BUZZER, HIGH);
  }
}
```

> super bien, ahora dame el codigo completo pero mas chevere, ya que quiero agregar un boton para alternar entre los sensores mas no que el display alterne solo, quiero que cuando oprima el boton, el display rote la info de los sensore, ayudame con eso, dime donde conecto las 4 patas del boton y eso

Conecta el pulsador a `GPIO 14` y `GND`. No necesita resistencia externa: el código usa la resistencia interna del ESP32.

En un botón de cuatro patas, las dos patas de un mismo lado ya están unidas internamente. Ponlo atravesando la ranura central de la protoboard:

```text
Pata de un lado del botón  → GPIO 14
Pata del lado opuesto      → GND

Las otras dos patas pueden quedar sin conectar.
```

Cada pulsación cambia manualmente la pantalla: distancia/presión → temperatura/humedad → luz → vuelve a empezar.

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <Adafruit_BMP085.h>
#include <DHT.h>

// HC-SR04
const int PIN_TRIG = 18;
const int PIN_ECHO = 19;

// DHT22
const int PIN_DHT = 27;
#define TIPO_DHT DHT22

// Fotoresistor SunFounder
const int PIN_FOTORESISTOR = 34;

// RGB LED SunFounder: R, G, B y VCC
const int PIN_RGB_ROJO = 25;
const int PIN_RGB_VERDE = 26;
const int PIN_RGB_AZUL = 32;

// Active Buzzer SunFounder V1.0
const int PIN_BUZZER = 33;

// Botón pulsador
const int PIN_BOTON = 14;

// LCD y sensores
LiquidCrystal_I2C lcd(0x27, 16, 2);
Adafruit_BMP085 bmp;
DHT dht(PIN_DHT, TIPO_DHT);

// Variables del DHT22
float humedad = NAN;
float temperatura = NAN;
unsigned long ultimaLecturaDHT = 0;

// Variables de alerta policial
unsigned long ultimoCambioAlerta = 0;
unsigned long ultimoCambioBuzzer = 0;
bool luzRoja = true;
bool buzzerSonando = false;

// Variables del botón y pantalla
int pantalla = 0;
int ultimaLecturaBoton = HIGH;
int estadoBoton = HIGH;
unsigned long ultimoRebote = 0;

float medirDistancia() {
  digitalWrite(PIN_TRIG, LOW);
  delayMicroseconds(2);

  digitalWrite(PIN_TRIG, HIGH);
  delayMicroseconds(10);
  digitalWrite(PIN_TRIG, LOW);

  long duracion = pulseIn(PIN_ECHO, HIGH, 30000);

  if (duracion == 0) {
    return -1;
  }

  return duracion * 0.0343 / 2.0;
}

// RGB de ánodo común:
// HIGH apaga un color y LOW enciende un color.
void apagarRGB() {
  digitalWrite(PIN_RGB_ROJO, HIGH);
  digitalWrite(PIN_RGB_VERDE, HIGH);
  digitalWrite(PIN_RGB_AZUL, HIGH);
}

// El Active Buzzer de este módulo trabaja invertido:
// HIGH = apagado, LOW = suena.
void apagarAlerta() {
  apagarRGB();
  digitalWrite(PIN_BUZZER, HIGH);
}

void actualizarAlerta(float distancia) {
  bool objetoCerca = (distancia >= 0 && distancia <= 5.0);

  if (!objetoCerca) {
    apagarAlerta();
    buzzerSonando = false;
    return;
  }

  // Luces policiales: alterna rojo y azul.
  if (millis() - ultimoCambioAlerta >= 250) {
    ultimoCambioAlerta = millis();
    luzRoja = !luzRoja;
  }

  if (luzRoja) {
    digitalWrite(PIN_RGB_ROJO, LOW);
    digitalWrite(PIN_RGB_VERDE, HIGH);
    digitalWrite(PIN_RGB_AZUL, HIGH);
  } else {
    digitalWrite(PIN_RGB_ROJO, HIGH);
    digitalWrite(PIN_RGB_VERDE, HIGH);
    digitalWrite(PIN_RGB_AZUL, LOW);
  }

  // Beep beep beep: prende y apaga el buzzer cada 140 ms.
  if (millis() - ultimoCambioBuzzer >= 140) {
    ultimoCambioBuzzer = millis();
    buzzerSonando = !buzzerSonando;
  }

  if (buzzerSonando) {
    digitalWrite(PIN_BUZZER, LOW);
  } else {
    digitalWrite(PIN_BUZZER, HIGH);
  }
}

void revisarBoton() {
  int lectura = digitalRead(PIN_BOTON);

  // Detecta cambios y elimina rebotes eléctricos.
  if (lectura != ultimaLecturaBoton) {
    ultimoRebote = millis();
  }

  if (millis() - ultimoRebote > 50) {
    if (lectura != estadoBoton) {
      estadoBoton = lectura;

      // Al presionar, el botón queda en LOW.
      if (estadoBoton == LOW) {
        pantalla++;

        if (pantalla > 2) {
          pantalla = 0;
        }
      }
    }
  }

  ultimaLecturaBoton = lectura;
}

void mostrarPantalla(float distancia, float presion, int porcentajeLuz) {
  lcd.clear();

  if (pantalla == 0) {
    // Pantalla 1: distancia y presión
    lcd.setCursor(0, 0);

    if (distancia < 0) {
      lcd.print("D: Fuera rango");
    } else {
      lcd.print("D:");
      lcd.print(distancia, 1);
      lcd.print(" cm");
    }

    lcd.setCursor(0, 1);

    if (distancia >= 0 && distancia <= 5.0) {
      lcd.print("!! ALERTA !!");
    } else {
      lcd.print("P:");
      lcd.print(presion, 1);
      lcd.print(" hPa");
    }

  } else if (pantalla == 1) {
    // Pantalla 2: temperatura y humedad
    lcd.setCursor(0, 0);

    if (isnan(temperatura) || isnan(humedad)) {
      lcd.print("Error DHT22");
    } else {
      lcd.print("T:");
      lcd.print(temperatura, 1);
      lcd.print(" C");

      lcd.setCursor(0, 1);
      lcd.print("H:");
      lcd.print(humedad, 1);
      lcd.print(" %");
    }

  } else {
    // Pantalla 3: fotoresistor
    lcd.setCursor(0, 0);
    lcd.print("Luz: ");
    lcd.print(porcentajeLuz);
    lcd.print("%");

    lcd.setCursor(0, 1);

    if (porcentajeLuz < 30) {
      lcd.print("Ambiente oscuro");
    } else if (porcentajeLuz < 70) {
      lcd.print("Luz media");
    } else {
      lcd.print("Ambiente claro");
    }
  }
}

void setup() {
  Serial.begin(115200);

  pinMode(PIN_TRIG, OUTPUT);
  pinMode(PIN_ECHO, INPUT);
  pinMode(PIN_FOTORESISTOR, INPUT);

  pinMode(PIN_RGB_ROJO, OUTPUT);
  pinMode(PIN_RGB_VERDE, OUTPUT);
  pinMode(PIN_RGB_AZUL, OUTPUT);

  pinMode(PIN_BUZZER, OUTPUT);

  // El botón usa GND al presionarse.
  pinMode(PIN_BOTON, INPUT_PULLUP);

  apagarAlerta();

  // I2C: LCD y BMP180
  Wire.begin(21, 22);

  lcd.init();
  lcd.backlight();

  dht.begin();

  lcd.setCursor(0, 0);
  lcd.print("Iniciando...");

  if (!bmp.begin(BMP085_STANDARD, &Wire)) {
    lcd.setCursor(0, 1);
    lcd.print("BMP180 no OK");

    while (true) {
      delay(10);
    }
  }

  lcd.clear();
  lcd.print("Sensores listos");
  delay(1500);
}

void loop() {
  float distancia = medirDistancia();
  float presion = bmp.readPressure() / 100.0F;

  int luz = analogRead(PIN_FOTORESISTOR);
  int porcentajeLuz = (luz * 100) / 4095;

  actualizarAlerta(distancia);
  revisarBoton();

  // El DHT22 se lee cada dos segundos.
  if (millis() - ultimaLecturaDHT >= 2000) {
    ultimaLecturaDHT = millis();

    humedad = dht.readHumidity();
    temperatura = dht.readTemperature();

    Serial.print("Temperatura: ");
    Serial.print(temperatura);
    Serial.print(" C | Humedad: ");
    Serial.print(humedad);
    Serial.print(" % | Luz: ");
    Serial.print(porcentajeLuz);
    Serial.println("%");
  }

  mostrarPantalla(distancia, presion, porcentajeLuz);

  delay(100);
}
```

> Porfa, porfa, aquí pasa. Necesito, necesito arreglar la luz, cál la luz. Porque, bueno, el código que detecta la cantidad de luz. Porque, porque cuando no hay luz, o sea, está totalmente, cuando está totalmente destapado, ¿no? O sea, está recibiendo luz, o lo alumbro, lo alumbro, no, no es interna, me dice luz 0%, ambiente oscuro, cuando de hecho está recibiendo el 100% de luz. Cuando le quito la luz y le tapo con las manos, me dice luz 60% luz media. Entonces, está detectándolo al revés. Está detectando que cuando no hay luz, me dice que sí hay luz, y cuando sí hay luz, me dice que no hay luz. Entonces, arréglame la luz, cala luz para que quede bien, ¿ok? Gracias. O sea, pásame el código, todo el código de nuevo. El botón ya está funcionando, por cierto.

Sí: el fotoresistor está conectado bien, pero su salida analógica viene invertida. Se corrige calculando el porcentaje al revés: lectura `0` será `100%` de luz.

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <Adafruit_BMP085.h>
#include <DHT.h>

// HC-SR04
const int PIN_TRIG = 18;
const int PIN_ECHO = 19;

// DHT22
const int PIN_DHT = 27;
#define TIPO_DHT DHT22

// Fotoresistor SunFounder
const int PIN_FOTORESISTOR = 34;

// RGB LED SunFounder: R, G, B y VCC
const int PIN_RGB_ROJO = 25;
const int PIN_RGB_VERDE = 26;
const int PIN_RGB_AZUL = 32;

// Active Buzzer SunFounder V1.0
const int PIN_BUZZER = 33;

// Botón pulsador
const int PIN_BOTON = 14;

// LCD y sensores
LiquidCrystal_I2C lcd(0x27, 16, 2);
Adafruit_BMP085 bmp;
DHT dht(PIN_DHT, TIPO_DHT);

// Variables del DHT22
float humedad = NAN;
float temperatura = NAN;
unsigned long ultimaLecturaDHT = 0;

// Variables de alerta policial
unsigned long ultimoCambioAlerta = 0;
unsigned long ultimoCambioBuzzer = 0;
bool luzRoja = true;
bool buzzerSonando = false;

// Variables del botón y pantalla
int pantalla = 0;
int ultimaLecturaBoton = HIGH;
int estadoBoton = HIGH;
unsigned long ultimoRebote = 0;

float medirDistancia() {
  digitalWrite(PIN_TRIG, LOW);
  delayMicroseconds(2);

  digitalWrite(PIN_TRIG, HIGH);
  delayMicroseconds(10);
  digitalWrite(PIN_TRIG, LOW);

  long duracion = pulseIn(PIN_ECHO, HIGH, 30000);

  if (duracion == 0) {
    return -1;
  }

  return duracion * 0.0343 / 2.0;
}

// RGB de ánodo común: HIGH apaga y LOW enciende.
void apagarRGB() {
  digitalWrite(PIN_RGB_ROJO, HIGH);
  digitalWrite(PIN_RGB_VERDE, HIGH);
  digitalWrite(PIN_RGB_AZUL, HIGH);
}

// Active Buzzer invertido: HIGH apagado, LOW sonando.
void apagarAlerta() {
  apagarRGB();
  digitalWrite(PIN_BUZZER, HIGH);
}

void actualizarAlerta(float distancia) {
  bool objetoCerca = (distancia >= 0 && distancia <= 5.0);

  if (!objetoCerca) {
    apagarAlerta();
    buzzerSonando = false;
    return;
  }

  // Luces policiales rojo-azul.
  if (millis() - ultimoCambioAlerta >= 250) {
    ultimoCambioAlerta = millis();
    luzRoja = !luzRoja;
  }

  if (luzRoja) {
    digitalWrite(PIN_RGB_ROJO, LOW);
    digitalWrite(PIN_RGB_VERDE, HIGH);
    digitalWrite(PIN_RGB_AZUL, HIGH);
  } else {
    digitalWrite(PIN_RGB_ROJO, HIGH);
    digitalWrite(PIN_RGB_VERDE, HIGH);
    digitalWrite(PIN_RGB_AZUL, LOW);
  }

  // Beep beep beep.
  if (millis() - ultimoCambioBuzzer >= 140) {
    ultimoCambioBuzzer = millis();
    buzzerSonando = !buzzerSonando;
  }

  if (buzzerSonando) {
    digitalWrite(PIN_BUZZER, LOW);
  } else {
    digitalWrite(PIN_BUZZER, HIGH);
  }
}

void revisarBoton() {
  int lectura = digitalRead(PIN_BOTON);

  if (lectura != ultimaLecturaBoton) {
    ultimoRebote = millis();
  }

  if (millis() - ultimoRebote > 50) {
    if (lectura != estadoBoton) {
      estadoBoton = lectura;

      if (estadoBoton == LOW) {
        pantalla++;

        if (pantalla > 2) {
          pantalla = 0;
        }
      }
    }
  }

  ultimaLecturaBoton = lectura;
}

void mostrarPantalla(float distancia, float presion, int porcentajeLuz) {
  lcd.clear();

  if (pantalla == 0) {
    // Distancia y presión
    lcd.setCursor(0, 0);

    if (distancia < 0) {
      lcd.print("D: Fuera rango");
    } else {
      lcd.print("D:");
      lcd.print(distancia, 1);
      lcd.print(" cm");
    }

    lcd.setCursor(0, 1);

    if (distancia >= 0 && distancia <= 5.0) {
      lcd.print("!! ALERTA !!");
    } else {
      lcd.print("P:");
      lcd.print(presion, 1);
      lcd.print(" hPa");
    }

  } else if (pantalla == 1) {
    // Temperatura y humedad
    lcd.setCursor(0, 0);

    if (isnan(temperatura) || isnan(humedad)) {
      lcd.print("Error DHT22");
    } else {
      lcd.print("T:");
      lcd.print(temperatura, 1);
      lcd.print(" C");

      lcd.setCursor(0, 1);
      lcd.print("H:");
      lcd.print(humedad, 1);
      lcd.print(" %");
    }

  } else {
    // Luz
    lcd.setCursor(0, 0);
    lcd.print("Luz: ");
    lcd.print(porcentajeLuz);
    lcd.print("%");

    lcd.setCursor(0, 1);

    if (porcentajeLuz < 30) {
      lcd.print("Ambiente oscuro");
    } else if (porcentajeLuz < 70) {
      lcd.print("Luz media");
    } else {
      lcd.print("Ambiente claro");
    }
  }
}

void setup() {
  Serial.begin(115200);

  pinMode(PIN_TRIG, OUTPUT);
  pinMode(PIN_ECHO, INPUT);
  pinMode(PIN_FOTORESISTOR, INPUT);

  pinMode(PIN_RGB_ROJO, OUTPUT);
  pinMode(PIN_RGB_VERDE, OUTPUT);
  pinMode(PIN_RGB_AZUL, OUTPUT);

  pinMode(PIN_BUZZER, OUTPUT);
  pinMode(PIN_BOTON, INPUT_PULLUP);

  apagarAlerta();

  // I2C: LCD y BMP180
  Wire.begin(21, 22);

  lcd.init();
  lcd.backlight();
  dht.begin();

  lcd.setCursor(0, 0);
  lcd.print("Iniciando...");

  if (!bmp.begin(BMP085_STANDARD, &Wire)) {
    lcd.setCursor(0, 1);
    lcd.print("BMP180 no OK");

    while (true) {
      delay(10);
    }
  }

  lcd.clear();
  lcd.print("Sensores listos");
  delay(1500);
}

void loop() {
  float distancia = medirDistancia();
  float presion = bmp.readPressure() / 100.0F;

  // El módulo entrega 0 con mucha luz y valores altos en oscuridad.
  int lecturaLuz = analogRead(PIN_FOTORESISTOR);
  int porcentajeLuz = 100 - ((lecturaLuz * 100) / 4095);

  // Evita valores fuera del rango 0-100.
  porcentajeLuz = constrain(porcentajeLuz, 0, 100);

  actualizarAlerta(distancia);
  revisarBoton();

  // Lectura DHT22 cada dos segundos.
  if (millis() - ultimaLecturaDHT >= 2000) {
    ultimaLecturaDHT = millis();

    humedad = dht.readHumidity();
    temperatura = dht.readTemperature();

    Serial.print("Temperatura: ");
    Serial.print(temperatura);
    Serial.print(" C | Humedad: ");
    Serial.print(humedad);
    Serial.print(" % | Luz: ");
    Serial.print(porcentajeLuz);
    Serial.println("%");
  }

  mostrarPantalla(distancia, presion, porcentajeLuz);

  delay(100);
}
```

Ahora, al iluminar el fotoresistor debe aparecer un porcentaje alto y `Ambiente claro`; al taparlo, un porcentaje bajo y `Ambiente oscuro`.

> Muchacho, ahora quiero arreglar la parte de la luz porque realmente ya no quiero saber cuánta luz, porque resulta que el fotoreceptor está midiendo una unidad que se llama Lux. Lo que yo quiero que haga es, es que me dé la unidad de watts sobre metro cuadrado, porque con eso voy a hacer otras cositas. Entonces quiero que ahí donde me dice luz 90% o etcétera, me diga esa unidad. Y por favor, a la derecha esa unidad local, watts sobre metro cuadrado. Muy bien. Código completo.

Ojo: el fotoresistor no mide lux ni W/m² directamente; entrega una señal analógica. Este código muestra una **estimación de irradiancia**: `0 W/m2` en oscuridad y hasta `1000 W/m2` con máxima luz detectada. Para una medición científica real necesitarías calibrarlo con un medidor de irradiancia.

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <Adafruit_BMP085.h>
#include <DHT.h>

// HC-SR04
const int PIN_TRIG = 18;
const int PIN_ECHO = 19;

// DHT22
const int PIN_DHT = 27;
#define TIPO_DHT DHT22

// Fotoresistor SunFounder
const int PIN_FOTORESISTOR = 34;

// RGB LED SunFounder: R, G, B y VCC
const int PIN_RGB_ROJO = 25;
const int PIN_RGB_VERDE = 26;
const int PIN_RGB_AZUL = 32;

// Active Buzzer SunFounder V1.0
const int PIN_BUZZER = 33;

// Botón pulsador
const int PIN_BOTON = 14;

// Referencia de irradiancia para 100% de luz.
// 1000 W/m2 equivale aproximadamente a sol directo intenso.
const float IRRADIANCIA_MAXIMA = 1000.0;

// LCD y sensores
LiquidCrystal_I2C lcd(0x27, 16, 2);
Adafruit_BMP085 bmp;
DHT dht(PIN_DHT, TIPO_DHT);

// Variables del DHT22
float humedad = NAN;
float temperatura = NAN;
unsigned long ultimaLecturaDHT = 0;

// Variables de alerta policial
unsigned long ultimoCambioAlerta = 0;
unsigned long ultimoCambioBuzzer = 0;
bool luzRoja = true;
bool buzzerSonando = false;

// Variables del botón y pantalla
int pantalla = 0;
int ultimaLecturaBoton = HIGH;
int estadoBoton = HIGH;
unsigned long ultimoRebote = 0;

float medirDistancia() {
  digitalWrite(PIN_TRIG, LOW);
  delayMicroseconds(2);

  digitalWrite(PIN_TRIG, HIGH);
  delayMicroseconds(10);
  digitalWrite(PIN_TRIG, LOW);

  long duracion = pulseIn(PIN_ECHO, HIGH, 30000);

  if (duracion == 0) {
    return -1;
  }

  return duracion * 0.0343 / 2.0;
}

// RGB de ánodo común: HIGH apaga y LOW enciende.
void apagarRGB() {
  digitalWrite(PIN_RGB_ROJO, HIGH);
  digitalWrite(PIN_RGB_VERDE, HIGH);
  digitalWrite(PIN_RGB_AZUL, HIGH);
}

// Active Buzzer invertido: HIGH apagado, LOW sonando.
void apagarAlerta() {
  apagarRGB();
  digitalWrite(PIN_BUZZER, HIGH);
}

void actualizarAlerta(float distancia) {
  bool objetoCerca = (distancia >= 0 && distancia <= 5.0);

  if (!objetoCerca) {
    apagarAlerta();
    buzzerSonando = false;
    return;
  }

  if (millis() - ultimoCambioAlerta >= 250) {
    ultimoCambioAlerta = millis();
    luzRoja = !luzRoja;
  }

  if (luzRoja) {
    digitalWrite(PIN_RGB_ROJO, LOW);
    digitalWrite(PIN_RGB_VERDE, HIGH);
    digitalWrite(PIN_RGB_AZUL, HIGH);
  } else {
    digitalWrite(PIN_RGB_ROJO, HIGH);
    digitalWrite(PIN_RGB_VERDE, HIGH);
    digitalWrite(PIN_RGB_AZUL, LOW);
  }

  // Beep beep beep.
  if (millis() - ultimoCambioBuzzer >= 140) {
    ultimoCambioBuzzer = millis();
    buzzerSonando = !buzzerSonando;
  }

  if (buzzerSonando) {
    digitalWrite(PIN_BUZZER, LOW);
  } else {
    digitalWrite(PIN_BUZZER, HIGH);
  }
}

void revisarBoton() {
  int lectura = digitalRead(PIN_BOTON);

  if (lectura != ultimaLecturaBoton) {
    ultimoRebote = millis();
  }

  if (millis() - ultimoRebote > 50) {
    if (lectura != estadoBoton) {
      estadoBoton = lectura;

      if (estadoBoton == LOW) {
        pantalla++;

        if (pantalla > 2) {
          pantalla = 0;
        }
      }
    }
  }

  ultimaLecturaBoton = lectura;
}

void mostrarPantalla(float distancia, float presion, float irradiancia) {
  lcd.clear();

  if (pantalla == 0) {
    // Distancia y presión
    lcd.setCursor(0, 0);

    if (distancia < 0) {
      lcd.print("D: Fuera rango");
    } else {
      lcd.print("D:");
      lcd.print(distancia, 1);
      lcd.print(" cm");
    }

    lcd.setCursor(0, 1);

    if (distancia >= 0 && distancia <= 5.0) {
      lcd.print("!! ALERTA !!");
    } else {
      lcd.print("P:");
      lcd.print(presion, 1);
      lcd.print(" hPa");
    }

  } else if (pantalla == 1) {
    // Temperatura y humedad
    lcd.setCursor(0, 0);

    if (isnan(temperatura) || isnan(humedad)) {
      lcd.print("Error DHT22");
    } else {
      lcd.print("T:");
      lcd.print(temperatura, 1);
      lcd.print(" C");

      lcd.setCursor(0, 1);
      lcd.print("H:");
      lcd.print(humedad, 1);
      lcd.print(" %");
    }

  } else {
    // Irradiancia estimada
    lcd.setCursor(0, 0);
    lcd.print("I:");
    lcd.print(irradiancia, 1);
    lcd.print(" W/m2");

    lcd.setCursor(0, 1);
    lcd.print("Irrad. estimada");
  }
}

void setup() {
  Serial.begin(115200);

  pinMode(PIN_TRIG, OUTPUT);
  pinMode(PIN_ECHO, INPUT);
  pinMode(PIN_FOTORESISTOR, INPUT);

  pinMode(PIN_RGB_ROJO, OUTPUT);
  pinMode(PIN_RGB_VERDE, OUTPUT);
  pinMode(PIN_RGB_AZUL, OUTPUT);

  pinMode(PIN_BUZZER, OUTPUT);
  pinMode(PIN_BOTON, INPUT_PULLUP);

  apagarAlerta();

  // I2C: LCD y BMP180
  Wire.begin(21, 22);

  lcd.init();
  lcd.backlight();
  dht.begin();

  lcd.setCursor(0, 0);
  lcd.print("Iniciando...");

  if (!bmp.begin(BMP085_STANDARD, &Wire)) {
    lcd.setCursor(0, 1);
    lcd.print("BMP180 no OK");

    while (true) {
      delay(10);
    }
  }

  lcd.clear();
  lcd.print("Sensores listos");
  delay(1500);
}

void loop() {
  float distancia = medirDistancia();
  float presion = bmp.readPressure() / 100.0F;

  // El módulo entrega 0 con mucha luz
  // y valores altos cuando está oscuro.
  int lecturaLuz = analogRead(PIN_FOTORESISTOR);

  int porcentajeLuz = 100 - ((lecturaLuz * 100) / 4095);
  porcentajeLuz = constrain(porcentajeLuz, 0, 100);

  // Convierte el porcentaje a irradiancia estimada.
  float irradiancia = (porcentajeLuz * IRRADIANCIA_MAXIMA) / 100.0;

  actualizarAlerta(distancia);
  revisarBoton();

  if (millis() - ultimaLecturaDHT >= 2000) {
    ultimaLecturaDHT = millis();

    humedad = dht.readHumidity();
    temperatura = dht.readTemperature();

    Serial.print("Temperatura: ");
    Serial.print(temperatura);
    Serial.print(" C | Humedad: ");
    Serial.print(humedad);
    Serial.print(" % | Irradiancia: ");
    Serial.print(irradiancia, 1);
    Serial.println(" W/m2");
  }

  mostrarPantalla(distancia, presion, irradiancia);

  delay(100);
}
```

En la pantalla ahora verás, por ejemplo: `I:900.0 W/m2`.

> Listo. Lo que pasa es que el sensor que tenemos, el fotorresistor, nos arroja una señal analógica. Esa es la que debemos convertir a lux y después convertir de lux a, pues al valor de radiación, que es watt sobre metro cuadrado. Eso es lo que necesitamos hacer, Arnold hazlo.

<details><summary>2 previous messages</summary>

> Voy a ajustar la conversión de forma honesta: para pasar de la señal analógica a lux necesitamos la curva/calibración del fotoresistor; y de lux a W/m² debemos fijar el tipo de luz de referencia. Voy a dejar ambos valores configurables en el código.
>
> <details><summary>Searched the web</summary>
>
> Searched the web for `site:docs.sunfounder.com Photoresistor Module V1.0 analog ESP32 resistance`
>
> Searched the web for `site:docs.sunfounder.com Photoresistor Module V1.0 analog ESP32 resistance`
>
> </details>
</details>

Sí, el flujo correcto es:

```text
Señal analógica → lux calibrados → W/m²
```

Pero falta un dato indispensable: el módulo SunFounder no trae una conversión oficial de `analogRead()` a lux. Solo entrega una señal relativa; de hecho, su salida analógica baja cuando hay más luz. [Documentación de SunFounder](https://docs.sunfounder.com/projects/umsk/es/latest/01_components_basic/11-component_photoresistor.html)

Para darte el código completo con valores reales, necesito que me digas estos dos valores del Monitor Serie:

1. `lecturaLuz` cuando lo tapes completamente con la mano.
2. `lecturaLuz` cuando esté iluminado en el lugar donde vas a medir.

Y también: ¿esa luz será principalmente sol directo? Si sí, usaremos aproximadamente:

```text
W/m² = lux / 93
```

No conviene inventar los lux desde `0–4095`, porque entonces los W/m² serían falsos. Con esas dos lecturas te lo calibro y te paso el código completo.

> lo minimo que le vi es 90 W/m^2 y lo maximo que le vi es 1000 w/m^2 se queda ahi, ese es su limite

Perfecto. Entonces la conversión quedará limitada entre:

```text
Oscuridad: 90 W/m2
Máxima luz: 1000 W/m2
```

Como tu fotoresistor entrega valores pequeños con más luz, el código convierte esa lectura invertida a ese rango.

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <Adafruit_BMP085.h>
#include <DHT.h>

// HC-SR04
const int PIN_TRIG = 18;
const int PIN_ECHO = 19;

// DHT22
const int PIN_DHT = 27;
#define TIPO_DHT DHT22

// Fotoresistor SunFounder
const int PIN_FOTORESISTOR = 34;

// RGB LED SunFounder: R, G, B y VCC
const int PIN_RGB_ROJO = 25;
const int PIN_RGB_VERDE = 26;
const int PIN_RGB_AZUL = 32;

// Active Buzzer SunFounder V1.0
const int PIN_BUZZER = 33;

// Botón pulsador
const int PIN_BOTON = 14;

// Rango de irradiancia configurado
const float IRRADIANCIA_MINIMA = 90.0;    // W/m2
const float IRRADIANCIA_MAXIMA = 1000.0;  // W/m2

// LCD y sensores
LiquidCrystal_I2C lcd(0x27, 16, 2);
Adafruit_BMP085 bmp;
DHT dht(PIN_DHT, TIPO_DHT);

// Variables del DHT22
float humedad = NAN;
float temperatura = NAN;
unsigned long ultimaLecturaDHT = 0;

// Variables de alerta policial
unsigned long ultimoCambioAlerta = 0;
unsigned long ultimoCambioBuzzer = 0;
bool luzRoja = true;
bool buzzerSonando = false;

// Variables del botón y pantalla
int pantalla = 0;
int ultimaLecturaBoton = HIGH;
int estadoBoton = HIGH;
unsigned long ultimoRebote = 0;

float medirDistancia() {
  digitalWrite(PIN_TRIG, LOW);
  delayMicroseconds(2);

  digitalWrite(PIN_TRIG, HIGH);
  delayMicroseconds(10);
  digitalWrite(PIN_TRIG, LOW);

  long duracion = pulseIn(PIN_ECHO, HIGH, 30000);

  if (duracion == 0) {
    return -1;
  }

  return duracion * 0.0343 / 2.0;
}

// RGB de ánodo común: HIGH apaga y LOW enciende.
void apagarRGB() {
  digitalWrite(PIN_RGB_ROJO, HIGH);
  digitalWrite(PIN_RGB_VERDE, HIGH);
  digitalWrite(PIN_RGB_AZUL, HIGH);
}

// Active Buzzer invertido: HIGH apagado, LOW sonando.
void apagarAlerta() {
  apagarRGB();
  digitalWrite(PIN_BUZZER, HIGH);
}

void actualizarAlerta(float distancia) {
  bool objetoCerca = (distancia >= 0 && distancia <= 5.0);

  if (!objetoCerca) {
    apagarAlerta();
    buzzerSonando = false;
    return;
  }

  if (millis() - ultimoCambioAlerta >= 250) {
    ultimoCambioAlerta = millis();
    luzRoja = !luzRoja;
  }

  if (luzRoja) {
    digitalWrite(PIN_RGB_ROJO, LOW);
    digitalWrite(PIN_RGB_VERDE, HIGH);
    digitalWrite(PIN_RGB_AZUL, HIGH);
  } else {
    digitalWrite(PIN_RGB_ROJO, HIGH);
    digitalWrite(PIN_RGB_VERDE, HIGH);
    digitalWrite(PIN_RGB_AZUL, LOW);
  }

  // Beep beep beep.
  if (millis() - ultimoCambioBuzzer >= 140) {
    ultimoCambioBuzzer = millis();
    buzzerSonando = !buzzerSonando;
  }

  if (buzzerSonando) {
    digitalWrite(PIN_BUZZER, LOW);
  } else {
    digitalWrite(PIN_BUZZER, HIGH);
  }
}

void revisarBoton() {
  int lectura = digitalRead(PIN_BOTON);

  if (lectura != ultimaLecturaBoton) {
    ultimoRebote = millis();
  }

  if (millis() - ultimoRebote > 50) {
    if (lectura != estadoBoton) {
      estadoBoton = lectura;

      if (estadoBoton == LOW) {
        pantalla++;

        if (pantalla > 2) {
          pantalla = 0;
        }
      }
    }
  }

  ultimaLecturaBoton = lectura;
}

void mostrarPantalla(float distancia, float presion, float irradiancia) {
  lcd.clear();

  if (pantalla == 0) {
    // Distancia y presión
    lcd.setCursor(0, 0);

    if (distancia < 0) {
      lcd.print("D: Fuera rango");
    } else {
      lcd.print("D:");
      lcd.print(distancia, 1);
      lcd.print(" cm");
    }

    lcd.setCursor(0, 1);

    if (distancia >= 0 && distancia <= 5.0) {
      lcd.print("!! ALERTA !!");
    } else {
      lcd.print("P:");
      lcd.print(presion, 1);
      lcd.print(" hPa");
    }

  } else if (pantalla == 1) {
    // Temperatura y humedad
    lcd.setCursor(0, 0);

    if (isnan(temperatura) || isnan(humedad)) {
      lcd.print("Error DHT22");
    } else {
      lcd.print("T:");
      lcd.print(temperatura, 1);
      lcd.print(" C");

      lcd.setCursor(0, 1);
      lcd.print("H:");
      lcd.print(humedad, 1);
      lcd.print(" %");
    }

  } else {
    // Irradiancia
    lcd.setCursor(0, 0);
    lcd.print("I:");
    lcd.print(irradiancia, 0);
    lcd.print(" W/m2");

    lcd.setCursor(0, 1);
    lcd.print("Radiacion solar");
  }
}

void setup() {
  Serial.begin(115200);

  pinMode(PIN_TRIG, OUTPUT);
  pinMode(PIN_ECHO, INPUT);
  pinMode(PIN_FOTORESISTOR, INPUT);

  pinMode(PIN_RGB_ROJO, OUTPUT);
  pinMode(PIN_RGB_VERDE, OUTPUT);
  pinMode(PIN_RGB_AZUL, OUTPUT);

  pinMode(PIN_BUZZER, OUTPUT);
  pinMode(PIN_BOTON, INPUT_PULLUP);

  apagarAlerta();

  // I2C: LCD y BMP180
  Wire.begin(21, 22);

  lcd.init();
  lcd.backlight();
  dht.begin();

  lcd.setCursor(0, 0);
  lcd.print("Iniciando...");

  if (!bmp.begin(BMP085_STANDARD, &Wire)) {
    lcd.setCursor(0, 1);
    lcd.print("BMP180 no OK");

    while (true) {
      delay(10);
    }
  }

  lcd.clear();
  lcd.print("Sensores listos");
  delay(1500);
}

void loop() {
  float distancia = medirDistancia();
  float presion = bmp.readPressure() / 100.0F;

  // El fotoresistor entrega:
  // 0 con más luz y 4095 con menos luz.
  int lecturaLuz = analogRead(PIN_FOTORESISTOR);

  // Convierte la señal analógica a irradiancia de 90 a 1000 W/m2.
  float proporcionLuz = 1.0 - (lecturaLuz / 4095.0);

  float irradiancia = IRRADIANCIA_MINIMA +
                      (proporcionLuz *
                      (IRRADIANCIA_MAXIMA - IRRADIANCIA_MINIMA));

  irradiancia = constrain(irradiancia,
                          IRRADIANCIA_MINIMA,
                          IRRADIANCIA_MAXIMA);

  actualizarAlerta(distancia);
  revisarBoton();

  if (millis() - ultimaLecturaDHT >= 2000) {
    ultimaLecturaDHT = millis();

    humedad = dht.readHumidity();
    temperatura = dht.readTemperature();

    Serial.print("Temperatura: ");
    Serial.print(temperatura);
    Serial.print(" C | Humedad: ");
    Serial.print(humedad);
    Serial.print(" % | Radiacion: ");
    Serial.print(irradiancia, 1);
    Serial.println(" W/m2");
  }

  mostrarPantalla(distancia, presion, irradiancia);

  delay(100);
}
```

Ahora, con máxima luz, se acercará a `1000 W/m2`; al taparlo, bajará hasta `90 W/m2`.

> recuerdame las paticas de todo que lo desarme y lo voy as volver a armar, donde conecto que con que?

Estas son todas las conexiones del proyecto actual con ESP32:

| Componente | Pata del componente | Va conectada a |
|---|---|---|
| HC-SR04 | `VCC` | `5V` / `VIN` del ESP32 |
| HC-SR04 | `GND` | `GND` |
| HC-SR04 | `TRIG` | GPIO `18` |
| HC-SR04 | `ECHO` | GPIO `19`, **pasando por divisor de voltaje** |
| DHT22 | `VCC` / `+` | `3.3V` |
| DHT22 | `GND` / `-` | `GND` |
| DHT22 | `DATA` / `OUT` | GPIO `27` |
| BMP180 | `VCC` | `3.3V` |
| BMP180 | `GND` | `GND` |
| BMP180 | `SDA` | GPIO `21` |
| BMP180 | `SCL` | GPIO `22` |
| LCD I2C | `GND` | `GND` |
| LCD I2C | `VCC` | `5V` / `VIN` |
| LCD I2C | `SDA` | GPIO `21` |
| LCD I2C | `SCL` | GPIO `22` |
| Fotoresistor SunFounder | `VCC` | `3.3V` |
| Fotoresistor SunFounder | `GND` | `GND` |
| Fotoresistor SunFounder | `AO` / `S` | GPIO `34` |
| Fotoresistor SunFounder | `DO` | Sin conectar |
| RGB LED SunFounder | `VCC` | `3.3V` |
| RGB LED SunFounder | `R` | GPIO `25` |
| RGB LED SunFounder | `G` | GPIO `26` |
| RGB LED SunFounder | `B` | GPIO `32` |
| Active Buzzer SunFounder | `Signal` / `S` | GPIO `33` |
| Active Buzzer SunFounder | `VCC` | `3.3V` |
| Active Buzzer SunFounder | `GND` | `GND` |
| Botón | Una pata de un lado | GPIO `14` |
| Botón | Pata del lado opuesto | `GND` |

Todos los `GND` van unidos: puedes usar una misma línea negativa de la protoboard.

Para el botón de cuatro patas: ponlo atravesando la ranura central de la protoboard. Solo usa una pata de cada lado; las otras dos son duplicadas.

Importante para el HC-SR04: entre `ECHO` y GPIO 19 debe ir un divisor para que no entren 5V al ESP32:

```text
ECHO ── 1 kΩ ──┬── GPIO 19
               |
              2 kΩ
               |
              GND
```

El LCD y el BMP180 comparten I2C: ambos llevan `SDA` a GPIO 21 y `SCL` a GPIO 22.
