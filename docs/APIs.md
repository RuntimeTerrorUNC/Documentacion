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
  "username": "String (Entre 1 y 25 caracteres)",
  "password": "String (Mínimo 8 caracteres)",
  "email": "String (Único globalmente, no puede repetirse)"
}
```

Respuestas:
- 201 Created
```json
{
  "id": 1,
  "username": "entrenador",
  "email": "entrenador@example.com",
  "mensaje": "Usuario registrado exitosamente."
}
```

- 400 Bad Request
```json
{ "detail": "El nombre de usuario o correo electrónico ya está registrado." }
```

- 422 Unprocessable Entity
```json
{ "detail": "Errores de validación del body" }
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
  "user_id": "Integer (Identificador del usuario)",
  "mensaje": "String (Confirmación de autenticación exitosa)",
  "tiene_club": "Integer o null (Identificador del club del usuario, si existe)"
}
```

El token vence a las 24 horas.

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
  "id": "Integer",
  "name": "String (Nombre del comportamiento consultado)",
  "script": "String (Código fuente Python almacenado para el comportamiento)"
}
```

- 403 Forbidden
```json
{ "detail": "El usuario autenticado no tiene un club." }
```

- 404 Not Found
```json
{ "detail": "Comportamiento no existe" }
```

## GET /api/comportamientos
Headers:
- Authorization: Bearer <token_jwt>

Body:
- Vacío.

Respuestas:
- 200 OK
```json
{
  "comportamientos": [
    { "id": 1, "nombre": "Presión Alta" }
  ]
}
```

Si no hay comportamientos, la lista `comportamientos` está vacía. Si el usuario no tiene club, responde `403 Forbidden` con `{"detail":"El usuario autenticado no tiene un club."}`.

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
{ "error": "String (Bloquea la acción e informa si el club quedaría sin comportamientos, o si el comportamiento se encuentra actualmente asignado a un jugador en una convocatoria de liga o en un partido activo)" }
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
{ "error": "String (Bloquea la acción si el comportamiento está asignado a un jugador en una convocatoria de liga o en un partido activo)" }
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
  "name": "String (Entre 1 y 50 caracteres; único dentro del club)",
  "speed": "Integer (Entre 20 y 100)",
  "strength": "Integer (Entre 20 y 100)",
  "power": "Integer (Entre 20 y 100)",
  "agility": "Integer (Entre 20 y 100)",
  "control": "Integer (Entre 20 y 100)"
}
```

La suma de las cinco estadísticas debe ser exactamente 300.

Respuestas:
- 201 Created
```json
{ "id": 1, "name": "Delantero", "created": true }
```

- 400 Bad Request
```json
{ "detail": "Las estadísticas deben sumar 300; suman <total>." }
```

- 403 Forbidden
```json
{ "detail": "El usuario autenticado no tiene un club." }
```

- 409 Conflict
```json
{ "detail": "Ya existe un jugador con ese nombre en tu club." }
```

Los errores de validación del body responden `422 Unprocessable Entity`.


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
    "id": 1,
    "name": "string",
    "speed": "int",
    "strength": "int",
    "power": "int",
    "agility": "int",
    "control": "int",
    "club_id": "int"
  }
]
```

```json
{ "detail": "El usuario autenticado no tiene un club." }
```
La respuesta es `403 Forbidden` si el usuario no tiene club. Si no hay jugadores, responde `200 OK` con `[]`.

CLUB

## POST /api/clubes
Headers:
- Content-Type: multipart/form-data
- Authorization: Bearer <token_jwt>

Body:
- `name`: texto obligatorio, entre 3 y 100 caracteres.
- `photo`: archivo JPEG/JPG obligatorio, máximo 1 MB.

Respuestas:
- 201 Created
```json
{ "name": "Club Ejemplo", "photo": "static/uploads/<archivo>.jpg", "id": 3, "owner_id": 1 }
```

- 400 Bad Request
```json
{ "detail": "La foto debe ser un archivo de imagen válido (JPG/JPEG) o el archivo supera el límite de 1 MB." }
```

- 403 Forbidden
```json
{ "detail": "El usuario ya posee un club registrado. Solo se permite un club por usuario." }
```

- 409 Conflict
```json
{ "detail": "El nombre de club ya está en uso." }
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
    "name": "Club Ejemplo",
    "photo": "static/uploads/<archivo>.jpg",
    "id": 3,
    "owner_id": 1
  }
]
```

La respuesta puede ser `[]` si no hay clubes ajenos para listar. Se puede enviar `?nombre=<texto>` para filtrar, aunque actualmente el filtro falla porque consulta un atributo `Club.nombre` que no existe en el modelo (el atributo se llama `name`).

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

- 403 Forbidden
```json
{ "error": "String (Bloquea la consulta cuando la liga es privada y el usuario autenticado no participa en ella)" }
```

LIGAS

## POST /api/ligas/
Headers:
- Content-Type: application/json
- Authorization: Bearer <token_jwt>

Body:
```json
{
  "name": "String (Entre 1 y 25 caracteres; único globalmente)",
  "password": "String o null (Opcional; si se informa, entre 6 y 50 caracteres)",
  "duracion_partido": "Integer (Mayor que 0)",
  "cantidad_equipos": "Integer (Entre 4 y 50)",
  "jugadores_convocados": [
    {
      "id_jugador": "Integer",
      "rol": "String (Debe indicarse 'titular' o 'suplente')",
      "comportamiento": "String o null (Obligatorio para titulares; no permitido para suplentes)"
    }
  ],
  "formacion_tactica": "Integer (Mayor o igual que 1)"
}
```

Debe incluir exactamente seis jugadores distintos del club del usuario. Se requieren tres titulares con comportamiento asignado; los suplentes no deben tenerlo. Una liga con contraseña no vacía es privada; sin contraseña es pública. El club creador se registra en la convocatoria al crear la liga.

Respuestas:
- 201 Created
```json
{ "id_liga": 1, "owner_id": 3 }
```
`owner_id` contiene el identificador del club creador.

- 400 Bad Request
```json
{ "detail": "La convocatoria no es válida o el usuario no tiene un club asociado." }
```

- 401 Unauthorized
```json
{ "detail": "Token ausente, inválido o vencido." }
```

- 409 Conflict
```json
{ "detail": "El nombre de la liga ya está en uso" }
```

Los errores de validación del body responden `422 Unprocessable Entity`.

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
  }]

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

La implementación exige que los seis jugadores pertenezcan al club retador y que cada uno tenga un comportamiento existente asignado. El club rival debe tener al menos seis jugadores. La solicitud vence a los cinco minutos.

Respuestas:
- 200 OK
```json
{ "id_solicitud_amistoso": "Integer", "mensaje": "String (Notifica al Usuario Retador que la invitación fue enviada exitosamente al rival y confirma el inicio del temporizador de espera)" }
```

- 400 Bad Request
```json
{ "detail": { "error": "No cumple con los requisitos mínimos de convocatoria o se enviaron jugadores duplicados." } }
```

- 400 Bad Request
```json
{ "detail": "El club rival no cumple con los requisitos mínimos de convocatoria" }
```

- 404 Not Found
```json
{ "detail": { "error": "El club rival seleccionado ya no existe o no fue encontrado en la base de datos" } }
```

Si el usuario no tiene club, responde `400 Bad Request`; los errores de autenticación responden `401 Unauthorized`. FastAPI envuelve el detalle de las excepciones en la propiedad `detail`.

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

- 400 Bad Request
```json
{ "error": "String (Informa que el jugador entrante no está disponible o que el jugador titular indicado no está actualmente en cancha)" }
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

## WebSocket de notificaciones de amistosos (implementado)

**Endpoint:** `ws://<host>/ws/{user_id}?token=<token_jwt>`

El `user_id` debe coincidir con el usuario incluido en el JWT. Si el token no es válido o no corresponde al usuario de la ruta, el servidor cierra la conexión con el código `1008`. Al llegar una solicitud de amistoso, el usuario rival conectado recibe un mensaje JSON como el siguiente:

```json
{
  "tipo": "NUEVA_SOLICITUD_AMISTOSO",
  "id_solicitud": 1,
  "id_club_retador": 3,
  "expira_en_segundos": 300,
  "mensaje": "El club 3 te ha enviado un desafío amistoso."
}
```

El cliente debe mantener la conexión abierta; los mensajes de texto que envía se reciben para mantener el socket activo, pero no ejecutan operaciones.

## WebSocket de partido en vivo (pendiente de implementación)

### Explicación rápida: por qué se separa en tres tipos

En un WebSocket de partido en vivo no conviene mandar un solo tipo de mensaje gigante todo el tiempo, porque hay datos que no cambian nunca durante el partido y otros que cambian constantemente. Por eso se separan en tres categorías:

- **Mensaje inicial:** se envía una sola vez, apenas el cliente se conecta. Es todo lo que no cambia durante el partido (cancha, clubes, plantel, duración, cuándo son las pausas).
- **Estado en vivo:** se envía muchas veces por segundo mientras el partido corre. Es la "foto" del momento actual (dónde está la pelota, dónde está cada jugador, el marcador, el reloj).
- **Eventos puntuales:** se envían solo quando pasa algo específico (un gol, el inicio de una pausa, una sustitución, el fin del partido). No se repiten en cada tick como el estado en vivo, solo aparecen una vez cuando el hecho ocurre.

Esto evita mandar información redundante todo el tiempo (por ejemplo, no tiene sentido reenviar los límites de la cancha 10 veces por segundo si nunca cambian) y separa claramente "esto es un dato que se actualiza" de "esto es un hecho que ocurrió".

**Endpoint propuesto:** `websocket/partidos/{id_partido}`  
**Fin de partido:**

```json
{ "tipo": "evento_fin_partido", "goles_local": Integer, "goles_visitante": Integer, "resultado": "local" | "visitante" | "empate" }
```

**Cierre de conexión:** el servidor cierra el socket automáticamente al enviar `evento_fin_partido`. El cliente puede desconectarse en cualquier momento sin afectar la simulación, ya que este canal es solo de lectura (no se manda nada desde el cliente).


## API COMPORTAMIENTO

### Contrato — Librería de Comportamientos (Primitivas y Compuestos)

define las funciones disponibles dentro del sandbox donde se ejecuta el script Python de cada jugador. No son endpoints HTTP: son funciones que se llaman directamente desde el código del comportamiento, evaluadas en cada tick del motor de simulación.

Se dividen en tres niveles:

- **Atómicas:** las únicas dos acciones reales que puede ejecutar un jugador en un tick.
- **Info:** funciones de solo lectura para consultar el estado del partido.
- **Compuestas:** funciones de ayuda de más alto nivel, construidas combinando atómicas + sensores, para facilitarle el trabajo al usuario.

**Regla general:** por tick, un jugador solo puede ejecutar una acción (atómica o compuesta — las compuestas terminan llamando internamente a una atómica). Si el script llama a más de una acción en el mismo tick, aplica la regla de validación que se defina (ej. se ejecuta la primera y se ignoran las siguientes).

## 1. Primitivas Atómicas

### `correr(x, y, z)`

**Parámetros:**

- `x, y (Float)`: coordenadas del punto de la cancha hacia el que se quiere correr.
- `z (Integer, 0–100)`: porcentaje de la velocidad máxima del jugador (según su skill de Velocidad) que se usará en este tick.

**Comportamiento:** mueve al jugador un paso en dirección a (x, y), a la fracción de velocidad indicada por z.

**Nota:** z = 0 equivale a quedarse quieto; no hace falta una primitiva separada para eso.

### `patearConFuerza(x, y, z)`

**Parámetros:**

- `x, y (Float)`: coordenadas del punto de la cancha hacia donde se dirige la pelota.
- `z (Integer, 0–100)`: fuerza del pateo (a mayor fuerza, mayor velocidad y distancia recorrida por la pelota).

**Comportamiento:** patea la pelota hacia (x, y) con la fuerza z, siempre que el jugador esté dentro del rango de contacto con la pelota (ver sensor `puedo_patear()`).

**Falla silenciosa o error:** si se llama sin estar en rango de la pelot el jugador no hace nada [correr(jugador.x, jugador.y, 0)]

## 2. Información de la cancha

Todos son de solo lectura, no consumen la acción del tick — se pueden llamar todas las veces que se necesite antes de decidir qué acción ejecutar. NO es como una api no se le devuelve al usuario, estas funciones se las puede llamar en el script

| Función | Devuelve | Descripción |
|---|---|---|
| `puedo_patear()` | `bool` | Si el jugador está lo suficientemente cerca de la pelota como para patearla. |
| `mi_posicion()` | `(x, y)` | Coordenadas actuales del propio jugador. |
| `posicion_pelota()` | `(x, y)` | Coordenadas actuales de la pelota. |
| `rival_mas_cercano()` | `jugador {id, x, y}` | Datos del rival más próximo al jugador. |
| `companero_mas_cercano()` | `jugador {id, x, y}` | Ídem, del propio equipo. |
| `jugadores_propios()` | `lista[jugador]` | Lista de los jugadores propios en cancha con su posición. |
| `jugadores_rivales()` | `lista[jugador]` | Lista de los jugadores rivales en cancha con su posición. |
| `posicion_arco_propio()` | `(x, y)` | Coordenadas del centro del propio arco. |
| `posicion_arco_rival()` | `(x, y)` | Coordenadas del centro del arco rival. |
| `tiempo_restante()` | `Integer` | Segundos restantes para que termine el partido. |
| `marcador()` | `(goles_propios, goles_rivales)` | Marcador actual del partido. |
| `id_propio()` | `int` | identificación del jugador quien llama la función |
| `distancia_a(x,y)` | `output: Float` | Distancia entre el jugador y un punto (x, y) |

## 3. Comportamientos Compuestos (helpers de la librería)

Cada uno se ofrece ya implementado para que el usuario no tenga que escribirlo desde cero, pero se construye únicamente con las atómicas y sensores de arriba.

### `marcar(id_jugador_rival, referencia)`

- **Parámetros:** id_jugador_rival (Integer) — el jugador rival a marcar.
- **Referencia:** (enum)[pelota|arco]
- **Comportamiento:** calcula el punto medio entre el rival y la referencia elegida (pelota o arco propio), y corre hacia ahí.
- **Se basa en:** jugadores_rivales() (o el id recibido) + posicion_arco_propio() + correr().

### `perseguir_jugador(id)`

- **Parámetros:** id (Integer) — jugador propio o rival a perseguir.
- **Comportamiento:** ejecuta correr() hacia la posición actual de ese jugador, recalculando en cada tick porque el objetivo se mueve.
- **Se basa en:** jugadores_propios()/jugadores_rivales() + correr().

### `perseguir_pelota()`

- **Comportamiento:** ejecuta correr() hacia posicion_pelota() en cada tick.
- **Se basa en:** posicion_pelota() + correr().

### `rematar_al_arco()`

- **Comportamiento:** si puedo_patear() es verdadero, ejecuta patearConFuerza() hacia posicion_arco_rival() con fuerza máxima (z = 100).
- **Se basa en:** puedo_patear() + posicion_arco_rival() + patearConFuerza().

### `pasar_a(id_companero)`

- **Parámetros:** id_companero (Integer) — el compañero al que se le pasa la pelota.
- **Comportamiento:** ejecuta patearConFuerza() hacia la posición actual de ese compañero, con una fuerza z fija
- **Se basa en:** jugadores_propios() +  patearConFuerza().
- con una fuerza z de 15

### `despejar()`

- **Comportamiento:** ejecuta patearConFuerza() con fuerza máxima en dirección contraria al propio arco, sin apuntar a un jugador específico — pensado como acción defensiva de urgencia.
- **Se basa en:** posicion_arco_propio() + patearConFuerza().

### `cubrir_posicion(x, y)`

- **Parámetros:** x, y (Float) — coordenada táctica fija.
- **Comportamiento:** ejecuta correr() hacia (x, y) y se mantiene ahí (disciplina posicional), sin perseguir la pelota.
- **Se basa en:** correr().

### `evadir(id_rival)`

- **Parámetros:** id_rival (Integer) — el rival del que se quiere alejar.
- **Comportamiento:** ejecuta correr() en dirección contraria a la posición de ese rival, típicamente mientras el jugador tiene la pelota (regate simple).
- **Se basa en:** jugadores_rivales() + mi_posicion() + correr().

### `transportar_balon(x, y)`

- **Parámetros:** x, y (Float) — destino hacia donde se quiere llevar la pelota.
- **Comportamiento:** si puedo_patear(), ejecuta patearConFuerza(x, y, z_bajo) (toque suave hacia adelante); en !puedo_patear() caso, ejecuta correr(x, y, 100) para perseguir la pelota a fondo.
- **Se basa en:** puedo_patear() + patearConFuerza() + correr().
- definir el valor de z_bajo (fuerza del toque suave)  por defecto 15

## Reglas de ejecución

**Resolución de disputa:** si más de un jugador (propio o rival) tiene puedo_patear() == true y ambos ejecutan patearConFuerza() en el mismo tick, se resuelve por mayor skill de Poder; en caso de empate, se decide aleatoriamente (50/50). El/los jugadores que pierden la disputa no ejecutan el pateo ese tick (equivalente a correr(jugador.x, jugador.y, 0)).

**Timeout:** 100ms (1/10 seg) por tick. Si el script de un jugador tarda más que eso en devolver una acción, ese tick el jugador no hace nada (equivalente a correr(jugador.x, jugador.y, 0)).

**Modelo de ejecución — sin estado (stateless):** el script no conserva memoria entre ticks. Cada tick se evalúa como una ejecución nueva e independiente: no existen variables que persistan de un tick al siguiente ni se recuerda ninguna decisión tomada previamente.
