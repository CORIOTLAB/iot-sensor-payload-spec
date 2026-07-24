# Protocolo de Datos para Sensores IoT

Versión 1.0

## Propósito

Este documento define un contrato de datos binario ligero para transmitir observaciones de sensores IoT desde nodos de campo hacia un backend. El objetivo es que firmware y plataforma usen el mismo diccionario de variables, tipos wire, factores de escala y metadatos mínimos, sin ambigüedad.

## Enfoque del documento

Esta versión se centra en el **diccionario de variables** y en los **metadatos mínimos necesarios** para interpretar un payload.

Quedan fuera de este documento:
- Protocolos de red y transporte como LoRaWAN, MQTT o NB-IoT.
- Lógica de negocio, alertas, analítica o modelos.
- Inventario operativo de activos, nomenclaturas locales y gestión de campo.
- OTA, downlinks y configuración remota.

## Modelo mínimo

### ObservedProperty

Una variable medida identificada por un `property_code` estático.

### Observation

Una medición compuesta por:
- `property_code`
- `value_wire`
- `timestamp` del mensaje

### Metadatos

Información de contexto necesaria para interpretar la observación, por ejemplo:
- versión del protocolo,
- origen del dato,
- tipo de dato wire,
- factor de escala,
- estado de calidad.

## Metadatos obligatorios

Los siguientes campos deben estar disponibles ya sea en el payload, en el catálogo estático o en la configuración del backend.

| Campo | Tipo | Origen recomendado | Descripción |
|---|---|---|---|
| `protocol_version` | uint8 | Payload | Versión del protocolo entendida por el nodo. |
| `timestamp` | uint32 o uint64 | Payload o red | Marca de tiempo de la observación. |
| `id_application` | uint16 | Payload | Identificador único del activo o nodo. |
| `id_sensor` | uint8 | Payload | Identificador del sensor dentro de la aplicación. |
| `property_code` | uint8 o uint16 | Payload | Código de la variable observada. |
| `wire_type` | catálogo | Catálogo | Tipo de dato binario usado para codificar el valor. |
| `scale_factor` | catálogo | Catálogo | Factor de escala usado para codificación/decodificación. |
| `unit` | catálogo | Catálogo | Unidad física del valor real. |
| `quality_flag` | uint8 opcional | Payload o backend | Indicador de validez, error o dato estimado. |
| `calibration_version` | string o uint16 opcional | Backend | Versión de calibración aplicada al sensor. |

## Regla de separación

Usar esta regla para decidir si un campo pertenece al diccionario o a los metadatos:

- Si responde **qué significa el valor**, pertenece al diccionario.
- Si responde **de qué dispositivo proviene, cómo se obtuvo o en qué contexto se midió**, pertenece a metadatos.

Ejemplos:
- `air_temp`, `unit`, `scale_factor` → diccionario.
- `id_application`, `id_sensor`, `timestamp`, `quality_flag` → metadatos.
- nombre de finca, ubicación textual, estado del activo → inventario/registro, no payload.

## Reglas de codificación

Codificación:

`value_wire = round(value_real * scale_factor)`

Decodificación:

`value_real = value_wire / scale_factor`

Reglas:
- `int16` e `int32` admiten negativos.
- `uint16` solo admite valores no negativos.
- Valores fuera del rango wire definido para la propiedad deben marcarse como inválidos en el backend.

## Diccionario de variables

| Código | Nombre | Descripción | Dominio | Unidad | Tipo wire | Factor escala | Rango wire | Resolución |
|---|---|---|---|---|---|---|---|---|
| 10 | `air_temp` | Temperatura del aire en el entorno del nodo | ambiental | °C | int16 | 10 | -400 a 600 | 0.1 |
| 11 | `air_hum` | Humedad relativa del aire | ambiental | % | uint16 | 10 | 0 a 1000 | 0.1 |
| 12 | `air_press` | Presión atmosférica al nivel del sensor | ambiental | hPa | uint16 | 10 | 8000 a 11000 | 0.1 |
| 13 | `solar_rad` | Radiación solar global incidente sobre el sensor | ambiental | W/m² | uint16 | 1 | 0 a 1500 | 1 |
| 20 | `water_ph` | Potencial de hidrógeno del agua | agua | pH | uint16 | 100 | 0 a 1400 | 0.01 |
| 21 | `water_turb` | Turbidez del agua | agua | NTU | uint16 | 10 | 0 a 40000 | 0.1 |
| 22 | `water_cond` | Conductividad eléctrica del agua | agua | µS/cm | uint16 | 1 | 0 a 5000 | 1 |
| 23 | `water_do` | Oxígeno disuelto en el agua | agua | mg/L | uint16 | 100 | 0 a 2000 | 0.01 |
| 30 | `lat` | Latitud geográfica del nodo (WGS84) | geo | grados | int32 | 100000 | -9000000 a 9000000 | ≈1e-5 |
| 31 | `lon` | Longitud geográfica del nodo (WGS84) | geo | grados | int32 | 100000 | -18000000 a 18000000 | ≈1e-5 |
| 32 | `alt` | Altitud sobre el nivel medio del mar | geo | m | int16 | 10 | -1000 a 60000 | 0.1 |
| 40 | `soil_moist` | Humedad volumétrica del suelo en el punto de muestreo | suelo | % vol | uint16 | 10 | 0 a 600 | 0.1 |
| 41 | `soil_temp` | Temperatura del suelo a profundidad de referencia | suelo | °C | int16 | 10 | -100 a 500 | 0.1 |
| 50 | `co2` | Concentración de CO₂ en el aire | aire_calidad | ppm | uint16 | 1 | 400 a 5000 | 1 |
| 51 | `pm25` | Material particulado fino PM2.5 | aire_calidad | µg/m³ | uint16 | 10 | 0 a 5000 | 0.1 |
| 52 | `pm10` | Material particulado grueso PM10 | aire_calidad | µg/m³ | uint16 | 10 | 0 a 6000 | 0.1 |
| 53 | `no2` | Concentración de NO₂ en el aire | aire_calidad | µg/m³ | uint16 | 10 | 0 a 5000 | 0.1 |
| 54 | `o3` | Concentración de ozono troposférico | aire_calidad | µg/m³ | uint16 | 10 | 0 a 3000 | 0.1 |
| 55 | `co` | Concentración de CO en el aire | aire_calidad | ppm | uint16 | 10 | 0 a 3000 | 0.1 |


## Versionado

Reglas mínimas:
- Un `property_code` publicado no debe cambiar de significado.
- No se debe cambiar `wire_type`, `unit` ni `scale_factor` de una propiedad existente.
- Si una propiedad requiere cambio incompatible, debe crearse un nuevo código.
- Los códigos retirados no deben reutilizarse.
- La cabecera del mensaje debe indicar `protocol_version`.
