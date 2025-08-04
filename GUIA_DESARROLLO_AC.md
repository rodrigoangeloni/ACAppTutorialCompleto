# 🚀 Guía Definitiva para el Desarrollo de Aplicaciones en Assetto Corsa

¡Bienvenido, desarrollador! Esta guía consolida la información de la documentación oficial y tutoriales para ofrecerte un punto de partida centralizado y claro para crear tus propias aplicaciones para Assetto Corsa.

---

## 🏁 Primeros Pasos: Tu "Hola Mundo" en AC

Antes de escribir código, es fundamental entender el ecosistema y el flujo de trabajo.

### 1. Estructura de Carpetas

-   **Instalación de AC:** Generalmente en `C:\Program Files (x86)\Steam\steamapps\common\assettocorsa`.
-   **Carpeta de Apps Python:** Aquí es donde vivirán tus aplicaciones: `\assettocorsa\apps\python\`.
-   **Carpeta de Logs:** Crucial para la depuración, se encuentra en `C:\Users\TU_USUARIO\Documents\Assetto Corsa\logs\`.
    -   `log.txt`: Log general del juego. Aquí verás los errores de carga de tu app.
    -   `py_log.txt`: Log específico para los mensajes que envíes desde tu código Python.

### 2. Instalación y Activación de una App

1.  **Copia la carpeta** de tu aplicación (ej. `MiApp`) dentro de `\assettocorsa\apps\python\`.
2.  **Inicia Assetto Corsa**.
3.  Ve a **Opciones > General**.
4.  **Marca la casilla** junto al nombre de tu aplicación para activarla.
5.  Una vez en una sesión de conducción, encontrarás tu app en la **barra lateral derecha** para mostrarla en pantalla.

### 3. Flujo de Trabajo y Depuración

-   **No reinicies el juego entero:** Para ver los cambios en tu código, solo necesitas salir de la sesión de conducción (volver al menú principal), modificar tu archivo `.py`, y volver a entrar a una sesión.
-   **Usa la depuración:**
    -   `ac.log("mensaje")`: Escribe en el archivo `py_log.txt`. Ideal para registrar datos de forma persistente.
    -   `ac.console("mensaje")`: Escribe en la consola del juego (se abre con la tecla **Home**). Perfecto para ver valores en tiempo real sin salir de la partida.

---

## ⚙️ Tipos de Aplicaciones y Métodos de Desarrollo

Existen dos enfoques principales para crear aplicaciones para AC:

### 1. Aplicaciones In-Game (con la API de Python)

Son aplicaciones que se ejecutan directamente dentro del juego, mostrando ventanas e información sobre la interfaz de usuario (UI).

#### Estructura Básica del Proyecto

-   `\apps\python\MiApp\MiApp.py`
-   `\apps\python\MiApp\lib\` (opcional, para librerías como `sim_info.py`)

#### Ciclo de Vida de la Aplicación

Tu código se estructura en funciones que el juego llama automáticamente:

```python
import ac
import acsys

# --- Variables Globales (para compartir entre funciones) ---
appWindow = 0
mi_etiqueta_de_velocidad = 0

# --- Se ejecuta una vez al iniciar la sesión ---
def acMain(ac_version):
    global appWindow, mi_etiqueta_de_velocidad

    # 1. Crear la ventana de la app
    appWindow = ac.newApp("Mi Primera App")
    ac.setSize(appWindow, 200, 150)

    # 2. Añadir widgets (controles)
    mi_etiqueta_de_velocidad = ac.addLabel(appWindow, "Velocidad: 0 km/h")
    ac.setPosition(mi_etiqueta_de_velocidad, 10, 40)

    ac.log("Mi App se ha iniciado correctamente")
    return "Mi Primera App" # Nombre que aparece en el menú

# --- Se ejecuta en cada frame del juego ---
def acUpdate(deltaT):
    # deltaT es el tiempo en segundos desde el último frame

    # 1. Leer datos del simulador
    velocidad = ac.getCarState(0, acsys.CS.SpeedKMH)

    # 2. Actualizar la UI
    ac.setText(mi_etiqueta_de_velocidad, f"Velocidad: {velocidad:.0f} km/h")

# --- Se ejecuta al finalizar la sesión ---
def acShutdown():
    ac.log("Mi App se está cerrando")
    # Aquí puedes guardar datos si es necesario
```

#### Acceso a Datos

-   **API de Python (`ac.getCarState`)**: Ofrece acceso a los datos más comunes (velocidad, RPM, marcha, tiempos, etc.).
-   **Memoria Compartida (`sim_info.py`)**: Para acceder a **toda** la información de la simulación (temperaturas, daños, etc.), debes usar un wrapper de la memoria compartida.
    1.  Descarga `sim_info.py` y `_ctypes.pyd`.
    2.  Crea una carpeta `lib` dentro de tu app y colócalos ahí.
    3.  Añade la librería a tu script:
        ```python
        import sys
        # Añade la ruta de tu librería
        sys.path.insert(0, 'apps/python/MiApp/lib')
        from sim_info import info
        ```
    4.  Accede a los datos: `combustible = info.physics.fuel`

### 2. Aplicaciones Externas (Telemetría Remota UDP)

Este método es para aplicaciones que se ejecutan en el mismo PC o en otro dispositivo de la red (como un dashboard en una tablet). La comunicación se realiza por red (UDP).

#### Flujo de Comunicación

1.  **Handshake (Saludo):**
    -   Tu aplicación (cliente) envía un paquete al PC con Assetto Corsa (servidor) en el puerto **9996**.
    -   El paquete inicial debe tener `operationId = 0` (HANDSHAKE).
    -   El servidor responde con datos del coche, piloto y pista.
2.  **Suscripción:**
    -   El cliente envía un segundo paquete para suscribirse a las actualizaciones.
    -   `operationId = 1` (SUBSCRIBE_UPDATE): Para recibir datos de telemetría completos en cada frame.
    -   `operationId = 2` (SUBSCRIBE_SPOT): Para recibir solo eventos importantes (como el final de una vuelta).
3.  **Recepción de Datos:**
    -   El servidor envía continuamente la estructura `RTCarInfo` con todos los datos de telemetría.
4.  **Desconexión:**
    -   El cliente envía un paquete con `operationId = 3` (DISMISS) para finalizar la comunicación.

---

## 📚 Referencia de Datos: La Memoria Compartida

La memoria compartida es la **fuente de la verdad** para todos los datos de la simulación. Es un espacio de memoria donde el juego escribe la información en tiempo real. Se divide en tres grandes bloques:

### 1. `SPageFilePhysics` (`acpmf_physics`)
Datos de alta frecuencia que cambian constantemente.
- **Entradas:** `gas`, `brake`, `steerAngle`
- **Motor:** `rpms`, `gear`, `fuel`
- **Movimiento:** `speedKmh`, `velocity`, `accG`
- **Ruedas:** `wheelSlip`, `wheelsPressure`, `tyreCoreTemperature`, `suspensionTravel`
- **Otros:** `carDamage`, `drs`, `kersCharge`, `turboBoost`

### 2. `SPageFileGraphic` (`acpmf_graphics`)
Datos de sesión y UI, de frecuencia media.
- **Estado:** `status` (Live, Pausa, Replay), `session` (Carrera, Práctica, etc.)
- **Tiempos:** `currentTime`, `lastTime`, `bestTime`, `split`
- **Sesión:** `completedLaps`, `position`, `numberOfLaps`, `sessionTimeLeft`
- **Pista:** `normalizedCarPosition`, `isInPit`, `flag` (banderas)

### 3. `SPageFileStatic` (`acpmf_static`)
Datos estáticos que no cambian durante la sesión.
- **Versiones:** `smVersion`, `acVersion`
- **Info Sesión:** `numCars`, `track`, `carModel`
- **Info Jugador:** `playerName`, `playerNick`
- **Info Coche:** `maxRpm`, `maxFuel`, `maxTorque`
- **Condiciones:** `airTemp`, `roadTemp`

---

## 🔗 Recursos Adicionales

-   **Custom Shaders Patch (CSP):** Es un mod muy popular que expande enormemente las capacidades de Assetto Corsa, incluyendo la adición de **muchas funciones nuevas a la API de Python**. Si planeas un desarrollo avanzado, es muy recomendable investigar la [documentación de la API de Python de CSP](https://github.com/ac-custom-shaders-patch/acc-extension-config/wiki/Python-Apps-%E2%80%93-New-functions).

¡Feliz desarrollo! 👨‍💻