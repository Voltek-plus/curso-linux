¡Claro! Aquí tienes el contenido que proporcionaste, estructurado y formateado como un archivo `README.md` profesional y fácil de seguir.

He organizado el tutorial en secciones numeradas, he utilizado bloques de código con resaltado de sintaxis, y he añadido notas y advertencias para mejorar la legibilidad y la comprensión.

---

# Configuración del Entorno para Hadoop en Ubuntu

Este tutorial te guiará a través del proceso de instalación y configuración de Apache Hadoop en un entorno Ubuntu.

## Requisitos Previos

*   **Sistema Operativo:** Ubuntu 18.04 64-bit (también compatible con Ubuntu 14.04 y 16.04, 32/64-bit).
*   Se asume que ya tienes una instalación limpia de Ubuntu.

---

## Índice

1.  [Creación de un Usuario Hadoop](#1-creación-de-un-usuario-hadoop)
2.  [Preparación del Sistema y Herramientas](#2-preparación-del-sistema-y-herramientas)
3.  [Instalación y Configuración de SSH](#3-instalación-y-configuración-de-ssh)
4.  [Instalación del Entorno Java (JDK)](#4-instalación-del-entorno-java-jdk)
5.  [Instalación de Hadoop 3.1.3](#5-instalación-de-hadoop-313)
6.  [Configuración y Ejecución de Hadoop](#6-configuración-y-ejecución-de-hadoop)
    *   [Modo Standalone (No Distribuido)](#modo-standalone-no-distribuido)
    *   [Modo Pseudo-Distribuido](#modo-pseudo-distribuido)
7.  [Iniciar y Detener Hadoop](#7-iniciar-y-detener-hadoop)
8.  [Instalación de un Clúster Completo](#8-instalación-de-un-clúster-completo)

---

## 1. Creación de un Usuario Hadoop

Si no creaste un usuario llamado `hadoop` durante la instalación de Ubuntu, sigue estos pasos.

Abre una terminal (`Ctrl` + `Alt` + `T`) y ejecuta el siguiente comando para crear el nuevo usuario:

```bash
sudo useradd -m hadoop -s /bin/bash
```

> **Notas sobre los comandos:**
> *   `sudo`: Permite ejecutar comandos con privilegios de administrador. Te pedirá la contraseña de tu usuario actual.
> *   **Contraseñas en la terminal:** Al escribir una contraseña en la terminal de Linux, no verás ningún carácter (ni siquiera `*`). Simplemente escribe la contraseña y pulsa `Enter`.
> *   **Atajos para Copiar y Pegar:** En la terminal de Ubuntu, los atajos son `Ctrl` + `Shift` + `C` para copiar y `Ctrl` + `Shift` + `V` para pegar.

A continuación, establece una contraseña para el nuevo usuario. Puedes usar `hadoop` como contraseña para simplificar.

```bash
sudo passwd hadoop
```

Agrega el usuario `hadoop` al grupo `sudo` para darle privilegios de administrador y evitar problemas de permisos.

```bash
sudo adduser hadoop sudo
```

Finalmente, **cierra la sesión actual** (haz clic en el menú de la esquina superior derecha y selecciona "Cerrar sesión") y vuelve a iniciar sesión con el usuario `hadoop`.

## 2. Preparación del Sistema y Herramientas

### Actualizar APT

Una vez que hayas iniciado sesión como `hadoop`, abre una terminal y actualiza la lista de paquetes del sistema.

```bash
sudo apt-get update
```

### (Opcional) Solucionar Problemas con las Fuentes de Software

Si durante la actualización encuentras un error como `Hash checksum mismatch`, puedes solucionarlo cambiando el servidor de descargas.

1.  Abre **Configuración del sistema** (icono de engranaje).
2.  Selecciona **Software y actualizaciones**.
3.  En la pestaña "Software de Ubuntu", haz clic en el menú desplegable junto a "Descargar desde:" y selecciona **"Otro..."**.
4.  En la nueva ventana, busca y selecciona un servidor cercano o uno conocido por su fiabilidad (ej. `mirrors.aliyun.com` o `mirrors.163.com`) y haz clic en **"Seleccionar servidor"**.
5.  Cierra la ventana de "Software y actualizaciones". El sistema te pedirá recargar la información de los paquetes. Haz clic en **"Recargar"** y espera a que el proceso termine.
6.  Vuelve a ejecutar `sudo apt-get update`.

### Instalar Editor de Texto (Vim)

Se recomienda instalar un editor de texto de terminal como `vim`. Si no estás familiarizado con `vim`, puedes usar `gedit` en su lugar.

```bash
sudo apt-get install vim
```

> **Uso Básico de Vim:**
> *   **Modo Normal:** El modo por defecto para navegar. Pulsa `Esc` en cualquier momento para volver a este modo.
> *   **Modo Inserción:** Para escribir texto. Pulsa la tecla `i` en modo normal para entrar.
> *   **Guardar y Salir:** En modo normal, escribe `:wq` y pulsa `Enter` para guardar los cambios y salir.

## 3. Instalación y Configuración de SSH

Hadoop necesita SSH para gestionar sus nodos, incluso en modo de un solo nodo. Ubuntu ya incluye el cliente SSH; solo necesitamos instalar el servidor.

```bash
sudo apt-get install openssh-server
```

Una vez instalado, puedes probar la conexión localmente:

```bash
ssh localhost
```

La primera vez te pedirá confirmar la autenticidad del host (escribe `yes`) y luego la contraseña del usuario `hadoop`. Para evitar tener que escribir la contraseña cada vez, configuraremos un inicio de sesión sin clave.

```bash
# Sal del ssh si sigues conectado
exit

# Navega al directorio .ssh (si no existe, el comando 'ssh localhost' anterior lo habrá creado)
cd ~/.ssh/

# Genera un par de claves RSA. Simplemente presiona Enter en todas las preguntas.
ssh-keygen -t rsa

# Añade la clave pública a la lista de claves autorizadas
cat ./id_rsa.pub >> ./authorized_keys
```

Ahora, si vuelves a ejecutar `ssh localhost`, deberías poder iniciar sesión sin que te pida la contraseña.

## 4. Instalación del Entorno Java (JDK)

Hadoop 3.1.3 requiere **JDK 1.8 o superior**.

1.  Descarga el paquete de instalación de JDK 1.8. Puedes encontrar `jdk-8u162-linux-x64.tar.gz` en línea o usar el proporcionado por tu tutorial. Asumiremos que el archivo se ha descargado en el directorio `~/Downloads`.

2.  Crea un directorio para el JDK y descomprime el archivo.

```bash
# Crea el directorio de destino
sudo mkdir -p /usr/lib/jvm

# Navega al directorio de descargas
cd ~/Downloads/

# Descomprime el archivo JDK en el directorio de destino
sudo tar -zxvf ./jdk-8u162-linux-x64.tar.gz -C /usr/lib/jvm
```

3.  Configura las variables de entorno de Java. Abre el archivo `.bashrc` con `vim`:

```bash
vim ~/.bashrc
```

4.  Añade las siguientes líneas al principio del archivo. Pulsa `i` para entrar en modo inserción.

```bash
export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_162
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

5.  Guarda y cierra el archivo (pulsa `Esc` y luego escribe `:wq`).

6.  Aplica los cambios a la sesión actual de la terminal:

```bash
source ~/.bashrc
```

7.  Verifica que Java se ha instalado correctamente:

```bash
java -version
```

Deberías ver una salida similar a esta:

```
java version "1.8.0_162"
Java(TM) SE Runtime Environment (build 1.8.0_162-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.162-b12, mixed mode)
```

## 5. Instalación de Hadoop 3.1.3

1.  Descarga `hadoop-3.1.3.tar.gz` desde el [sitio web oficial de Apache Hadoop](https://hadoop.apache.org/releases.html) o la fuente proporcionada en tu tutorial. Asumiremos que está en `~/Downloads`.

2.  Instala Hadoop en el directorio `/usr/local`.

```bash
# Descomprime el archivo en /usr/local
sudo tar -zxf ~/Downloads/hadoop-3.1.3.tar.gz -C /usr/local

# Navega al directorio de instalación
cd /usr/local/

# Renombra la carpeta a 'hadoop' para simplificar
sudo mv ./hadoop-3.1.3/ ./hadoop

# Asigna la propiedad de la carpeta al usuario 'hadoop'
sudo chown -R hadoop ./hadoop
```

3.  Verifica que la instalación se ha realizado correctamente:

```bash
/usr/local/hadoop/bin/hadoop version
```

## 6. Configuración y Ejecución de Hadoop

### Modo Standalone (No Distribuido)

Este es el modo por defecto. No requiere configuración adicional y es útil para la depuración.

Ejecutemos un ejemplo para ver cómo funciona. Usaremos el ejemplo `grep` que busca un patrón en los archivos de configuración de Hadoop.

```bash
cd /usr/local/hadoop

# Crea un directorio de entrada
mkdir ./input

# Copia los archivos de configuración XML al directorio de entrada
cp ./etc/hadoop/*.xml ./input

# Ejecuta el trabajo MapReduce (grep)
./bin/hadoop jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar grep ./input ./output 'dfs[a-z.]+'

# Muestra los resultados
cat ./output/*
```

> **Importante:** Hadoop no sobrescribe los directorios de salida. Si quieres volver a ejecutar el ejemplo, primero debes eliminar el directorio `output`: `rm -r ./output`.

### Modo Pseudo-Distribuido

En este modo, todos los demonios de Hadoop (NameNode, DataNode, etc.) se ejecutan como procesos Java separados en un único nodo.

Necesitamos modificar dos archivos de configuración: `core-site.xml` y `hdfs-site.xml`.

1.  **Edita `core-site.xml`:**

    ```bash
    vim ./etc/hadoop/core-site.xml
    ```

    Añade la siguiente configuración entre las etiquetas `<configuration>` y `</configuration>`:

    ```xml
    <property>
        <name>hadoop.tmp.dir</name>
        <value>file:/usr/local/hadoop/tmp</value>
        <description>Abase for other temporary directories.</description>
    </property>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    ```

2.  **Edita `hdfs-site.xml`:**

    ```bash
    vim ./etc/hadoop/hdfs-site.xml
    ```

    Añade la siguiente configuración:

    ```xml
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/usr/local/hadoop/tmp/dfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/usr/local/hadoop/tmp/dfs/data</value>
    </property>
    ```

3.  **Formatea el NameNode:**
    Este paso inicializa el sistema de archivos HDFS. **¡Solo se debe hacer la primera vez!**

    ```bash
    ./bin/hdfs namenode -format
    ```

    Si todo va bien, verás un mensaje que contiene `Storage directory ... has been successfully formatted`.

    > **Solución de Problemas:** Si recibes un error `Error: JAVA_HOME is not set`, significa que la variable de entorno no se ha configurado correctamente. Ve al directorio de instalación de Hadoop y edita el archivo `./etc/hadoop/hadoop-env.sh`. Descomenta y modifica la línea `export JAVA_HOME` para que apunte a tu directorio de Java, por ejemplo: `export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_162`.

4.  **Inicia los demonios de Hadoop:**

    ```bash
    ./sbin/start-dfs.sh
    ```

5.  **Verifica que los procesos se están ejecutando:**
    Usa el comando `jps` (Java Virtual Machine Process Status Tool).

    ```bash
    jps
    ```

    Deberías ver los procesos `NameNode`, `DataNode` y `SecondaryNameNode` en la lista.

    > **Solución de Problemas:** Si `DataNode` no se inicia, puedes intentar la siguiente solución (¡cuidado, esto eliminará todos los datos en HDFS!):
    > ```bash
    > ./sbin/stop-dfs.sh
    > rm -r ./tmp
    > ./bin/hdfs namenode -format
    > ./sbin/start-dfs.sh
    > ```

6.  **Accede a la Interfaz Web:**
    Abre tu navegador y ve a `http://localhost:9870` para ver el estado del NameNode y el DataNode.

7.  **Ejecuta el ejemplo en modo pseudo-distribuido:**
    Ahora, los datos de entrada y salida estarán en HDFS.

    ```bash
    # Crea un directorio de usuario en HDFS
    ./bin/hdfs dfs -mkdir -p /user/hadoop

    # Crea un directorio de entrada dentro del directorio de usuario
    ./bin/hdfs dfs -mkdir input

    # Sube los archivos de configuración locales a HDFS
    ./bin/hdfs dfs -put ./etc/hadoop/*.xml input

    # Ejecuta el trabajo MapReduce (los datos se leen y escriben en HDFS)
    ./bin/hadoop jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar grep input output 'dfs[a-z.]+'

    # Muestra los resultados directamente desde HDFS
    ./bin/hdfs dfs -cat output/*
    ```

    > **Nota:** Al igual que antes, si quieres volver a ejecutar el trabajo, primero debes eliminar el directorio `output` de HDFS: `./bin/hdfs dfs -rm -r output`.

## 7. Iniciar y Detener Hadoop

*   **Para iniciar Hadoop (NameNode y DataNode):**

    ```bash
    /usr/local/hadoop/sbin/start-dfs.sh
    ```

*   **Para detener Hadoop:**

    ```bash
    /usr/local/hadoop/sbin/stop-dfs.sh
    ```

    **Importante:** La próxima vez que inicies el sistema, no necesitas volver a formatear el NameNode. Simplemente ejecuta `start-dfs.sh`.

## 8. Instalación de un Clúster Completo

Para configurar un clúster de Hadoop con múltiples nodos, el proceso es más complejo. Puedes consultar tutoriales especializados para ello, como el **Tutorial de Configuración de Instalación de Hadoop Cluster (Hadoop 3.1.3)**.
