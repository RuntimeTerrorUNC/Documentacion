# Casos de Uso
## Menú de Sesión
1. Autenticar Usuario-
2. Registrar Usuario
3. Cerrar sesión
4. Crear club

## Menú del Club
5. Cambiar nombre club
6. Cambiar avatar 
7. Crear jugador
8. Crear comportamiento
9. Listar comportamiento
10. Ver comportamiento
11. Modificar comportamiento
12. Eliminar comportamiento
13. Listar jugadores

## Menú de juego
14. Listar ligas 
15. Unirse a liga 
16. Salir de liga 
17. Eliminar liga
18. Ver clubes
19. Invitar a Partido Amistoso
20. Unirse a partido amistoso 
21. Crear liga 
22. Iniciar liga

## Durante la liga
23. Ver partido
24. Dirigir partido
25. Cambia comportamiento de jugador durante partido
26. Ver jugadores de otro club 
27. Ver plantel de un club en una liga X
28. Ver plantel de mi club en una liga X
29. Cambiar jugador durante partido
30. Ver Tabla de liga
31. Ver Tabla global
32. Ver Partidos jugados 
33. Ver Fixture 
34. Cambiar Plantel Titular de una Liga




## Caso de Uso 1:

Título:  Autenticar Usuario

Actor: Usuario

Precondición: El Usuario no está Logueado

Caso de Exito (Flujo Principal):
1. El usuario ingresa credenciales user y password.
2. El sistema chequea las credenciales.
3. El Usuario comienza a jugar.


Casos Excepcionales:

2. Los datos ingresados no cumplen los requisitos &rArr; El sistema solicita reingresar los datos inválidos.


--- 

## Caso de Uso 2:

Título:  Registrar Usuario

Actor: Usuario

Precondición: El Usuario no está Autenticado ni Registrado

Caso de Exito (Flujo Principal):
1. El Usuario ingresa un nombre, email y password.
2. El sistema valida los datos ingresados.
3. El sistema muestra pantalla de inicio  
Casos Alternativos: 

Casos Excepcionales:

2. Los datos ingresados no cumplen los requisitos (casilla vacias, nombre ya registrado, email ya registrado, password incorrecto) &rArr; El sistema informa al usuario los datos que no cumplen y le solicita que ingrese nuevamente.

---
## Caso de Uso 3:

Título:  Cerrar Sesión

Actor: Usuario

Precondición:  El Usuario debe esta Logueado; Tener Club

Caso de Exito (Flujo Principal):
1. El usuario elige cerrar sesión
2. Se le muestra un mensaje con la opción de confirmar o cancelar
3. El usuario elige la opción confirmar
4. El sistema redirige al usuario a la página de autenticación

Casos Alternativos: 
3. El usuario cancela la acción cancelar
4. El Sistema quita la ventana de cierre de sesión y el usuario sigue jugando

---
## Caso de Uso 4:

Título:  Crear Club

Actor: Usuario

Precondición: El Usuario está autenticado y No tener club

Caso de Exito (Flujo Principal):
1. Usuario ingresa un nombre y un avatar
2. El sistema valida los datos ingresados y notifica que el club fue creado.
3. El Usuario ingresa al Home

Casos Excepcionales:
- 2.a) El nombre ingresado no cumple con las restricciones del campo (casilla vacías, caracteres invalido, nombre ya registrado) &rArr; Se informa al usuario que ingrese nuevamente un nombre.
- 2.b) El avatar ingresado no cumple con las restricciones (casilla vacías, formato invalido) &rArr;Se informa al usuario que el dato no cumple con tal restricción y se le pide que ingrese nuevamente.

## Menú del Club
---
## Caso de Uso 5:

Título:  Cambiar Nombre del Club

Actor: Usuario

Precondición: Usuario autenticado y con club registrado.

Caso de Exito (Flujo Principal):
1. El usuario solicita cambiar el nombre de su club.
2. El sistema despliega un campo requiriendo el nuevo nombre.
3. El usuario ingresa el nuevo nombre.
4. El sistema valida que el texto cumpla con la longitud permitida, no contenga caracteres inválidos y garantice la unicidad global del nombre frente a otros clubes existentes.
5. El sistema actualiza el nombre del club en la base de datos.
6. El sistema notifica que el cambio se realizó correctamente.

Casos Excepcionales:

- 4.a) Nombre duplicado &rArr; El sistema rechaza el cambio, informa que el nombre ya está en uso por otro club bajo la regla de unicidad global y solicita uno nuevo.
- 4.b) Caracteres inválidos / Longitud excedida &rArr; El sistema rechaza el cambio, especifica la regla de formato infringida y solicita corregir el texto.

---
## Caso de Uso 6:

Título:  Cambiar Avatar

Actor: Usuario

Precondición: Usuario autenticado y con club registrado.

Caso de Exito (Flujo Principal):
1. El usuario solicita actualizar el avatar de su club.
2. El sistema solicita la carga de un archivo de imagen.
3. El usuario selecciona y sube un archivo.
4. El sistema valida el archivo subido, procesa y reemplaza el avatar anterior y le informa que la actualización fue exitosa.

Casos Excepcionales:
- 4.a) La extensión del archivo no es válida &rArr; El sistema rechaza la carga, le informa que solo admite formato .jpg y le solicita que suba nuevamente el archivo
- 4.b) El archivo excede el peso máximo &rArr; El sistema bloquea la carga, le informa el límite en megabytes(MB) y solicita subir un archivo más liviano

---
## Caso de Uso 7:

Título:  Crear Jugador

Actor: Usuario

Precondición: Usuario Autenticado; Tener Club;

Caso de Exito (Flujo Principal):
1. El Usuario solicita la creación de un nuevo jugador.
2. El Sistema solicita el nombre del jugador y la asignación numérica para sus 5 skills (Poder, Agilidad, Control, Velocidad y Fuerza).
3. El Usuario ingresa el nombre y distribuye 300 puntos entre los atributos.
4. El Sistema confirma los datos ingresados, crea y registra al jugador en el club y confirma la creación al usuario

Casos Excepcionales:
- 4.a) La suma de las skills es superior a 300 pts &rArr; El sistema le notifica la diferencia de puntos y solicita reajustar los valores.
- 4.b) Nombre duplicado &rArr; El sistema le informa que el nombre ya existe en el club y solicita uno nuevo.
- 4.c) Límite de jugadores alcanzado &rArr; El sistema le informa que alcanzó el límite de jugadores y le sugiere que elimine un jugador para crear al nuevo jugador

---

---
## Caso de Uso 8:

Título:  Crear Comportamiento

Actor: Usuario

Precondición: Usuario Autenticado; Tener Club

Caso de Exito (Flujo Principal):
1. El Usuario solicita crear un nuevo comportamiento.
2. El Sistema despliega el formulario requiriendo nombre y el código Python del comportamiento.
3. El Usuario ingresa el nombre y escribe el comportamiento de Python.
4. El Sistema valida la entrada, almacena el nuevo comportamiento y confirma la creación.

Casos Excepcionales:
- 4.a) Nombre duplicado &rArr; El sistema rechaza el registro, informa el conflicto de nombre y solicita ingresar uno diferente.
- 4.b) Código no válido / Primitivas prohibidas &rArr; El sistema rechaza el código, informa el error de sintaxis y solicita corregirlo.

---
## Caso de Uso 9:

Título:  Listar Comportamientos

Actor: Usuario

Precondición: Usuario Autenticado; Tener Club

Caso de Exito (Flujo Principal):
1. El usuario solicita listar comportamientos
2. El sistema lista todos lo comportamientos de su club con su respectivo nombre

Casos Alternativos: 

Casos Excepcionales:

---
## Caso de Uso 10:

Título:  Ver Comportamiento

Actor: Usuario

Precondición: Usuario Autenticado; Tener Club

Caso de Exito (Flujo Principal):
1. El usuario solicita ver un comportamiento
2. El sistema solicita el nombre del comportamiento especifico 
3. El usuario ingresa el nombre del comportamiento
4. El sistema valida el nombre y le muestra el codigo python del comportamiento

Casos Excepcionales:

4. El nombre del comportamiento no existe &rArr; El sistema le informa que el comportamiento no existe y que vuelva a ingresar un nombre especifico
---
## Caso de Uso 11:

Título:  Modificar Comportamiento

Actor: Usuario

Precondición: Usuario autenticado ; Tener Club ; Al menos dos comportamientos (¿2?)

Caso de Exito (Flujo Principal):
1. El usuario solicita modificar un comportamiento.
2. El sistema le pide ingrese el nombre del comportamiento
3. El usuario ingresa el nombre del comportamiento
4. El sistema verifica que el comportamiento exista y que el numero de comportamientos sea >1. Espera confirmacion del usuario
5. El usuario modifica el código y confirma la modificación.
6. El sistema modifica permanentemente el comportamiento.

Casos Alternativos: 

Casos Excepcionales:

- El comportamiento no existe &rArr; El sistema indica la falla en el paso número 4. Espera que el usuario seleccione otro comportamiento.
- La nueva sintaxis es inválida &rArr; El sistema indica que hay errores de sintaxis

---
## Caso de Uso 12:

Título:  Eliminar Comportamiento

Actor: Usuario

Precondición: Usuario autenticado, con club registrado y con al menos dos comportamientos.

Caso de Exito (Flujo Principal):
1. El Usuario solicita eliminar un comportamiento.
2. El Sistema pide el NOMBRE del comportamiento
3. El Usuario pone el nombre del comportamiento a eliminar
4. El Sistema verifica que el comportamiento exista y que el numero de comportamientos sea >1. Espera confirmacion del usuario
5. El Sistema verifica que ningún jugador convocado tenga este comportamiento
6. El usuario confirma la eliminación del comportamiento
7. El sistema elimina permanentemente el comportamiento.


Casos Alternativos: 

Casos Excepcionales:
- El comportamiento no existe &rArr;  El sistema indica la falla en el paso número 4. Espera que el usuario seleccione otro comportamiento.

---
## Caso de Uso 13:

Título:  Listar Jugadores

Actor: Usuario

Precondición: Usuario Logueado; Tener Club.

Caso de Exito (Flujo Principal):
1. El usuario solicita listar jugadores
2. El sistema lista todos lo jugadores de su club con sus respectivos nombres y skills

Casos Alternativos: 

Casos Excepcionales:

---
## Caso de Uso 14:

Título:  Listar Ligas

Actor: Usuario

Precondición: El usuario debe tener club y estar logueado 

Caso de Exito (Flujo Principal):
1. El usuario solicita listar ligas activas
2. El sistema lista todas las ligas activas del juego con su respectiva id_liga, nombre, clubes esperando(ligas a la que si se puede unir o sea que no empezaron )

Casos Alternativos: 

Casos Excepcionales:

---
## Caso de Uso 15:

Título:  Unirse a Liga

Actor: Usuario

Precondición:  Usuario autenticado, con club registrado y 6 jugadores disponibles

Caso de Exito (Flujo Principal):
1. El usuario selecciona una liga de la lista de ligas.
2. El sistema verifica que no se exceda la capacidad de la liga y pide la contraseña de la misma al usuario.
3. El usuario ingresa la contraseña de la liga (en caso de ser necesaria) y confirma la petición
4. El sistema verifica la contraseña y solicita una lista de convocados al usuario.
5. El usuario selecciona sus jugadores titulares y suplentes y envía la lista del equipo.
6. El sistema actualiza la lista de clubes en la liga e ingresa al usuario y a sus jugadores.


Casos Alternativos: 

Casos Excepcionales:
- La capacidad de la liga está al maximo. &rArr; El sistema envia un mensaje al usuario en el paso número 2 indicando que la liga está completa.
- La cantidad de jugadores es insuficiente &rArr;
- Contraseña de liga incorrecta &rArr; El sistema envía en el paso 4 un mensaje (“CONTRASEÑA INCORRECTA. INTENTE NUEVAMENTE”) al usuario.
- Usuario ingreso menos de 6 jugadores -> el sistema informa que necesita convocar a 6 jugadores 3 titulares con comportamientos y 3 suplentes

---
## Caso de Uso 16:

Título: Salir de Liga
Actor: Usuario
Precondición: Usuario autenticado; inscripto en la liga; la liga no ha generado su fixture (no inició); no ser el creador de la liga.

Caso de Éxito (Flujo Principal):
1. El usuario elige la opción "Salir de Liga".
2. El sistema muestra un aviso indicando que perderá su lugar en la sala.
3. El usuario confirma la acción.
4. El sistema remueve al club y a sus jugadores convocados de la liga, liberando el cupo, y redirige al usuario a la pantalla principal.

Casos Alternativos:
3.(a) El usuario rechaza la confirmación &rArr; El sistema cierra el aviso sin hacer cambios.

Casos Excepcionales:
- El usuario es el creador de la liga &rArr;El sistema bloquea la acción e indica que debe usar "Eliminar Liga" en su lugar.
- La liga ya generó su fixture (ya inició) &rArr; El sistema bloquea la salida indicando que el torneo ya está en curso.
---
## Caso de Uso 17:

Título:  Eliminar Liga

Actor: Usuario

Precondición: Estar Autenticado; Tener Club; Haber creado la Liga; ¿Que no haya iniciado?

Caso de Exito (Flujo Principal):
1. El usuario elige la opción de eliminar liga
2. El sistema le muestra un aviso para confirmar o rechazar 
3. El usuario elige la opción confirmar
4. El Sistema elimina la liga y redirige al usuario a la página principal

Casos Alternativos: 
3\.(a) El Usuario elige la opción rechazar &rArr; El Sistema quita el aviso de Eliminar liga.
Casos Excepcionales:

---
## Caso de Uso 18:

Título:  Ver Clubes

Actor: Usuario

Precondición:  Estar Autenticado; Tener Club;

Caso de Exito (Flujo Principal):
1. El usuario solicita listar clubes
2. El sistema lista todos lo clubes del juego con us respectivo nombre y logo

Casos Alternativos: 

Casos Excepcionales:

---
## Caso de Uso 19:

Título:  Invitar a Partido Amistoso

Actor: Usuario Retador

Precondición: Usuario autenticado y con club registrado.

Caso de Exito (Flujo Principal):
1. El Usuario Retador selecciona la sección "Clubes".
2. El sistema muestra la lista de clubes disponibles.
3. El Usuario Retador selecciona un club rival y pulsa "Partido Amistoso".
4. El sistema solicita seleccionar 6 jugadores, formación táctica y comportamientos iniciales.
5. El Usuario Retador configura su alineación y confirma la solicitud.
6. El sistema envía la invitación al Usuario Rival e inicia el temporizador de espera.
7. El sistema notifica al Usuario Retador que la solicitud fue enviada exitosamente.

Casos Alternativos: 
7\.(a) La solicitud es rechazada o expira:
1. El sistema recibe la notificación de rechazo o fin del temporizador e informa al Usuario retador que el partido no se llevará a cabo.

Casos Excepcionales:
3\.(a) El club rival no cumple los requisitos mínimos de plantel:
1. El sistema valida el rival y bloquea la acción informando que no tiene plantel suficiente.
2. El flujo regresa al paso 2.
5\.(a) El Usuario Retador no cumple con los requisitos mínimos de convocatoria:
1. El sistema detecta que faltan jugadores o comportamientos requeridos. Deshabilita el botón de confirmación y muestra el motivo.
2. El usuario cancela la acción y el caso de uso finaliza.

---
## Caso de Uso 20:

Título:  Unirse a Partido Amistoso

Actor: Usuario

Precondición: Usuario autenticado con club registrado y una invitación activa recibida.

Caso de Exito (Flujo Principal):
1. El sistema notifica al Usuario la recepción de un desafío con temporizador de respuesta.
2. El Usuario abre la notificación y selecciona "Aceptar Invitación".
3. El sistema solicita al Usuario seleccionar 6 jugadores, formación táctica y comportamientos iniciales.
4. El Usuario configura su alineación y confirma.
5. El sistema valida la información e inicia el partido para ambos usuarios.

Casos Alternativos: 
2\(a). El Usuario Rival declina la invitación:
1. El Usuario selecciona "Rechazar".
2. El sistema notifica al Usuario Retador la declinación.

2\(b). El temporizador expira sin respuesta:
1. El sistema detecta el vencimiento del tiempo límite. Descarta la invitación y notifica al Usuario Retador.

Casos Excepcionales:
- 4\.(a) El Usuario no cuenta con los requisitos mínimos para armar la plantilla:
- 1. El sistema detecta que el Usuario no puede completar los 6 convocados o tácticas. Cancela la aceptación automáticamente y notifica al retador la falta de plantel.

---
## Caso de Uso 21:

Título:  Crear Liga

Actor: Usuario

Precondición: El usuario debe estar autenticado y tener un club

Caso de Exito (Flujo Principal):
1. El Usuario elige crear una liga
2. El Sistema muestra al usuario los campos requeridos para crear una liga: nombre de la liga, contraseña(opcional), duración de partido, cantidad de equipos
3. El Usuario completa los datos requeridos y da sus 6 jugadores respectivamente 3 titulares(con comportamiento) y sus 3 suplentes
4. El Sistema crea la sala de la liga

Casos Alternativos: 

Casos Excepcionales:
- Los datos ingresados no cumplen los requisitos / Faltan &rArr; El sistema solicita reingresar los datos inválidos.

---
## Caso de Uso 22:

Título: Iniciar Liga
Actor: Usuario (creador de la liga)
Precondición: Usuario autenticado; ser el creador legítimo de la liga; la liga está en estado de espera (no iniciada).

Caso de Éxito (Flujo Principal):
1. El usuario (creador) elige la opción "Iniciar Liga".
2. El sistema verifica que se cumpla el mínimo de 3 clubes inscriptos.
3. El sistema bloquea nuevos ingresos, genera el fixture de todos contra todos e inicializa la tabla de posiciones en cero.
4. El sistema confirma el inicio y el torneo continúa automáticamente.

Casos Excepcionales:
- El usuario no es el creador de la liga &rArr; El sistema bloquea la acción.
- No se alcanzó el mínimo de 3 clubes &rArr; El sistema cancela la operación, informa el motivo y mantiene la sala en espera.
- La liga ya había sido iniciada previamente &rArr; El sistema informa que la acción ya fue realizada.

## Durante La Liga
---
## Caso de Uso 23:

Título:  Ver Partido (de liga?)

Actor: Usuario

Precondición: El Partido debe estar Iniciado / tiempo Prematch; El usuario debe encontrarse en el menú de una liga y no debe ser dueño del ninguno de los clubes que juegan el partido.

Caso de Exito (Flujo Principal):

Casos Alternativos: 
1. El usuario selecciona el partido que quiere ver
2. El sistema provee la interfaz para que el usuario pueda ver el partido

Casos Excepcionales:

---
## Caso de Uso 24:

Título:  Dirigir Partido (de liga?)

Actor: Usuario

Precondición: El Partido debe estar Iniciado. El usuario debe encontrarse en el menú de una liga. El usuario debe ser dueño de alguno de los clubes que juegan ese partido.

Caso de Exito (Flujo Principal):
1. El usuario selecciona el partido que quiere dirigir
2. El sistema provee la interfaz para que el usuario pueda ver el partido e interactuar con los jugadores (cambiar formacion, comportamientos y hacer los cambios)

Casos Alternativos: 

Casos Excepcionales:

---
## Caso de Uso 25:

Título:  Cambiar Comportamiento de Jugador Durante Partido

Actor: Usuario

Precondición: Estar Dirigiendo/Jugando un partido.

Caso de Exito (Flujo Principal):
1. El usuario selecciona al  jugador que desea cambiar el comportamiento
2. El sistema le muestra al usuario qué comportamientos están disponibles
3. El usuario elige el comportamiento del jugador
4. El sistema cambia el comportamiento del jugador y cierra el menú de comportamientos

Casos Alternativos: 
3\.(a) El jugador selecciona el botón “atrás” &rArr; El sistema no realiza cambios y cierra el menú de comportamientos

Casos Excepcionales:

---
## Caso de Uso 26:

Título:  Ver Jugadores De Otro Club

Actor: Usuario

Precondición: Usuario Autenticado

Caso de Exito (Flujo Principal):
1. El Usuario selecciona el club deseado en la sección de “Clubes” y solicita los datos del club
2. El Sistema recopila los datos del club (sus ultimos 5 partidos jugados, nombre, avatar y plantel) y muestra por pantalla.

Casos Alternativos: 

Casos Excepcionales:
- El club no tiene jugadores aún &rArr; La sección jugadores indicará lo siguiente: “El club no ha fichado jugadores aún”

---
## Caso de Uso 27:

Título:  Ver Plantel de un club en una liga

Actor: Usuario

Precondición:  El usuario debe tener club y estar logueado 

Caso de Exito (Flujo Principal):
1. El usuario solicita listar los jugadores de otro club
2. El sistema solicita el nombre del club para listar los jugadores
3. El usuario ingresa el nombre del club
4. El sistema lista todos los jugadores de ese club con su respectivo nombre y skills

Casos Alternativos: 

Casos Excepcionales:
- 3\.(a) El nombre ingresado del club no existe &rArr; el sistema informa que no hay ningún club con ese nombre


---
## Caso de Uso 28:

Título:  Ver Plantel de mi club en una Liga

Actor: Usuario

Precondición: Estar logueado ; El usuario debe tener club; El Usuario debe pertenecer a la liga. 

Caso de Exito (Flujo Principal):
1. El usuario solicita ver su plantel en una liga
2. El sistema solicita una liga, (para devolver el plantel que esté jugando esa liga)
3. El usuario ingresa el nombre de la liga
4. El sistema devuelve el plantel de su club que juega en la liga que solicitó el usuario 

Casos Alternativos: 

Casos Excepcionales:
- El nombre de la liga que ingreso el usuario no existe &rArr; el sistema informa que no se encontró ninguna liga con ese nombre 

---
## Caso de Uso 29:

Título:  Cambiar Jugador Durante Partido

Actor: Usuario

Precondición: Estar Jugando/Dirigiendo un partido; Tener cambios disponibles

Caso de Exito (Flujo Principal):
1. El usuario selecciona al jugador que desea cambiar
2. El sistema le muestra al jugador que jugadores están disponibles para el cambio
3. El usuario elige el jugador que reemplaza al actual 
4. El usuario selecciona el comportamiento que tendrá el jugador entrante.
5. El sistema cierra el menú de elección y cambia de jugador cuando ocurra el evento de “pausa de hidratación” o “medio tiempo”

Casos Alternativos: 

Casos Excepcionales:

---
## Caso de Uso 30:

Título:  Ver Tabla de Liga

Actor: Usuario

Precondición: Usuario Logueado; Club registrado; Usuario en el lobby de una liga en la que participa.

Caso de Exito (Flujo Principal):
1. Usuario solicita ver la tabla de una liga.
2. El sistema muestra la tabla de posiciones de la liga según los resultados obtenidos y los criterios de clasificación establecidos con los siguientes campos:Número de posición,club,cantidad de partidos jugados,cantidad de partidos ganados,cantidad de partidos perdidos,cantidad de partidos empatados,goles a favor,goles en contra,goles de diferencia y puntos.

Casos Alternativos: 

Casos Excepcionales:

---
## Caso de Uso 31:

Título: Ver Tabla Global
Actor: Usuario
Precondición: Usuario autenticado; tener club registrado.

Caso de Éxito (Flujo Principal):
1. El usuario solicita ver el ranking global.
2. El sistema muestra la clasificación de todos los clubes, ordenada por puntos obtenidos, goles a favor, con partidos jugados, ganados, empatados, perdidos

---
## Caso de Uso 32:

Título:  Ver Partidos Jugados

Actor: Usuario

Precondición: Usuario autenticado; Club registrado.

Caso de Exito (Flujo Principal):
1. El usuario solicita ver su historial de partidos completo
2. El sistema solicita los datos de partidos jugados de ese usuario y los muestra en pantalla

Casos Alternativos: 

Casos Excepcionales:
- El usuario no ha jugado ningun partido &rArr; El sistema muestra un mensaje de error al usuario indicándole que no ha jugado ningún partido aun.

---
## Caso de Uso 33:

Título:  Ver Fixture

Actor: Usuario

Precondición: Usuario Logueado; Club Registrado ; Usuario está en el lobby de una liga que participa 

Caso de Exito (Flujo Principal):
1. Usuario solicita ver el fixture completo de una liga.
2. El sistema muestra las fechas,los equipos que disputan y su resultado(si es que ya sucedieron) o el resultado en vivo si es que se está jugando. Para los no diputados se indica que están pendientes.

Casos Alternativos: 

Casos Excepcionales:

---
## Caso de Uso 34:

Título: Cambiar Plantel Titular de una Liga
Actor: Usuario
Precondición: El usuario está inscripto en la liga; no hay un partido de esa liga en curso en este momento.

Caso de Éxito (Flujo Principal):
1. El usuario solicita modificar su plantel para una liga en la que participa.
2. El sistema muestra la convocatoria actual (titulares, suplentes, formación y comportamientos).
3. El usuario selecciona 3 titulares (con comportamiento), 3 suplentes y la formación táctica.
4. El sistema guarda los cambios, que se aplicarán a partir del próximo partido pendiente de esa liga.

Casos Excepcionales:
- Hay un partido de esta liga en curso &rArr; El sistema bloquea el cambio e informa que debe esperar a que finalice.
- La estructura de titulares/suplentes es inválida o falta el comportamiento de algún titular &rArr; El sistema informa el error específico.
