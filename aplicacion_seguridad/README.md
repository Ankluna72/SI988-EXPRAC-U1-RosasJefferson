# EXAMEN PRÁCTICO DE LA UNIDAD I
## SI-988 · SOLUCIONES MÓVILES II

* **Docente:** Dr. Oscar Juan Jimenez Flores
* **Estudiante:** Jefferson Rosas Chambilla
* **Código:** 2021072618
* **Usuario GitHub:** [Ankluna72](https://github.com/Ankluna72)
* **Repositorio oficial:** [SI988-EXPRAC-U1-RosasJefferson](https://github.com/Ankluna72/SI988-EXPRAC-U1-RosasJefferson.git)
* **Tipo de Examen:** **TIPO 1 — LOGIN, BLOQUEO POR INTENTOS Y CIERRE ANTE HTTP 401**

---

## Resumen Ejecutivo de Implementación (20 / 20 pts)

Se implementó de manera robusta y desacoplada la arquitectura completa de autenticación y control de sesión para la aplicación móvil del guardia de seguridad (`aplicacion_seguridad`), cumpliendo estrictamente con los 5 ítems evaluados en la rúbrica del **Tipo 1**:

```
                                      ┌────────────────────────────┐
                                      │   LoginScreen (View UI)    │
                                      │   - Validación inline      │
                                      │   - Feedback de carga      │
                                      └─────────────┬──────────────┘
                                                    │
                                                    ▼
                                      ┌────────────────────────────┐
                                      │  AuthController (VM)       │
                                      │  - Bloqueo 3 intentos (4to)│
                                      │  - Estado reactivo         │
                                      └─────────────┬──────────────┘
                                                    │
                                                    ▼
                                      ┌────────────────────────────┐
                                      │  AuthRepository            │
                                      │  - Serialización estricta  │
                                      │  - Política de roles       │
                                      └─────────────┬──────────────┘
                                                    │
                                                    ▼
                                      ┌────────────────────────────┐
                                      │  AuthInterceptor (Base)    │
                                      │  - Captura HTTP 401        │
                                      │  - Purga local automática  │
                                      │  - 0 reintentos automáticos│
                                      └─────────────┬──────────────┘
                                                    │
                                                    ▼
                                      ┌────────────────────────────┐
                                      │  SecureStorageService      │
                                      │  - Bóveda cifrada tokens   │
                                      └────────────────────────────┘
```

---

## Detalle de Ítems Desarrollados y Archivos Afectados

### 1. Validación de Formulario y Feedback en UI (4 puntos)
* **Archivos afectados:**
  - `lib/features/auth/presentation/screens/login_screen.dart`
* **Implementación técnica:**
  - Validación antes de procesar el envío: el identificador no puede estar vacío y debe tener al menos 3 caracteres; la contraseña debe tener como mínimo 6 caracteres (`validator`).
  - Si los datos son inválidos, se muestran mensajes descriptivos de error *inline* en color rojo y se aborta el envío antes de llamar a la red.
  - Al ingresar credenciales válidas, el botón se deshabilita inmediatamente y muestra un indicador de carga (`CircularProgressIndicator`) mientras se procesa la solicitud HTTP.

### 2. Bloqueo de Intentos de Autenticación (4 puntos)
* **Archivos afectados:**
  - `lib/features/auth/presentation/controllers/auth_controller.dart`
* **Implementación técnica:**
  - Implementado estrictamente en la capa de lógica de negocio / ViewModel (`AuthController`), no como texto cosmético.
  - Se administra el acumulador `_intentosFallidos` (máximo 3 intentos).
  - **Condición de interrupción:** Al registrarse el cuarto intento consecutivo (`estaBloqueado == true`), la función `iniciarSesion()` aborta de inmediato **sin emitir ninguna llamada de red al servicio (zero calls)** y presenta un mensaje explícito de bloqueo temporal al usuario.
  - Ante un inicio de sesión exitoso, el contador se restablece automáticamente a 0.

### 3. Cierre de Sesión ante Error HTTP 401 Unauthorized (4 puntos)
* **Archivos afectados:**
  - `lib/core/network/auth_interceptor.dart`
  - `lib/features/auth/data/repositories/auth_repository.dart`
  - `lib/main.dart`
* **Implementación técnica:**
  - Se diseñó la clase desacoplada `AuthInterceptor` extendiendo `http.BaseClient` para actuar como middleware/filtro en todas las peticiones con sesión activa.
  - Cuando una petición con cabecera `Authorization` recibe un código HTTP 401:
    1. **Purga inmediata:** Ejecuta `_secureStorage.limpiarSesion()`, dejando los tokens nulos en el dispositivo.
    2. **Fuerza sesión cerrada:** Dispara reactivamente el callback `onUnauthorized` que cambia el estado de la app a `unauthenticated`.
    3. **Bloqueo de reintentos:** Retorna la respuesta de inmediato sin realizar reintentos automáticos.

### 4. Tres Pruebas Unitarias del Login en Terminal (4 puntos)
* **Archivos afectados:**
  - `test/examen_tipo1_test.dart`
* **Pruebas ejecutadas:**
  - **4.a (Estado inicial):** Comprueba que al instanciar el servicio y controlador no existe ninguna sesión previa ni tokens almacenados.
  - **4.b (Autenticación exitosa):** Ante credenciales correctas, el estado pasa a autenticado, se almacenan los tokens en la bóveda segura y se procesa un único envío.
  - **4.c (Autenticación rechazada):** Ante respuesta 401/error del backend, el estado refleja error y se garantiza que no se reintente la llamada (exactamente 1 llamada procesada).

### 5. Dos Pruebas Unitarias del Bloqueo y de la Respuesta 401 (4 puntos)
* **Archivos afectados:**
  - `test/examen_tipo1_test.dart`
* **Pruebas ejecutadas:**
  - **5.a (Verificación de bloqueo):** Demuestra mediante un mock/spy que tras acumular 3 fallos consecutivos, el **cuarto intento registra exactamente 0 llamadas al servicio (zero calls)**.
  - **5.b (Verificación de HTTP 401):** Inyecta un código HTTP 401 simulado y demuestra que el token queda totalmente nulo/limpio en el almacenamiento seguro, notificando el cierre de sesión y registrando un único envío sin reintentos.

---

## Instrucciones de Ejecución

### 1. Ejecutar las Pruebas Unitarias en Consola
Para ejecutar las 5 pruebas oficiales del examen práctico:

```powershell
flutter test test/examen_tipo1_test.dart
```

Para ejecutar toda la suite de pruebas del proyecto (16 pruebas):
```powershell
flutter test
```

### 2. Análisis Estático de Código
Garantizar 0 errores y 0 advertencias con el linter de Flutter:

```powershell
flutter analyze
```

### 3. Formateo de Código
```powershell
dart format lib test
```

### 4. Desplegar y Emular la Aplicación
```powershell
flutter run -d emulator-5554
```
*(O seleccionar dispositivo `windows` o `chrome` según se requiera).*
