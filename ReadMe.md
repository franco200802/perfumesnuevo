# Documentación técnica — TP3 HogarPay (rama `dev`)
> Proyecto: **HogarPay** — Sistema de gestión de facturas de servicios del hogar  
> Stack: React + TypeScript + Vite (frontend) con Axios para comunicarse con una API REST

---

## Índice

1. [Estructura general del proyecto](#1-estructura-general-del-proyecto)
2. [Punto de entrada — `main.tsx`](#2-punto-de-entrada--maintsx)
3. [Enrutamiento principal — `App.tsx`](#3-enrutamiento-principal--apptsx)
4. [Tipos e interfaces — `types/index.ts`](#4-tipos-e-interfaces--typesindexts)
5. [Contexto de autenticación — `context/AuthContext.tsx`](#5-contexto-de-autenticación--contextauthcontexttsx)
6. [Componentes — `components/`](#6-componentes--components)
   - [Navbar.tsx](#navbartsx)
   - [PrivateRoute.tsx](#privateroute-tsx)
   - [ConfettiEffect.tsx](#confettieffecttsx)
7. [Servicios — `services/`](#7-servicios--services)
   - [api.ts](#apits)
   - [authService.ts](#authservicets)
   - [facturaService.ts](#facturaservicets)
   - [alertaService.ts](#alertaservicets)
   - [estadisticaService.ts](#estadisticaservicets)
   - [servicioService.ts](#servicioservicets)
   - [solicitudService.ts](#solicitudservicets)
   - [usuarioService.ts](#usuarioservicets)
8. [Páginas — `pages/`](#8-páginas--pages)
   - [LoginPage.tsx](#loginpagetsx)
   - [RegisterPage.tsx](#registerpagetsx)
   - [ProfilePage.tsx](#profilepagetsx)
   - [FacturasPage.tsx](#facturaspagetsx)
   - [AlertasPage.tsx](#alertaspagetsx)
   - [DashboardPage.tsx](#dashboardpagetsx)
   - [MiCuentaPage.tsx](#micuentapagetsx)
   - [SolicitudesAdminPage.tsx](#solicitudesadminpagetsx)
   - [NotFoundPage.tsx](#notfoundpagetsx)

---

## 1. Estructura general del proyecto

```
frontend/
├── index.html
├── vite.config.ts
├── package.json
└── src/
    ├── main.tsx              ← punto de entrada React
    ├── App.tsx               ← árbol de rutas principal
    ├── index.css / App.css   ← estilos globales
    ├── types/
    │   └── index.ts          ← todas las interfaces TypeScript
    ├── context/
    │   └── AuthContext.tsx   ← estado global de sesión/autenticación
    ├── components/
    │   ├── Navbar.tsx        ← barra de navegación
    │   ├── PrivateRoute.tsx  ← protección de rutas
    │   └── ConfettiEffect.tsx← efecto visual de celebración
    ├── services/
    │   ├── api.ts            ← instancia Axios centralizada
    │   ├── authService.ts
    │   ├── facturaService.ts
    │   ├── alertaService.ts
    │   ├── estadisticaService.ts
    │   ├── servicioService.ts
    │   ├── solicitudService.ts
    │   └── usuarioService.ts
    └── pages/
        ├── LoginPage.tsx
        ├── RegisterPage.tsx
        ├── ProfilePage.tsx
        ├── FacturasPage.tsx
        ├── AlertasPage.tsx
        ├── DashboardPage.tsx
        ├── MiCuentaPage.tsx
        ├── SolicitudesAdminPage.tsx
        └── NotFoundPage.tsx
```

---

## 2. Punto de entrada — `main.tsx`

Archivo mínimo que monta la app React en el DOM.

| Función / llamada | Descripción |
|---|---|
| `createRoot(document.getElementById('root')!)` | Crea el root de React sobre el `<div id="root">` del `index.html` |
| `.render(<StrictMode><App /></StrictMode>)` | Renderiza `App` envuelta en `StrictMode` para detectar problemas en desarrollo |

---

## 3. Enrutamiento principal — `App.tsx`

Define todas las rutas de la SPA usando `react-router-dom`.

### Componente `RoleRedirect`

```ts
const RoleRedirect: React.FC = () => { ... }
```

Redirige a la ruta correcta según el rol del usuario al acceder a `/`.

| Condición | Redirección |
|---|---|
| No hay usuario logueado | `/login` |
| Usuario con rol `ADMIN` | `/dashboard` |
| Usuario con rol `CLIENTE` | `/facturas` |

### Componente `App`

Árbol de rutas que:
- Envuelve todo con `<AuthProvider>` (estado global de sesión)
- Renderiza `<Navbar>` en todas las páginas
- Define las siguientes rutas:

| Ruta | Componente | Acceso |
|---|---|---|
| `/login` | `LoginPage` | Público |
| `/registro` | `RegisterPage` | Solo `ADMIN` |
| `/perfil` | `ProfilePage` | Autenticado |
| `/facturas` | `FacturasPage` | Autenticado |
| `/alertas` | `AlertasPage` | Autenticado |
| `/dashboard` | `DashboardPage` | Solo `ADMIN` |
| `/mi-cuenta` | `MiCuentaPage` | Autenticado |
| `/solicitudes` | `SolicitudesAdminPage` | Solo `ADMIN` |
| `/` | `RoleRedirect` | Redirige según rol |
| `*` | `NotFoundPage` | Cualquier ruta no encontrada |

---

## 4. Tipos e interfaces — `types/index.ts`

Define todas las entidades de datos del sistema.

### `LoginRequest`
Credenciales enviadas al backend para autenticar.
```ts
{ email: string; password: string }
```

### `LoginResponse`
Respuesta del backend tras el login exitoso. Se guarda en `localStorage`.
```ts
{ token: string; dni: string; email: string; rol: string }
```

### `Usuario`
Representa un usuario del sistema.
```ts
{ dni: string; email: string; rol: string; activo?: boolean }
```

### `Servicio`
Medidor/servicio de un cliente (luz, gas, agua).
```ts
{ numeroMedidor: string; tipoServicio: string; cliente: Usuario }
```

### `Factura`
Factura emitida para un servicio.
```ts
{ nroFactura: number; monto: number; fechaVencimiento: string; estado: 'PENDIENTE' | 'PAGADA' | 'VENCIDA'; servicio: Servicio }
```

### `Pago`
Registro de transacción de pago de una factura.
```ts
{ nroTransaccion: number; fechaPago: string; pdfUrl: string; estado: 'EXITOSO' | 'FALLIDO' | 'PENDIENTE'; factura: Factura }
```

### `Alerta`
Alerta generada por consumo anómalo.
```ts
{ id: number; tipoAlerta: string; mensaje: string; consumoActual: number; consumoPromedio: number; porcentajeDesvio: number; reconocida: boolean; fechaCreacion: string }
```

### `Estadisticas`
Métricas globales del sistema para el dashboard admin.
```ts
{ totalUsuarios: number; totalServicios: number; totalFacturas: number; montoTotalPagado: number; facturasPorEstado: Record<string, number> }
```

### `SolicitudCambio`
Solicitud enviada por un cliente para cambiar email o contraseña.
```ts
{ id: number; cliente: Usuario; campo: string; valorNuevo: string; estado: 'PENDIENTE' | 'APROBADA' | 'RECHAZADA'; fechaCreacion: string; fechaResolucion?: string; adminResolutor?: Usuario }
```

---

## 5. Contexto de autenticación — `context/AuthContext.tsx`

Provee el estado global de sesión a toda la app mediante React Context.

### Funciones internas (helpers privados)

#### `decodeJwtPayload(token: string)`
Decodifica manualmente el payload de un JWT (sin librerías externas).

- Toma la segunda parte del token (separada por `.`)
- Convierte de Base64URL a Base64 estándar
- Parsea el JSON resultante
- **Retorna**: `Record<string, unknown> | null`

#### `getRolFromToken(token?: string)`
Extrae el campo `rol` del payload JWT.

- Llama a `decodeJwtPayload`
- **Retorna**: `string | null` (el rol del usuario o null si no puede decodificar)

#### `isTokenExpired(token?: string)`
Verifica si un token JWT ya expiró comparando el campo `exp` con la fecha actual.

- **Retorna**: `true` si el token está vencido o es inválido, `false` si sigue vigente

### `AuthProvider` (componente)

Proveedor del contexto. Inicializa el estado leyendo desde `localStorage`.

**Estado inicial**: Lee `localStorage.getItem('auth')`, valida que el token no esté expirado. Si está expirado, lo elimina y arranca con sesión nula.

#### `login(userData: LoginResponse)`
- Guarda `userData` en `localStorage` bajo la clave `'auth'`
- Actualiza el estado `user`
- Memorizado con `useCallback`

#### `logout()`
- Elimina la clave `'auth'` de `localStorage`
- Pone `user` en `null`
- Memorizado con `useCallback`

#### `isAdmin` (valor derivado)
- Valor booleano calculado con `useMemo`
- Llama a `getRolFromToken(user?.token)` y verifica si el rol es `'ADMIN'`
- Se recalcula solo cuando cambia `user.token`

### `useAuth()`
Hook personalizado para consumir el contexto desde cualquier componente.
- Lanza un error si se usa fuera de `AuthProvider`
- **Retorna**: `{ user, login, logout, isAdmin }`

---

## 6. Componentes — `components/`

### `Navbar.tsx`

Barra de navegación superior. Se oculta completamente si no hay usuario logueado (`if (!user) return null`).

#### `handleLogout()`
- Llama a `logout()` del contexto
- Redirige a `/login` con `useNavigate`

**Menú según rol:**

| Rol `ADMIN` | Rol `CLIENTE` |
|---|---|
| Mi Perfil | Mi Perfil |
| Mi Cuenta | Mis Facturas |
| Facturas | Mis Alertas |
| Alertas | Mi Cuenta |
| Dashboard | — |
| Solicitudes | — |
| Registrar Usuario | — |

---

### `PrivateRoute.tsx`

Componente de orden superior (HOC) para proteger rutas.

**Props:**
- `children`: El componente a renderizar si se cumplen las condiciones
- `requiredRole?`: Rol requerido para acceder (opcional)

**Lógica:**
1. Si no hay usuario → redirige a `/login`
2. Si se especificó `requiredRole` y el rol del usuario no coincide → redirige a `/perfil`
3. Si todo está bien → renderiza `children`

---

### `ConfettiEffect.tsx`

Efecto visual de confeti que se muestra al pagar una factura exitosamente.

#### `useEffect` (al montar)
Genera un array de 50 partículas con propiedades aleatorias:
- `x`: posición horizontal aleatoria (0–100%)
- `color`: uno de 6 colores predefinidos
- `delay`: retardo de animación (0–1s)
- `size`: tamaño (4–12px)

**Comportamiento**: se renderiza como overlay fijo sobre toda la pantalla (`position: fixed`, `z-index: 9999`). Cada partícula cae con la animación CSS `confetti-fall` en 2.5 segundos. No intercepta clicks (`pointerEvents: none`).

---

## 7. Servicios — `services/`

Todos los servicios usan la instancia centralizada `api` (Axios). Cada función es `async` y retorna los datos tipados directamente.

---

### `api.ts`

Instancia base de Axios configurada para toda la aplicación.

**Configuración base:**
```ts
baseURL: '/api'   // todas las URLs son relativas a /api
```

#### Interceptor de request
Se ejecuta antes de cada petición HTTP:
- Lee `localStorage.getItem('auth')`
- Si existe un token JWT, lo inyecta en el header `Authorization: Bearer <token>`

#### Interceptor de response (manejo de errores)
Se ejecuta cuando el backend responde:
- **Respuesta exitosa**: la deja pasar sin modificar
- **Error 401 o 403**: elimina `auth` de `localStorage` y redirige al navegador a `/login` (excepto si ya está en `/login`)
- **Otros errores**: los rechaza (`Promise.reject`) para que cada servicio los maneje

---

### `authService.ts`

#### `login(credentials: LoginRequest): Promise<LoginResponse>`
- Hace `POST /auth/login` con `{ email, password }`
- **Retorna**: token JWT + datos básicos del usuario

---

### `facturaService.ts`

#### `getFacturasByCliente(dni, page?, size?, estado?): Promise<PaginatedResponse<Factura>>`
- Hace `GET /facturas/cliente/{dni}` con parámetros de paginación
- `estado` es opcional: filtra por `PENDIENTE`, `PAGADA` o `VENCIDA`
- **Retorna**: objeto paginado con `content[]`, `totalPages`, `totalElements`, `number`

#### `pagarFactura(nroFactura: number)`
- Hace `POST /pagos/{nroFactura}`
- Registra el pago de una factura específica
- **Retorna**: datos del pago creado

#### `crearFactura(dto)`
- Hace `POST /facturas`
- `dto` contiene: `dniCliente`, `numeroMedidor`, `monto`, `fechaVencimiento`
- **Retorna**: la `Factura` creada

---

### `alertaService.ts`

#### `getAlertasActivas(dni: string): Promise<Alerta[]>`
- Hace `GET /alertas/cliente/{dni}`
- **Retorna**: lista de alertas activas (no reconocidas) del cliente

#### `reconocerAlerta(id: number): Promise<Alerta>`
- Hace `PUT /alertas/{id}/reconocer`
- Marca la alerta como "descartada/reconocida"
- **Retorna**: la `Alerta` actualizada

---

### `estadisticaService.ts`

#### `getEstadisticas(): Promise<Estadisticas>`
- Hace `GET /estadisticas`
- Solo accesible para admins
- **Retorna**: métricas globales del sistema

---

### `servicioService.ts`

#### `getServicios(): Promise<Servicio[]>`
- Hace `GET /servicios`
- **Retorna**: todos los servicios del sistema

#### `getServiciosByCliente(dni: string): Promise<Servicio[]>`
- Hace `GET /servicios/cliente/{dni}`
- **Retorna**: medidores/servicios asignados a un cliente específico

#### `createServicio(servicio: Servicio): Promise<Servicio>`
- Hace `POST /servicios`
- Crea y asigna un nuevo medidor a un cliente
- **Retorna**: el `Servicio` creado

#### `deleteServicio(numeroMedidor: string): Promise<void>`
- Hace `DELETE /servicios/{numeroMedidor}`
- Elimina un servicio/medidor del sistema

---

### `solicitudService.ts`

#### `crearSolicitud(campo, valorNuevo): Promise<SolicitudCambio>`
- Hace `POST /solicitudes`
- Crea una solicitud de cambio de datos de cuenta (`email` o `password`)
- **Retorna**: la `SolicitudCambio` creada

#### `getMisSolicitudes(): Promise<SolicitudCambio[]>`
- Hace `GET /solicitudes/mis-solicitudes`
- **Retorna**: historial de solicitudes del cliente autenticado

#### `getSolicitudesPendientes(): Promise<SolicitudCambio[]>`
- Hace `GET /solicitudes/pendientes`
- Solo accesible para admins
- **Retorna**: todas las solicitudes en estado `PENDIENTE`

#### `aprobarSolicitud(id: number): Promise<SolicitudCambio>`
- Hace `PATCH /solicitudes/{id}/aprobar`
- El admin aprueba la solicitud; el backend aplica el cambio en el usuario
- **Retorna**: la `SolicitudCambio` actualizada

#### `rechazarSolicitud(id: number): Promise<SolicitudCambio>`
- Hace `PATCH /solicitudes/{id}/rechazar`
- El admin rechaza la solicitud sin aplicar cambios
- **Retorna**: la `SolicitudCambio` actualizada

---

### `usuarioService.ts`

#### `actualizarPerfil(campo, valorNuevo): Promise<void>`
- Hace `PATCH /usuarios/perfil`
- Permite a un admin actualizar directamente su propio `email` o `password`

#### `getUsuarios(): Promise<Usuario[]>`
- Hace `GET /usuarios`
- **Retorna**: lista de todos los usuarios del sistema

#### `createUsuario(usuario): Promise<Usuario>`
- Hace `POST /usuarios`
- Crea un nuevo usuario con `dni`, `email`, `passwordHash` y `rol`
- **Retorna**: el `Usuario` creado

#### `updateUsuario(dni, datos): Promise<Usuario>`
- Hace `PUT /usuarios/{dni}`
- Actualiza `email`, `passwordHash` y/o `rol` de un usuario
- **Retorna**: el `Usuario` actualizado

#### `deleteUsuario(dni: string): Promise<void>`
- Hace `DELETE /usuarios/{dni}`
- Elimina un usuario del sistema

---

## 8. Páginas — `pages/`

---

### `LoginPage.tsx`

Página pública de inicio de sesión.

**Estado local:**
- `form`: `{ email, password }` — campos del formulario
- `error`: mensaje de error a mostrar
- `loading`: deshabilita el botón mientras espera respuesta

#### `handleChange(e)`
Actualiza el campo correspondiente en `form` al escribir en los inputs.

#### `handleSubmit(e)`
- Previene el submit por defecto del formulario
- Activa `loading`
- Llama a `loginService(form)` del servicio
- Si tiene éxito: llama a `login(data)` del contexto y navega a `/perfil`
- Si falla: extrae el mensaje de error de la respuesta y lo muestra

---

### `RegisterPage.tsx`

Página exclusiva para admins para crear nuevos usuarios.

**Estado local:**
- `form`: `{ dni, email, passwordHash, rol }` — campos del formulario
- `error` / `success`: mensajes de retroalimentación
- `loading`: estado de carga

#### `handleChange(e)`
Actualiza cualquier campo del formulario (inputs y select).

#### `handleSubmit(e)`
- Llama a `createUsuario(form)` del servicio
- Si tiene éxito: muestra mensaje de éxito y limpia el formulario
- Si falla: muestra el mensaje de error del backend

---

### `ProfilePage.tsx`

Muestra los datos del perfil del usuario autenticado.

**Estado local:**
- `editing`: booleano que alterna entre vista y edición
- `form`: `{ email, passwordHash }` — campos editables
- `error` / `success`: mensajes de retroalimentación
- `loading`: estado de carga

#### `handleEdit()`
Activa el modo edición, inicializa `form` con el email actual y limpia mensajes.

#### `handleCancel()`
Desactiva el modo edición y limpia mensajes.

#### `handleChange(e)`
Actualiza el campo del formulario al escribir.

#### `handleSubmit(e)`
- Construye el payload: siempre incluye `email` y `rol`; incluye `passwordHash` solo si no está vacío
- Llama a `updateUsuario(user.dni, payload)`
- Si tiene éxito: actualiza la sesión en el contexto con el nuevo email y muestra mensaje de éxito
- Si falla: muestra el error del backend

**Diferencia por rol:**
- `ADMIN`: puede editar directamente desde esta página
- `CLIENTE`: solo puede ver sus datos; se lo redirige a `/mi-cuenta` para solicitar cambios

---

### `FacturasPage.tsx`

Página central de gestión de facturas. Tiene comportamiento distinto según el rol.

**Estado local principal:**
- `facturas`: lista de facturas cargadas
- `page` / `totalPages`: control de paginación
- `loading`: estado de carga
- `showConfetti`: activa el efecto de celebración al pagar
- `filtroEstado`: filtro por estado de factura

**Estado adicional para admins:**
- `showForm`: muestra/oculta el formulario de creación
- `usuarios` / `servicios`: listas para los selects del formulario
- `selectedDni` / `verDni`: DNI del cliente seleccionado para crear o ver facturas
- `buscarCliente` / `buscarVer`: filtros de búsqueda en los selects
- `formData`: campos del formulario de nueva factura
- `showCrearMedidor` / `nuevoMedidor`: estado del sub-formulario inline para crear medidores

#### `clientesFiltrados` (memo)
Filtra la lista de usuarios excluyendo admins y aplicando el texto de búsqueda.

#### `clientesVerFiltrados` (memo)
Filtra la lista de usuarios para el selector "ver facturas de".

#### `cargar()`
Carga las facturas del DNI activo (propio si es cliente, seleccionado si es admin).  
Llama a `getFacturasByCliente` con paginación y filtro de estado.

#### `useEffect` — recarga automática
Llama a `cargar()` cuando cambian `page`, `filtroEstado` o `dniParaCargar`.

#### `useEffect` — carga usuarios (solo admin)
Al montar la página como admin, llama a `getUsuarios()`.

#### `useEffect` — carga servicios del cliente seleccionado (solo admin)
Cuando `selectedDni` cambia, llama a `getServiciosByCliente(selectedDni)` para llenar el select de medidores.

#### `handlePagar(nro: number)`
- Llama a `pagarFactura(nro)`
- Si tiene éxito: activa `showConfetti` durante 3 segundos y recarga la lista
- Si falla: muestra `alert`

#### `handleCrearFactura(e)`
- Llama a `crearFactura` con los datos del formulario
- Si tiene éxito: muestra mensaje verde, limpia el formulario y recarga si corresponde
- Si falla: muestra mensaje rojo

**Sub-formulario inline para crear medidor:**
Al detectar que un cliente no tiene medidores, ofrece un botón "Asignar uno ahora" que muestra un formulario embebido. Al guardarlo, llama a `createServicio` y luego recarga la lista de servicios.

---

### `AlertasPage.tsx`

Muestra las alertas de consumo anómalo del usuario autenticado.

**Estado local:**
- `alertas`: lista de alertas activas
- `loading`: estado de carga

#### `cargar()`
Llama a `getAlertasActivas(user.dni)` y actualiza el estado.

#### `handleReconocer(id: number)`
- Llama a `reconocerAlerta(id)`
- Recarga la lista de alertas

**Vista:**
- Si no hay alertas: muestra un mensaje positivo (✅)
- Si hay alertas: muestra tarjetas con tipo, mensaje, desvío, consumo actual vs promedio y botón "Descartar"

---

### `DashboardPage.tsx`

Panel de estadísticas globales. Solo accesible para admins.

**Estado local:**
- `stats`: objeto `Estadisticas | null`
- `loading`: estado de carga

#### `useEffect` (al montar)
Llama a `getEstadisticas()` y actualiza `stats`.

**Vista:**
- Grid de 4 tarjetas: total usuarios, total servicios, total facturas, monto cobrado
- Lista de facturas agrupadas por estado con sus conteos

---

### `MiCuentaPage.tsx`

Permite cambiar email o contraseña. El comportamiento difiere según el rol.

**Estado local:**
- `solicitudes`: historial de solicitudes del cliente
- `campo`: `'email' | 'password'` — campo a modificar
- `valorNuevo` / `confirmacion`: valores del formulario
- `error`: validación local
- `toast`: notificación temporal (5 segundos)

#### `mostrarToast(mensaje, tipo)`
Activa el toast con mensaje y estilo (`'ok'` = verde, `'err'` = rojo). Se oculta automáticamente tras 5 segundos.

#### `cargar()`
Llama a `getMisSolicitudes()` y actualiza el historial (solo relevante para clientes).

#### `handleSubmit(e)`
Validaciones locales:
- Si `campo === 'password'`, verifica que `valorNuevo === confirmacion`
- Verifica que el valor no esté vacío

Comportamiento según rol:
- **Admin**: llama a `actualizarPerfil(campo, valorNuevo)` para aplicar el cambio directamente
- **Cliente**: llama a `crearSolicitud(campo, valorNuevo)` para enviarla a revisión del admin

#### `estadoBadge(estado)`
Función auxiliar que retorna un `<span>` con la clase CSS y texto del badge según el estado de la solicitud.

**Vista para clientes:** además del formulario, muestra una tabla con el historial de solicitudes anteriores indicando en la columna "Notificación" si fue aprobada, rechazada o está pendiente.

---

### `SolicitudesAdminPage.tsx`

Panel de administración de solicitudes de cambio de cuenta. Solo accesible para admins.

**Estado local:**
- `solicitudes`: lista de solicitudes pendientes
- `loading`: estado de carga
- `toast`: notificación temporal

#### `mostrarToast(msg, tipo)`
Igual que en `MiCuentaPage`: muestra una notificación que se oculta tras 5 segundos.

#### `cargar()`
Llama a `getSolicitudesPendientes()` y actualiza la lista.

#### `handleAprobar(s: SolicitudCambio)`
- Llama a `aprobarSolicitud(s.id)`
- Muestra toast verde indicando que el cambio fue aplicado y el cliente fue notificado
- Recarga la lista
- En caso de error, muestra el mensaje del backend

#### `handleRechazar(s: SolicitudCambio)`
- Llama a `rechazarSolicitud(s.id)`
- Muestra toast rojo indicando que la solicitud fue denegada
- Recarga la lista
- En caso de error, muestra el mensaje del backend

**Vista:** tabla con columnas Cliente, DNI, Campo, Nuevo valor (oculto si es contraseña), Fecha y acciones Aprobar/Rechazar.

---

### `NotFoundPage.tsx`

Página 404. Se renderiza para cualquier ruta no definida (`path="*"`). Muestra un mensaje de "página no encontrada".

---

## Flujo general de la aplicación

```
Usuario accede a cualquier ruta
        ↓
PrivateRoute verifica si hay sesión válida en AuthContext
        ↓
    ¿Logueado?
   /         \
  No          Sí → ¿tiene el rol requerido?
  ↓                    /              \
/login               No            Sí → renderiza la página
                      ↓
                  /perfil (redirect)
```

**Flujo de login:**
```
LoginPage → authService.login() → POST /api/auth/login
         → AuthContext.login() → guarda en localStorage
         → navega a /perfil
```

**Flujo de pago de factura:**
```
FacturasPage → pagarFactura(nro) → POST /api/pagos/{nro}
             → ConfettiEffect durante 3s
             → recarga lista de facturas
```

**Flujo de solicitud de cambio (cliente):**
```
MiCuentaPage → crearSolicitud() → POST /api/solicitudes
             → aparece en SolicitudesAdminPage del admin
             → admin aprueba → PATCH /api/solicitudes/{id}/aprobar
             → backend aplica el cambio en el usuario
             → estado cambia a APROBADA en MiCuentaPage
```
