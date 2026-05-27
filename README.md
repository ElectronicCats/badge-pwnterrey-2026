# Badge PWNTerrey

El **El Badge de Pwnterrey** es un dispositivo de hardware abierto y portátil diseñado para la auditoría de redes inalámbricas y seguridad de radiofrecuencia (WiFi, Bluetooth y NFC). Esta versión utiliza el potente y nuevo chip **ESP32-C5 (Arquitectura RISC-V)**, configurada específicamente para operar en modo **"Headless"** (sin pantalla), permitiendo la interacción total mediante consola serie (CLI) e integrando sensores y emuladores como NFC y GPS externo.

***

## 🚀 Características Principales

### 📶 Conectividad Inalámbrica Avanzada
- **Procesador ESP32-C5**: Chip de doble banda Wi-Fi 6 (2.4 GHz y 5 GHz) y Bluetooth 5 (LE) integrado.
- **Auditoría WiFi**: Soporte para escaneo de APs, rastreo de estaciones, sniffing de paquetes, inyección de tramas de desautenticación, spam de beacons y captura de handshakes.
- **Bluetooth (BLE)**: Capacidad para ejecutar ataques de spam BLE (`blespam`).

### 🏷️ Periféricos y Sensores Integrados
- **Emulador NFC (NT3H2111)**: Chip NTAG I2C 2k que permite actuar en **Modo Tarjeta**. El ESP32 emula una etiqueta pasiva para transferir datos NDEF (URLs, textos, vCards, redes WiFi) a teléfonos móviles o Flipper Zero.
- **Receptor GPS**: Interfaz serial dedicada para capturar coordenadas geográficas en tiempo real (Wardriving) y parsear sentencias NMEA.

### ⚙️ Optimización Headless (Sin Pantalla)
- **Bajo Consumo**: Al eliminar pantallas TFT/OLED y botones físicos, se reduce la huella energética y se maximiza la estabilidad.
- **Control Serial Total**: Una robusta interfaz de comandos (CLI) que permite controlar el 100% de las funciones del dispositivo.

***

## 📋 Especificaciones Técnicas

| Componente | Especificación |
| :--- | :--- |
| **Microcontrolador** | ESP32-C5 ECO1 (Arquitectura RISC-V de 32 bits) |
| **Wi-Fi** | Doble banda 2.4 GHz y 5 GHz (Wi-Fi 6 / 802.11ax) |
| **Bluetooth** | Bluetooth 5.0 (Low Energy - LE) |
| **NFC** | Chip **NT3H2111 (NTAG I2C 2k)** emulador de etiqueta pasiva |
| **GPS** | Interfaz serie dedicada para receptor GPS externo (UART1) |
| **Interfaz de Control**| CLI por Puerto Serie USB (115200 baudios) |
| **Estabilidad** | PSRAM desactivada, Flash configurada a 40MHz |

***

## 🔌 Interfaces y Pinout del ESP32-C5

El circuito integrado ESP32-C5 en su configuración Marauder utiliza los siguientes pines clave:

### 🌐 Conexión NFC (I2C)
El chip NTAG NT3H2111 está conectado a través del bus I2C:
- **SDA**: GPIO 2 (Pin 4)
- **SCL**: GPIO 3 (Pin 5)
- **Field Detect (FD)**: GPIO 6 (Pin 8) *(Detecta cuando un lector NFC externo energiza el campo inductivo)*

### 🛰️ Conexión GPS (UART1)
La comunicación con el módulo GPS se realiza mediante los pines:
- **GPS TX (MCU)**: GPIO 4 *(Hacia el pin RX del GPS)*
- **GPS RX (MCU)**: GPIO 5 *(Desde el pin TX del GPS)*
- **GPS ON/Power**: GPIO 7 *(Control de encendido del módulo GPS)*

***

## ⚡ Primeros Pasos

### 1. Alimentación y Conexión
Puedes alimentar y controlar tu adaptador de dos formas:
- **Conexión Directa USB**: Conéctalo a tu computadora usando un cable USB-C. El puerto creará un puerto COM serial.
- **Integración con Flipper Zero**: Conectado como un Addon GPIO al Flipper Zero, alimentado directamente desde los pines de 3.3V/5V del Flipper y controlado desde su aplicación de terminal.

### 2. Abrir la Terminal Serial
El dispositivo se comunica a **115200 baudios**, sin control de flujo (8N1).

#### 🐧 Linux / macOS
Usa el comando `screen` o `minicom`. Primero identifica el puerto (`/dev/ttyUSBX` o `/dev/ttyACM0`):
```bash
# Con screen (para salir: Ctrl+A, luego K)
screen /dev/ttyUSB0 115200

# Con minicom
sudo minicom -b 115200 -D /dev/ttyUSB0
```

#### 🪟 Windows
Usa clientes como **PuTTY** o **TeraTerm**:
- **Puerto**: COMx (Ver en administrador de dispositivos)
- **Baudrate**: 115200
- **Data bits**: 8
- **Parity**: None
- **Stop bits**: 1
- **Flow control**: None

***

## 🏷️ Control del Emulador NFC

El módulo NFC funciona exclusivamente en **modo emulación de etiqueta (Tag)**. No es un lector; actúa como una tarjeta inteligente pasiva que almacena registros NDEF.

### 🔍 Diagnóstico e Información
- **Escanear bus NFC**:
  ```bash
  nfc scan
  ```
  Realiza un escaneo de hardware (discovery) en los pares de pines del bus I2C para verificar la dirección del chip (`0x55` o `0x2A`).
- **Leer contenido actual de la etiqueta**:
  ```bash
  nfc read
  ```
  Muestra en hexadecimal y ASCII los bloques del 1 al 8 grabados en la memoria del chip NFC.

### ✍️ Escritura de Registros NDEF
Puedes programar el emulador NFC para inyectar diferentes cargas útiles:

- **🔗 Enlace Web (URL)**:
  ```bash
  nfc -u https://electroniccats.com
  ```
- **📝 Texto Plano**:
  ```bash
  nfc -t "Hola Hackers del ESP32-C5"
  ```
- **📇 Tarjeta de Contacto (vCard)**:
  ```bash
  nfc -v "Nombre,Telefono,Email"
  # Ejemplo:
  nfc -v "Electronic Cats,5551234567,info@electroniccats.com"
  ```
- **📶 Credenciales de Red WiFi**:
  ```bash
  nfc -w "SSID,Password,Autenticacion"
  # Ejemplo:
  nfc -w "MiRedSegura,Clave1234,WPA2"
  ```

Una vez ejecutado el comando con éxito, cualquier dispositivo móvil o lector NFC (como un Flipper Zero) que se acerque al sensor detectará la etiqueta y ejecutará la acción (abrir el link, importar el contacto o conectarse al WiFi).

***

## ⚙️ Configuración y Compilación de Firmware

Si deseas compilar o realizar modificaciones en el firmware, debes seguir estas recomendaciones específicas para el hardware ESP32-C5:

Clonar el [repositorio](https://github.com/ElectronicCats/badge-pwnterrey-2026)

### 🛠️ Configuración en configs.h
Asegúrate de que en el archivo [configs.h](file:///run/media/omaro/8C96D9BF96D9AA4A/ESP32Marauder/ESP32Marauder/esp32_marauder/configs.h) la definición activa para la placa sea:
```cpp
#define MARAUDER_FLIPPER_C5
```
Esta definición configurará automáticamente la ausencia de pantalla (`TFT_CS -1`), habilitará `HAS_BT`, `HAS_GPS` y `HAS_NFC` con sus respectivos mapeos de pines.

### ⚠️ Parámetros de Compilación en Arduino IDE
Debido al estado actual del soporte del chip, aplica las siguientes configuraciones de carga:
- **Placa**: ESP32C5 Dev Module
- **Arquitectura**: RISC-V (Compiler optimization)
- **Flash Frequency**: **40MHz** (Recomendado para máxima estabilidad)
- **Partition Scheme**: Minimal SPIFFS / No OTA (Para maximizar el espacio de programa y SPIFFS)
- **PSRAM**: Desactivada (Disabled)
- **Modo de Carga**: Inserte el dispositivo en modo Bootloader manteniendo presionado el botón Boot mientras conecta el USB si no es detectado automáticamente.

***

## ⌨️ Consola de Comandos Completa (CLI Reference)

A continuación se detalla la lista de comandos disponibles en el intérprete de comandos serial:

### 🛠️ Comandos de Administración

| Comando | Sintaxis | Descripción |
| :--- | :--- | :--- |
| `help` | `help` | Muestra la lista de comandos con su formato de ayuda. |
| `info` | `info` | Muestra información del sistema, compilación y hardware. |
| `reboot` | `reboot` | Reinicia el microcontrolador. |
| `channel` | `channel [-s <canal>]` | Muestra o cambia el canal WiFi activo. |
| `settings` | `settings [-s <setting> <enable/disable>]` | Muestra, modifica o reinicia (`-r`) los ajustes internos. |
| `spiffs` | `spiffs [ls / read <file> / rm <file> / format]`| Administra la memoria flash interna (SPIFFS). |
| `led` | `led -s <hex color> / -p <rainbow>` | Cambia el color del led de estado. |

### 📶 Auditoría WiFi (Sniffing & Scanning)

| Comando | Sintaxis | Descripción |
| :--- | :--- | :--- |
| `scanap` | `scanap` | Busca puntos de acceso (APs) inalámbricos cercanos. |
| `scansta` | `scansta` | Busca estaciones y clientes conectados. |
| `stopscan` | `stopscan` | Detiene inmediatamente cualquier escaneo en curso. |
| `list` | `list [-a / -s / -c / -t]` | Muestra listas cargadas (`-a` APs, `-s` SSIDs, `-c` Clientes). |
| `select` | `select -a/-s/-c <indices>` | Selecciona elementos por índice para interactuar. |
| `clearlist`| `clearlist -a/-c/-s` | Limpia los registros de la memoria RAM. |
| `ssid` | `ssid -a [-g <count> / -n <name>]` | Agrega SSIDs a la lista (`-g` aleatorios, `-n` específicos). |
| `sniffraw` | `sniffraw` | Captura y vuelca todos los paquetes raw recibidos. |
| `sniffbeacon`| `sniffbeacon` | Captura tramas beacon. |
| `sniffprobe`| `sniffprobe` | Captura peticiones probe request de clientes. |
| `sniffpwn` | `sniffpwn` | Sniffer enfocado en capturas de handshakes WPA. |
| `sniffpmkid`| `sniffpmkid [-c <channel>]` | Captura asociaciones PMKID para auditoría WPA2. |

### 💥 Comandos de Ataque WiFi y Bluetooth

| Comando | Sintaxis | Descripción |
| :--- | :--- | :--- |
| `attack` | `attack -t <deauth/beacon/probe/rickroll>` | Inicia ataque de desautenticación, spam de beacons, etc. |
| `blespam` | `blespam -t <apple/google/samsung/windows/all>`| Inicia campañas de spam publicitario Bluetooth BLE. |
| `wardrive` | `wardrive [-s]` | Inicia el modo Wardriving clásico sincronizado con GPS. |


***

## ⚠️ Advertencia de Seguridad

Esta herramienta ha sido diseñada exclusivamente para **fines educativos, pruebas de penetración ética y auditorías de seguridad autorizadas**. 

El uso de este dispositivo para interferir, interceptar o atacar redes e infraestructura ajena sin el consentimiento previo por escrito del propietario es **estrictamente ilegal**. Los desarrolladores y colaboradores no se hacen responsables del uso indebido de esta herramienta ni de los daños que este pueda causar.

## 📖 Documentación Adicional

- [Marauder App](https://github.com/ElectronicCats/marauder-ui-android-app)
- [Marauder UI Pro](https://github.com/ElectronicCats/marauder-ui-pro)
- [Firmware](https://github.com/ElectronicCats/pwnterrey-2026_Firmware)

## 🤝 Contribuciones

Este es un proyecto de hardware abierto. Las contribuciones son bienvenidas:

1. Haz un fork del repositorio.
2. Crea una nueva rama para tu feature (`git checkout -b feature/AmazingFeature`)
3. Haz commit de tus cambios (`git commit -m 'Add some AmazingFeature'`)
4. Haz push a tu rama (`git push origin feature/AmazingFeature`)
5. Abre un Pull Request

## 📝 Licencia

Este proyecto es de hardware abierto. Consulta los archivos de licencia para más detalles.

## 🏢 Electronic Cats

Desarollado con ❤️ por [Electronic Cats](https://www.electroniccats.com/)

<a href="https://github.com/sponsors/ElectronicCats">
<img src="https://electroniccats.com/wp-content/uploads/2020/07/Badge_GHS.png" height="104" />
</a>

Electronic Cats invierte tiempo y recursos en proporcionar este diseño de hardware abierto.
Por favor, apoya a Electronic Cats y al hardware abierto comprando productos de Electronic Cats.

## 📞 Contacto y soporte 

- **Website**: [https://www.electroniccats.com/](https://www.electroniccats.com/)
- **GitHub**: [ElectronicCats](https://github.com/ElectronicCats)
- **Issues**: Usa la sección de Issues en GitHub para reportar problemas o sugerir mejoras

## 🙏 Agradecimientos

Gracias a toda la comunidad de hardware abierto y a todas las personas que hacen posibles proyectos como este.

## 🤝 Credits & Ecosystem
- **Hardware & Support**: [Electronic Cats](https://electroniccats.com) 😼
- **Original Firmware**: [justcallmekoko](https://github.com/justcallmekoko)
- **UI Maintenance**: [michelangelomo](https://github.com/michelangelomo) & [Mikystars](https://github.com/Mikystars)

---
*Developed for the Offensive Security community.* 💀

