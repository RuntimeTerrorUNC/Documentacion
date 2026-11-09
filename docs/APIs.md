# Contrato de la API

Referencia revisada contra los routers de `Backend/app/routers`, los routers WebSocket registrados en `app/main.py` y los modelos de `app/schemas.py`. Los ejemplos muestran la forma de los datos; donde se indica `string` o `integer`, el valor es ilustrativo.

## Convenciones

- Base HTTP local: `http://localhost:8000`.
- Base WebSocket local: `ws://localhost:8000`.
- Los endpoints protegidos usan `Authorization: Bearer <token_jwt>`.
- El token de acceso vence a las 24 horas. Los endpoints WebSocket reciben el mismo token mediante `?token=<token_jwt>`.
- FastAPI devuelve normalmente errores como `{ "detail": "..." }`; los errores de validación del body son `422` con el detalle estándar de FastAPI. Algunos endpoints amistosos devuelven un `detail` que a su vez contiene un objeto `error`.
- No existe un endpoint REST de logout en el backend actual.

## Salud

### `GET /`

No requiere autenticación. Respuesta `200`:

```json
{ "mensaje": "Backend operativo" }
```

## Autenticación y cuenta

### `POST /api/auth/registro`

Crea una cuenta. Body JSON (`UserRegisterRequest`):

```json
{
  "username": "entrenador",
  "email": "entrenador@example.com",
  "password": "clave1234"
}
```

- `username`: 1–25 caracteres.
- `email`: dirección válida.
- `password`: mínimo 4 caracteres según el schema actual.
- `201`: `{ "id": 1, "username": "entrenador", "email": "entrenador@example.com", "mensaje": "..." }`.
- `400`: username o email ya registrado (`detail`).
- `422`: body inválido.

Cuando `VERIFICACION_EMAIL_ACTIVA` está habilitada, se programa el envío del correo de verificación; si no, la cuenta se crea verificada.

### `POST /api/auth/login`

Body JSON: `{ "username": "...", "password": "..." }`.

- `200`: `{ "token": "...", "user_id": 1, "mensaje": "autenticación exitosa", "tiene_club": 10 }`. `tiene_club` es el ID del club o `null`.
- `400`: campos vacíos, respuesta `{ "error": "Hay campos vacios" }`.
- `401`: credenciales incorrectas, respuesta `{ "error": "Nombre de usuario o contraseña incorrectos" }`.
- `403`: cuenta sin verificar, respuesta `{ "error": "Tenés que verificar tu email antes de iniciar sesión." }`.

### `GET /api/auth/verificar-email?token=<token>`

Consume el token enviado por correo y redirige al frontend a `/verificacion?estado=ok|expirado|invalido`. No requiere bearer token.

### `POST /api/auth/reenviar-verificacion`

Body JSON: `{ "email": "entrenador@example.com" }`.

- `200`: `{ "mensaje": "Si el email esta registrado y sin verificar, te enviamos un nuevo enlace" }`.
- La respuesta es neutra tanto si el email no existe como si no corresponde reenviar.

### `POST /api/auth/forgot-password`

Body JSON: `{ "email": "entrenador@example.com" }`.

- `200`: `{ "mensaje": "Si el correo está registrado, recibirás un enlace para restablecer tu contraseña" }`.
- La respuesta no revela si el correo está registrado.

### `POST /api/auth/reset-password`

Body JSON: `{ "token": "<token_recuperacion>", "new_password": "nueva-clave" }`. El schema requiere una contraseña de al menos 4 caracteres.

- `200`: `{ "mensaje": "Contraseña actualizada exitosamente." }`.
- `400`: token inválido o expirado (`detail`).
- `404`: usuario del token no encontrado (`detail`).
- `422`: body inválido.

### `GET /api/users/{id}`

Requiere bearer token y permite consultar únicamente el perfil del propio usuario.

- `200`: `{ "id": 1, "username": "entrenador", "email": "entrenador@example.com" }`.
- `403`: el ID solicitado no coincide con el usuario autenticado.
- `401`: token ausente, inválido o vencido.

### `PATCH /api/users/{id}`

Requiere bearer token y solo permite modificar el perfil propio. Body JSON con uno o más campos opcionales:

```json
{ "username": "nuevo_nombre", "email": "nuevo@example.com", "password": "nueva-clave-segura" }
```

- `username`: 1–25 caracteres.
- `email`: dirección válida.
- `password`: mínimo 8 caracteres.
- `200`: perfil `{ "id": 1, "username": "...", "email": "..." }`.
- `400`: no se envió ningún cambio.
- `403`: se intenta modificar otro usuario.
- `409`: username o email ya registrado.
- `422`: body inválido.

Los cambios se envían a confirmar por correo; el endpoint responde con el perfil vigente antes de que el usuario confirme el enlace.

### `GET /api/users/confirmar-cambios?token=<token>`

Confirma los cambios del perfil y redirige al frontend a `/perfil?actualizacion=ok|invalido`. No requiere bearer token.

## Clubes

### `POST /api/clubes/`

Requiere bearer token y `multipart/form-data`:

- `name`: texto obligatorio, longitud 3–100 caracteres en este endpoint.
- `photo`: archivo obligatorio con MIME `image/jpeg`, máximo 1 MiB.

Respuesta `201` con `ClubResponse`:

```json
{ "name": "Club Ejemplo", "photo": "static/uploads/<archivo>.jpg", "id": 10, "owner_id": 1 }
```

Al crear el club también se crean sus comportamientos predeterminados.

- `400`: formato JPEG o tamaño inválido.
- `403`: el usuario ya tiene un club.
- `409`: nombre ya utilizado.
- `422`: campos/form-data inválidos.

`photo` es la ruta que guarda el backend. Este contrato no implica que esa ruta esté publicada como URL HTTP.

### `GET /api/clubes`

Requiere bearer token. Opcional: `?nombre=<texto>`. Responde `200` con una lista de `ClubResponse` de clubes ajenos al usuario:

```json
[{ "name": "Club Ejemplo", "photo": "static/uploads/x.jpg", "id": 10, "owner_id": 2 }]
```

Sin resultados, devuelve `[]`. **Incidencia conocida:** cuando se especifica `nombre`, la implementación filtra por `Club.nombre`, atributo que no existe en el modelo (el campo se llama `name`), por lo que ese filtro puede producir `500`.

### `GET /api/clubes/{id_club}`

Requiere bearer token y que el usuario tenga un club propio.

- `200`: `{ "id": 10, "nombre_club": "Club Ejemplo", "avatar_url": "static/uploads/x.jpg" }`.
- `403`: usuario sin club.
- `404`: club solicitado inexistente.

## Jugadores

### `POST /api/jugadores`

Requiere bearer token. Body JSON:

```json
{ "name": "Delantero", "speed": 60, "strength": 60, "power": 60, "agility": 60, "control": 60 }
```

Cada atributo debe estar entre 20 y 100 y la suma debe ser exactamente 300.

- `201`: `{ "id": 1, "name": "Delantero", "created": true }`.
- `400`: suma de estadísticas distinta de 300.
- `403`: el usuario no tiene club.
- `409`: ya existe ese nombre en el club.
- `422`: body inválido.

### `GET /api/jugadores`

Requiere bearer token. Lista los jugadores del club del usuario con los campos `id`, `name`, `speed`, `strength`, `power`, `agility`, `control` y `club_id`.

- `200`: lista, posiblemente vacía.
- `403`: el usuario no tiene club.

## Comportamientos

### `POST /api/comportamientos`

Requiere bearer token. Body JSON: `{ "name": "Presionar", "script": "...código Python..." }`. El nombre debe tener 1–100 caracteres no blancos; `script` no puede estar vacío.

- `201`: `{ "id": 1, "name": "Presionar", "script": "..." }`.
- `400`: script rechazado por las validaciones del motor.
- `403`: el usuario no tiene club.
- `409`: ya existe ese nombre para el club.
- `422`: body inválido.

### `GET /api/comportamientos`

Requiere bearer token. Respuesta `200`: `{ "comportamientos": [{ "id": 1, "nombre": "Presionar" }] }`. `403` si el usuario no tiene club.

### `GET /api/comportamientos/{name}`

Requiere bearer token. Obtiene un comportamiento del club propio.

- `200`: `{ "id": 1, "name": "Presionar", "script": "..." }`.
- `403`: el usuario no tiene club.
- `404`: comportamiento no encontrado.

### `PUT /api/comportamientos/{nombre_comportamiento}`

Requiere bearer token. Body JSON, con al menos uno de los campos:

```json
{ "name": "Nuevo nombre", "script": "nuevo código Python" }
```

Ambos campos son opcionales; `name` debe tener 1–100 caracteres no blancos y `script` no puede estar vacío. Actualiza los campos enviados.

- `200`: comportamiento actualizado `{ "id": 1, "name": "...", "script": "..." }`.
- `400`: script inválido.
- `403`: usuario sin club.
- `404`: comportamiento no encontrado.
- `409`: comportamiento usado por una liga aún activa o nombre duplicado.
- `422`: body inválido o sin campos.

### `DELETE /api/comportamientos/{nombre_comportamiento}`

Requiere bearer token. Elimina el comportamiento del club propio.

- `204`: sin body.
- `403`: usuario sin club.
- `404`: comportamiento no encontrado.
- `409`: el comportamiento está asignado a un jugador.

### `POST /api/comportamientos/{id_comportamiento}/probar`

Requiere bearer token. No recibe body; simula 20 ticks con un jugador y un compañero ficticios.

- `200`: `{ "ancho_cancha": number, "alto_cancha": number, "ticks": 20, "frames": [{ "tick": 0, "jugador": { "x": number, "y": number }, "companero": { "x": number, "y": number }, "pelota": { "x": number, "y": number } }] }`.
- `400`: script almacenado inválido.
- `403`: usuario sin club.
- `404`: comportamiento no encontrado en el club.

## Amistosos

### `POST /api/clubes/{id_club_rival}/amistosos/solicitar`

Requiere bearer token. Body JSON:

```json
{
  "formacion_tactica": 1,
  "jugadores_convocados": [
    { "id_jugador": 1, "rol": "titular", "comportamiento": "Presionar" },
    { "id_jugador": 2, "rol": "titular", "comportamiento": "Presionar" },
    { "id_jugador": 3, "rol": "titular", "comportamiento": "Presionar" },
    { "id_jugador": 4, "rol": "suplente", "comportamiento": null },
    { "id_jugador": 5, "rol": "suplente", "comportamiento": null },
    { "id_jugador": 6, "rol": "suplente", "comportamiento": null }
  ]
}
```

`formacion_tactica`: `1` equilibrada, `2` defensiva o `3` ofensiva. Deben ser seis jugadores del club retador, sin IDs duplicados, con tres titulares (cada uno con comportamiento válido) y tres suplentes (sin comportamiento). El club rival debe contar con al menos seis jugadores. La invitación vence en cinco minutos.

- `200`: `{ "id_solicitud_amistoso": 1, "mensaje": "..." }`.
- `400`: usuario sin club, club rival con menos de seis jugadores o convocatoria inválida.
- `404`: club rival no encontrado.
- `422`: body inválido.

### `GET /api/amistosos/solicitudes/`

Requiere bearer token. Lista las invitaciones pendientes y no vencidas dirigidas al club del usuario.

- `200`: `{ "lista_amistosos": [{ "id_solicitud": 1, "id_club_retador": 2, "nombre_club_retador": "Rival FC", "avatar": "...", "tiempo_restante": 123 }] }`.
- `404`: el usuario no tiene club.

### `POST /api/amistosos/solicitudes/{id_solicitud}/aceptar`

Requiere bearer token. Body igual al de solicitud de amistoso, con la convocatoria del club que acepta.

- `200`: `{ "id_solicitud": 1, "id_partido": 5, "formacion_equipo_0": 1, "formacion_equipo_1": 2, "convocados_equipo_0": [...], "convocados_equipo_1": [...] }`.
- `400`: solicitud no respondible/convocatoria inválida o solicitud sin convocatoria válida.
- `404`: club no encontrado para el usuario o solicitud vencida/no encontrada.
- `422`: body inválido.

Al aceptar, se crea y lanza el partido; la respuesta contiene la formación y convocatorias de ambos equipos.

### `POST /api/amistosos/solicitudes/{id_solicitud}/rechazar`

Requiere bearer token. Sin body.

- `200`: `{ "id_solicitud_amistoso": 1, "mensaje": "Se rechazo con exito" }`.
- `400`: solicitud no pendiente/no dirigida al usuario o ya vencida.
- `404`: el usuario no tiene club.

## Ligas

En las convocatorias de liga se requieren seis jugadores distintos del club: tres `titular` con nombre de comportamiento y tres `suplente` sin comportamiento. La formación admite `1`, `2` o `3`.

### `POST /api/ligas/`

Requiere bearer token. Body JSON:

```json
{
  "name": "Liga Norte",
  "password": null,
  "duracion_partido": 2,
  "cantidad_equipos": 4,
  "formacion_tactica": 1,
  "jugadores_convocados": [
    { "id_jugador": 1, "rol": "titular", "comportamiento": "Presionar" }
  ]
}
```

El array debe contener exactamente seis convocados; el ejemplo de arriba es parcial. `name`: 1–25 caracteres; `password`: opcional, si se informa 6–50 caracteres; `duracion_partido` > 0; `cantidad_equipos`: 4–50.

- `201`: `{ "id_liga": 1, "owner_id": 10, "lista_clubes_participantes": [{ "name": "Club Ejemplo", "photo": "..." }] }`. `owner_id` es el ID del club creador.
- `400`: usuario sin club o convocatoria/jugador/comportamiento inválidos.
- `409`: nombre de liga ya utilizado.
- `422`: body inválido.

Sin contraseña, la liga es pública; con contraseña no vacía, es privada.

### `POST /api/ligas/{id_liga}/unirse`

Requiere bearer token. Body JSON:

```json
{
  "contrasenia": null,
  "formacion_tactica": 1,
  "jugadores_convocados": [
    { "id_jugador": 1, "rol": "titular", "comportamiento": "Presionar" }
  ]
}
```

El array debe contener seis jugadores válidos; el ejemplo es parcial. `contrasenia` se requiere para ligas privadas.

- `200`: `{ "id_liga": 1, "id_club": 10, "nombre_club": "Club Ejemplo", "formacion_tactica": 1, "jugadores_convocados": [{ "id_detalle": 1, "id_jugador": 1, "rol": "titular", "id_comportamiento": 5 }] }`.
- `401`: contraseña incorrecta para liga privada.
- `403`: liga completa.
- `404`: liga inexistente.
- `409`: liga fuera del estado de espera o club ya inscripto.
- `422`: body inválido.

### `GET /api/ligas?limit=20&skip=0`

Requiere bearer token y club propio. `limit`: 1–20; `skip`: >= 0.

- `200`: `{ "lista_liga": [{ "id": 1, "name": "Liga Norte", "duracion_partido": 2, "maxima_cantidad_clubes": 4, "cantidad_de_equipos_insc": 3, "tipo": "publica" }] }`.
- La lista incluye ligas no finalizadas, incluso completas o iniciadas, para que se pueda acceder al lobby. Las terminadas/eliminadas no se incluyen.
- `404`: usuario sin club.
- `422`: query inválida.

### `GET /api/ligas/{id_liga}/alineacion`

Requiere bearer token. Consulta la convocatoria del club del usuario en la liga.

- `200`: `{ "id_liga": 1, "id_club": 10, "formacion_tactica": 1, "jugadores_convocados": [{ "id_jugador": 1, "rol": "titular", "comportamiento": "Presionar" }] }`.
- `403`: el usuario no tiene club o su club no participa en la liga.
- `404`: liga inexistente.

### `PUT /api/ligas/{id_liga}/alineacion`

Requiere bearer token. Body `ConvocatoriaCreate`: `formacion_tactica` (`1`, `2` o `3`) y `jugadores_convocados` (exactamente seis, con la estructura descrita arriba).

- `200`: devuelve el mismo esquema `AlineacionLigaResponse` con la alineación actualizada.
- `400`: jugadores/roles/comportamientos inválidos.
- `403`: club inexistente o no inscripto en la liga.
- `404`: liga inexistente.
- `422`: body inválido.
- `500`: no se pudo actualizar la alineación.

### `GET /api/ligas/{id_liga}/lobby`

Requiere bearer token y un club propio. Solo responde si la liga está iniciada.

- `200`: `{ "Datos": { "id": 1, "name": "Liga Norte", "duracion_partido": 2, "maxima_cantidad_clubes": 4, "cantidad_de_equipos_insc": 4, "tipo": "publica" }, "fixture": [...], "tablas_posiciones": [...] }`.
- Cada fixture incluye fecha, horario, partido, IDs/nombres/avatares de los dos clubes, estado (`EN_ESPERA`, `EN_VIVO` o `FINALIZADO`) y goles.
- Cada posición incluye `id_club`, `nombre`, `avatar`, `pj`, `pg`, `pe`, `pp`, `gf`, `gc`, `dif` y `pts`.
- `400`: liga todavía no iniciada.
- `403`: usuario sin club.
- `404`: liga inexistente.

### `PUT /api/ligas/{id_liga}/iniciar`

Requiere bearer token; solo el club creador puede iniciar. Se requieren al menos tres clubes inscriptos y no superar el máximo de equipos.

- `200`: `{ "mensaje": "Liga iniciada correctamente", "id_liga": 1 }`.
- `400`: liga no está en espera, creador no inscripto, cantidad insuficiente/excedida u otra precondición inválida.
- `403`: usuario no es creador.
- `404`: liga o club inexistentes.
- Al iniciar se genera el fixture, se programa la ejecución automática y la liga permanece en el listado para acceder al lobby.

### `PUT /api/ligas/{id_liga}/{id_usuario}`

Abandona una liga. Requiere bearer token y que `id_usuario` coincida con el usuario autenticado. Sin body.

- `200`: `{ "message": "Eliminado correctamente" }`.
- `400`: liga iniciada, el creador intenta abandonar o precondición inválida.
- `403`: ID de usuario no coincide.
- `404`: club/liga inexistente o club no inscripto.

### `DELETE /api/ligas/{id_liga}`

Cancela/elimina una liga en espera. Solo el club creador puede hacerlo. Sin body.

- `200`: `{ "message": "Liga eliminada correctamente", "id_liga": 1 }`.
- `400`: liga ya iniciada/finalizada.
- `403`: usuario no es creador.
- `404`: liga o club inexistentes.

No existe actualmente un endpoint REST `/api/ligas/{id_liga}/tabla`; la tabla se entrega dentro de la respuesta del lobby.

## Partidos

### `POST /api/partidos/{id_partido}/sustituciones`

Requiere bearer token. Body JSON:

```json
{ "id_jugador_sale": 1, "id_jugador_entra": 4 }
```

Encola como máximo una sustitución por equipo para la próxima pausa disponible.

- `200`: `{ "id_partido": 5, "id_jugador_sale": 1, "id_jugador_entra": 4, "tick_aplicacion": 150, "mensaje": "..." }`.
- `400`: IDs/jugadores no elegibles, jugador que sale no está en cancha o entrante no es suplente.
- `403`: usuario sin club o sin autoridad sobre el partido/jugador.
- `404`: partido inexistente o no está en curso.
- `409`: no quedan pausas o ya hay una sustitución pendiente para ese equipo.
- `422`: body inválido.

### `PUT /api/partidos/{id_partido}/jugadores/{id_jugador}/comportamiento`

Requiere bearer token. Body JSON; indicar al menos uno de los campos:

```json
{ "id_comportamiento": 2 }
```

o `{ "nombre": "Presionar" }`.

- `200`: `{ "id_partido": 5, "id_jugador": 1, "id_comportamiento": 2, "nombre": "Presionar", "mensaje": "Comportamiento actualizado para el próximo tick." }`.
- `400`: no se indicó ID ni nombre.
- `403`: usuario sin club, no controla un equipo del partido o el jugador no pertenece a su club.
- `404`: partido/runtime/jugador/comportamiento inexistente o fuera del partido.
- `422`: body inválido.

## WebSockets

Todos los canales a continuación usan JWT en `?token=...`. Los canales son de lectura: el cliente puede enviar texto para mantener abierta la conexión, pero no para invocar acciones de juego.

### Notificaciones de amistosos: `WS /ws/{user_id}`

El ID de la ruta debe coincidir con `user_id` dentro del JWT. Al destinatario de un desafío se le envía:

```json
{ "tipo": "NUEVA_SOLICITUD_AMISTOSO", "id_solicitud": 1, "id_club_retador": 10, "expira_en_segundos": 300, "mensaje": "..." }
```

Al retador pueden llegar eventos `desafio_aceptado` o `desafio_rechazado` con los datos del partido/club rival.

### Partidos propios activos: `WS /ws/mis-partidos-activos`

Al conectar, envía un array de partidos activos del club asociado al usuario. Cada elemento contiene `id_partido`, `id_club_local`, `nombre_local`, `id_club_visitante`, `nombre_visitante`, `goles_local`, `goles_visitante` y `estado`. El array se vuelve a enviar cuando cambia el listado de partidos activos.

### Listado de ligas: `WS /ws/ligas`

Envía inicialmente `{ "tipo": "listado", "ligas": [...] }`. Cada liga incluye `id_liga`, `nombre`, `cantidad_equipos`, `cupos_ocupados`, `duracion_partido`, `es_privada` y `estado_liga`. Luego transmite eventos como `evento_liga_creada`, `evento_liga_actualizada` y `evento_liga_no_disponible`. Las ligas completas o iniciadas no se retiran del listado; las finalizadas o eliminadas sí.

### Lobby de liga: `WS /ws/ligas/{id_liga}`

Envía al conectar un `snapshot_liga` con `id_liga`, `cupos_ocupados`, datos serializados de la liga y `lista_liga` de clubes participantes. Después puede enviar snapshots actualizados, ocupación, estado y eventos de partido.

### Descubrimiento de partidos en vivo: `WS /websocket/partidos-en-vivo`

Envía `{ "tipo": "PARTIDOS_EN_VIVO", "partidos": [...] }` al conectar. Notifica nuevas disponibilidades con `PARTIDO_DISPONIBLE` y finalizaciones con `PARTIDO_NO_DISPONIBLE`.

### Espectador de partido: `WS /websocket/partidos/{id_partido}`

Solo admite partidos actualmente activos. Al conectar envía un `PARTIDO_INICIAL` con cancha, clubes, formaciones y alineaciones; luego un `PARTIDO_ESTADO` y los estados/eventos en vivo. Un partido finalizado o no disponible devuelve `ERROR_PARTIDO` y cierra con código `4404`.

Los mensajes de los jugadores participantes incluyen estados `PARTIDO_INICIADO`, `PARTIDO_TICK`, `PARTIDO_PAUSADO`, `PARTIDO_REANUDADO` y `PARTIDO_FINALIZADO`. Si hay definición por penales, también incluyen `PENALES_INICIADOS`, `PENAL_RONDA_INICIADA`, `PENAL_TICK` y `PENAL_RONDA_FINALIZADA`; esos mensajes contienen el estado (incluida la posición de la pelota) de la ronda actual.

## Librería de comportamientos (no son endpoints HTTP)

Los scripts de los jugadores se ejecutan en el sandbox del motor. Las primitivas y sensores no son rutas de la API.

### Primitivas

- `correr(x, y, z)`: se mueve hacia las coordenadas indicadas; `z` es la intensidad de 0 a 100. Con `z = 0`, el jugador queda quieto.
- `patearConFuerza(x, y, z)`: patea hacia las coordenadas indicadas con fuerza de 0 a 100; requiere estar en rango de la pelota.

### Sensores

| Función | Resultado | Uso |
|---|---|---|
| `puedo_patear()` | `bool` | Comprueba si el jugador está en rango para patear. |
| `mi_posicion()` | `(x, y)` | Posición del jugador actual. |
| `posicion_pelota()` | `(x, y)` | Posición de la pelota. |
| `rival_mas_cercano()` | jugador `{id, x, y}` | Rival más próximo. |
| `companero_mas_cercano()` | jugador `{id, x, y}` | Compañero más próximo. |
| `jugadores_propios()` | lista de jugadores | Jugadores propios en cancha. |
| `jugadores_rivales()` | lista de jugadores | Rivales en cancha. |
| `posicion_arco_propio()` | `(x, y)` | Centro del arco propio. |
| `posicion_arco_rival()` | `(x, y)` | Centro del arco rival. |
| `tiempo_restante()` | entero | Tiempo restante. |
| `marcador()` | `(goles_propios, goles_rivales)` | Marcador actual. |
| `id_propio()` | entero | ID del jugador que ejecuta el script. |
| `distancia_a(x, y)` | float | Distancia a un punto. |

### Funciones compuestas disponibles

- `marcar(id_jugador_rival, referencia)`: se ubica entre el rival y una referencia (`pelota` o `arco`).
- `perseguir_jugador(id)`: corre hacia el jugador indicado.
- `perseguir_pelota()`: corre hacia la pelota.
- `rematar_al_arco()`: si puede patear, dispara al arco rival con fuerza máxima.
- `pasar_a(id_companero)`: pasa al compañero con fuerza predeterminada 15.
- `despejar()`: despeja con fuerza máxima en dirección contraria al arco propio.
- `cubrir_posicion(x, y)`: corre a una posición táctica.
- `evadir(id_rival)`: se aleja del rival indicado.
- `transportar_balon(x, y)`: se mueve hacia el destino y usa un toque suave predeterminado en 15 cuando puede patear.

### Reglas del motor

- El script es stateless entre ticks: no conserva variables entre ejecuciones.
- Cada jugador dispone de una acción por tick; las funciones compuestas terminan usando las primitivas.
- El timeout documentado del motor es 100 ms por tick; si se excede, el jugador no realiza acción.
- Si dos jugadores intentan patear en disputa, se prioriza mayor atributo de Poder; en empate, decide el azar.
