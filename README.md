# Control de 2 OLED SSD1306 Plantilla

Plantilla para controlar **dos pantallas OLED SSD1306 de 128x64 píxeles mediante I2C** utilizando una sola tarjeta de desarrollo.

El proyecto permite mostrar contenido independiente en cada pantalla al mismo tiempo, combinando **imágenes en formato bitmap y texto**. El sketch puede utilizarse como base para crear interfaces, fichas informativas, menús, indicadores, animaciones simples, presentaciones de datos o cualquier proyecto que necesite dos pantallas OLED independientes.

El ejemplo está desarrollado para una **ESP32-S3 Super Mini**, aunque puede adaptarse a otras tarjetas compatibles con Arduino.

---

## Características

- Control de **2 pantallas OLED SSD1306 128x64** simultáneamente.
- Comunicación mediante **I2C**.
- Una pantalla utiliza la dirección `0x3C`.
- La segunda pantalla utiliza la dirección `0x3D`.
- Ambas pantallas comparten las mismas líneas SDA y SCL.
- Permite mostrar contenido diferente en cada OLED.
- Visualización de imágenes mediante **bitmaps**.
- Bitmaps almacenados en memoria mediante `PROGMEM`.
- Visualización de texto y datos.
- Cambio automático de contenido mediante intervalos de tiempo.
- Posibilidad de intercambiar el contenido entre ambas pantallas.
- Diseñado para servir como **plantilla modificable para otros proyectos**.

---

## Hardware utilizado

- ESP32-S3 Super Mini
- 2 × OLED SSD1306 128x64 I2C
  (NOTA: la dirección o Address de las OLED se modifica por hardware, es necesario cambiar de lugar una resistencia para configurar
   la dirección 0x3C o 0x3D).
- Cables de conexión
- Fuente de alimentación mediante USB

Las dos pantallas deben tener **direcciones I2C diferentes**.

En este proyecto se utilizan:

```text
OLED A → 0x3C
OLED B → 0x3D
```

---

## Conexiones

Las dos pantallas comparten el mismo bus I2C.

| ESP32-S3 Super Mini | OLED A | OLED B |
|---|---|---|
| 3.3V | VCC | VCC |
| GND | GND | GND |
| GPIO 8 | SDA | SDA |
| GPIO 9 | SCL | SCL |

El sketch utiliza:

```cpp
#define SDA_PIN 8
#define SCL_PIN 9
```

y el bus I2C se inicializa mediante:

```cpp
Wire.begin(SDA_PIN, SCL_PIN);
```

---

## Direcciones I2C

Cada pantalla necesita una dirección diferente para poder controlarse independientemente dentro del mismo bus.

El proyecto utiliza:

```cpp
#define DIRECCION_OLED_A 0x3C
#define DIRECCION_OLED_B 0x3D
```

De esta manera, el microcontrolador puede enviar información específicamente a cada OLED aunque ambas compartan los mismos cables SDA y SCL.

Si tienes dudas sobre las direcciones de tus pantallas, puedes utilizar un **I2C Scanner** para comprobarlas antes de ejecutar el proyecto.

---

## Librerías necesarias

El proyecto utiliza las siguientes librerías:

```cpp
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
```

### Wire

Gestiona la comunicación I2C.

### Adafruit GFX

Proporciona las funciones gráficas necesarias para dibujar texto, imágenes y otros elementos.

### Adafruit SSD1306

Permite controlar las pantallas OLED basadas en el controlador SSD1306.

Las librerías de Adafruit pueden instalarse desde el **Library Manager del Arduino IDE**.

---

## Configuración de las pantallas

Las dos pantallas tienen una resolución de:

```text
128 × 64 píxeles
```

En el sketch se define mediante:

```cpp
#define ANCHO_PANTALLA 128
#define ALTO_PANTALLA 64
```

Posteriormente se crea un objeto independiente para cada OLED:

```cpp
Adafruit_SSD1306 pantallaA(
  ANCHO_PANTALLA,
  ALTO_PANTALLA,
  &Wire,
  OLED_RESET
);

Adafruit_SSD1306 pantallaB(
  ANCHO_PANTALLA,
  ALTO_PANTALLA,
  &Wire,
  OLED_RESET
);
```

Esto permite utilizar comandos independientes como:

```cpp
pantallaA.clearDisplay();
pantallaA.display();
```

y:

```cpp
pantallaB.clearDisplay();
pantallaB.display();
```

---

## Bitmaps

Las imágenes utilizadas por el proyecto se almacenan como arreglos de bytes.

Ejemplo:

```cpp
const unsigned char PROGMEM imagenA[] = {

  // Pegar aquí el bitmap

};
```

El uso de:

```cpp
PROGMEM
```

permite almacenar los datos de las imágenes en la memoria Flash del microcontrolador.

Para una pantalla completa, las imágenes deben prepararse con una resolución de:

```text
128 × 64 píxeles
```

y convertirse a un bitmap monocromático compatible con `Adafruit_GFX`.

---

## Mostrar una imagen

Una imagen puede enviarse a la primera OLED utilizando:

```cpp
pantallaA.clearDisplay();

pantallaA.drawBitmap(
  0,
  0,
  imagenA,
  128,
  64,
  SSD1306_WHITE
);

pantallaA.display();
```

Para mostrar otra imagen en la segunda OLED:

```cpp
pantallaB.clearDisplay();

pantallaB.drawBitmap(
  0,
  0,
  imagenB,
  128,
  64,
  SSD1306_WHITE
);

pantallaB.display();
```

Cada pantalla mantiene su propio contenido.

---

## Mostrar texto

También es posible utilizar una de las pantallas para mostrar información mediante texto.

Ejemplo:

```cpp
pantallaB.clearDisplay();

pantallaB.setTextSize(1);
pantallaB.setTextColor(SSD1306_WHITE);
pantallaB.setCursor(0, 0);

pantallaB.println("Nombre: Ejemplo");
pantallaB.println("Dato 1: 123");
pantallaB.println("Dato 2: 456");
pantallaB.println("Estado: OK");

pantallaB.display();
```

Esto permite utilizar una OLED para mostrar una imagen y la otra para mostrar información relacionada con ella.

---

## Funcionamiento del proyecto

El sketch funciona como una presentación automática de diferentes elementos.

Durante cada etapa:

```text
┌─────────────────┐       ┌─────────────────┐
│                 │       │                 │
│     OLED A      │       │     OLED B      │
│                 │       │                 │
│     BITMAP      │       │      DATOS      │
│                 │       │                 │
└─────────────────┘       └─────────────────┘
       0x3C                      0x3D
```

Se muestra una imagen en una pantalla y simultáneamente información relacionada en la otra.

Después de un intervalo determinado, ambas pantallas cambian de contenido:

```text
Elemento 1
   ↓
espera
   ↓
Elemento 2
   ↓
espera
   ↓
Elemento 3
   ↓
espera
   ↓
...
```

Al terminar la secuencia, el ciclo vuelve a comenzar.

---

## Tiempo entre elementos

El tiempo durante el cual permanece visible cada conjunto de información puede controlarse mediante:

```cpp
delay(5000);
```

`5000` corresponde a:

```text
5000 ms = 5 segundos
```

Por ejemplo:

```cpp
delay(3000);
```

mantendría el contenido durante aproximadamente 3 segundos.

Este valor puede modificarse dependiendo de las necesidades del proyecto.

---

## Intercambio de pantallas

La plantilla también permite cambiar el papel de las pantallas durante la secuencia.

Por ejemplo:

```text
Primera parte

OLED A → Imagen
OLED B → Información
```

y posteriormente:

```text
Segunda parte

OLED A → Información
OLED B → Imagen
```

Esto permite crear cambios visuales dentro de una presentación sin modificar las conexiones físicas de las OLED.

---

## Personalización

El objetivo principal de este proyecto es funcionar como una **plantilla**.

Puedes reemplazar los bitmaps incluidos por tus propias imágenes y modificar completamente los textos mostrados.

Por ejemplo, el sistema puede adaptarse para mostrar:

- Personajes o avatares y sus estadísticas
- Productos y especificaciones
- Sensores y mediciones
- Iconos y estados del sistema
- Información de dispositivos
- Menús
- Datos de videojuegos
- Fichas informativas
- Animaciones simples
- Interfaces para proyectos electrónicos
- Información obtenida mediante sensores
- Proyectos educativos

La lógica de las dos pantallas puede conservarse mientras se reemplaza únicamente el contenido.

---

## Estructura básica

La idea general del programa es:

```cpp
// Limpiar OLED A
pantallaA.clearDisplay();

// Dibujar imagen
pantallaA.drawBitmap(...);

// Actualizar OLED A
pantallaA.display();


// Limpiar OLED B
pantallaB.clearDisplay();

// Escribir información
pantallaB.setCursor(...);
pantallaB.println(...);

// Actualizar OLED B
pantallaB.display();


// Mantener contenido
delay(5000);
```

Después se repite el mismo procedimiento con el siguiente conjunto de información.

---

## Adaptación a otras placas

El proyecto fue desarrollado utilizando una:

```text
ESP32-S3 Super Mini
```

con:

```text
SDA → GPIO 8
SCL → GPIO 9
```

En otras placas ESP32 pueden utilizarse otros GPIO compatibles con I2C modificando:

```cpp
#define SDA_PIN 8
#define SCL_PIN 9
```

---

## Arduino Uno

La lógica del proyecto también puede adaptarse a un Arduino Uno, pero hay que tener en cuenta las diferencias de hardware.

En Arduino Uno el bus I2C utiliza:

```text
SDA → A4
SCL → A5
```

En lugar de:

```cpp
Wire.begin(SDA_PIN, SCL_PIN);
```

se utiliza:

```cpp
Wire.begin();
```

### Importante

Una OLED SSD1306 de 128×64 necesita un framebuffer de aproximadamente **1024 bytes**.

Dos instancias de `Adafruit_SSD1306` pueden necesitar aproximadamente:

```text
1024 + 1024 = 2048 bytes
```

El Arduino Uno dispone de solamente **2 KB de SRAM**, por lo que controlar dos buffers completos simultáneamente con esta misma estructura puede provocar problemas de memoria.

Por este motivo, para utilizar esta plantilla sin modificaciones importantes es recomendable una placa con mayor cantidad de RAM, como ESP32 o ESP32-S3.

---

## Organización recomendada

Si deseas utilizar este proyecto como base para uno propio, puedes seguir esta estructura:

```text
1. Configurar SDA y SCL
2. Configurar las direcciones I2C
3. Crear los objetos de ambas OLED
4. Añadir los bitmaps
5. Inicializar el bus I2C
6. Inicializar las dos pantallas
7. Mostrar imagen en una OLED
8. Mostrar información en la otra OLED
9. Esperar el tiempo establecido
10. Cambiar al siguiente contenido
11. Repetir el ciclo
```

---

## Notas

- Ambas OLED pueden compartir SDA y SCL porque utilizan direcciones I2C diferentes.
- Comprueba que una pantalla responda en `0x3C` y la otra en `0x3D`.
- Los bitmaps deben tener el formato adecuado para `Adafruit_GFX`.
- Utiliza `clearDisplay()` antes de preparar el siguiente contenido.
- Utiliza `display()` para transferir el framebuffer a la pantalla correspondiente.
- Si una OLED no aparece correctamente, comprueba primero las direcciones mediante un I2C Scanner.
- Los pines SDA y SCL pueden cambiar dependiendo de la placa utilizada.

---

## Autor

**Uriel Engine**

Proyecto creado como plantilla y ejemplo educativo para el manejo simultáneo de múltiples pantallas OLED mediante I2C.

---

## Licencia

El código de esta plantilla puede utilizarse, modificarse y adaptarse para proyectos personales y educativos.

Los gráficos, imágenes o datos que cada usuario decida añadir al proyecto son responsabilidad de sus respectivos autores y propietarios.
