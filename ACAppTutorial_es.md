# Primeros pasos con el desarrollo de aplicaciones para Assetto Corsa

Empezar con el desarrollo de aplicaciones para Assetto Corsa no es difícil, pero es más complicado de lo que debería ser. La sabiduría popular parece ser leer las aplicaciones de ejemplo que vienen con Assetto Corsa (en `/apps/python/`, hay una aplicación de Chat y un gMeter), o leer el código de algunas de las aplicaciones que se comparten en los foros.

Esto ciertamente funciona, y así es como aprendí, pero requiere más prueba y error de lo que debería ser necesario. Por ejemplo, al principio de leer el foro oí hablar de algo llamado `py_log.txt`, but no estaba claro dónde encontrarlo.

En este documento, descorro el telón y muestro explícitamente lo básico. Asumo que ya sabes algo de Python. Aparte de una pequeña nota sobre el tratamiento de las variables globales en Python, no cubro nada de Python.

A menudo, escribiré `AC` como abreviatura de `Assetto Corsa`.

A continuación, voy a desarrollar una aplicación llamada `appName`. Por supuesto, deberías nombrar tu aplicación con algo más descriptivo.

## Preliminares

### Ubicaciones de interés
Hay dos carpetas que debes conocer:

*   El directorio de instalación de Assetto Corsa.
    Lo más probable es que se encuentre en tu directorio de Steam. En Windows, esto se ve como `C:\Program Files (x86)\Steam\steamapps\common\assettocorsa`. Es posible que hayas elegido instalar AC en otro lugar, y confío en que puedas encontrar dónde está.
*   El directorio de documentos de Assetto Corsa. En Windows, esto se ve como `C:\Usuario\Mis Documentos\Assetto Corsa\`, donde `Usuario` se reemplaza con tu nombre de cuenta de Windows.

Cada uno de estos contiene al menos una subcarpeta de interés:

*   En el directorio de instalación, la subcarpeta interesante es `/apps/`. Aquí es donde se coloca la aplicación.
*   En el directorio de documentos, la subcarpeta de interés es `/logs/`. En este directorio encontrarás tanto `log.txt` como `py_log.txt`.

`log.txt` es donde AC registra todo sobre la ejecución de AC mismo. A veces contendrá información relevante sobre tu aplicación que AC registra automáticamente.
`py_log.txt` es donde AC coloca las cadenas de texto que las aplicaciones en ejecución solicitan explícitamente que se registren.

### Flujo de trabajo básico

El flujo de trabajo para probar una aplicación no es el mejor. Puede ser lento cuando estás haciendo muchos cambios, especialmente si cometes errores de sintaxis o lógicos. Intenta ser cuidadoso y asegúrate de que el código que intentas ejecutar es correcto. Quizás quieras investigar algo como pylint para poder encontrar errores en el código sin tener que ejecutar Assetto Corsa.

Si algo está mal en tu código, podría aparecer un mensaje de error automáticamente en la consola del juego o en `py_log.txt`. Siempre aparecerá en `log.txt`. La mejor manera de encontrar un error en `log.txt` es buscando el nombre de tu aplicación, en este caso `appName`, y leyendo la salida circundante.

Llamo a un evento en pista una sesión. Normalmente pruebo en modo de práctica en una pista corta como silverstone-international, pero no debería importar cuál elijas.

Así es como pruebo mi aplicación:

*   Edito el código fuente, luego ejecuto AC y comienzo una sesión.
*   Si no hay errores y la aplicación estaba previamente activa en la pantalla, seguirá estándolo. Si algo salió mal, habrá desaparecido y habrá un mensaje en algún lugar sobre por qué. Si ha ocurrido un error, termino la sesión. De lo contrario, continúo.
*   Hago lo que necesito para probar el comportamiento de la aplicación.
*   Cuando descubro que necesito hacer cambios, termino la sesión.

En cualquier caso - un error o un cambio deseado - no tienes que salir de Assetto Corsa. En su lugar, puedes hacer alt-tab, cambiar el código y luego comenzar una nueva sesión. Esto es más rápido que iniciar y salir continuamente de la aplicación principal de Assetto Corsa. Sigue siendo un poco pesado tener que salir y reiniciar sesiones, así que como señalé antes, intenta ser cuidadoso de que al menos la sintaxis de tu código sea correcta antes de probarlo. De lo contrario, esperas mientras inicias una sesión solo para descubrir que tu aplicación no se ha cargado.

Es un hábito mío revisar siempre la consola al inicio de una sesión. Si algo en mi aplicación está mal, es probable que haya un mensaje en la consola sobre lo que salió mal. También suele ocurrir que el mensaje en la consola contenga el número de línea en mi aplicación donde se encontró el error. Si este no es el caso, el error podría estar en `py_log.txt` o `log.txt`. De nuevo, tienes una buena oportunidad de encontrar un número de línea específico allí, o en el peor de los casos un mensaje de error útil.

## Poniendo en marcha una aplicación

Crea una carpeta en `/apps/python/` con el nombre `appname`. Dentro de `/apps/python/appname/`, crea un archivo `appname.py`. Abre el archivo para editarlo.

### Una aplicación muy básica

Primero, ten en cuenta que una aplicación básica todavía necesita algunas importaciones. No seguiré incrustando esto en los fragmentos de código a continuación, así que ten en cuenta que cada aplicación debería comenzar con las siguientes importaciones:

    import sys
    import ac
    import acsys 

Las aplicaciones más básicas solo requieren unas pocas líneas de código. La arquitectura de plugins de AC ejecutará automáticamente ciertas funciones en las que espera encontrar tu código. Para comenzar una aplicación, debes definir una función de la siguiente manera:

    def acMain(ac_version):
        ...

El código de tu aplicación irá en lugar de los puntos suspensivos. Por ahora, insertaremos el código mínimo:

	def acMain(ac_version):
    	appWindow = ac.newApp("appName")
    	ac.setSize(appWindow, 200, 200)
    	return "appName"

En realidad, el mínimo indispensable es probablemente solo la declaración de retorno, pero esa aplicación no es nada interesante.

Si ejecutas Assetto Corsa y comienzas una sesión, encontrarás en la barra lateral de aplicaciones una entrada llamada `appName`. Si la activas, verás un widget muy básico que consiste en una ventana de aplicación de 200x200 con el nombre `appName` en la parte superior. Esto es lo que deberías ver en la barra lateral de aplicaciones:

![image](https://raw.github.com/ckendell/ACAppTutorial/master/images/sidebar.png?raw=true)
	
### ac.log y ac.console
La función `ac.log` escribe en el archivo `py_log.txt` que mencioné anteriormente.

La función `ac.console` escribe en la consola de Assetto Corsa. Para abrir la consola, presiona la tecla `Home` en tu teclado. Presiona la tecla de nuevo para cerrar la consola.

La forma en que puedes usar estas funciones es bastante similar, así que piénsalo de esta manera:

*   Usa `ac.log` cuando quieras que el texto persista después de que la sesión haya terminado. Esto es útil si quieres depurar la aplicación a través de muchas declaraciones impresas.
*   Usa `ac.console` cuando quieras leer la salida durante la sesión. Al abrir la consola del juego puedes ver inmediatamente los mensajes. Sí, puedes hacer alt-tab y ver el `py_log.txt` mientras la sesión todavía está en marcha, pero no es tan agradable.

Estas funciones son buenos objetivos para volcar información sobre la que no estás muy seguro. Úsalas para averiguar exactamente qué está haciendo un trozo de código.

### Extendiendo la aplicación básica

Cambiemos el código a lo siguiente:

	def acMain(ac_version):
    	appWindow = ac.newApp("appName")
    	ac.setSize(appWindow, 200, 200)

		ac.log("¡Hola, mundo de las aplicaciones de Assetto Corsa!")
		ac.console("¡Hola, consola de Assetto Corsa!")
    	return "appName"

Inicia una nueva sesión y comprueba que ves el texto `¡Hola, consola de Assetto Corsa!` en la consola cuando presionas la tecla `Home`. Además, asegúrate de que `¡Hola, mundo de las aplicaciones de Assetto Corsa!` ha aparecido en el archivo `py_log.txt`.

Como era de esperar, ambos deberían ser el caso. No hubo trucos aquí. Una cosa importante a tener en cuenta es que tu aplicación no tiene uso exclusivo ni de la consola ni del archivo de registro de python. Otras aplicaciones que hayas instalado también podrían estar enviando texto a la consola o al registro de python. Puedes desactivar todas las demás aplicaciones, o prefijar todos los mensajes con una cadena única, por ejemplo, `*** Mensaje de appName:` para que puedas encontrar rápidamente la salida de tu aplicación.

### Añadiendo etiquetas a la ventana de tu aplicación

Si tienes una ventana de aplicación en tu pantalla, probablemente quieras mostrar alguna información dentro de ella. Para hacerlo, podemos añadir etiquetas a la ventana:

	l_lapcount = ac.addLabel(appWindow, "Vueltas: 0");
	ac.setPosition(l_lapcount, 3, 30)

Recuerda, la ventana de tu aplicación es un widget de 200x200. Parte de este espacio está ocupado por el encabezado, donde la etiqueta `appName` aparece automáticamente. Por eso establecí la etiqueta en la posición 30 verticalmente. La establecí en 3 horizontalmente para desplazarla ligeramente del borde.

El código ahora debería verse así:

	def acMain(ac_version):
    	appWindow = ac.newApp("appName")
    	ac.setSize(appWindow, 200, 200)

		ac.log("¡Hola, mundo de las aplicaciones de Assetto Corsa!")
		ac.console("¡Hola, consola de Assetto Corsa!")

		l_lapcount = ac.addLabel(appWindow, "Vueltas: 0");
		ac.setPosition(l_lapcount, 3, 30)
    	return "appName"

y la ventana de la aplicación debería verse así:

![image](https://raw.github.com/ckendell/ACAppTutorial/master/images/barebones.png?raw=true)

### Hacia una aplicación más realista

Hasta ahora, nuestra aplicación consiste solo en la ventana de la aplicación y una etiqueta estática. Añadamos un comportamiento dinámico.

La función `acMain` ha configurado nuestra ventana de aplicación. Para hacer algo con ella, debemos usar una función adicional `acUpdate`.

Una cosa importante a tener en cuenta es que vamos a necesitar acceder a la etiqueta `l_lapcount` desde dentro de `acUpdate` si queremos colocar información dinámica en ella. Hasta ahora, la etiqueta ha sido una variable local de `acMain`. Como no somos nosotros quienes llamamos a `acUpdate`, no podemos pasarle la etiqueta como parámetro. En su lugar, debemos hacer de `l_lapcount` una variable global. Para ello, defínela fuera de `acMain`. Luego, dentro de `acMain` debemos informar a la función que `l_lapcount` es una variable global. Si olvidamos hacer esto, crearemos una variable local `l_lapcount` dentro de `acMain` que ocultará la variable global, y cualquier cambio que hagamos dentro de `acMain` no será visible fuera de ella. Lo más importante, si olvidamos hacer esto, la etiqueta real que colocamos en la ventana de la aplicación en `acUpdate` no estaría disponible desde `acUpdate`.

También añadiremos una variable global `lapcount` que solo necesita ser accesible dentro de `acUpdate`. El código debería verse así:

	l_lapcount=0
	lapcount=0

	def acMain(ac_version):
		global l_lapcount

    	appWindow = ac.newApp("appName")
    	ac.setSize(appWindow, 200, 200)

		ac.log("¡Hola, mundo de las aplicaciones de Assetto Corsa!")
		ac.console("¡Hola, consola de Assetto Corsa!")

		l_lapcount = ac.addLabel(appWindow, "Vueltas: 0");
		ac.setPosition(l_lapcount, 3, 30)
    	return "appName"

	def acUpdate(deltaT):
		global l_lapcount, lapcount
		laps = ac.getCarState(0, acsys.CS.LapCount)
    	if laps > lapcount:
        	lapcount = laps        	        	
        	ac.setText(l_lapcount, "Vueltas: {}".format(lapcount))

`acUpdate` toma un parámetro que es ???(Suposición: ?mili?segundos desde la última vez que se llamó).

Ten en cuenta que dentro de `acUpdate` hacemos una llamada `ac.getCarState(0, acsys.CS.LapCount)`. Esto puede parecer confuso al principio ya que nunca expliqué nada al respecto, pero es solo otra función disponible a través de nuestra importación de `ac` y `acsys`. No quiero duplicar la documentación oficial, así que por favor mira la sección de recursos al final de esta guía para un enlace a la documentación oficial. Eventualmente, deberías leerla para saber qué han puesto a disposición los desarrolladores de Assetto Corsa para el desarrollo de aplicaciones, pero no es importante en este momento. Puedes continuar con esta guía.

Ahora, después de completar una vuelta, la ventana de tu aplicación se verá así:

![image](https://raw.github.com/ckendell/ACAppTutorial/master/images/after_lap.png?raw=true)

y así sucesivamente a medida que completes vueltas.

Podrías, por supuesto, también registrar esta información en la consola o en el registro de python:

	ac.log("{} vueltas completadas".format(lapcount))
    ac.console("{} vueltas completadas".format(lapcount))

## Funciones adicionales llamadas por Assetto Corsa

Una función adicional a tener en cuenta es `acShutdown`, que se llama cuando finaliza la sesión.

    def acShutdown():
       ...

Querrás añadir dentro de `acShutdown` cualquier código que deba completarse antes de que tu aplicación salga. Por ejemplo, si hay modificaciones pendientes en la base de datos, querrás asegurarte de confirmarlas y cerrar de forma segura la conexión a la base de datos.

También puedes registrar callbacks para ciertos eventos. Dos de los que conozco son `ac.addOnAppActivatedListener` y `ac.addOnAppDismissedListener`. Por ejemplo, podrías definir una función `on_activation` y registrarla llamando a

	ac.addOnAppActivatedListener(appWindow, on_activation)

Parece que `acUpdate` siempre se llama, incluso cuando la aplicación está descartada. Si esto no es deseable, el modismo sería verificar una bandera dentro de `acUpdate` que se establece en el callback registrado con `ac.addOnAppActivatedListener`, para que solo ejecutes código cuando la aplicación está activada.

## Accediendo a la Memoria Compartida

La API de Python nos da acceso a algunas cosas interesantes, pero hay mucho más a lo que podrías necesitar acceso para hacer tu aplicación. Para llegar a esta información adicional, debes acceder a la estructura de memoria compartida que Assetto Corsa pone a disposición. Aquí hay un tutorial rápido sobre cómo hacerlo:

*   Añade un directorio dentro de `/apps/python/appname/` con un nombre de tu elección. Yo uso `third_party`.
*   Añade a este directorio `_ctypes.pyd` y `sim_info.py`.
    Consulta [Memoria compartida para aplicaciones Python (sim_info.py) para AC v0.20](http://www.assettocorsa.net/forum/index.php?threads/shared-memory-for-python-applications-sim_info-py-for-ac-v0-20.11382/) para saber dónde obtener estos archivos.
*   Inserta el directorio `third_party` en el entorno de python antes de usar la declaración de importación:
    `sys.path.insert(len(sys.path), 'apps/python/appname/third_party')`.
*   Importa desde `sim_info.py` usando `from sim_info import info`.

Después de hacerlo, puedes acceder a cualquier información de la memoria compartida usando, por ejemplo, `info.physics.fuel`.

**Ejercicio**: Añade una etiqueta adicional a la ventana de la aplicación y llénala con el combustible actual en el tanque. Actualiza el valor dinámicamente a lo largo de la sesión con un período más corto que una vez por vuelta.

Si puedes completar esto, has entendido todo lo que intenté comunicar al escribir este documento.

# Suficiente, por ahora

Eso debería cubrir lo básico y ponerte en marcha para escribir aplicaciones de Assetto Corsa. Todavía hay mucho que necesitarás aprender para hacer una aplicación no trivial, pero creo que lo que cubrí es suficiente para empezar. Si hay interés, podría escribir algunos tutoriales más avanzados en el futuro. Por ahora, gracias por leer y diviértete.

# Recursos

## Recursos Oficiales
[Documentación Python de Assetto Corsa](http://www.assettocorsa.net/forum/index.php?threads/python-doc.517/)
[Referencia de Memoria Compartida](http://www.assettocorsa.net/forum/index.php?threads/shared-memory-reference.3352/)
[Foro: Discusiones de Modding - Lenguaje de Programación - Apps - Temas de GUI](http://www.assettocorsa.net/forum/index.php?forums/programming-language-apps-gui-themes.22/)

## Recursos Notables de la Comunidad
[Memoria compartida para aplicaciones Python (sim_info.py) para AC v0.20](http://www.assettocorsa.net/forum/index.php?threads/shared-memory-for-python-applications-sim_info-py-for-ac-v0-20.11382/)
**NB: Creo que quieres usar len(sys.path) como parámetro para sys.path.insert, no 0.**

# Licencia del Documento

Este trabajo está licenciado bajo la Licencia Internacional Creative Commons Atribución-NoComercial 4.0. Para ver una copia de esta licencia, visita http://creativecommons.org/licenses/by-nc/4.0/.