# Protocolo de Datos para Sensores IoT – v1.0

## 1. Propósito y Alcance

### 1.1 Objetivo
<!-- Describir brevemente qué resuelve el protocolo y en qué contexto se usa -->

### 1.2 Alcance
<!-- Qué tipos de nodos/sensores y redes cubre (LoRaWAN, WiFi, etc.) -->

### 1.3 Audiencia
<!-- Roles: desarrolladores de firmware, backend, científicos de datos, etc. -->

---

## 2. Terminología y Modelo Conceptual

### 2.1 Definiciones
- **Thing**
- **Sensor**
- **ObservedProperty**
- **Observation**
- **FeatureOfInterest**
- **Payload**
- **Gateway**
- **Backend / Plataforma Nube**

### 2.2 Relación entre Entidades
<!-- Describir brevemente y luego referenciar un diagrama conceptual -->
- Thing – Sensor – ObservedProperty – Observation – FeatureOfInterest

### 2.3 Convenciones Generales
- Unidades
- Zona horaria y formato de tiempo
- Sistema de coordenadas (para geo)
- Notación de tipos (uint8, float32, etc.)

---

## 3. Catálogo de Propiedades Observadas (ObservedProperty)

### 3.1 Estructura de la Tabla de Propiedades

Cada propiedad observada (ObservedProperty) se describe mediante la siguiente estructura estándar:

- **Código**: Identificador numérico único dentro del protocolo.
- **Nombre**: Nombre canónico de la propiedad (snake_case, sin espacios).
- **Descripción**: Explicación breve y precisa de lo que se mide.
- **Dominio**: Categoría temática (ambiental, agua, geo, suelo, etc.).
- **Unidad**: Unidad física empleada (en sistema SI o derivadas cuando sea posible).
- **Tipo de dato**: Tipo de dato utilizado en el payload (p. ej. float32, int16).
- **Rango esperado**: Rango operativo típico en campo, útil para validación.
- **Resolución recomendada**: Resolución mínima que se recomienda reportar.

### 3.2 Tabla Maestra de Propiedades – Versión 1.0

La siguiente tabla define las propiedades estándar para la versión 1.0 del protocolo.  
Los códigos son estaticos y no deben reutilizarse para significados distintos.

### 3.2 Tabla Maestra de Propiedades – Versión 1.0

La siguiente tabla define las propiedades estándar para la versión 1.0 del protocolo.  
Los códigos son estaticos y no deben reutilizarse para significados distintos.

| Código | Nombre          | Descripción                                                     | Dominio      | Unidad      | Tipo wire | Factor escala | Rango wire           | Resolución |
|--------|-----------------|-----------------------------------------------------------------|--------------|-------------|-----------|---------------|----------------------|------------|
| 10     | air_temp        | Temperatura del aire en el entorno inmediato del cultivo       | ambiental    | °C          | int16     | 10            | -400 a 600           | 0.1        |
| 11     | air_hum         | Humedad relativa del aire                                      | ambiental    | %           | uint16    | 10            | 0 a 1000             | 0.1        |
| 12     | air_press       | Presión atmosférica al nivel del sensor                        | ambiental    | hPa         | uint16    | 10            | 8000 a 11000         | 0.1        |
| 13     | solar_rad       | Radiación solar global incidente sobre el sensor               | ambiental    | W/m²        | uint16    | 1             | 0 a 1500             | 1          |
| 20     | water_ph        | Potencial de hidrógeno del agua                                | agua         | pH          | uint16    | 100           | 0 a 1400             | 0.01       |
| 21     | water_turb      | Turbidez del agua                                              | agua         | NTU         | uint16    | 10            | 0 a 40000            | 0.1        |
| 22     | water_cond      | Conductividad eléctrica del agua                               | agua         | µS/cm       | uint16    | 1             | 0 a 5000             | 1          |
| 23     | water_do        | Oxígeno disuelto en el agua                                    | agua         | mg/L        | uint16    | 100           | 0 a 2000             | 0.01       |
| 30     | lat             | Latitud geográfica del sensor/nodo                             | geo          | grados      | int32     | 100000        | -9000000 a 9000000   | ≈1e-5      |
| 31     | lon             | Longitud geográfica del sensor/nodo                            | geo          | grados      | int32     | 100000        | -18000000 a 18000000 | ≈1e-5      |
| 32     | alt             | Altitud sobre el nivel medio del mar                           | geo          | m           | int16     | 10            | -1000 a 60000        | 0.1        |
| 40     | soil_moist      | Humedad volumétrica del suelo en el punto de muestreo          | suelo        | % vol       | uint16    | 10            | 0 a 600              | 0.1        |
| 41     | soil_temp       | Temperatura del suelo a profundidad de referencia              | suelo        | °C          | int16     | 10            | -100 a 500           | 0.1        |
| 50     | co2             | Concentración de CO₂ en el aire                                | aire_calidad | ppm         | uint16    | 1             | 400 a 5000           | 1          |
| 51     | pm25            | Material particulado fino PM2.5                                | aire_calidad | µg/m³       | uint16    | 10            | 0 a 5000             | 0.1        |
| 52     | pm10            | Material particulado grueso PM10                               | aire_calidad | µg/m³       | uint16    | 10            | 0 a 6000             | 0.1        |
| 53     | no2             | Concentración de NO₂ en el aire                                | aire_calidad | µg/m³       | uint16    | 10            | 0 a 5000             | 0.1        |
| 54     | o3              | Concentración de ozono troposférico                            | aire_calidad | µg/m³       | uint16    | 10            | 0 a 3000             | 0.1        |
| 55     | co              | Concentración de CO en el aire                                 | aire_calidad | ppm         | uint16    | 10            | 0 a 3000             | 0.1        |

> **Nota de codificación**: `valor_wire = round(valor_real × factor_escala)`.  
> **Nota de decodificación**: `valor_real = valor_wire / factor_escala`.  
> **Nota de tipos**: `int16` y `int32` admiten negativos; `uint16` solo positivos.


> Nota: los rangos esperados son orientativos y pueden ajustarse según el contexto agroclimático y normativas de calidad de aire.  
> La resolución recomendada indica la granularidad mínima útil; el hardware puede tener resolución interna distinta.


## 4. Identificadores de Entidades

Esta sección define cómo se representan las entidades principales del modelo (Thing, Sensor y FeatureOfInterest) mediante identificadores numéricos en el payload, y cómo se recomienda nombrarlas lógicamente fuera del payload (en configuración, documentación, base de datos).

### 4.1 Identificadores de Thing

Los **Thing** representan nodos, estaciones, boyas, parcelas u otros activos físicos o lógicos a los que se asocian sensores.

#### 4.1.1 Campo `id_thing` en el payload

- **Nombre de campo**: `id_thing`  
- **Tipo de dato**: `uint16`  
- **Rango válido**: `1` a `65535`  
- **Valor reservado**: `0` se reserva para indicar “sin asignar” o mensajes de prueba.

Este identificador es puramente numérico y debe ser único dentro de la red gestionada por el backend.

#### 4.1.2 Reglas de asignación de `id_thing`

- La asignación de `id_thing` la realiza el sistema de gestión (backend, servidor de configuración o script de provisión).  
- Cada Thing activo debe tener un `id_thing` único.  
- Se recomienda mantener un registro maestro (por ejemplo, en una base de datos o archivo de configuración) que relacione:
  - `id_thing` → nombre lógico (por ejemplo: `1 → "estacion_central_finca_x"`).  
- Una vez asignado, el `id_thing` no debe reutilizarse para otro Thing, incluso si el dispositivo original se retira.

#### 4.1.3 Nomenclatura lógica de Things (fuera del payload)

Además del `id_thing` numérico, se recomienda asignar un identificador lógico legible:

- Formato sugerido:  
  - `site/station_code` (por ejemplo: `finca_x/estacion_01`).  
- Este nombre NO viaja en el payload binario; se usa en:
  - Archivos de configuración.  
  - Consolas web y dashboards.  
  - Documentación técnica.  

La relación entre `id_thing` y el nombre lógico debe mantenerse consistente y versionada.

---

### 4.2 Identificadores de Sensor

Los **Sensor** representan los dispositivos o módulos que miden una o más propiedades observadas (ObservedProperty) asociadas a un Thing.

#### 4.2.1 Campo `id_sensor` en el payload

- **Nombre de campo**: `id_sensor`  
- **Tipo de dato**: `uint8`  
- **Rango válido**: `1` a `255`  
- **Valor reservado**: `0` se reserva para indicar “sin asignar” o uso interno.

El `id_sensor` es único dentro de cada Thing, pero puede repetirse entre Things distintos (por ejemplo, muchos Things pueden tener un `id_sensor = 1` que representa el “módulo ambiental”).

#### 4.2.2 Asociación Thing–Sensor

La combinación `(id_thing, id_sensor)` identifica de forma única un sensor físico o lógico en la red.

- Ejemplo:  
  - `(id_thing = 1, id_sensor = 1)` → módulo ambiental de la estación 1.  
  - `(id_thing = 1, id_sensor = 2)` → módulo de calidad de agua de la estación 1.  


#### 4.2.3 Sensores virtuales o agregados

En algunos casos, un “sensor” no es un dispositivo físico único, sino:

- Un cálculo derivado (por ejemplo, índice de confort, índice de estrés hídrico).  
- Un agregado (por ejemplo, promedio horario de `air_temperature`).

Para estos casos:

- Se puede reservar un rango específico de `id_sensor` (por ejemplo, 240–255) para sensores virtuales.  
- En la documentación del sistema se debe indicar claramente:
  - La fórmula o algoritmo que genera la medición.  
  - Las propiedades de entrada que utiliza (por ejemplo, `air_temperature`, `relative_humidity`).  

Estos sensores virtuales pueden o no ser enviados en el payload; también pueden generarse solo en el backend, para esto informar a el dev (Juan Carlos / Kevin).

---

### 4.3 Identificadores de FeatureOfInterest

El **FeatureOfInterest** representa el objeto del mundo real al que se refiere la medición (por ejemplo, un lago especifico, un pozo, un aeroponico).

El uso explícito de identificadores de FeatureOfInterest en el payload es opcional y depende de la complejidad del despliegue de sensores.

#### 4.3.1 Campo opcional `id_feature_of_interest`

Si se utiliza en el payload:

- **Nombre de campo**: `id_feature_of_interest`  
- **Tipo de dato**: `uint16`  
- **Rango válido**: `1` a `65535`  
- **Valor reservado**: `0` indica que la observación se refiere al propio Thing.

Este campo puede enviarse:

- En la cabecera del mensaje (si todas las mediciones se refieren al mismo FeatureOfInterest).  
- O dentro de cada bloque de medición (si diferentes mediciones en el mismo mensaje corresponden a FeatureOfInterest distintos).  

#### 4.3.2 Gestión de FeatureOfInterest en el backend

Aunque `id_feature_of_interest` no se use en el payload, en el backend:

- Se tiene tabla en el con:
  - `id_feature_of_interest`, nombre, descripción, geometría (punto, línea o polígono), metadatos.  
- Hay relaciones establecidas
  - `Thing` → FeatureOfInterest por defecto (por ejemplo, el aeroponico donde está instalado).  
  - Observations → FeatureOfInterest (explícito o derivado).  

Esto facilita análisis espaciales y consultas del tipo:
- “Dame todas las observaciones de calidad de agua para el lago X en la última semana”.

---


## 5. Tipos de Datos y Convenciones Binarias

Esta sección define los tipos de datos primitivos admitidos en el payload, la convención de orden de bytes (endianness) y cómo representar valores especiales como errores o datos faltantes.

### 5.1 Tipos Primitivos

El protocolo utiliza un conjunto reducido de tipos para simplificar las implementaciones en microcontroladores y en el backend:

- **uint8**  
  - Entero sin signo de 8 bits.  
  - Rango: 0 a 255.  
  - Uso típico: códigos de propiedad (`property_code`), identificadores de sensor (`id_sensor`), contadores pequeños, flags compactos.

- **uint16**  
  - Entero sin signo de 16 bits.  
  - Rango: 0 a 65535.  
  - Uso típico: identificadores de Thing (`id_thing`), identificadores de FeatureOfInterest (`id_feature_of_interest`), contadores mayores.

- **uint32**  
  - Entero sin signo de 32 bits.  
  - Rango: 0 a 4 294 967 295.  
  - Uso típico: marcas de tiempo (`timestamp` en segundos desde época UNIX), contadores globales.

- **float32 (IEEE 754)**  
  - Número en coma flotante de 32 bits, estándar IEEE 754.  
  - Uso típico: valores de medición (`result`), como `air_temperature`, `water_ph`, `air_co2`, etc.  

- **Flags / bitfields (uint8 o uint16)**  
  - Se utilizan campos enteros en los que cada bit representa un estado booleano.  
  - Ejemplo (campo `status_flags` de 8 bits):  
    - bit 0: batería baja  
    - bit 1: error de sensor  
    - bit 2: datos saturados  
    - bits 3–7: reservados para uso futuro  

Los tipos utilizados en cada campo se especifican en las tablas de formato de payload (sección 6).

### 5.2 Endianness

Para garantizar interoperabilidad entre diferentes plataformas (microcontroladores, servidores, lenguajes de programación), el protocolo define un orden de bytes único para todos los campos multibyte.

- **Regla global**: todos los enteros (`uint16`, `uint32`) y `float32` se codifican en **big endian**.  
- “Big endian” significa que el byte más significativo (MSB) se envía primero en el payload.

Ejemplos:

- Un `uint16` con valor decimal 1 se codifica como: `0x00 0x01`.  
- Un `uint32` con valor decimal 1 se codifica como: `0x00 0x00 0x00 0x01`.  
- Un `float32` se codifica según IEEE 754 y se envían sus 4 bytes en orden big endian (MSB primero).

> Nota: Todas las implementaciones de firmware deben codificar uint16, uint32 y float32 usando big endian, independientemente del endianness nativo de la plataforma.
    El presente estándar incluye funciones de referencia en C/C++ para dicha conversión, que deben usarse o replicarse funcionalmente en todas las implementaciones de nodos.

### 5.3 Representación de Valores Especiales

Para manejar situaciones de datos faltantes, errores de sensor o valores fuera de rango, se adoptan las siguientes convenciones:

- **NaN (Not a Number) en `float32`**  
  - Cuando una medición no está disponible o resulta inválida después de una validación local, el microcontrolador puede enviar el valor `NaN` en el campo `result` (tipo `float32`).  
  - El interpreta `NaN` como “sin valor válido” y descartar.

- **Valores fuera de rango**  
  - Cada propiedad observada tiene un “rango esperado” documentado en la tabla de propiedades (sección 3).  
  - En el microcontrolador se recomienda:
    - Limitar los valores a un rango físico razonable (por ejemplo, no enviar temperaturas < -50 °C o > 100 °C).  
    - Opcionalmente, usar `NaN` cuando la lectura sea claramente inválida (sensor desconectado, error de lectura).  
  - En el backend:
    - Se marcan como sospechosos los valores fuera del rango esperado y se descartan.

- **Códigos de error explícitos**  
  - Cuando se necesite codificar un estado de error específico, se usan campos de flags (`status_flags`).  
  - `status_flags` (uint8):
    - bit 0 = 1: batería baja.  
    - bit 1 = 1: error de sensor en la última lectura.  
    - bit 2 = 1: valor saturado (límite del sensor).  
    - bits 3–7: reservados.  
  - El significado de cada bit está documentado en una tabla específica de la sección 6 o anexos.

- **Valores reservados en enteros**  
  - En campos identificadores (`id_thing`, `id_sensor`, `id_feature_of_interest`), el valor `0` se reserva para indicar “no asignado” o uso especial y no debe usarse como ID válido.

Estas convenciones deben ser aplicadas tanto en el firmware de los nodos como en la lógica de decodificación y validación en el backend.


## 6. Formato de Payload

El mensaje binario enviado por cada nodo se compone de una **cabecera fija** seguida de un **bloque de mediciones** repetido N veces (donde N = `num_measurements`).  
El objetivo es tener un formato compacto, fácil de codificar en microcontroladores y de decodificar en el backend.

### 6.1 Visión General del Mensaje

Estructura general:

- Cabecera (tamaño fijo).  
- Bloque de medición 1.  
- Bloque de medición 2.  
- …  
- Bloque de medición N.

En pseudoestructura:

```text
Header {
    version          : uint8
    id_thing         : uint16
    id_sensor        : uint8
    timestamp        : uint32
    num_measurements : uint8
}

Measurement[i] {
    property_code    : uint8
    value            : float32
}


## 7. Ejemplos de Mensajes

### 7.1 Ejemplo 1: Temperatura del Aire
- Descripción en texto
- Tabla de campos con valores
- Representación hexadecimal
- Interpretación campo por campo

### 7.2 Ejemplo 2: Temperatura del Aire + pH del Agua
- Descripción en texto
- Tabla de campos con valores
- Representación hexadecimal
- Interpretación campo por campo

### 7.3 Ejemplo 3: Paquete con GPS
- Descripción en texto
- Tabla de campos con valores
- Representación hexadecimal
- Interpretación campo por campo

---

## 8. Reglas de Calidad de Datos

### 8.1 Validación en el Microcontrolador
- Rangos duros por propiedad
- Manejo de errores de sensor

### 8.2 Validación en Backend
- Detección de outliers
- Comprobaciones de consistencia (ej. salto brusco de pH)

### 8.3 Reglas de Reintento / Retransmisión
- Opcional: cómo manejar pérdidas de datos

---

## 9. Versionado y Compatibilidad

### 9.1 Esquema de Versionado
- Semántica de versión_protocolo
- Cambios mayores vs menores

### 9.2 Compatibilidad Hacia Atrás
- Cómo debe comportarse el backend cuando recibe versiones antiguas
- Campos reservados

### 9.3 Plan de Migración
- Cómo introducir nuevas propiedades o cambios sin romper nodos existentes

---

## 10. Anexos

### 10.1 Tabla Completa de Códigos de Propiedades
<!-- Copia completa del catálogo de ObservedProperty -->

### 10.2 Ejemplos de Código
- Pseudocódigo de codificación en C/C++
- Pseudocódigo de decodificación en Python/Node

### 10.3 Diagramas
- Diagrama del flujo de datos (nodo → gateway → nube)
- Diagrama del modelo conceptual (Thing–Sensor–Observation–ObservedProperty–FeatureOfInterest)
