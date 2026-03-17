# Protocolo de Datos para Sensores IoT – v1.0

## 1. Propósito y Alcance

### 1.1 Objetivo

Este documento define un protocolo binario ligero para la transmisión estandarizada de datos de sensores IoT desde nodos de campo hacia una plataforma en la nube.

El protocolo establece:

- Un catálogo fijo de variables (ObservedProperty) con nombres, unidades, tipos de dato y factores de escala.
- Una estructura de mensaje binario (payload) compacta y determinista.
- Funciones de referencia para codificación en nodos (C/C++) y decodificación en backend (Python).

El objetivo es que cualquier programador que trabaje en el sistema —tanto en firmware como en backend— use exactamente el mismo contrato de datos, sin ambigüedades ni interpretaciones locales.

---

### 1.2 Alcance

El protocolo aplica a:

- **Nodos IoT** basados en microcontroladores de baja potencia (por ejemplo, ESP32, STM32, Arduino, etc.).
- **Redes de transmisión** con ancho de banda limitado o costoso, incluyendo:
  - LoRaWAN (principal caso de uso).
  - WiFi, NB-IoT, Sigfox u otros enlaces LPWAN.
- **Variables de monitoreo** de los dominios: ambiental, agua, geolocalización, suelo y calidad del aire (ver sección 3).
- **Plataformas backend** que reciban, decodifiquen y almacenen los mensajes (por ejemplo, TTN, ChirpStack, AWS IoT, Azure IoT Hub, etc.).

Fuera del alcance de este documento:

- Protocolos de red (LoRaWAN, MQTT, etc.); este protocolo define solo el contenido del payload.
- Lógica de negocio, alertas o modelos de análisis sobre los datos.
- Configuración remota de nodos (OTA, downlinks).

---

## 2. Terminología y Modelo Conceptual

### 2.1 Definiciones

- **Application**: Activo físico o lógico que agrupa uno o más sensores bajo una misma identidad en la red (por ejemplo, estación ambiental, boya de calidad de agua, nodo IoT en parcela). Se identifica en el payload mediante `id_application`.

- **Sensor**: Dispositivo físico o módulo que realiza mediciones de una o más propiedades observadas (por ejemplo, módulo de temperatura/humedad DHT22, sonda de pH, módulo GPS). Se identifica en el payload mediante `id_sensor`.

- **ObservedProperty**: Propiedad del mundo real que se mide (por ejemplo, `air_temp`, `water_ph`, `lat`). Cada propiedad tiene un código numérico único (`property_code`), unidad, tipo de dato wire y factor de escala definidos en el catálogo (sección 3).

- **Observation**: Medición concreta de una ObservedProperty en un momento dado. Está compuesta por: `property_code`, `value` (entero escalado), y el `timestamp` de la cabecera del mensaje.

- **Payload**: Mensaje binario generado por un nodo siguiendo este protocolo. Contiene una cabecera y uno o más bloques de medición.

- **Gateway**: Dispositivo o servicio que recibe los payloads desde los nodos (por ejemplo, gateway LoRaWAN) y los reenvía al backend sin modificar el contenido binario.

- **Backend / Plataforma nube**: Sistema que recibe los payloads del gateway, los decodifica siguiendo este protocolo y almacena las observaciones resultantes para su análisis y visualización.

---

## 3. Catálogo de Propiedades Observadas (ObservedProperty)

### 3.1 Estructura de la Tabla de Propiedades

Cada propiedad observada se describe con los siguientes campos:

- **Código**: Identificador numérico único y estático dentro del protocolo. Es el valor que viaja en el payload como `property_code`.
- **Nombre**: Nombre canónico en snake_case, en inglés. Es el nombre que se usa en código, base de datos y APIs.
- **Descripción**: Explicación breve de lo que se mide.
- **Dominio**: Categoría temática de la propiedad (`ambiental`, `agua`, `geo`, `suelo`, `aire_calidad`).
- **Unidad**: Unidad física del valor real (después de aplicar el factor de escala en decodificación).
- **Tipo wire**: Tipo de dato entero usado en el payload binario (`int16`, `uint16`, `int32`).
- **Factor escala**: Número por el que se multiplica el valor real antes de enviarlo. En decodificación se divide.
- **Rango wire**: Rango válido del valor ya escalado (el que viaja en el payload).
- **Resolución**: Granularidad mínima útil del valor real.

---

### 3.2 Tabla Maestra de Propiedades – Versión 1.0

Los códigos son estáticos y no deben reutilizarse para significados distintos.

> **Codificación**: `valor_wire = round(valor_real × factor_escala)`
> **Decodificación**: `valor_real = valor_wire / factor_escala`
> `int16` e `int32` admiten negativos. `uint16` solo admite positivos.

| Código | Nombre       | Descripción                                              | Dominio      | Unidad   | Tipo wire | Factor escala | Rango wire           | Resolución |
|--------|--------------|----------------------------------------------------------|--------------|----------|-----------|---------------|----------------------|------------|
| 10     | air_temp     | Temperatura del aire en el entorno del nodo              | ambiental    | °C       | int16     | 10            | -400 a 600           | 0.1        |
| 11     | air_hum      | Humedad relativa del aire                                | ambiental    | %        | uint16    | 10            | 0 a 1000             | 0.1        |
| 12     | air_press    | Presión atmosférica al nivel del sensor                  | ambiental    | hPa      | uint16    | 10            | 8000 a 11000         | 0.1        |
| 13     | solar_rad    | Radiación solar global incidente sobre el sensor         | ambiental    | W/m²     | uint16    | 1             | 0 a 1500             | 1          |
| 20     | water_ph     | Potencial de hidrógeno del agua                          | agua         | pH       | uint16    | 100           | 0 a 1400             | 0.01       |
| 21     | water_turb   | Turbidez del agua                                        | agua         | NTU      | uint16    | 10            | 0 a 40000            | 0.1        |
| 22     | water_cond   | Conductividad eléctrica del agua                         | agua         | µS/cm    | uint16    | 1             | 0 a 5000             | 1          |
| 23     | water_do     | Oxígeno disuelto en el agua                              | agua         | mg/L     | uint16    | 100           | 0 a 2000             | 0.01       |
| 30     | lat          | Latitud geográfica del nodo (WGS84)                      | geo          | grados   | int32     | 100000        | -9000000 a 9000000   | ≈1e-5      |
| 31     | lon          | Longitud geográfica del nodo (WGS84)                     | geo          | grados   | int32     | 100000        | -18000000 a 18000000 | ≈1e-5      |
| 32     | alt          | Altitud sobre el nivel medio del mar                     | geo          | m        | int16     | 10            | -1000 a 60000        | 0.1        |
| 40     | soil_moist   | Humedad volumétrica del suelo en el punto de muestreo    | suelo        | % vol    | uint16    | 10            | 0 a 600              | 0.1        |
| 41     | soil_temp    | Temperatura del suelo a profundidad de referencia        | suelo        | °C       | int16     | 10            | -100 a 500           | 0.1        |
| 50     | co2          | Concentración de CO₂ en el aire                          | aire_calidad | ppm      | uint16    | 1             | 400 a 5000           | 1          |
| 51     | pm25         | Material particulado fino PM2.5                          | aire_calidad | µg/m³    | uint16    | 10            | 0 a 5000             | 0.1        |
| 52     | pm10         | Material particulado grueso PM10                         | aire_calidad | µg/m³    | uint16    | 10            | 0 a 6000             | 0.1        |
| 53     | no2          | Concentración de NO₂ en el aire                          | aire_calidad | µg/m³    | uint16    | 10            | 0 a 5000             | 0.1        |
| 54     | o3           | Concentración de ozono troposférico                      | aire_calidad | µg/m³    | uint16    | 10            | 0 a 3000             | 0.1        |
| 55     | co           | Concentración de CO en el aire                           | aire_calidad | ppm      | uint16    | 10            | 0 a 3000             | 0.1        |

> **Nota**: los rangos wire son orientativos. Valores fuera de rango deben tratarse como inválidos en el backend (ver sección 5.3).

---

### 3.3 Reglas para la Evolución del Catálogo

1. **Añadir nuevas propiedades**
   - Asignar un código numérico no usado previamente.
   - El nombre debe ser único, en snake_case y en inglés.
   - Completar todos los campos de la tabla (descripción, dominio, unidad, tipo wire, factor escala, rango wire, resolución).
   - Incrementar la versión menor del protocolo (por ejemplo, v1.0 → v1.1).

2. **No reutilización de códigos**
   - Un código retirado no puede asignarse a otra propiedad.
   - Las propiedades retiradas se marcan como `DEPRECATED desde vX.Y` y permanecen en el anexo histórico.

3. **No modificar propiedades existentes**
   - No se puede cambiar el tipo wire, factor escala ni unidad de una propiedad ya publicada.
   - Si se requiere un cambio de ese tipo, se crea una nueva propiedad con nuevo código y se depreca la anterior.

4. **Control de versiones**
   - Toda modificación del catálogo incrementa al menos la versión menor del protocolo.
   - El campo `version` en la cabecera del payload indica qué versión del catálogo entiende el nodo.

---

## 4. Identificadores de Entidades

Esta sección define cómo se representan las entidades principales del modelo (Application y Sensor) mediante identificadores numéricos en el payload, y cómo se recomienda nombrarlas lógicamente fuera del payload (en configuración, documentación y base de datos).

---

### 4.1 Identificadores de Application

Las **Applications** representan nodos, estaciones, boyas u otros activos físicos o lógicos a los que se asocian sensores.

#### 4.1.1 Campo `id_application` en el payload

- **Nombre de campo**: `id_application`
- **Tipo de dato**: `uint16`
- **Rango válido**: `1` a `65535`
- **Valor reservado**: `0` se reserva para mensajes de prueba o nodos sin asignar.

Este identificador es puramente numérico y debe ser único dentro de la red gestionada por el backend.

#### 4.1.2 Reglas de asignación

- La asignación de `id_application` la realiza el sistema de gestión (backend o script de provisión), no el nodo.
- Cada Application activa debe tener un `id_application` único.
- Una vez asignado, el `id_application` no debe reutilizarse para otra Application, incluso si el dispositivo original se retira.
- Se debe mantener un registro maestro que relacione:

| id_application | Nombre lógico           | Ubicación        | Estado   |
|----------------|-------------------------|------------------|----------|
| 1              | estacion_finca_x_lote_a | Finca X, Lote A  | activo   |
| 2              | boya_lago_principal     | Lago Principal   | activo   |
| 3              | nodo_aeroponico_01      | Invernadero 1    | retirado |

#### 4.1.3 Nomenclatura lógica (fuera del payload)

El nombre lógico de una Application NO viaja en el payload. Se usa en:

- Archivos de configuración del nodo.
- Consolas web y dashboards.
- Documentación técnica y registros de campo.

Formato sugerido: `site_code/node_code`
Ejemplo: `finca_x/estacion_01`, `lago_principal/boya_02`

---

### 4.2 Identificadores de Sensor

Los **Sensors** representan los dispositivos o módulos físicos que miden una o más propiedades observadas dentro de una Application.

#### 4.2.1 Campo `id_sensor` en el payload

- **Nombre de campo**: `id_sensor`
- **Tipo de dato**: `uint8`
- **Rango válido**: `1` a `239`
- **Rango reservado**: `240` a `255` reservado para sensores virtuales (ver 4.2.3).
- **Valor reservado**: `0` se reserva para indicar "sin asignar" o uso interno.

El `id_sensor` es único dentro de cada Application, pero puede repetirse entre Applications distintas.

#### 4.2.2 Asociación Application–Sensor

La combinación `(id_application, id_sensor)` identifica de forma única un sensor en la red.

| id_application | id_sensor | Tipo de sensor         | ObservedProperties              |
|----------------|-----------|------------------------|---------------------------------|
| 1              | 1         | Módulo ambiental DHT22 | air_temp, air_hum               |
| 1              | 2         | Sonda pH Atlas         | water_ph                        |
| 1              | 3         | Módulo GPS NEO-6M      | lat, lon, alt                   |
| 2              | 1         | Módulo calidad agua    | water_ph, water_cond, water_do  |

Se recomienda mantener esta tabla de configuración en el backend para relacionar sensores físicos con las propiedades que miden.

#### 4.2.3 Sensores virtuales o agregados

Un sensor virtual es aquel cuyo valor no proviene directamente de hardware, sino de:

- Un cálculo derivado (por ejemplo, índice de calor a partir de `air_temp` y `air_hum`).
- Un agregado (por ejemplo, promedio horario de `air_temp`).
- Una estimación por modelo (por ejemplo, evapotranspiración).

Reglas para sensores virtuales:

- Usar `id_sensor` en el rango `240–255`.
- Documentar explícitamente la fórmula o algoritmo que genera el valor.
- Indicar las ObservedProperties de entrada que utiliza.
- Los sensores virtuales pueden generarse en el nodo o exclusivamente en el backend; en ambos casos deben registrarse en la tabla de configuración.

| id_sensor | Nombre          | Tipo     | Fórmula / Origen                              |
|-----------|-----------------|----------|-----------------------------------------------|
| 240       | heat_index      | derivado | f(air_temp, air_hum) – fórmula Rothfusz       |
| 241       | air_temp_avg_1h | agregado | promedio de air_temp en ventana de 60 minutos |

---
