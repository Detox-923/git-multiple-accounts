# Configuración de varias cuentas de git en un mismo PC

Para realizar esta configuración, lo primero que necesitamos es crear nuestras claves ssh, una para cada cuenta.
Para esto, usaremos los siguientes comandos:

## 1. Creación de claves ssh

Este comando es valido para cuanlquier sistema operativo. Y para facilidad, lo ideal sería que se ejecute dentro del directorio ssh de su sistema.

```bash
ssh-keygen -t rsa -b 4096 -C "user@mail.com" -f id_rsa_detox
```

Al ejecutar el comando, nos pedirá ingresar una "contraseña" o frase de seguridad, esta es opcional, pero puedes definir lo que quieras.
Al terminar de ejecutarse, esto creará dos archivos, uno con extensión .pub correspondiente a la llave publica y otro sin extensión, este último es la clave ssh.

Explicación de sus parámetros:

- -t rsa: Especifica el tipo de clave (en este caso, rsa).
- -b 4096: Define el tamaño de la clave en bits (4096 bits).
- -C "<user@mail.com>": Añade un comentario, generalmente el correo electrónico para identificar la clave.
- -f ~/.ssh/id_rsa_detox: Define el archivo de salida para almacenar la clave privada. La clave pública se almacenará en un archivo con el mismo nombre, pero con la extensión .pub.

## 2. Agregar claves al agente ssh

Para agregar las claves ssh al agente, primero debemos verificar que este este ejecutandose. Aquí depenede del sistema operativo.
En caso de estar usando windows, puedes verificarlo desde la powershell usando el comando:

```bash
Get-Service -Name ssh-agent
```

Si estás usando macOS o Linux, puedes usar el comando:

```bash
ps aux | grep ssh-agent
```

Si el comando no retorna nada (macOS o Linux) o muestran un estado de apagado o detenido (Windows), debemos encender el agente. Para esto, debemos ejecutar los siguientes comandos, dependiendo del sistema operativo.

Para Windows, primero configuramos el arranque automático del agente y luego lo encendemos:

```bash
Set-Service -Name ssh-agent -StartupType Automatic
```

```bash
Start-Service -Name ssh-agent
```

Para macOS y Linux, debes ejecutar el comando:

```bash
eval "$(ssh-agent -s)"
```

Una vez arrancado el agente, procedemos a agregar las claves ssh. Para esto ejecutamos el comando:

```bash
ssh-add ~/.ssh/id_rsa
```

Para macOS, el comando es un poco diferente:

```bash
ssh-add -K ~/.ssh/id_rsa
```

## 3. Agregar llaves publicas en Github

Para esto Vamos a GitHub > settings > SSH and GPG keys > New SSH key.
Agregamos un título y copiamos el contenido del archivo **.pub** al cuadro de texto llamado key.
Repetimos este proceso para todas las cuentas que queramos configurar y que ya le hemos creados los archivos ssh necesarios.

## 4. Crear o editar archivo config ssh

Primero, debemos mover nuestros archivos ssh generamos en el paso 1, dentro del directorio ssh.
Después, buscamos un archivo llamado **config**. En caso existir, se debe de crear uno.
Dentro de este debemos definir las siguientes configuraciones, reemplazando los datos según sea necesario:

```ini
# Configuración para la cuenta 1
Host user1-github.com
  User          User1
  HostName      github.com
  IdentityFile  ~/.ssh/id_rsa_user1

# Configuración para la cuenta 2
Host user2-github.com
  User          User2
  HostName      github.com
  IdentityFile  ~/.ssh/id_rsa_user2

# Agregar las cuantas que se necesiten...

# Configuraciones globales
Host *
  AddKeysToAgent            yes       # Añadir claves al agente SSH (si tienes passphrase)
  IdentitiesOnly            yes       # Asegura usar solo las claves especificadas
  PreferredAuthentications  publickey # Solo autenticación con claves públicas
```

Podemos tomar de referencia el archivo **config.example** que se encuentra dentro de este repositorio.
De esta configuración, es importante tener en cuenta lo definido en **Host**, ya que se usará más adelante.

## 5. Crear directorios para cada usuario

Para que este proceso funcione de forma correcta, debemos crear un directorio especifico donde vamos manipular git con cada usuario.
Por ejemplo, si queremos realizar la configuración para dos cuentas, vamos a crear una carpeta llamada "Proyectos" y dentro de ella colocamos una carpeta para cada usuarios.
De tal manera que podría quedar algo así: Disk:/ruta/Proyectos/User1/ y Disk:/ruta/Proyectos/User2/

## 6. Crear o editar archivos .gitconfig

Ubicamos dentro de nuestro equipo el archivo .gitconfig. En caso de que no exista, lo creamos.

- En Linux y macOS: ~/.gitconfig
- Windows: C:\Users\<tu-usuario>\.gitconfig

De la misma manera, creamos tantos archivos, como cuentas queramos manejar, con el siguiente formato: **.gitconfig.[user]**
Donde "user" hace referencia al usuario que queremos configurar.

Dentro de los archivos escribiremos las siguientes configuraciones, cambiando los datos según corresponda:

```ini
[user]
    email = user1@mail.com
    name = User1

[url "git@user1-github.com"]
    insteadOf = git@github.com
```

Esto lo hacemos para todos los archivos que se crearon para cada usuario que vamos a manejar.
Tener muy en cuenta que el valor que definas dentro de la directiva **[url "git@user1-github.com"]** debe coincider con el valor definido en **Host** en el archivo config ssh que menciona en el paso 4.
Como en este caso, el valor en ambos valores es **user1-github.com**.
Podemos usar de referencia el archivo **.gitconfig.user1.example** de este repositorio.

Ahora. Dentro del archivo **.gitconfig**, vamos a escribir las siguientes configuraciones:

```ini
[include]
    path = ~/.gitconfig.user1

[includeIf "gitdir:Disk:/ruta/User1/"]
    path = ~/.gitconfig.user1

[includeIf "gitdir:Disk:/ruta/User2/"]
    path = ~/.gitconfig.user2

[init]
    defaultBranch = main
```

Con la directiva [Include] se le indica a git que cualquier configuración dentro de ~/.gitconfig.user1 se aplicará a todos los repositorios, a menos que esté sobrescrita por configuraciones más específicas.

La directiva [includeIf "gitdir:..."] permite cargar configuraciones específicas cuando trabajas en repositorios dentro de ciertas carpetas.
En este caso, cada vez que trabajes en proyectos dentro de Disk:/ruta/User1/, Git cargará también las configuraciones en ~/.gitconfig.user1

Se puede usar de referencia el archivo **.gitconfig.example** de este repositorio.

## 7. Usar git

Ya para finalizar este proceso, el último paso es probar las configuraciones.
Para esto, podemos ir a github y clonar un repositorio usando la opción SSH.
Al clonar el repositorio demos cambiar el host que nos entrega github por el que definimos en nuestras configuraciones.
Por ejemplo, si github nos muestra este comando para realizar la clonación:

```bash
git@github.com:User1/project-test.git
```

Demos cambiarlo por el siguiente:

```bash
git@user1-github.com:User1/project-test.git
```

Debemos hacer lo mismo para si queremos, por ejemplo, enlazar un proyecto local con uno remoto.

Con esto terminamos este tutorial, espero les sirva y no olviden pasar por el video donde explico cómo realizar esta configuración en windows.
