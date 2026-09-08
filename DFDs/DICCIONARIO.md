# Diccionario de datos

Usuario: id_usuario, username, email

Credenciales de registro o inicio de sesión: username, password, email opcional

Credenciales con hash: username, password_hash, email

Token de sesión: token (JWT), asociado a id_usuario

Club: id_club, nombre

Datos del club: id_club, nombre, avatar

Cambio de nombre de club: nuevo_nombre

Cambio de avatar de club: avatar (archivo)

Liga: id_liga, nombre


Datos para crear una liga: nombre, contrasena opcional, duracion_partido, cantidad_equipos

Jugador: id_jugador, nombre

Datos del jugador: id_jugador, nombre, skills (fuerza, velocidad, control, poder, agilidad)

Comportamiento: id_comportamiento, nombre

Datos del comportamiento: nombre, codigo_python

Convocatoria: lista de 6 objetos { id_jugador, rol, comportamiento } y formacion_tactica (entero). rol puede ser titular o suplente; comportamiento aplica solo a titulares.

Partido: id_partido, equipo_local, equipo_visitante, estado

Datos completos del partido: id_partido, equipo_local, equipo_visitante, estado, goles_local, goles_visitante, fecha

Solicitud de amistoso: id_solicitud_amistoso, id_club_rival

Fixture: lista de { id_partido, fecha, equipo_local, equipo_visitante, estado, goles_local, goles_visitante }

Tabla de liga: lista de { numero_posicion, club, partidos_jugados, partidos_ganados, partidos_perdidos, partidos_empatados, goles_a_favor, goles_en_contra, goles_diferencia, puntos }

Tabla global: lista de { numero_posicion, club, partidos_jugados, goles_a_favor, partidos_ganados, partidos_empatados, partidos_perdidos, puntos }

Plantel propio en una liga: lista de 6 { id_jugador, nombre, rol (titular/suplente), skills, comportamiento_asignado }

Plantel de un club rival en una liga: lista de 6 { id_jugador, nombre, skills }, sin comportamiento_asignado porque esa información no se expone del rival
