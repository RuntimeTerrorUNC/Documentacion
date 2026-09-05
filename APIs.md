NOTAS:
- Probablemente hay endpoints de casos de uso que ya no existen, como "eliminar jugador".
- En algunas ocasiones se devuelve el id de algunos objetos, por ejemplo el id del usuario o del club.
- Pregunta: el cambio de comportamiento se hace a través de la API, no a través de WebSocket, para no complejizar mucho ni sobrecargar el socket. La contrapartida es que un usuario que cambie muchas veces el comportamiento en muy poco tiempo puede sufrir una pequeña demora.

API CONTRATO

LOGIN / REGISTRARSE

## POST /api/auth/registro
Headers:
- Content-Type: application/json

Body:
```json
{
  "username": "String (Alfanumérico, no modificable a futuro)",
  "password": "String (Mínimo 8 caracteres)",
  "email": "String (Único globalmente, no puede repetirse)"
}
```

Respuestas:
- 201 Created
```json
{ "mensaje": "String", "id_usuario": "Integer" }
```

- 400 Bad Request
```json
{ "error": "String (Descripción del campo faltante o formato inválido)" }
```

- 409 Conflict
```json
{ "error": "String (Notifica que el username ya existe)" }
```

## POST /api/auth/login
Headers:
- Content-Type: application/json

Body:
```json
{
  "username": "String (Nombre de usuario ingresado)",
  "password": "String (Contraseña ingresada)"
}
```

Respuestas:
- 200 OK
```json
{
  "token": "String (Token de sesión JWT para autorizar el comienzo del juego)",
  "mensaje": "String (Confirmación de autenticación exitosa)"
}
```

- 400 Bad Request
```json
{ "error": "String (Informa que hay casillas vacías y solicita ingresar los datos nuevamente)" }
```

- 401 Unauthorized
```json
{ "error": "String (Informa que el nombre de usuario o la contraseña son incorrectos y solicita ingresarlos nuevamente)" }
```

## POST /api/auth/logout
Headers:
- Authorization: Bearer <token_jwt>

Body:
- Vacío. La validación de mostrar el mensaje y elegir "cancelar" ocurre estrictamente en la interfaz gráfica del Frontend. La petición a la API solo se dispara cuando el usuario hace clic en "confirmar".

Respuestas:
- 200 OK
```json
{ "mensaje": "String (Confirma que la sesión se cerró exitosamente y el token en el backend fue invalidado, permitiendo al sistema redirigir a la página de autenticación)" }
```

- 401 Unauthorized
```json
{ "error": "String (Informa que la acción falló porque el usuario no estaba autenticado correctamente o su token ya había expirado)" }
```

COMPORTAMIENTOS

## POST /api/comportamientos
Headers:
- Content-Type: application/json
- Authorization: Bearer <token_jwt>

Body:
```json
{
  "nombre": "String (Único dentro del club del usuario)",
  "codigo_python": "String (Sintaxis válida, limitado al uso de primitivas autorizadas como correr y patear)"
}
```

Respuestas:
- 201 Created
```json
{ "mensaje": "String", "id_comportamiento": "Integer" }
```

- 400 Bad Request
```json
{ "error": "String (Notifica el error de sintaxis en el código o el uso de primitivas prohibidas)" }
```

- 409 Conflict
```json
{ "error": "String (Notifica el conflicto al existir ya un comportamiento con ese nombre en el club)" }
```

## GET /api/comportamientos/{nombre_comportamiento}
Headers:
- Authorization: Bearer <token_jwt>

Body:
- Vacío. El nombre del comportamiento se envía como parámetro en la URL.

Respuestas:
- 200 OK
```json
{
  "nombre": "String (Nombre del comportamiento consultado)",
  "codigo_python": "String (Código fuente Python almacenado para el comportamiento)"
}
```

- 404 Not Found
```json
{ "error": "String (Informa de que no se encontró ningún comportamiento con ese nombre en el club)" }
```

## GET /api/comportamientos
Headers:
- Authorization: Bearer <token_jwt>

Body:
- Vacío.

Respuestas:
- 200 OK
```json
[
  {
    "id_comportamiento": "Integer",
    "nombre": "String (Nombre de cada comportamiento perteneciente al club del usuario)"
  }
]
```

- 404 Not Found
```json
{ "error": "String (Informa que el club no posee ningún comportamiento creado hasta el momento)" }
```

## DELETE /api/comportamientos/{nombre_comportamiento}
Headers:
- Authorization: Bearer <token_jwt>

Body:
- Vacío. El nombre del comportamiento se envía como parámetro en la URL.

Respuestas:
- 200 OK
```json
{ "mensaje": "String (Confirma la eliminación permanente del comportamiento del club)" }
```

- 403 Forbidden
```json
{ "error": "String (Bloquea la acción e informa si el club quedaría con menos de 1 comportamiento registrado, o si el comportamiento se encuentra actualmente asignado a un jugador convocado o en partido activo)" }
```

- 404 Not Found
```json
{ "error": "String (Informa que no se encontró ningún comportamiento con ese nombre registrado en el club del usuario)" }
```

## PUT /api/comportamientos/{nombre_comportamiento}
Nota: solo modifica el código Python, no el nombre.

Headers:
- Content-Type: application/json
- Authorization: Bearer <token_jwt>

Body:
```json
{
  "codigo_python": "String (Nuevo código fuente en Python con sintaxis válida, restringido al uso exclusivo de las primitivas correr y patear)"
}
```

Respuestas:
- 200 OK
```json
{ "mensaje": "String (Confirma la modificación permanente del comportamiento)" }
```

- 400 Bad Request
```json
{ "error": "String (Indica la falla notificando que la nueva sintaxis es inválida o que se utilizaron primitivas prohibidas)" }
```

- 403 Forbidden
```json
{ "error": "String (Bloquea la acción e informa que el club debe tener estrictamente más de 1 comportamiento registrado para poder modificarlo)" }
```

- 404 Not Found
```json
{ "error": "String (Indica la falla notificando que el comportamiento no existe y espera que el usuario seleccione otro)" }
```

JUGADORES

## POST /api/jugadores
Headers:
- Content-Type: application/json
- Authorization: Bearer <token_jwt>

Body:
```json
{
  "nombre": "String (Único dentro del plantel del club)",
  "skills": {
    "fuerza": "Integer (Valor numérico)",
    "velocidad": "Integer (Valor numérico)",
    "control": "Integer (Valor numérico)",
    "poder": "Integer (Valor numérico)",
    "agilidad": "Integer (Valor numérico)"
  }
}
```

Respuestas:
- 201 Created
```json
{ "mensaje": "String (Confirma la creación exitosa y el registro del nuevo jugador en el plantel)", "id_jugador": "Integer" }
```

- 400 Bad Request
```json
{ "error": "String (Notifica la diferencia de puntos si la suma de las skills es distinta a 300 exactos y solicita reajustar los valores)" }
```

- 403 Forbidden
```json
{ "error": "String (Bloquea la acción informando que se alcanzó la capacidad máxima de 50 jugadores en la base de datos e invita a eliminar uno)" }
```

- 409 Conflict
```json
{ "error": "String (Rechaza el registro notificando que el nombre ya existe en el club y solicita uno nuevo)" }
```


## GET /api/jugadores
Headers:
- Authorization: Bearer <token_jwt>

Body:
- Vacío.

Respuestas:
- 200 OK
```json
[
  {
    "id_jugador": "Integer",
    "nombre": "String (Nombre del jugador)",
    "skills": {
      "fuerza": "Integer (Atributo de fuerza)",
      "velocidad": "Integer (Atributo de velocidad)",
      "control": "Integer (Atributo de control)",
      "poder": "Integer (Atributo de poder)",
      "agilidad": "Integer (Atributo de agilidad)"
    }
  }
]
```

- 404 Not Found
```json
{ "error": "String (Informa que el usuario no tiene un club registrado o que aún no posee jugadores en su plantel)" }
```

CLUB

## POST /api/clubes
Headers:
- Content-Type: application/json
- Authorization: Bearer <token_jwt>

Body:
```json
{
  "nombre": "String (Nombre del club a crear)",
  "avatar": "archivo.jpg"
}
```

Respuestas:
- 201 Created
```json
{ "id_club": "Integer", "mensaje": "String (Confirma los datos ingresados y notifica que el club fue creado exitosamente)" }
```

- 400 Bad Request
```json
{ "error": "String (Informa que las casillas están vacías, que el nombre posee caracteres inválidos, o que el avatar tiene un formato inválido, solicitando reingreso)" }
```

- 400 Bad Request
```json
{ "error": "String (Informa que el archivo excede el tamaño máximo permitido o que la extensión debe ser .jpg)" }
```

- 403 Forbidden
```json
{ "error": "String (Bloquea la creación informando que el usuario autenticado ya posee un club asociado)" }
```

- 409 Conflict
```json
{ "error": "String (Informa que el nombre del club ingresado ya está registrado en el sistema y solicita ingresar uno diferente)" }
```

## PUT /api/clubes/mi-club/nombre
Headers:
- Content-Type: application/json
- Authorization: Bearer <token_jwt>

Body:
```json
{
  "nuevo_nombre": "String (Nuevo nombre del club, debe cumplir con la longitud permitida y no contener caracteres inválidos)"
}
```

Respuestas:
- 200 OK
```json
{ "mensaje": "String (Notifica que el cambio de nombre se realizó y actualizó correctamente en la base de datos)" }
```

- 400 Bad Request
```json
{ "error": "String (Rechaza el cambio especificando la regla de formato infringida, ya sea por caracteres inválidos o longitud excedida, y solicita corregir el texto)" }
```

- 409 Conflict
```json
{ "error": "String (Rechaza el cambio informando que el nombre ya está en uso por otro club, garantizando la regla de unicidad global, y solicita uno nuevo)" }
```

## PUT /api/clubes/mi-club/avatar
Headers:
- Content-Type: multipart/form-data
- Authorization: Bearer <token_jwt>

Body:
```json
{ "avatar": "archivo.jpg" }
```

Respuestas:
- 200 OK
```json
{ "mensaje": "String (Confirma la actualización exitosa visualizando que el sistema procesó la imagen, reemplazó el avatar anterior y asoció el nuevo archivo al club)" }
```

- 400 Bad Request
```json
{ "error": "String (Rechaza o bloquea la carga advirtiendo que la extensión no es válida, permitiendo solo .jpg, o informa que el archivo excede el límite de peso máximo en MB, solicitando subir uno más liviano)" }
```

OTROS CLUBES

## GET /api/clubes
Headers:
- Authorization: Bearer <token_jwt>

Body:
- Vacío.

Respuestas:
- 200 OK
```json
[
  {
    "id_club": "Integer",
    "nombre": "String (Nombre del club)",
    "logo": "Archivo.jpg"
  }
]
```

- 403 Forbidden
```json
{ "error": "String (Bloquea la acción informando que el usuario autenticado no cumple con la precondición de tener un club propio registrado en el sistema para poder ver el listado)" }
```

- 404 Not Found
```json
{ "error": "String (Informa que por algún motivo excepcional no se encontró ningún club registrado en la base de datos para listar)" }
```

## GET /api/clubes/{nombre_club}/jugadores
Headers:
- Authorization: Bearer <token_jwt>

Body:
- Vacío. El nombre del club se envía como parámetro en la URL.

Respuestas:
- 200 OK
```json
[
  {
    "id_jugador": "Integer",
    "nombre": "String (Nombre del jugador)",
    "skills": {
      "fuerza": "Integer (Atributo de fuerza)",
      "velocidad": "Integer (Atributo de velocidad)",
      "control": "Integer (Atributo de control)",
      "poder": "Integer (Atributo de poder)",
      "agilidad": "Integer (Atributo de agilidad)"
    }
  }
]
```

- 404 Not Found
```json
{ "error": "String (Informa que no hay ningún club registrado con ese nombre en el sistema)" }
```

## GET /api/ligas/{nombre_liga}/mi-plantel
Headers:
- Authorization: Bearer <token_jwt>

Body:
- Vacío. El nombre de la liga se envía como parámetro en la URL.

Respuestas:
- 200 OK
```json
[
  {
    "id_jugador": "Integer",
    "nombre": "String",
    "rol": "String (Indica si el jugador es uno de los 3 titulares o 3 suplentes seleccionados para esta liga)",
    "skills": {
      "fuerza": "Integer",
      "velocidad": "Integer",
      "control": "Integer",
      "poder": "Integer",
      "agilidad": "Integer"
    },
    "comportamiento_asignado": "String (Nombre del script Python asignado al jugador)"
  }
]
```

- 404 Not Found
```json
{ "error": "String (Informa que no se encontró ninguna liga con el nombre ingresado)" }
```

## GET /api/ligas/{nombre_liga}/clubes/{nombre_club}/plantel
Headers:
- Authorization: Bearer <token_jwt>

Body:
- Vacío. El nombre de la liga y el nombre del club rival se envían como parámetros en la URL.

Respuestas:
- 200 OK
```json
[
  {
    "id_jugador": "Integer",
    "nombre": "String (Nombre del jugador rival)",
    "skills": {
      "fuerza": "Integer",
      "velocidad": "Integer",
      "control": "Integer",
      "poder": "Integer",
      "agilidad": "Integer"
    }
  }
]
```

- 404 Not Found
```json
{ "error": "String (Informa que no se encontró ninguna liga con ese nombre, o que el club solicitado no participa en ella)" }
```

LIGAS

## POST /api/ligas
Headers:
- Content-Type: application/json
- Authorization: Bearer <token_jwt>

Body:
```json
{
  "nombre": "String (Único globalmente, no puede estar registrado por otra liga activa)",
  "contrasena": "String (Opcional, si no se envía la liga será de acceso público)",
  "duracion_partido": "Integer (Minutos de duración, debe estar dentro de los parámetros permitidos)",
  "cantidad_equipos": "Integer (Tope máximo de clubes que podrán unirse a la sala)",
  "jugadores_convocados": [
    {
      "id_jugador": "Integer",
      "rol": "String (Debe indicarse 'titular' o 'suplente')",
      "comportamiento": "String (Debe indicarse el nombre del comportamiento necesariamente para jugar partidos)"
    }
  ],
  "formacion_tactica": "Integer (Código numérico de la formación táctica predefinida por el sistema)"
}
```

Nota: al crear una liga también se está uniendo como participante.

Respuestas:
- 201 Created
```json
{ "mensaje": "String (Confirma que los datos son correctos, que la sala fue creada y el sistema queda a la espera de clubes)", "id_liga": "Integer" }
```

- 400 Bad Request
```json
{ "error": "String (Informa exactamente qué dato está mal ingresado: casillas vacías, contraseña inválida, duración de partido fuera de rango o cantidad de equipos mal definida, y solicita reingreso)" }
```

- 403 Forbidden
```json
{ "error": "String (Bloquea la acción si el usuario autenticado intenta crear la liga sin tener previamente un club registrado)" }
```

- 409 Conflict
```json
{ "error": "String (Informa que el nombre de la liga ingresado ya se encuentra registrado en el sistema y solicita uno diferente)" }
```

## DELETE /api/ligas/{id_liga}
Headers:
- Authorization: Bearer <token_jwt>

Body:
- Vacío. La advertencia visual de aceptar o rechazar ocurre estrictamente en la interfaz del Frontend; la petición a la API se dispara únicamente cuando el usuario elige la opción "confirmar".

Respuestas:
- 200 OK
```json
{ "mensaje": "String (Confirma la eliminación permanente de la liga e instruye al cliente web para redirigir al creador y a los demás usuarios a la página principal)" }
```

- 403 Forbidden
```json
{ "error": "String (Bloquea la acción informando que el usuario autenticado no cumple la precondición de ser el creador legítimo de la liga para poder eliminarla)" }
```

- 404 Not Found
```json
{ "error": "String (Informa que no se encontró ninguna liga activa asociada a ese identificador en el sistema)" }
```

- 409 Conflict
```json
{ "error": "String (Bloquea la eliminación notificando que la liga ya ha iniciado su curso automático y no puede ser borrada mientras los partidos se estén ejecutando)" }
```

## POST /api/ligas/{id_liga}/unirse
Headers:
- Content-Type: application/json
- Authorization: Bearer <token_jwt>

Body:
```json
{
  "contrasenia": "String (Requerida si la liga posee protección, de lo contrario nula o vacía)",
  "jugadores_convocados": [
    {
      "id_jugador": "Integer",
      "rol": "String (Debe indicarse 'titular' o 'suplente')",
      "comportamiento": "String (Debe indicarse el nombre del comportamiento necesariamente para jugar partidos)"
    }
  ],
  "formacion_tactica": "Integer (Código numérico de la formación táctica predefinida por el sistema)"
}
```

Nota: el array debe contener exactamente 6 objetos (3 titulares y 3 suplentes). Solo los titulares deben llevar comportamiento asignado.

Respuestas:
- 200 OK
```json
{ "mensaje": "String (Confirma que la lista de clubes en la liga fue actualizada y el usuario ingresó exitosamente junto con sus jugadores)" }
```

- 400 Bad Request
```json
{ "error": "String (Informa que la cantidad de jugadores es insuficiente o que la estructura de titulares/suplentes es inválida)" }
```

- 401 Unauthorized
```json
{ "error": "String ('CONTRASEÑA INCORRECTA. INTENTE NUEVAMENTE')" }
```

- 403 Forbidden
```json
{ "error": "String (Envía un mensaje indicando que la liga está completa al haber alcanzado su capacidad máxima)" }
```

- 409 Conflict
```json
{ "error": "String (Informa que el club del usuario ya se encuentra registrado en esta liga)" }
```

## POST /api/ligas/{id_liga}/iniciar
Headers:
- Authorization: Bearer <token_jwt>

Body:
- Vacío. El identificador de la liga se envía en la URL y la acción no requiere parámetros adicionales.

Respuestas:
- 200 OK
```json
{ "mensaje": "String (Confirma que la liga inició, bloquea nuevos ingresos e inicializa la tabla de posiciones en cero. Notifica la creación del fixture de n(n-1)/2 partidos distribuidos por fechas y confirma que el torneo continuará automáticamente.)" }
```

- 403 Forbidden
```json
{ "error": "String (Bloquea la acción informando que el usuario autenticado no cumple la precondición de ser el creador legítimo de la liga para poder iniciarla)" }
```

- 404 Not Found
```json
{ "error": "String (Informa que no se encontró ninguna liga en estado de espera asociada a ese identificador)" }
```

- 409 Conflict
```json
{ "error": "String (Bloquea el arranque informando que no se llegó al mínimo de participantes requeridos de 3 clubes, cancelando la operación y manteniendo la sala en espera, o notifica que la liga ya había sido iniciada previamente)" }
```

## GET /api/ligas/{id_liga}/tabla
Headers:
- Authorization: Bearer <token_jwt>

Body:
- Vacío. El identificador de la liga se envía como parámetro en la URL.

Respuestas:
- 200 OK
```json
[
  {
    "numero_posicion": "Integer (Calculado jerárquicamente por puntos, luego goles de diferencia y goles a favor)",
    "club": "String (Nombre del club)",
    "partidos_jugados": "Integer",
    "partidos_ganados": "Integer",
    "partidos_perdidos": "Integer",
    "partidos_empatados": "Integer",
    "goles_a_favor": "Integer",
    "goles_en_contra": "Integer",
    "goles_diferencia": "Integer",
    "puntos": "Integer"
  }
]
```

- 403 Forbidden
```json
{ "error": "String (Bloquea la consulta informando que el usuario no participa en esta liga, respetando la precondición del lobby)" }
```

- 404 Not Found
```json
{ "error": "String (Informa que no se encontró ninguna liga asociada a ese identificador)" }
```

## GET /api/ligas/activas
Headers:
- Authorization: Bearer <token_jwt>

Body:
- Vacío.

Respuestas:
- 200 OK
```json
[
  {
    "id_liga": "Integer",
    "nombre": "String (Nombre de la liga)",
    "clubes_maximos": "Integer (La máxima cantidad de clubes que se pueden unir)",
    "clubes_esperando": "Integer (Cantidad de clubes inscriptos actualmente a la espera de que se complete el cupo máximo para iniciar)"
  }
]
```

- 403 Forbidden
```json
{ "error": "String (Bloquea la consulta informando que el usuario no cumple con la precondición de poseer un club registrado para poder explorar el listado)" }
```

- 404 Not Found
```json
{ "error": "String (Informa que actualmente no existen ligas en la plataforma que estén en fase de espera y disponibles para unirse)" }
```

## GET /api/ligas/{id_liga}/fixture
Headers:
- Authorization: Bearer <token_jwt>

Body:
- Vacío. El identificador de la liga se envía como parámetro en la URL.

Respuestas:
- 200 OK
```json
[
  {
    "id_partido": "Integer",
    "fecha": "Integer (Número de la jornada o fecha correspondiente a este cruce)",
    "equipo_local": "String (Nombre del club)",
    "equipo_visitante": "String (Nombre del club)",
    "estado": "String ('pendiente', 'en vivo' o 'finalizado')",
    "goles_local": "Integer (Nulo si el partido está 'pendiente', numérico si está 'en vivo' o 'finalizado')",
    "goles_visitante": "Integer (Nulo si el partido está 'pendiente', numérico si está 'en vivo' o 'finalizado')"
  }
]
```

- 403 Forbidden
```json
{ "error": "String (Bloquea la consulta informando que el usuario no cumple con la precondición de poseer un club o no participa en esta liga específica)" }
```

- 404 Not Found
```json
{ "error": "String (Informa que no se encontró ninguna liga asociada a ese identificador en el sistema)" }
```

## POST /api/ligas/{id_liga}/salir
Headers:
- Authorization: Bearer <token_jwt>

Body:
- Vacío. La advertencia visual informando que perderá su lugar y la posterior decisión de confirmar o rechazar ocurren estrictamente en la interfaz del Frontend; la petición a la API se dispara únicamente cuando el usuario elige la opción "confirmar".

Respuestas:
- 200 OK
```json
{ "mensaje": "String (Notifica que la salida fue exitosa, confirmando que el sistema removió al club y a sus 6 jugadores convocados, liberando el cupo en la sala para que el cliente redirija a la pantalla principal)" }
```

- 403 Forbidden
```json
{ "error": "String (Bloquea la salida individual informando que al ser el creador legítimo no puede simplemente abandonar la sala, y le indica que debe utilizar la acción 'Eliminar Liga' si desea desarmar la competición)" }
```

- 404 Not Found
```json
{ "error": "String (Informa que no se encontró ninguna liga asociada a ese identificador o que el usuario no se encuentra actualmente inscripto en ella)" }
```

- 409 Conflict
```json
{ "error": "String (Bloquea la acción notificando que la liga ya ha comenzado con su fixture generado, indicando que inició su curso automático y los partidos deben jugarse obligatoriamente)" }
```

AMISTOSOS

## POST /api/clubes/{id_club_rival}/amistosos/solicitar
Headers:
- Content-Type: application/json
- Authorization: Bearer <token_jwt>

Body:
```json
{
  "jugadores_convocados": [
    {
      "id_jugador": "Integer",
      "rol": "String (Debe indicarse 'titular' o 'suplente')",
      "comportamiento": "String (Nombre comportamiento)"
    }
  ],
  "formacion_tactica": "Integer (Código numérico de la formación táctica predefinida por el sistema)"
}
```

Nota: el array debe contener exactamente 6 jugadores requeridos (3 titulares y 3 suplentes) para superar la validación.

Respuestas:
- 200 OK
```json
{ "id_solicitud_amistoso": "Integer", "mensaje": "String (Notifica al Usuario Retador que la invitación fue enviada exitosamente al rival y confirma el inicio del temporizador de espera)" }
```

- 400 Bad Request
```json
{ "error": "String (No cumple con los requisitos mínimos de convocatoria)" }
```

- 400 Bad Request
```json
{ "error": "String (El club rival no cumple con los requisitos mínimos de convocatoria)" }
```

- 404 Not Found
```json
{ "error": "String (Informa que el club rival seleccionado ya no existe o no fue encontrado en la base de datos)" }
```

## POST /api/amistosos/solicitudes/{id_solicitud}/aceptar
Headers:
- Content-Type: application/json
- Authorization: Bearer <token_jwt>

Body:
```json
{
  "jugadores_convocados": [
    {
      "id_jugador": "Integer",
      "rol": "String (Debe indicarse 'titular' o 'suplente')",
      "comportamiento": "String (Nombre comportamiento)"
    }
  ],
  "formacion_tactica": "Integer (Código numérico de la formación táctica predefinida por el sistema)"
}
```

Nota: el array debe contener exactamente 6 jugadores requeridos (3 titulares y 3 suplentes) para superar la validación.

Respuestas:
- 200 OK
```json
{ "id_partido": "Integer", "mensaje": "String (Confirma que la información es válida, que el partido se ha iniciado para ambos usuarios y provee el identificador del encuentro)" }
```

- 400 Bad Request
```json
{ "error": "String (Detecta que el Usuario no cuenta con los requisitos mínimos para armar la plantilla, cancela la aceptación y emite automáticamente la notificación al retador sobre la falta de plantel)" }
```

- 404 Not Found
```json
{ "error": "String (Informa que la invitación ya no existe debido a que el temporizador expiró sin respuesta, descartando la acción)" }
```

- 409 Conflict
```json
{ "error": "String (Bloquea la acción informando que la solicitud ya había sido respondida o procesada previamente)" }
```

## POST /api/amistosos/solicitudes/{id_solicitud}/rechazar
Headers:
- Authorization: Bearer <token_jwt>

Body:
- Vacío.

Respuestas:
- 200 OK
```json
{ "mensaje": "String (Confirma la declinación de la invitación e instruye al backend a notificar al Usuario Retador sobre el rechazo)" }
```

- 404 Not Found
```json
{ "error": "String (Informa que la invitación ya caducó por tiempo límite o no fue encontrada)" }
```

Nota: el temporizador que expira sin respuesta no tiene un contrato REST directo; es un proceso autónomo controlado por el reloj del servidor.

PARTIDO

## PUT /api/partidos/{id_partido}/jugadores/{id_jugador}/comportamiento
Headers:
- Content-Type: application/json
- Authorization: Bearer <token_jwt>

Body:
- El caso alternativo de presionar "atrás" se maneja estrictamente en el Frontend cerrando el menú localmente, por lo que no genera ninguna petición a la API.

```json
{
  "nuevo_comportamiento": "String (Nombre del comportamiento disponible y previamente creado que se desea asignar al jugador)"
}
```

Respuestas:
- 200 OK
```json
{ "mensaje": "String (Confirma que el comportamiento del jugador fue cambiado exitosamente en la memoria del partido, instruyendo a la interfaz a cerrar el menú de comportamientos)" }
```

- 403 Forbidden
```json
{ "error": "String (Bloquea la acción informando que el usuario no cumple la precondición de ser el dueño del jugador que intenta modificar)" }
```

- 404 Not Found
```json
{ "error": "String (Indica que el comportamiento solicitado no existe en la base de datos del club o el partido ya no se encuentra activo)" }
```

## POST /api/partidos/{id_partido}/sustituciones
Headers:
- Content-Type: application/json
- Authorization: Bearer <token_jwt>

Body:
```json
{
  "id_jugador_sale": "Integer (Identificador del jugador titular que actualmente está en la cancha)",
  "id_jugador_entra": "Integer (Identificador del jugador que reemplaza al actual, elegido de la lista de suplentes)",
  "comportamiento": "String"
}
```

Respuestas:
- 200 OK
```json
{ "mensaje": "String (Confirma que el cambio fue registrado exitosamente y notifica que se ejecutará físicamente en la cancha cuando ocurra el evento de 'pausa de hidratación' o 'medio tiempo')" }
```

- 403 Forbidden
```json
{ "error": "String (Bloquea la petición informando que el usuario no es el dueño legítimo de este jugador)" }
```

- 409 Conflict
```json
{ "error": "String (Avisa al usuario que no tiene cambios disponibles en este momento, respetando la regla de que solo se permite 1 cambio por pausa y no son acumulables)" }
```

TABLA GLOBAL

## GET /api/ranking-global
Headers:
- Authorization: Bearer <token_jwt>

Body:
- Vacío.

Respuestas:
- 200 OK
```json
[
  {
    "numero_posicion": "Integer (Calculado jerárquicamente por puntos, luego goles a favor)",
    "club": "String (Nombre del club)",
    "partidos_jugados": "Integer (Totalidad de encuentros disputados, sumando ligas y amistosos)",
    "goles_a_favor": "Integer",
    "partidos_ganados": "Integer",
    "partidos_empatados": "Integer",
    "partidos_perdidos": "Integer",
    "puntos": "Integer (Calculado sumando 3 por victoria, 1 por empate y 0 por derrota)"
  }
]
```

- 403 Forbidden
```json
{ "error": "String (Bloquea la acción informando que el usuario autenticado no cumple con la precondición de poseer un club registrado en el sistema para acceder a las estadísticas globales)" }
```

## PUT /api/ligas/{id_liga}/mi-plantel
Headers:
- Content-Type: application/json
- Authorization: Bearer <token_jwt>

Body:
```json
{
  "jugadores_convocados": [
    {
      "id_jugador": "Integer",
      "rol": "String ('titular' o 'suplente')",
      "comportamiento": "String (Obligatorio solo si rol = 'titular')"
    }
  ],
  "formacion_tactica": "Integer (Código numérico de la formación táctica predefinida por el sistema)"
}
```

Nota: el array debe contener exactamente 6 objetos (3 titulares y 3 suplentes).

Respuestas:
- 200 OK
```json
{ "mensaje": "String (Confirma que el plantel de la liga fue actualizado y se aplicará a partir del próximo partido pendiente de esa liga)" }
```

- 400 Bad Request
```json
{ "error": "String (Informa que la estructura de titulares/suplentes es inválida, que falta el comportamiento de algún titular, o que el código de formación no existe)" }
```

- 403 Forbidden
```json
{ "error": "String (Bloquea la acción informando que el club del usuario no participa en esta liga)" }
```

- 404 Not Found
```json
{ "error": "String (Informa que no se encontró ninguna liga asociada a ese identificador)" }
```

- 409 Conflict
```json
{ "error": "String (Bloquea el cambio informando que hay un partido de esta liga en curso en este momento; debe esperar a que finalice para actualizar el plantel)" }
```

## WebSocket de partido en vivo

### Explicación rápida: por qué se separa en tres tipos

En un WebSocket de partido en vivo no conviene mandar un solo tipo de mensaje gigante todo el tiempo, porque hay datos que no cambian nunca durante el partido y otros que cambian constantemente. Por eso se separan en tres categorías:

- **Mensaje inicial:** se envía una sola vez, apenas el cliente se conecta. Es todo lo que no cambia durante el partido (cancha, clubes, plantel, duración, cuándo son las pausas).
- **Estado en vivo:** se envía muchas veces por segundo mientras el partido corre. Es la "foto" del momento actual (dónde está la pelota, dónde está cada jugador, el marcador, el reloj).
- **Eventos puntuales:** se envían solo quando pasa algo específico (un gol, el inicio de una pausa, una sustitución, el fin del partido). No se repiten en cada tick como el estado en vivo, solo aparecen una vez cuando el hecho ocurre.

Esto evita mandar información redundante todo el tiempo (por ejemplo, no tiene sentido reenviar los límites de la cancha 10 veces por segundo si nunca cambian) y separa claramente "esto es un dato que se actualiza" de "esto es un hecho que ocurrió".

**Endpoint:** `websocket/partidos/{id_partido}`  
**Autenticación:** token JWT

El servidor envía tres tipos de mensajes distintos, diferenciados por el campo `tipo`.

### 1. Mensaje inicial

> Se envía una sola vez, al conectarse.

Contiene todo lo que no cambia durante el partido: límites de la cancha, datos de los clubes, plantel convocado, duración total y en qué segundo ocurre cada pausa.

```json
{
  "tipo": "init",
  "id_partido": Integer,
  "cancha": { "ancho": Integer, "alto": Integer },
  "club_local": { "id_club": Integer, "nombre": String, "avatar": String (URL) },
  "club_visitante": { "id_club": Integer, "nombre": String, "avatar": String (URL) },
  "duracion_partido_seg": Integer,
  "pausas_programadas": [
    { "tipo": "hidratacion_1" | "medio_tiempo" | "hidratacion_2", "segundo": Integer }
  ],
  "plantel_local": [
    { "id_jugador": Integer, "nombre": String, "rol": "titular" | "suplente" }
  ],
  "plantel_visitante": [
    { "id_jugador": Integer, "nombre": String, "rol": "titular" | "suplente" }
  ]
}
```

### 2. Estado en vivo

Se envía repetidamente mientras el partido está en curso, por ejemplo, cada 150 ms. Contiene la "foto" del momento actual: posición de la pelota, posición y acción de cada jugador, marcador y reloj.

```json
{
  "tipo": "estado",
  "tiempo_transcurrido_seg": Integer,
  "estado_partido": "prematch" | "en_curso" | "pausa" | "finalizado",
  "pelota": { "x": Float, "y": Float, "posesion_id_jugador": Integer o null },
  "jugadores": [
    {
      "id_jugador": Integer,
      "id_club": Integer,
      "x": Float,
      "y": Float,
      "comportamiento": String
    }
  ],
  "goles_local": Integer,
  "goles_visitante": Integer
}
```

### 3. Eventos puntuales

Se envían solo cuando ocurre el hecho y no se repiten.

**Gol:**

```json
{ "tipo": "evento_gol", "id_club_anota": Integer, "id_jugador_anota": Integer, "goles_local": Integer, "goles_visitante": Integer, "tiempo_seg": Integer }
```

**Inicio de pausa:**

```json
{ "tipo": "evento_pausa_inicio", "pausa": String, "tiempo_seg": Integer }
```

**Fin de pausa / reanudación:**

```json
{ "tipo": "evento_pausa_fin", "tiempo_seg": Integer }
```

**Sustitución efectuada:**

```json
{ "tipo": "evento_sustitucion", "id_club": Integer, "id_jugador_sale": Integer, "id_jugador_entra": Integer }
```

**Fin de partido:**

```json
{ "tipo": "evento_fin_partido", "goles_local": Integer, "goles_visitante": Integer, "resultado": "local" | "visitante" | "empate" }
```

**Cierre de conexión:** el servidor cierra el socket automáticamente al enviar `evento_fin_partido`. El cliente puede desconectarse en cualquier momento sin afectar la simulación, ya que este canal es solo de lectura (no se manda nada desde el cliente).


