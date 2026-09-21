# Sistema de planificación de cargas para empresas de transporte terrestre

Aplicación multiplataforma para la gestión operativa de empresas de transporte terrestre de mercancías. Permite planificar pedidos y cargas, gestionar conductores y vehículos, coordinar colaboradores externos, comunicar incidencias y generar cartas de porte digitales.

El proyecto forma parte del Trabajo de Fin de Grado **«Sistema de planificación de cargas para empresas de transporte terrestre de mercancías»**, desarrollado por **Haritz Gómez Sarasola** en el Grado en Ingeniería Informática — Ingeniería de Software.

> Este repositorio contiene el código fuente del frontend Flutter, el backend FastAPI, las pruebas y la configuración necesaria para el desarrollo y el despliegue.

## Índice

- [Características](#características)
- [Arquitectura](#arquitectura)
- [Roles](#roles)
- [Tecnologías](#tecnologías)
- [Estructura del repositorio](#estructura-del-repositorio)
- [Requisitos previos](#requisitos-previos)
- [Configuración local](#configuración-local)
- [Ejecución](#ejecución)
- [Pruebas](#pruebas)
- [API](#api)
- [Despliegue](#despliegue)
- [Seguridad](#seguridad)
- [Limitaciones y trabajo futuro](#limitaciones-y-trabajo-futuro)
- [Autoría y uso de IA](#autoría-y-uso-de-ia)

## Características

### Gestión de tráfico

- Gestión de conductores y vehículos de la empresa.
- Creación de pedidos y de una o varias cargas asociadas.
- Definición de tipos de carga reutilizables.
- Planificación semanal mediante calendario.
- Asignación de cargas a conductores y vehículos.
- Validación de disponibilidad horaria y capacidad del vehículo.
- Cesión de cargas a empresas subcontratadas.
- Panel de control con el estado de las cargas y las incidencias.

### Operaciones del conductor

- Hoja de ruta con las cargas asignadas.
- Consulta de los detalles de cada carga.
- Confirmación de recogida y entrega.
- Apertura de la ruta en Google Maps.
- Registro y consulta de incidencias.

### Colaboración y comunicación

- Soporte multi-tenant: cada empresa solo accede a sus datos.
- Invitación de cargadores, subcontratados y conductores.
- Notificaciones push mediante Firebase Cloud Messaging.
- Actualización en tiempo real de las cargas del conductor y de las incidencias relevantes.

### Cartas de porte digitales

- Generación de documentos PDF con los datos de la carga y de los participantes.
- Inclusión de un código QR con la URL del documento almacenado.
- Almacenamiento en Firebase Storage.
- Conservación del *snapshot* de los datos necesarios para mantener la información histórica de la carta.
- Notificación a los usuarios correspondientes cuando el documento está disponible.

## Arquitectura

La solución utiliza una arquitectura cliente-servidor con tres subsistemas principales:

```text
Flutter (Web / Android / iOS)
        │ HTTPS + Bearer JWT
        ▼
FastAPI (routers → servicios → repositorios)
        │ Firebase Admin SDK
        ▼
Firebase (Authentication, Firestore, Storage y FCM)
```

### Frontend

El frontend sigue una organización **feature-first** y utiliza Flutter con Dart. La interfaz se estructura mediante una separación tipo MVVM:

- **View**: widgets y páginas de presentación.
- **ViewModel / estado**: `Provider` y `ChangeNotifier`.
- **Model / servicios**: modelos tipados y clientes para comunicarse con Firebase o la API.

Las interfaces se adaptan a móvil y escritorio mediante breakpoints, `LayoutBuilder`, `Expanded`, `Flexible` y `Wrap`. En móvil, las tablas se transforman en listas y la navegación utiliza un `Drawer`.

### Backend

El backend está implementado con FastAPI y se divide en capas:

1. **Routers**: definen los endpoints HTTP.
2. **Dependencias de seguridad**: validan el token Firebase, el tenant y el rol.
3. **Schemas Pydantic**: validan y serializan los datos.
4. **Servicios de aplicación**: contienen las reglas de negocio.
5. **Repositorios / CRUD**: encapsulan el acceso a Firestore.

El patrón Repository desacopla la lógica de negocio de la base de datos y facilita futuras migraciones.

### Persistencia y consistencia

Firestore utiliza subcolecciones y cierta desnormalización para reducir lecturas N+1. Las actualizaciones de varias cargas se realizan mediante **batches** para evitar estados intermedios inconsistentes. Las cartas de porte conservan un snapshot de los datos relevantes en el momento de crear la carga.

## Roles

- **Encargado de tráfico**: administra usuarios, flota, pedidos, planificación, cesiones, incidencias y cartas de porte.
- **Chófer**: consulta su hoja de ruta, actualiza el estado de sus cargas, consulta rutas y reporta incidencias.
- **Cargador**: crea y consulta pedidos y accede a las cartas de porte de sus cargas.
- **Subcontratado**: consulta las cargas cedidas y registra los recursos necesarios para realizarlas.

La autorización se basa en los *custom claims* de Firebase Authentication, que contienen el identificador de empresa y el rol del usuario.

## Tecnologías

| Área | Tecnología |
|---|---|
| Cliente | Flutter / Dart |
| API | Python / FastAPI |
| Validación | Pydantic |
| Autenticación | Firebase Authentication |
| Base de datos | Cloud Firestore |
| Archivos | Firebase Storage |
| Notificaciones | Firebase Cloud Messaging |
| PDF | Jinja2 + WeasyPrint |
| Geocodificación | Photon / OpenStreetMap |
| Testing frontend | `flutter_test`, `integration_test`, Mockito |
| Testing backend | pytest |
| CI y calidad | GitHub Actions y SonarCloud |
| Despliegue | Firebase Hosting y Google Cloud Run |

## Estructura del repositorio

```text
.
├── frontend/              # Aplicación Flutter para web, Android e iOS
│   ├── lib/
│   │   ├── core/          # Modelos y widgets compartidos
│   │   ├── features/      # Módulos organizados por funcionalidad
│   │   └── app.dart
│   ├── test/              # Pruebas unitarias del frontend
│   └── integration_test/  # Pruebas de integración Flutter
├── backend/               # API FastAPI
│   ├── app/
│   │   ├── routers/       # Endpoints
│   │   ├── services/      # Lógica de negocio
│   │   ├── schemas/       # Modelos Pydantic
│   │   ├── repositories/  # Acceso a Firestore
│   │   └── tests/         # Pruebas backend
│   ├── requirements.txt
│   └── Dockerfile
├── memoria.md             # Memoria del Trabajo de Fin de Grado
└── README.md
```

Los nombres exactos de algunos módulos pueden variar según la evolución del proyecto; la separación de responsabilidades descrita es la utilizada por la aplicación.

## Requisitos previos

- Flutter SDK estable.
- Dart incluido con Flutter.
- Python 3.12 o superior.
- Node.js y npm, necesarios para Firebase CLI.
- Java 21, necesario para ejecutar el emulador de Firestore.
- Una cuenta y un proyecto de Firebase para desarrollo.
- Firebase CLI y, opcionalmente, FlutterFire CLI.

Instalación de herramientas:

```bash
npm install -g firebase-tools
dart pub global activate flutterfire_cli
firebase login
```

## Configuración local

### Backend

```bash
cd backend
python -m venv .venv

# Windows PowerShell
.\.venv\Scripts\Activate.ps1

# Linux / macOS
source .venv/bin/activate

pip install --upgrade pip
pip install -r requirements.txt
```

Crea un archivo `.env` a partir de `.env.example` y configura, como mínimo, las variables necesarias para Firebase Admin y el correo electrónico:

```env
FIREBASE_CREDENTIALS_PATH="ruta/al/archivo-de-credenciales.json"
FRONTEND_URL="http://localhost:5500"
EMAIL_USER="correo-del-sistema"
EMAIL_PASSWORD="app-password"
```

No subas nunca al repositorio las credenciales de Firebase, contraseñas ni archivos `.env`.

### Frontend

```bash
cd frontend
flutter pub get
```

Configura los proyectos Firebase de los flavors que vayas a utilizar mediante FlutterFire CLI. El proyecto contempla entornos `dev`, `stg` y `prod`, con configuraciones Firebase separadas:

```bash
flutterfire configure \
  --project=<proyecto-firebase-dev> \
  --out=lib/firebase_options_dev.dart
```

Crea `frontend/dart_defines.json` sin incluirlo en el control de versiones si contiene valores privados:

```json
{
  "API_BASE_URL": "http://127.0.0.1:8000",
  "SYNCFUSION_KEY": "TU_LICENCIA_LOCAL",
  "VAPID_PUBLIC_KEY": "TU_CLAVE_VAPID_LOCAL"
}
```

Para conectar un dispositivo físico, utiliza la IP local del ordenador en lugar de `127.0.0.1`. El backend debe escuchar en todas las interfaces (`0.0.0.0`) y ambos dispositivos deben estar en la misma red.

## Ejecución

### API local

Desde `backend/`:

```bash
uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
```

Para acceder desde un dispositivo físico:

```bash
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

### Flutter Web

Desde `frontend/`:

```bash
flutter run -d chrome --flavor dev --web-port=5500 \
  --dart-define-from-file=dart_defines.json
```

### Otros dispositivos

```bash
# Android Emulator: normalmente utiliza 10.0.2.2 para acceder al host
flutter run -d emulator-5554 \
  --dart-define=API_BASE_URL=http://10.0.2.2:8000

# Dispositivo físico
flutter run -d <device-id> \
  --dart-define=API_BASE_URL=http://<IP_LOCAL>:8000
```

## Pruebas

### Frontend

```bash
cd frontend
dart run build_runner build --delete-conflicting-outputs
flutter test --coverage
```

### Backend

```bash
cd backend
pytest app/tests/ -m "not integration" --cov=app
```

### Integración con Firestore Emulator

Con Firebase CLI, Node.js y Java instalados:

```bash
cd backend
firebase emulators:exec --project test-project --only firestore \
  "pytest app/tests/integration_tests.py -m integration --cov=app"
```

Las pruebas de integración verifican, entre otros aspectos, el flujo pedido → cargas → carta de porte, la persistencia del snapshot y el aislamiento entre empresas.

El pipeline de GitHub Actions separa las pruebas de Flutter, las pruebas unitarias de FastAPI y las pruebas de integración. Posteriormente SonarCloud combina los informes de cobertura y analiza bugs, vulnerabilidades, *code smells*, duplicaciones y mantenibilidad.

## API

FastAPI genera automáticamente la documentación OpenAPI. Con el backend en ejecución, se puede consultar en:

- Swagger UI: `http://localhost:8000/docs`
- ReDoc: `http://localhost:8000/redoc`

Los grupos principales de endpoints son:

| Ruta | Responsabilidad |
|---|---|
| `/auth` | Registro y datos de autenticación |
| `/trans` | Gestión de conductores |
| `/vehiculos` | Gestión de vehículos |
| `/external_users` | Invitación y gestión de cargadores y subcontratados |
| `/pedidos` | Creación y consulta de pedidos |
| `/cargas` | Planificación, cesiones, incidencias y cartas de porte |
| `/dashboard` | Consultas agregadas para el panel de control |

Las peticiones protegidas deben incluir un token de Firebase en formato Bearer:

```http
Authorization: Bearer <FIREBASE_ID_TOKEN>
```

## Despliegue

### Backend en Google Cloud Run

El backend se empaqueta mediante Docker y se despliega como un servicio de Cloud Run:

```bash
cd backend
gcloud auth login
gcloud config set project <ID_PROYECTO_GCP>

gcloud run deploy backend-tfg-transporte \
  --source . \
  --region europe-southwest1 \
  --allow-unauthenticated \
  --max-instances=2 \
  --set-env-vars="FRONTEND_URL=https://<proyecto>.web.app"
```

Las credenciales y secretos deben configurarse mediante Secret Manager o variables de entorno seguras, nunca mediante archivos incluidos en la imagen o en el repositorio.

### Frontend en Firebase Hosting

```bash
cd frontend
flutter clean
flutter pub get
flutter build web --wasm --dart-define-from-file=dart_defines.json
firebase deploy --only hosting
```

El backend y el frontend se despliegan por separado: una actualización de la interfaz no obliga a redesplegar la API.

## Seguridad

- Firebase Authentication gestiona la identidad y emite tokens JWT.
- FastAPI verifica la firma, la vigencia, el `company_id` y el rol en cada petición protegida.
- El aislamiento multi-tenant se aplica en la capa de servicios y repositorios.
- Las operaciones se realizan mediante HTTPS en los entornos desplegados.
- Se utilizan políticas CORS para restringir los orígenes autorizados.
- Existe middleware de limitación de peticiones (*rate limiting*).
- Las reglas de Firestore restringen las lecturas directas del cliente a los casos estrictamente necesarios, como el perfil del usuario, las incidencias y las cargas del conductor.
- No deben almacenarse secretos, credenciales de Firebase, contraseñas ni tokens en el repositorio.

## Limitaciones y trabajo futuro

El alcance actual no incluye facturación, pagos, publicación en Play Store o App Store ni optimización automática de rutas. La planificación es manual.

Entre las ampliaciones previstas se encuentran:

- Geolocalización en tiempo real de la flota.
- Evidencias fotográficas al confirmar una entrega.
- Interacción por voz para los conductores.
- Planificación avanzada y optimización de rutas mediante Vehicle Routing Problem y OR-Tools.
- Ampliación de las funcionalidades del módulo móvil del conductor.

## Autoría y uso de IA

Proyecto realizado por **Haritz Gómez Sarasola**.

Durante el desarrollo se utilizaron GitHub Copilot, Claude, Gemini y Figma Make como herramientas de apoyo para programación, diseño, estructura, corrección y metodología. El código y la documentación fueron revisados y adaptados al proyecto por el autor.

## Licencia

Este repositorio corresponde a un Trabajo de Fin de Grado. Si no se especifica una licencia en el repositorio, todos los derechos quedan reservados por el autor.
