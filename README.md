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
Los códigos son estables y no deben reutilizarse para significados distintos.

### 3.2 Tabla Maestra de Propiedades – Versión 1.0

La siguiente tabla define las propiedades estándar para la versión 1.0 del protocolo.  
Los códigos son estables y no deben reutilizarse para significados distintos.

| Código | Nombre              | Descripción                                                      | Dominio        | Unidad    | Tipo dato | Rango esperado          | Resolución recomendada |
|--------|---------------------|------------------------------------------------------------------|----------------|-----------|----------|-------------------------|------------------------|
| 10     | temperatura_aire    | Temperatura del aire en el entorno inmediato del cultivo        | ambiental      | °C        | float32  | -20 a 60                | 0.1                    |
| 11     | humedad_relativa    | Humedad relativa del aire                                       | ambiental      | %         | float32  | 0 a 100                 | 0.1                    |
| 12     | presion_atmosferica | Presión atmosférica al nivel del sensor                         | ambiental      | hPa       | float32  | 800 a 1100              | 0.1                    |
| 13     | radiacion_solar     | Radiación solar global incidente sobre el sensor                | ambiental      | W/m²      | float32  | 0 a 1500                | 1                      |
| 20     | ph_agua             | Potencial de hidrógeno del agua                                 | agua           | pH        | float32  | 0 a 14                  | 0.01                   |
| 21     | turbidez_agua       | Turbidez del agua                                               | agua           | NTU       | float32  | 0 a 4000                | 0.1–1                  |
| 22     | conductividad_agua  | Conductividad eléctrica del agua                                | agua           | µS/cm     | float32  | 0 a 5000                | 1                      |
| 23     | oxigeno_disuelto    | Concentración de oxígeno disuelto en el agua                    | agua           | mg/L      | float32  | 0 a 20                  | 0.1                    |
| 30     | latitud             | Latitud geográfica del sensor/nodo                              | geo            | grados    | float32  | -90 a 90                | ≈1e-5                  |
| 31     | longitud            | Longitud geográfica del sensor/nodo                             | geo            | grados    | float32  | -180 a 180              | ≈1e-5                  |
| 32     | altitud             | Altitud sobre el nivel medio del mar                            | geo            | m         | float32  | -100 a 6000             | 0.1–1                  |
| 40     | humedad_suelo_vol   | Humedad volumétrica del suelo en el punto de muestreo           | suelo          | % vol     | float32  | 0 a 60                  | 0.1                    |
| 41     | temperatura_suelo   | Temperatura del suelo a una profundidad de referencia           | suelo          | °C        | float32  | -10 a 50                | 0.1                    |
| 50     | co2_aire            | Concentración de dióxido de carbono en el aire                  | aire_calidad   | ppm       | float32  | 400 a 5000              | 1                      |
| 51     | pm2_5               | Concentración de material particulado fino PM2.5                | aire_calidad   | µg/m³     | float32  | 0 a 500                 | 0.1–1                  |
| 52     | pm10                | Concentración de material particulado grueso PM10               | aire_calidad   | µg/m³     | float32  | 0 a 600                 | 0.1–1                  |
| 53     | no2_aire            | Concentración de dióxido de nitrógeno en el aire                | aire_calidad   | ppb o µg/m³ | float32 | 0 a 500                 | 0.1–1                  |
| 54     | o3_aire             | Concentración de ozono troposférico en el aire                  | aire_calidad   | ppb o µg/m³ | float32 | 0 a 300                 | 0.1–1                  |
| 55     | co_aire             | Concentración de monóxido de carbono en el aire                 | aire_calidad   | ppm       | float32  | 0 a 300                 | 0.1–1                  |

> Nota: los rangos esperados son orientativos y pueden ajustarse según el contexto agroclimático y normativas de calidad de aire.  
> La resolución recomendada indica la granularidad mínima útil; el hardware puede tener resolución interna distinta.


## 4. Identificadores de Entidades

### 4.1 Identificadores de Thing
- Espacio de IDs (ej. uint16)
- Reglas de asignación
- Recomendaciones de nomenclatura lógica (fuera del payload)

### 4.2 Identificadores de Sensor
- Espacio de IDs (ej. uint8 por Thing)
- Asociación Thing–Sensor
- Casos especiales (sensores virtuales, agregados)

### 4.3 Identificadores de FeatureOfInterest
- Opcional: cómo se representan cuando se usan explícitamente

---

## 5. Tipos de Datos y Convenciones Binarias

### 5.1 Tipos Primitivos
- uint8
- uint16
- uint32
- float32 (IEEE 754)
- Flags/bitfields

### 5.2 Endianness
- Definición (big endian o little endian)
- Regla global del protocolo

### 5.3 Representación de Valores Especiales
- NaN
- Valores fuera de rango
- Códigos de error

---

## 6. Formato de Payload

### 6.1 Visión General del Mensaje
<!-- Describir la estructura general: cabecera + bloque de mediciones -->

### 6.2 Cabecera del Mensaje
- versión_protocolo
- id_thing
- id_sensor
- timestamp
- num_mediciones

#### 6.2.1 Tabla de Campos de Cabecera
<!-- Tabla con offset, longitud, campo, tipo, descripción -->

### 6.3 Bloque de Medición
- cod_observed_property
- valor

#### 6.3.1 Tabla de Campos de Medición
<!-- Tabla con offset relativo, longitud, campo, tipo, descripción -->

### 6.4 Extensiones Opcionales
- Flags de estado (batería, error de sensor, etc.)
- Campos reservados para futuras versiones

---

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
