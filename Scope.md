### Especificación Técnica Refinada – Proyecto SmartEdify

---

## 1. Arquitectura General

* **Frontend User Web**

  * Dominio: `www.smart-edify.com`
  * Landing page con botón en header que abre **modal de login**.
  * Tras login exitoso → dashboard que muestra conexiones del usuario.
  * No tiene CRUD.
  * Deploy en **Render** (Docker + Nginx).

* **Frontend Admin Web**

  * Dominio: `admin.smart-edify.com`
  * Login directo en pantalla dedicada.
  * Dashboard con:

    * **CRUD de usuarios** (crear, listar, editar, borrar).
    * Tabla de conexiones (todos los usuarios).
  * Deploy en **Render** (Docker + Nginx).

* **Backend API**

  * Dominio: `api.smart-edify.com`
  * Endpoints:

    * `POST /login` → valida usuario en tabla `users`. Si es correcto:

      * Genera mensaje con fecha/hora (zona Lima).
      * Guarda registro en tabla `connections`.
      * Devuelve mensaje.
    * `GET /connections?user_id=X` → lista conexiones de un usuario.
    * `GET /users` / `POST /users` / `PUT /users/:id` / `DELETE /users/:id` → CRUD de usuarios (solo admin).
    * `GET /health` → para Render.
  * Tecnologías: Node.js + Express + PostgreSQL.
  * Deploy en **Render** (Docker).

* **Base de Datos PostgreSQL**

  * Administrada por Render.
  * Tablas:

    ```sql
    CREATE TABLE users (
      id SERIAL PRIMARY KEY,
      username TEXT UNIQUE NOT NULL,
      password TEXT NOT NULL, -- en real debería ser hash
      role TEXT NOT NULL DEFAULT 'user' -- valores: user, admin
    );

    CREATE TABLE connections (
      id BIGSERIAL PRIMARY KEY,
      user_id INT REFERENCES users(id),
      message TEXT NOT NULL,
      created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
    );
    ```

* **Aplicación Móvil (Expo React Native)**

  * Splash 3 seg.
  * Login con modal igual a user web.
  * Dashboard → tabla de conexiones del usuario (sin CRUD).
  * APK generado con **Expo EAS Build**.

---

## 2. Lógica de Autenticación

1. Credenciales se validan en tabla `users`.
2. Si son correctas:

   * Se genera mensaje:

     ```
     Hola <username>, hoy es <weekday> y has ingresado a las <hh:mm> del <dd>: <mmmm>:<yyyy>
     ```
   * Se guarda en tabla `connections`.
   * Se responde con JSON `{ok:true,message:...,user:{id,username,role}}`.
3. Frontend guarda sesión (localStorage en webs, AsyncStorage en móvil).
4. Rol `admin` habilita acceso al CRUD.

---

## 3. Variables de Entorno

En Render → **Environment**:

* Generales:

  * `PORT=10000`
  * `TZ=America/Lima`

* Backend:

  * `DATABASE_URL` (Render la expone al enlazar DB).
  * `JWT_SECRET` (para emitir tokens si se desea extender seguridad).

* Webs:

  * `VITE_API_URL=https://api.smart-edify.com`

---

## 4. Diferencias entre Frontend User y Admin

| Aspecto   | User (www)                       | Admin (admin)              |
| --------- | -------------------------------- | -------------------------- |
| Acceso    | Modal login desde landing        | Página directa de login    |
| Dashboard | Tabla conexiones del propio user | CRUD + conexiones de todos |
| Roles     | Solo `user`                      | `admin`                    |

---

## 5. Flujo de Datos

1. Usuario ingresa credenciales en login.
2. Frontend → `POST /login` en API.
3. API valida → guarda conexión → responde mensaje.
4. Web/Móvil muestran mensaje en dashboard.
5. Admin puede además usar CRUD para gestionar `users`.

---

## 6. DNS y Dominios

En **Squarespace DNS**:

| Host  | Tipo  | Target en Render                        |
| ----- | ----- | --------------------------------------- |
| www   | CNAME | `<frontend-user-service>.onrender.com`  |
| admin | CNAME | `<frontend-admin-service>.onrender.com` |
| api   | CNAME | `<backend-service>.onrender.com`        |

Render generará SSL para cada subdominio.

---

## 7. Tareas del Equipo

1. **Backend**

   * Implementar API con Express.
   * Conexión a PostgreSQL.
   * Endpoints `/login`, `/users` (CRUD), `/connections`.

2. **Frontend User**

   * Landing con modal login.
   * Dashboard con tabla de conexiones del usuario logueado.

3. **Frontend Admin**

   * Login directo.
   * Dashboard con tabla de todos los usuarios y CRUD.

4. **Móvil**

   * Expo app con splash.
   * Login.
   * Dashboard con tabla de conexiones (sin CRUD).

---

## 8. Limitaciones de MVP Dummy

* Password en texto plano (para demo).
* Sin JWT/token persistente (sesión básica con localStorage o AsyncStorage).
* CORS abierto a dominios web configurados.
* Seguridad mínima, no apto para producción real.

---

¿Quieres que arme un **diagrama de arquitectura (cajas y flechas)** mostrando User Web, Admin Web, Backend API, DB y Móvil conectándose a Render?
