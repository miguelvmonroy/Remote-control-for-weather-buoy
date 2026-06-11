# Remote-control-for-weather-buoy


# MSP430 - Monitor y Control de Señales Boya / Hallow

## Descripción

Este proyecto implementa un sistema de monitoreo y control utilizando un microcontrolador MSP430. El programa supervisa el estado de dos señales externas denominadas **Boya** y **Hallow**, reflejando su condición mediante salidas digitales y generando pulsos alternados cuando se detecta la activación del botón de la boya.

El firmware trabaja mediante un ciclo principal continuo (*polling*), sin utilizar interrupciones ni temporizadores.

---

## Características

* Configuración del reloj del MSP430 a **1 MHz**.
* Lectura del estado de:

  * LED Hallow.
  * LED Boya.
  * Botón de encendido/apagado de la Boya.
* Generación de pulsos alternados A y B.
* Salidas de reloj disponibles para medición:

  * MCLK
  * SMCLK
  * ACLK
* Implementación simple basada en sondeo continuo (*polling*).

---

## Conexiones de Hardware

### Puerto 1

| Pin  | Función            | Dirección |
| ---- | ------------------ | --------- |
| P1.0 | Estado LED Boya    | Salida    |
| P1.1 | Botón Boya         | Entrada   |
| P1.2 | Estado LED Hallow  | Salida    |
| P1.5 | Lectura LED Hallow | Entrada   |

### Puerto 2

| Pin  | Función |
| ---- | ------- |
| P2.0 | MCLK    |
| P2.2 | ACLK    |

### Puerto 5

| Pin  | Función          | Dirección |
| ---- | ---------------- | --------- |
| P5.2 | Lectura LED Boya | Entrada   |

### Puerto 7

| Pin  | Función | Dirección |
| ---- | ------- | --------- |
| P7.4 | Pulso B | Salida    |
| P7.5 | Pulso A | Salida    |
| P7.7 | SMCLK   | Salida    |

---

## Funcionamiento

### 1. Monitoreo del LED Hallow

El estado de la entrada **P1.5** es leído continuamente.

* Si la entrada está activa:

  * Se activa la salida **P1.2**.
* Si la entrada está inactiva:

  * Se desactiva la salida **P1.2**.

---

### 2. Generación de Pulsos A y B

El estado del botón conectado a **P1.1** determina la generación de pulsos.

#### Botón desactivado

* Pulso A = OFF
* Pulso B = OFF

#### Botón activado

Se alternan las salidas:

* Pulso A ON / Pulso B OFF
* Pulso A OFF / Pulso B ON

La variable global `Bandera` controla la alternancia entre ambos estados.

---

### 3. Monitoreo del LED Boya

El estado de la entrada **P5.2** es monitoreado continuamente.

* Si la entrada está activa:

  * Se activa la salida **P1.0**.
* Si la entrada está inactiva:

  * Se desactiva la salida **P1.0**.

---

## Configuración del Reloj

El proyecto incluye dos funciones de configuración:

### Config_Reloj()

Configura el sistema a aproximadamente **25 MHz** utilizando el DCO.

### Config_Reloj_1M()

Configura el sistema a aproximadamente **1 MHz**.

Actualmente el programa utiliza:

```c
Config_Reloj_1M();
```

durante la inicialización.

---

## Estructura del Software

```text
main()
│
├── Config_Reloj_1M()
│
├── Config_Puertos()
│
└── while(1)
    ├── Monitoreo LED Hallow
    ├── Generación Pulsos A/B
    └── Monitoreo LED Boya
```

---

## Requisitos

### Hardware

* MSP430 compatible con UCS (Unified Clock System)
* Fuente de alimentación adecuada
* Pulsador para entrada Boya
* Señales de monitoreo para Boya y Hallow
* Osciloscopio (opcional para observar ACLK, SMCLK y MCLK)

### Software

* Code Composer Studio (CCS)
* MSP430 GCC Compiler
* Librería estándar MSP430:

```c
#include <msp430.h>
```

---

## Autor

Proyecto desarrollado para monitoreo y control de señales de estado de Boya y Hallow mediante plataforma MSP430.
