# PX4 Flight Lab

Herramienta local en Python para trabajar con PX4/Pixhawk sin instalar las dependencias Python de la aplicación de forma global en el sistema.

Incluye:

- conexión MAVLink por USB, radio SiK/telemetría y UDP/MAVROS;
- modo compartido para usar MAVROS + QGroundControl + PX4 Flight Lab;
- biblioteca recursiva de archivos `.params`;
- comparación, backup automático y carga de parámetros con el vehículo desarmado;
- listado y descarga de logs ULog desde la Pixhawk;
- analizador de `.ulg` con topics/campos y gráficas rápidas de actitud, rates, posición, velocidad y batería;
- exportación de topics a CSV.

## Primera ejecución en cada ordenador

Desde la carpeta del proyecto:

```bash
chmod +x setup.sh run.sh reset_local_env.sh
./setup.sh
```

El instalador prepara `.runtime/` y `.venv/` dentro de esta misma carpeta. No instala paquetes Python globales ni modifica el `PATH` del usuario.

Después, para abrir la aplicación:

```bash
./run.sh
```

Si copias el proyecto a otro ordenador, no copies `.venv/` ni `.runtime/`. Ejecuta allí una vez `./setup.sh` para que se prepare un entorno compatible con ese equipo.

## Modos de conexión

La aplicación detecta automáticamente `/dev/ttyACM*` y `/dev/ttyUSB*`. No es necesario escribir endpoints manualmente.

### USB

Perfil típico:

```text
USB · /dev/ttyACM0
```

Usa 115200 baud. Si QGroundControl tiene abierto directamente el mismo puerto USB, desconéctalo antes de usar este modo.

### Radio directa

Perfil típico:

```text
Radio · /dev/ttyUSB0
```

Usa 57600 baud. En este modo MAVROS no debe tener abierto `/dev/ttyUSB0`, y el AutoConnect SiK de QGroundControl tampoco debe apropiarse de esa radio.

### MAVROS UDP, herramienta sola

Con tu comando habitual:

```bash
ros2 launch mavros px4.launch \
  fcu_url:=/dev/ttyUSB0:57600 \
  gcs_url:=udp://@127.0.0.1:14550
```

selecciona:

```text
MAVROS UDP · herramienta sola
```

Este modo escucha UDP 14550. QGroundControl debe dejar libre ese puerto.

### MAVROS + QGroundControl + PX4 Flight Lab

Este es el modo recomendado cuando quieres mantener QGroundControl conectado a la vez.

Lanza MAVROS así:

```bash
ros2 launch mavros px4.launch \
  fcu_url:=/dev/ttyUSB0:57600 \
  gcs_url:=udp://@127.0.0.1:14551
```

Luego selecciona:

```text
MAVROS + QGroundControl · compartido
```

La aplicación crea un puente local:

```text
Pixhawk/Radio
     │
   MAVROS
  14555 -> 14551
             │
             ├── QGroundControl :14550
             └── PX4 Flight Lab :14552
```

QGroundControl puede mantener su UDP habitual en 14550.

## Archivos `.params`

La biblioteca hace búsqueda recursiva. Puedes seleccionar, por ejemplo, una carpeta:

```text
Ensayos/
├── vuelo_01.params
├── vuelo_02.params
└── PID/
    ├── test_A.params
    └── test_B.params
```

y aparecerán todos los `.params`, incluidos los de subcarpetas. La extensión se detecta sin distinguir mayúsculas/minúsculas.

La lista se actualiza automáticamente; ya no hay botón de refresco.

Al aplicar parámetros, la herramienta crea primero un backup en:

```text
<carpeta params>/backups/
```

La escritura se bloquea si PX4 informa de que el vehículo está armado.

## Logs

Los logs descargados se guardan automáticamente en:

```text
px4_qgc_tool/logs/
```

La descarga usa peticiones por bloques grandes en USB/UDP y bloques más pequeños en radio. Esto reduce mucho el tiempo muerto que tenía la versión anterior en USB y conserva reintentos selectivos para enlaces con pérdidas.

Puedes hacer doble clic sobre un log para descargarlo. Al terminar, se abre automáticamente en el analizador ULog.

## Analizador ULog

Además del selector completo de topic/campo, hay vistas rápidas para:

- actitud;
- velocidad angular;
- posición local;
- velocidad local;
- batería.

También puedes exportar cualquier topic a CSV.

## Restablecer el entorno local

Si quieres borrar únicamente el runtime y las dependencias locales de esta herramienta:

```bash
./reset_local_env.sh
```

Después `./run.sh` volverá a preparar el entorno en la siguiente ejecución.
