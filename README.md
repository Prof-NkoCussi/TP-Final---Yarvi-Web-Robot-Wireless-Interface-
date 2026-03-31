# TP-Final---Yarvi-Web-Robot-Wireless-Interface

# 🤖 Yarvi — Control de Robot Asistente por Red

Proyecto Integrador Anual — 
Redes I 5° Año · Colegio Técnico Provincial "Olga B. de Arko"
Prof. Nicolás A. Cussi

---

## 📋 Descripción

**Yarvi** es un mini robot (auto de 2 ruedas) controlado remotamente desde una aplicación web a través de una red local. El sistema integra conceptos de redes, programación y desarrollo de software en un proyecto IoT funcional.

El usuario envía comandos desde el navegador → el servidor los recibe vía HTTP/REST → los publica en un broker MQTT → el ESP32 los recibe por WiFi y mueve los motores.

## 🏗️ Arquitectura del Sistema

```
┌─────────────────┐
│   Navegador      │  ← Cliente (HTML + JS)
│   (Usuario)      │
└────────┬────────┘
         │  HTTP / REST / JSON
         │  Puerto 3000
         ▼
┌─────────────────┐
│  Servidor        │  ← Node.js + Express
│  Node.js         │
└────────┬────────┘
         │  MQTT publish
         │  Puerto 1883
         ▼
┌─────────────────┐
│  Broker          │  ← Mosquitto
│  Mosquitto       │
└────────┬────────┘
         │  MQTT subscribe
         │  WiFi (misma LAN)
         ▼
┌─────────────────┐
│  ESP32           │  ← Microcontrolador WiFi
│  (IP fija)       │     IP: 192.168.x.x
└────────┬────────┘
         │  PWM / GPIO
         ▼
┌─────────────────┐
│  Driver L9110S   │  → Motores TT
│  + Chasis        │  → Movimiento del robot
└─────────────────┘
```

## 🔧 Tecnologías Utilizadas

| Capa | Tecnología | Puerto | Función |
|------|-----------|--------|---------|
| Frontend | HTML + CSS + JavaScript | — | Interfaz de control del usuario |
| Backend | Node.js + Express | 3000 | Servidor REST, puente HTTP→MQTT |
| Broker | Mosquitto | 1883 | Intermediario de mensajes MQTT |
| Hardware | ESP32 (Arduino/PlatformIO) | — | Recibe comandos y controla motores |
| Drivers | L9110S | — | Controla los motores TT del chasis |

## 📁 Estructura del Proyecto

```
yarvi/
├── server/
│   ├── server.js          # Servidor Express + cliente MQTT
│   ├── package.json       # Dependencias del proyecto
│   └── public/
│       ├── index.html      # Interfaz web de control
│       ├── style.css       # Estilos de la interfaz
│       └── app.js          # Lógica del cliente (fetch a la API)
├── esp32/
│   └── yarvi_mqtt.ino      # Firmware del ESP32 (suscriptor MQTT)
├── docs/
│   ├── arquitectura.png    # Diagrama de arquitectura
│   └── fotos/              # Fotos del proceso de armado
├── README.md
└── .gitignore
```

## 🚀 Instalación y Puesta en Marcha

### Requisitos previos

- **Node.js** v18 o superior → [nodejs.org](https://nodejs.org)
- **Mosquitto** (broker MQTT) → [mosquitto.org](https://mosquitto.org)
- **Git** → [git-scm.com](https://git-scm.com)
- Red WiFi local (todos los dispositivos en la misma subred)

### Paso 1 — Clonar el repositorio

```bash
git clone https://github.com/tu-usuario/yarvi.git
cd yarvi/server
```

### Paso 2 — Instalar dependencias

```bash
npm install
```

### Paso 3 — Iniciar el broker Mosquitto

```bash
# En una terminal aparte
mosquitto -v
```

> El broker escucha en el puerto **1883** por defecto.

### Paso 4 — Iniciar el servidor

```bash
node server.js
```

> El servidor se levanta en `http://localhost:3000`

### Paso 5 — Verificar conectividad con el ESP32

```bash
ping 192.168.x.x    # Reemplazar con la IP fija del ESP32
```

### Paso 6 — Abrir la interfaz de control

Abrir el navegador en `http://localhost:3000` y probar los comandos del robot.

## 📡 API REST — Endpoints

| Endpoint | Método | Descripción | Body (JSON) | Respuesta (JSON) |
|----------|--------|-------------|-------------|------------------|
| `/api/robot/command` | `POST` | Envía un comando de movimiento | `{ "accion": "adelante" }` | `{ "success": true, "accion": "adelante" }` |
| `/api/robot/status` | `GET` | Consulta el estado del robot | — | `{ "online": true, "ultimaAccion": "..." }` |
| `/api/robot/stop` | `POST` | Detiene el robot inmediatamente | — | `{ "success": true, "accion": "stop" }` |

### Acciones disponibles

| Acción | Descripción |
|--------|-------------|
| `adelante` | Ambos motores hacia adelante |
| `atras` | Ambos motores en reversa |
| `izquierda` | Motor derecho avanza, izquierdo se detiene |
| `derecha` | Motor izquierdo avanza, derecho se detiene |
| `stop` | Detiene ambos motores |

### Ejemplo con `curl`

```bash
# Mover el robot hacia adelante
curl -X POST http://localhost:3000/api/robot/command \
  -H "Content-Type: application/json" \
  -d '{"accion": "adelante"}'

# Consultar estado
curl http://localhost:3000/api/robot/status
```

## 📨 Protocolo MQTT

| Elemento | Valor |
|----------|-------|
| **Broker** | Mosquitto (local) |
| **Puerto** | 1883 |
| **Publisher** | Servidor Node.js (al recibir un POST en la API) |
| **Subscriber** | ESP32 (escucha y ejecuta los comandos) |
| **Tópico principal** | `yarvi/comando` |
| **Tópico de estado** | `yarvi/estado` |

### ¿Por qué MQTT y no HTTP directo al ESP32?

1. **Desacoplamiento**: el ESP32 no necesita ser un servidor web; solo se suscribe a un tópico.
2. **Bajo consumo**: MQTT es un protocolo liviano, ideal para microcontroladores con recursos limitados.
3. **Broker como intermediario**: si el ESP32 se desconecta momentáneamente, el broker puede retener mensajes.

## 🔌 Hardware del Kit

| Componente | Cant. | Función |
|-----------|-------|---------|
| Chasis 2 motores TT + rueda loca | 1 | Estructura física |
| Portapilas 4xAA con switch | 1 | Alimentación de motores (6V) |
| ESP32 DevKit USB-C | 1 | Microcontrolador WiFi + MQTT |
| Driver de motores L9110S | 1 | Puente entre ESP32 y motores |
| Cables jumper H-H | varios | Conexiones |
| Pilas AA | 4 | Energía |

### Conexiones ESP32 → L9110S

| ESP32 Pin | L9110S Pin | Motor |
|-----------|-----------|-------|
| GPIO 25 | IA (Motor A) | Izquierdo - adelante |
| GPIO 26 | IB (Motor A) | Izquierdo - atrás |
| GPIO 27 | IA (Motor B) | Derecho - adelante |
| GPIO 14 | IB (Motor B) | Derecho - atrás |

## 🛡️ Consideraciones de Seguridad

El sistema actual es un prototipo educativo. Vulnerabilidades conocidas y posibles mejoras:

| Vulnerabilidad | Riesgo | Mejora propuesta |
|---------------|--------|-----------------|
| API sin autenticación | Cualquiera en la LAN puede enviar comandos | Agregar API Key o JWT |
| MQTT sin cifrado | Los mensajes viajan en texto plano | Configurar MQTT con TLS |
| HTTP sin cifrar | Las peticiones pueden ser interceptadas | Migrar a HTTPS con certificado |
| WiFi con contraseña débil | Acceso no autorizado a la red | Usar WPA3 y contraseña robusta |

## 🔍 Diagnóstico de Red

Herramientas útiles para verificar el funcionamiento del sistema:

```bash
# Verificar conectividad con el ESP32
ping 192.168.x.x

# Verificar que los puertos estén escuchando
netstat -an | grep 3000    # Servidor Express
netstat -an | grep 1883    # Broker MQTT

# Captura de tráfico (requiere Wireshark o tcpdump)
# Filtro sugerido en Wireshark: mqtt || http
```

## 🗂️ Contenidos Curriculares Integrados

### Redes I
- Direccionamiento IPv4 y subnetting
- Modelos OSI y TCP/IP
- Topologías de red
- TCP como protocolo de transporte
- HTTP/REST y APIs
- MQTT (pub/sub)
- Puertos de servicio
- Seguridad en redes
- Diagnóstico con herramientas

### Programación II
- JavaScript: variables, funciones, estructuras de control
- Fetch API y manejo de JSON
- Arquitectura cliente-servidor

### Software II
- Git: clone, add, commit, push, branch
- Documentación técnica (README)
- Resolución de conflictos en repositorios colaborativos

## 👥 Equipo

| Integrante | Rol | Rama |
|-----------|-----|------|
| Nombre 1 | — | `grupo-X` |
| Nombre 2 | — | `grupo-X` |
| Nombre 3 | — | `grupo-X` |

## 📄 Licencia

Proyecto educativo — Colegio Técnico Provincial "Olga B. de Arko", Ushuaia, Tierra del Fuego.

---

Redes I — Prof. Nicolás A. Cussi —
