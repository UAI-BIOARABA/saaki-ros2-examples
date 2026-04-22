<div align="center">

<h1> Ejemplos de ROS2 para Saaki - Unitree G1 </h1>

<p>
  <a href="README.md">English</a> |
  <a href="README_es.md">Español</a>
</p>

[![ROS 2 Humble](https://img.shields.io/badge/ROS2-Humble-22314E?logo=ros&logoColor=white)](https://docs.ros.org/en/humble/index.html)
[![Ubuntu 22.04](https://img.shields.io/badge/Ubuntu-22.04-E95420?logo=ubuntu&logoColor=white)](https://releases.ubuntu.com/22.04/)
[![C++17](https://img.shields.io/badge/C%2B%2B-17-00599C?logo=c%2B%2B&logoColor=white)](https://isocpp.org/)

[![Robot: Unitree G1](https://img.shields.io/badge/Robot-Unitree%20G1-0A66C2)](https://www.unitree.com/g1)
[![Status: Tested on G1](https://img.shields.io/badge/Status-Tested%20on%20Real%20Hardware-success)](#ejecución-y-verificacion)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
</div>

## 📖 Descripción

Este repositorio contiene un paquete de ROS 2 (`saaki_ros2_examples`) optimizado y configurado exclusivamente para controlar y monitorizar el robot humanoide **Unitree G1**.

**Créditos y Origen:** El código fuente de los ejemplos y la estructura base pertenecen a [Unitree Robotics](https://github.com/unitreerobotics/unitree_ros2). Este repositorio es una *adaptación* en el que se ha limpiado el código eliminando los scripts de otros modelos (Go2, B2, etc.) y se ha reestructurado el `CMakeLists.txt` para cumplir con los estándares de instalación de ejecutables de ROS 2 (permitiendo el uso nativo de `ros2 run`). Adicionalmente, podremos ir creando scripts propios a modo de ejemplo sin eliminar los originales.

---

## 🛠️ Requisitos Previos

* **Sistema Operativo:** Ubuntu 22.04 LTS
* **Middleware ROS:** ROS 2 Humble
* **Hardware:** Robot Unitree G1 (conexión por cable Ethernet)

---

## 📦 1. Instalación Base (Dependencias de Unitree)

Dado que este paquete depende de los mensajes oficiales del robot (`unitree_go`, `unitree_hg`, `unitree_api`), **es obligatorio** instalar y compilar el repositorio oficial de Unitree como capa base ("underlay") antes de compilar este repositorio.

### 1.1. Instalar CycloneDDS

El robot se comunica a través de CycloneDDS. En ROS 2 Humble, basta con instalar los binarios del sistema:
```bash
sudo apt install ros-humble-rmw-cyclonedds-cpp ros-humble-rosidl-generator-dds-idl libyaml-cpp-dev
```

### 1.2. Clonar y compilar los mensajes oficiales
No es necesario compilar todo el repositorio de Unitree, solo su espacio de trabajo de CycloneDDS:

```bash
# Clonar el repositorio oficial en tu directorio home (usa nuestro fork)
git clone https://github.com/UAI-BIOARABA/unitree_ros2

# Compilar los paquetes de mensajes
cd ~/unitree_ros2/cyclonedds_ws
colcon build
```

---

## 🎁 2. Instalación de este Paquete (Saaki Examples)

Una vez tienes la base de Unitree, puedes clonar y compilar este entorno de trabajo.

```bash
# Crear tu workspace si no lo tienes
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src

# Clonar este repositorio (le ponemos '_' en vez de '-' por estandares de ROS2)
git clone https://github.com/UAI-BIOARABA/saaki-ros2-examples.git saaki_ros2_examples

# Ir a la raíz del workspace
cd ~/ros2_ws

# IMPORTANTE: Cargar el entorno de Unitree ANTES de compilar
source ~/unitree_ros2/setup.sh

# Compilar este paquete
colcon build --symlink-install
```

---

## 🌐 3. Configuración de Red (Conexión al Robot)

Para que ROS 2 descubra al robot, tu PC debe estar en la misma subred y usar CycloneDDS correctamente.

### 1. Conecta el PC al robot mediante cable Ethernet.

### 2. Configura una IP estática en tu PC:

   - IP: 192.168.123.99

   - Máscara: 255.255.255.0

### 3. Edita el script de configuración oficial (~/unitree_ros2/setup.sh). Debe quedar algo así (cambia enp44s0 por el nombre de tu interfaz de red):

```sh
#!/bin/bash
echo "Setup unitree ros2 environment"
source /opt/ros/humble/setup.bash
source $HOME/unitree_ros2/cyclonedds_ws/install/setup.bash
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
export CYCLONEDDS_URI='<CycloneDDS><Domain><General><Interfaces>
                            <NetworkInterface name="enp44s0" priority="default" multicast="default" />
                        </Interfaces></General></Domain></CycloneDDS>'
```

---

## 🚀 4. Ejecución y verificación

Cada vez que abras una terminal nueva para trabajar con el robot, debes cargar ambos entornos en este orden:

```bash
# 1. Cargar dependencias y configuración de red de Unitree
source ~/unitree_ros2/setup.sh

# 2. Cargar tu espacio de trabajo
source ~/ros2_ws/install/setup.bash
```

### Nodos Disponibles
Puedes ejecutar cualquiera de los siguientes nodos usando el comando estándar de ROS 2:

### Lectura de estado:

- ros2 run saaki_ros2_examples read_low_state_hg (Lee el estado de bajo nivel de los motores y sensores).

- ros2 run saaki_ros2_examples read_wireless_controller (Lee los inputs del mando a distancia).

### Control del robot (¡Precaución! El robot se moverá):

- ros2 run saaki_ros2_examples g1_low_level_example (Control directo a bajo nivel).

- ros2 run saaki_ros2_examples g1_loco_client_example (Control de locomoción a alto nivel).

- ros2 run saaki_ros2_examples g1_arm_action_example (Ejemplo de acciones de los brazos).

- ros2 run saaki_ros2_examples g1_audio_client_example (Prueba del sistema de audio).

*(Ver el código fuente de cada script para más detalles sobre lo que hace cada ejemplo).*

---

## ⚠️ Solución de Problemas Comunes

- "No executable found" o "Package not found": Asegúrate de haber hecho source install/setup.bash en la raíz de ros2_ws.

- "CMake Error: Could not find unitree_hg": Olvidaste hacer source ~/unitree_ros2/setup.sh antes de ejecutar colcon build.

- Los tópicos no aparecen (ros2 topic list está vacío):

    1. Comprueba que el firewall de Ubuntu está desactivado (sudo ufw disable).

    2. Verifica que la IP local es 192.168.123.99.

    3. Asegúrate de que no tienes un ROS_DOMAIN_ID configurado que entre en conflicto con el del robot (por defecto el robot usa el ID 0 o ninguno).

---

## 🧑‍💻 Autores

- **Código base de los ejemplos:** [Unitree Robotics](https://github.com/unitreerobotics) &rarr; [unitree_ros2](https://github.com/unitreerobotics/unitree_ros2)
- **Project Manager:** [Juan Fernández](https://github.com/jfbioaraba)
- **Lead Developer:** [Andoni González](https://github.com/andoni92)

---

## Descargo de responsabilidad


Este software y los materiales asociados se proporcionan “tal cual”, sin garantías de ningún tipo, ni expresas ni implícitas, incluyendo —pero no limitándose a— garantías de comercialización, idoneidad para un propósito particular o ausencia de errores.
 
Los/as autores/as y Bioaraba – Instituto de Investigación Sanitaria no asumen responsabilidad alguna por el uso, la redistribución o la modificación de este repositorio ni por los posibles daños directos o indirectos derivados de su utilización.
 
Este proyecto tiene fines exclusivos de investigación y/o docencia. No está destinado a su uso clínico, diagnóstico, terapéutico ni asistencial, ni sustituye herramientas certificadas ni la evaluación profesional en entornos sanitarios.
 
Sin perjuicio de lo anterior, el uso del software y del sistema robótico se realizará bajo la responsabilidad de la persona usuaria, quien deberá verificar previamente su idoneidad para el fin concreto y adoptar todas las medidas de seguridad, supervisión y control necesarias en función del entorno y condiciones de uso.

La persona usuaria será responsable de las consecuencias derivadas de un uso inadecuado, negligente o no conforme, incluyendo los posibles daños personales o materiales que pudieran producirse.

En la máxima medida permitida por la legislación aplicable, Bioaraba no asumirá responsabilidad por daños derivados del uso del software o sistema robótico cuando estos provengan de una implementación defectuosa, una supervisión insuficiente o una decisión de uso inapropiada por parte de la persona usuaria, así como por la inobservancia de las medidas de seguridad recomendadas.
