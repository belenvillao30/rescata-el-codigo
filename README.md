# Operación: ¡Rescata el código! — versión multiequipo

Web estática lista para GitHub Pages + Supabase (PostgreSQL y Auth). **No basta con alojar el HTML**: Supabase permite que varios equipos compartan respuestas y puntajes, sin exponer las soluciones.

## Funciones

- 8 niveles, **17 preguntas** basadas en las versiones previas del juego.
- 4 equipos con nombre y **código de acceso privado**. Cada equipo responde desde su teléfono o computadora.
- Docente con correo y contraseña de Supabase Auth, visible únicamente si se autoriza su UUID en `game_admins`.
- Un cronograma sincronizado basado en la **hora del servidor**, sin que un equipo pueda avanzar temprano; tras contestar no se permite modificar la respuesta.
- Fácil: **30 segundos, +5 si acierta / −2 si falla**; intermedio: **60 segundos, +10 / −5**; avanzado: **90 segundos, +20 / −10**. No responder: **0**. Los puntajes pueden ser negativos: las penalizaciones se aplican al acumulado.
- El docente puede modificar **tiempo, recompensa y penalización pregunta por pregunta**, pero únicamente antes de iniciar la partida.
- Clasificación y respuestas por equipo, refrescadas cada 2,5 segundos. Los puntos solo aparecen en el marcador una vez que **termina** la pregunta.
- Reinicio de todas las respuestas y calificaciones desde el panel docente (con confirmación).

## Configuración (15 pasos)

1. Crea un proyecto en https://supabase.com/dashboard.
2. En **SQL Editor**, ejecuta el contenido completo de `supabase_setup.sql`, una sola vez en una base nueva.
3. En **Authentication > Providers > Email** comprueba que el ingreso por correo/contraseña esté habilitado.
4. En **Authentication > Users > Add user**, crea la cuenta del docente con correo/contraseña. Para la cuenta docente, puedes marcar el correo como confirmado.
5. En **SQL Editor**, ejecuta este SQL con el correo real del docente:
   ```sql
   insert into public.game_admins(user_id)
   select id from auth.users where email='TU_CORREO@DOMINIO.COM'
   on conflict do nothing;
   ```
6. En **Project Settings > API** (o API Keys), copia la URL del proyecto y la **clave pública anon / publishable**. **Nunca** utilices la clave `service_role` / secret.
7. Para evitar que cada alumno tenga que configurar Supabase, abre `index.html` y edita al principio del bloque `<script>`:
   ```javascript
   const CONFIG={url:'https://TU-PROYECTO.supabase.co',anonKey:'TU_CLAVE_PUBLICA'};
   ```
   Estas claves son públicas por diseño; la seguridad está en las funciones y permisos SQL.
8. Crea un repositorio de GitHub y sube `index.html` a la raíz. **No subas** correos con contraseñas ni claves secretas.
9. En GitHub: **Settings > Pages > Deploy from a branch > main / (root)**.
10. Abre la URL publicada `https://USUARIO.github.io/REPOSITORIO/`.
11. Ingresa con tu correo docente y contraseña.
12. **Antes de iniciar**, cambia los cuatro códigos desde el panel docente. Los códigos de demostración del SQL son `ALFA-271`, `BETA-438`, `GAMA-592` y `DELTA-864`: son públicos porque aparecen en el archivo de instalación, así que **no debes dejarlos para uso real**.
13. Comparte URL, nombre y código individual con cada equipo. Cada equipo responde en su navegador.
14. Configura tiempos, recompensa y penalizaciones si lo necesitas. Hazlo **antes de comenzar**.
15. Pulsa **Iniciar juego sincronizado**. Los equipos que se incorporen tarde verán la pregunta que está activa, no las anteriores.

## Reglas pedagógicas

La partida tiene 17 preguntas; la configuración inicial suma **17 minutos** cronometrados, más el tiempo de presentación y cierre. El reloj funciona desde que el docente inicia y **no se pausa** cuando alguien desconecta o cierra el navegador. Los cambios de pregunta son automáticos; no se muestra la explicación correcta hasta después de que termina cada pregunta y, por ahora, solo en el historial personal del equipo que la respondió. Todos pueden consultar el marcador.

Las preguntas inicialmente abiertas o de secuenciación se transformaron en elección múltiple para permitir calificación automática. Puedes revisar esas adaptaciones en `questions_public_preview.json` (sin respuestas) y en el SQL de instalación.

## Seguridad y limitaciones

- No hay credenciales ni soluciones correctas en el HTML servido a los estudiantes.
- Las tablas no conceden lectura directa a `anon`/`authenticated`; todo acceso ocurre por procedimientos que validan un token de sesión de equipo o la cuenta docente.
- La autenticación de equipo por código es práctica para un juego, pero **no es equivalente a una identidad estudiantil verificada**: comparte los códigos solo con su equipo.
- El navegador guarda la sesión del equipo solo en `sessionStorage` (hasta cerrar la pestaña); el progreso real permanece en la base de datos. Si se cierra la pestaña, basta volver a ingresar con el código.
- Para una implementación institucional con muchas personas o alto riesgo de ataques, conviene reforzar los límites de intentos y controles contra abuso mediante Edge Functions / rate limiting. Supabase puede tener cuotas y límites en su plan.
- **No probado en un proyecto Supabase activo** en esta sesión. Es una implementación lista para configurar y probar; haz una prueba real con cuatro navegadores antes de la clase.
- El juego usa consultas cada 2,5 segundos, no WebSockets. Hay un pequeño retraso en la actualización del marcador, aunque el tiempo de recepción de respuestas se valida en el servidor.
- Reiniciar elimina las respuestas previas de la partida. Exporta información antes de reiniciar si necesitas conservarla.
