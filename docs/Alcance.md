# Alcance del Proyecto: "FútBot" 
## Descripción General del Proyecto 
El proyecto consiste en el desarrollo de “FútBot”, un juego de fútbol en dos dimensiones (2D), multijugador y ejecutable en entorno web, que integra el ámbito futbolístico con la programación en el lenguaje Python. La dinámica del juego se basa en partidos de fútbol en tiempo real donde los futbolistas son entidades virtuales (bots). Dichas entidades ejecutan un comportamiento automatizado mediante scripts de Python redactados por el usuario y configurados con atributos personalizados, permitiendo el diseño e implementación de diversas estrategias tácticas.

## Gestión de Cuentas y Autenticación 
El registro e inicio de sesión en la plataforma requiere de un nombre, una contraseña y una dirección de correo electrónico con formato correcto. Se restringe la existencia de nombres duplicados y la creación de más de una cuenta por usuario.

## Funcionalidades del Sistema y Gestión del Club 
Cada usuario dispondrá de un club propio, con la facultad de personalizar su denominación e insignia (logo). Los nombres de los clubes deben ser únicos dentro del sistema y no se permitirá la creación de múltiples clubes por usuario.

Dentro del club, el usuario podrá realizar las siguientes configuraciones:
- Creación de jugadores: Se define el nombre del futbolista y la asignación de su conjunto de estadísticas denominadas PACSS. Dichas estadísticas se construyen mediante la distribución de 300 puntos entre cinco variables distintas, permitiendo configurar perfiles enfocados en potencia de disparo, velocidad u otros atributos.
- Creación de comportamientos: Consiste en la programación de scripts en lenguaje Python, utilizando conocimientos básicos y primitivas suministradas por la plataforma. Estos scripts definen la lógica y las acciones que los jugadores ejecutarán en la cancha en función de los eventos del partido.

## Modalidades de Juego y Sistema de Torneos 
Para acceder a la sección de juego, el club deberá contar con un mínimo de seis jugadores creados y un comportamiento registrado. Las modalidades disponibles corresponden a:
- Ligas: Torneos conformados por un mínimo de tres usuarios bajo el formato de competencia de todos contra todos. Se otorgan tres puntos por partido ganado, un punto por empate y cero puntos por derrota; resulta ganador el usuario que acumule mayor puntaje. Previo al inicio de la liga, el usuario debe seleccionar tres jugadores titulares, tres suplentes, la formación inicial y un comportamiento asignado a cada titular. Una vez iniciada la liga, el usuario no podrá retirarse de la misma; sin embargo, podrá inscribirse en otras ligas en paralelo, ya que el sistema ejecuta automáticamente los partidos utilizando los scripts y alineaciones predefinidos. Las ligas pueden configurarse con visibilidad pública o privada. Las ligas privadas restringen el acceso a usuarios no autorizados y no otorgan puntos para el escalafón (ranking) global.
- Partidos Amistosos: Modalidad uno contra uno accesible desde el menú de juego mediante invitación directa a otro club. Requiere la misma selección previa de titulares, suplentes, comportamiento y formación, y computa sus resultados para el ranking global (3 puntos si ganas, 1 si empatas y 0 si perdes).
- Espectador: Los usuarios pueden visualizar partidos en desarrollo pertenecientes a su misma liga en los que no participen como competidores, sin interferir en el desarrollo del encuentro.
- Ranking Global: Clasificación general de usuarios estructurada según la cantidad de puntos ganados, encuentros, total de goles convertidos.

## Dinámica y Reglas del Partido
 Los partidos tienen una duración fijada al crear la liga o amistoso. El usuario asume el rol exclusivo de director técnico, por lo que no controla manualmente a los futbolistas en tiempo real, sino que establece su comportamiento y gestiona las sustituciones.
- Estructura temporal y pausas: El partido se divide en 4 cuartos, con una pausa entre cada cuarto (3 pausas en total); primer cooling break, entretiempo y segundo cooling break. 
- Sistema de cambios de jugadores: Durante las tres pausas indicadas se activa el evento automático de sustitución de jugadores. La decisión del cambio debe ser configurada previamente por el usuario antes de que ocurra la pausa. Cada usuario dispone de un máximo de tres cambios por partido (uno por cada pausa), los cuales no son acumulables. Un jugador titular sustituido no puede reingresar al campo durante el mismo encuentro. Al finalizar el partido, las alineaciones y comportamientos se restablecen a la configuración inicial ingresada en el formulario de acceso.
- Ajustes de comportamiento: El usuario puede modificar el script de comportamiento de sus jugadores de forma individual y de manera ilimitada en cualquier momento del encuentro, permitiendo adaptar la estrategia en función del desarrollo del juego, los cambios del adversario y la cantidad de comportamientos programados disponibles.

## Exclusiones del Alcance (Límites del Proyecto)
 El sistema no contempla las siguientes funcionalidades:
- Reglamento arbitral: no existen tarjetas amarillas, tarjetas rojas, faltas, árbitros, penales, saques de banda (laterales) ni saques de meta.
- Gestión de datos de partido: no se incluye la grabación ni el almacenamiento de simulaciones de partidos para su posterior reproducción.
- Rendimiento físico: no existe un sistema de energía o desgaste físico; los jugadores mantienen el mismo rendimiento de forma indefinida a lo largo de los encuentros.
- Modulo económico e interacción social: no se implementan sistemas monetarios, compraventa de futbolistas, elementos cosméticos, listas de amigos ni salas de chat.
