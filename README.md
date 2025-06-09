# Práctica 7: Buses de Comunicación III (I2S)

## Descripción
En esta práctica nos enfocamos en el protocolo de comunicación **I2S (Inter-IC Sound)**, utilizado para transferir señales de audio digital entre dispositivos.

Se realizaron **dos ejercicios prácticos** para entender su funcionamiento:
1. Reproducción de audio almacenado en la memoria interna del ESP32.
2. Reproducción de un archivo de audio WAV desde una tarjeta SD externa.

Para facilitar la reproducción de audio de alta calidad, se utilizó el **módulo amplificador MAX98357A**, que convierte señales digitales I2S en señales analógicas para ser emitidas por un altavoz.  
El **ESP32-S3** fue elegido por su compatibilidad nativa con I2S, su capacidad de procesamiento y su flexibilidad en aplicaciones de audio.

---

## Ejercicio Práctico 1: Reproducción desde Memoria Interna

### Código: `main.cpp`
```cpp
#include "AudioGeneratorAAC.h" 
#include "AudioOutputI2S.h" 
#include "AudioFileSourcePROGMEM.h" 
#include "sampleaac.h" 

AudioFileSourcePROGMEM *in; 
AudioGeneratorAAC *aac; 
AudioOutputI2S *out; 

void setup() { 
    Serial.begin(115200); 
    in = new AudioFileSourcePROGMEM(sampleaac, sizeof(sampleaac)); 
    aac = new AudioGeneratorAAC(); 
    out = new AudioOutputI2S(); 
    out->SetGain(0.125); 
    out->SetPinout(40, 39, 38); 
    aac->begin(in, out); 
} 

void loop() { 
    if (aac->isRunning()) { 
        aac->loop(); 
    } else { 
        aac->stop(); 
        Serial.printf("Sound Generator\n"); 
        delay(1000); 
    } 
}
```

### Descripción
Este código reproduce un archivo de audio en formato AAC almacenado en la memoria interna del ESP32.
El archivo de audio está contenido en el archivo sampleaac.h.

El monitor serie mostrará repetidamente:

```Serial Monitor
Sound Generator
Sound Generator
Sound Generator
...
```

---

## Ejercicio Práctico 2: Reproducir un Archivo WAV desde Tarjeta SD

### Código: `main.cpp`

```cpp
#include "Audio.h"
#include "SD.h"
#include "FS.h"

// Definir los pines para ESP32-S3
#define SD_CS         10  
#define SPI_MOSI      11  
#define SPI_MISO      13  
#define SPI_SCK       12  
#define I2S_DOUT      6   
#define I2S_BCLK      5   
#define I2S_LRC       4   

Audio audio; 

void setup() { 
    pinMode(SD_CS, OUTPUT); 
    digitalWrite(SD_CS, HIGH); 
    SPI.begin(SPI_SCK, SPI_MISO, SPI_MOSI); 
    Serial.begin(115200); 
    SD.begin(SD_CS); 
    audio.setPinout(I2S_BCLK, I2S_LRC, I2S_DOUT); 
    audio.setVolume(10); // 0...21 

    audio.connecttoFS(SD, "patito_juan_short.wav"); 
} 
 
void loop() { 
    audio.loop(); 
} 
 
// Opcional: funciones para mostrar información del audio 
void audio_info(const char *info) { 
    Serial.print("info        "); Serial.println(info); 
} 

void audio_id3data(const char *info) { 
    Serial.print("id3data     "); Serial.println(info); 
} 

void audio_eof_mp3(const char *info) { 
    Serial.print("eof_mp3     "); Serial.println(info); 
} 

void audio_showstation(const char *info) { 
    Serial.print("station     "); Serial.println(info); 
} 

void audio_showstreaminfo(const char *info) { 
    Serial.print("streaminfo  "); Serial.println(info); 
} 

void audio_showstreamtitle(const char *info) { 
    Serial.print("streamtitle "); Serial.println(info); 
} 

void audio_bitrate(const char *info) { 
    Serial.print("bitrate     "); Serial.println(info); 
} 

void audio_commercial(const char *info) { 
    Serial.print("commercial  "); Serial.println(info); 
} 

void audio_icyurl(const char *info) { 
    Serial.print("icyurl      "); Serial.println(info); 
} 

void audio_lasthost(const char *info) { 
    Serial.print("lasthost    "); Serial.println(info); 
} 

void audio_eof_speech(const char *info) { 
    Serial.print("eof_speech  "); Serial.println(info); 
} 
```

### Descripcion

Este código permite reproducir un archivo de audio WAV guardado en una tarjeta SD externa.
Se configuran los pines del ESP32 para la comunicación SPI con la tarjeta SD y para la salida I2S hacia el amplificador MAX98357A.

El archivo de audio reproducido es patito_juan_short.wav almacenado en la tarjeta SD.

El loop mantiene la reproducción activa y opcionalmente se pueden mostrar datos del audio en el monitor serie, como:

-Información de bitrate.

-Nombre de la canción.

-Información ID3.

-Datos de transmisión.
