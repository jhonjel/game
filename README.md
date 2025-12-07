# Sistema de seguimiento de vehículos

Aplicación Ionic/Angular para gestionar vehículos, recorridos GPS y visualización en mapas usando Leaflet. [file:11][file:13]

## Características principales

- Autenticación de usuarios con roles (incluye rol visitante con funcionalidades limitadas). [file:12][file:13]
- Selección de vehículo y configuración de rutas asociadas. [file:11][file:13]
- Inicio, seguimiento y finalización de recorridos con registro periódico de posiciones GPS. [file:11]
- Visualización de rutas y recorridos en mapas (Leaflet + OpenStreetMap). [file:11][file:13]
- Mapa de vehículos activos mostrando su última posición y trayectoria reciente. [file:13]

## Arquitectura general

- **AppComponent (`app.component.ts`)**: raíz de la aplicación, define la estructura principal y controla la visibilidad del menú según el rol del usuario autenticado. [file:12]
- **Tabs & Rutas (`tabs.page.ts`, `tabs.routes.ts`, `app.routes.ts`)**: definen la navegación principal por pestañas y las rutas a páginas como login, registro, selección de vehículos, mapa de recorridos y mapa de vehículos. [file:11][file:13]
- **Páginas principales**:
  - `tab1.page.ts`: gestión/selección de vehículos. [file:11]
  - `tab2.page.ts`: seguimiento GPS de un vehículo, inicio/fin de recorridos y dibujo de ruta seleccionada. [file:11]
  - `tab3.page.ts`: funcionalidades adicionales relacionadas (por ejemplo, listado de recorridos o configuraciones). [file:11]
  - `mapa-vehiculos.ts`: mapa global de vehículos con recorridos activos. [file:13]
  - `rutas.page.ts`: administración y visualización de rutas disponibles. [file:11]
  - `login.page.ts` y `registro.page.ts`: autenticación y registro de usuarios. [file:11]
- **Modelos y servicios relacionados**:
  - `vehiculo.model.ts`: modelo de datos para vehículos. [file:11]
  - Servicios (no adjuntos pero usados): `VehiculoSeleccionadoService`, `VehiculosService`, `RecorridosService`, `RutasService`, `AuthService`. [file:11][file:13]
  - Uso de `environment.tokenSecret` como identificador de perfil para consumir la API de recorridos y rutas. [file:11][file:13]

## Flujo de autenticación y roles

- El usuario se autentica en `login.page.ts`, y la información se gestiona mediante `AuthService`. [file:11]
- `AppComponent` se suscribe al usuario actual y calcula si es visitante para habilitar o bloquear opciones del menú. [file:12]
- En `mapa-vehiculos.ts` se vuelve a comprobar si el usuario es visitante para condicionar ciertas acciones. [file:13]
- Guards como `auth-guard.ts`, `visitante-guard.ts` e `interceptor-guard.ts` se utilizan para proteger rutas y manejar tokens de autenticación en las peticiones HTTP. [file:11]

## Flujo de recorridos y GPS (Tab2)

- La página `Tab2Page` se encarga de:
  - Obtener el vehículo seleccionado desde `VehiculoSeleccionadoService`. [file:11]
  - Cargar las rutas disponibles desde `RutasService` usando el `perfil_id`. [file:11]
  - Cargar Leaflet dinámicamente (CSS + JS) y crear un mapa inicial centrado en unas coordenadas por defecto. [file:11]
  - Obtener la ubicación actual del navegador con `navigator.geolocation`, posicionar el marcador y guardar la posición inicial. [file:11]
  - Iniciar un recorrido llamando a `RecorridosService.iniciarRecorrido`, guardando el `recorridoActualId`. [file:11]
  - Activar `watchPosition` para registrar posiciones periódicamente, filtrando por precisión y distancia mínima recorrida y enviándolas con `RecorridosService.registrarPosicion`. [file:11]
  - Dibujar la trayectoria en el mapa mediante una polyline y mostrar la ruta seleccionada usando datos GeoJSON. [file:11]
  - Finalizar el recorrido con `RecorridosService.finalizarRecorrido`, deteniendo geolocalización e intervalos. [file:11]

## Mapa de vehículos activos

- La página `MapaVehiculosPage` (`mapa-vehiculos.ts`) se encarga de: [file:13]
  - Cargar Leaflet y mostrar un mapa inicial. [file:13]
  - Consultar la lista de vehículos desde `VehiculosService`. [file:13]
  - Obtener recorridos activos (sin `ts_fin`) por perfil con `RecorridosService.obtenerRecorridosPorPerfil`. [file:13]
  - Cargar posiciones de cada recorrido con `RecorridosService.obtenerPosiciones`. [file:13]
  - Normalizar las estructuras de coordenadas (`geom`, `geometry`, `coordinates`, etc.) mediante `normalizarPosicion`. [file:13]
  - Filtrar posiciones con coordenadas válidas y tomar la última posición como posición actual del vehículo. [file:13]
  - Crear marcadores en el mapa para cada vehículo y, si hay suficientes puntos, dibujar su trayectoria más reciente. [file:13]
  - Implementar actualización periódica para refrescar posiciones. [file:13]

## Control de acceso y experiencia de usuario

- Se usan guards de ruta para:
  - Permitir acceso solo a usuarios autenticados en ciertas secciones. [file:11]
  - Restringir opciones a los visitantes (por ejemplo, impedir iniciar nuevos recorridos o modificar ciertos datos). [file:11][file:13]
- Se emplean `ToastController` y `AlertController` para mostrar mensajes de error, advertencia y éxito (problemas de GPS, errores de API, confirmaciones de cierre de sesión, etc.). [file:11][file:13]

## Requisitos y tecnologías

- Ionic + Angular con componentes standalone (`standalone: true`). [file:11][file:13]
- Leaflet 1.9.x para mapas, cargado dinámicamente desde CDN. [file:11][file:13]
- API REST propia para:
  - Autenticación y gestión de usuarios. [file:11]
  - Vehículos, recorridos y rutas, identificados por `perfil_id` (environment). [file:11][file:13]
- Navegador con soporte de `navigator.geolocation` para el seguimiento GPS. [file:11]

## Servicios

- **AuthService (`auth.service.ts`)**  
  Gestiona autenticación, almacenamiento de tokens y rol del usuario (incluyendo rol visitante).  
  Expone métodos para login, logout, comprobar si el usuario está autenticado y obtener el rol actual. [file:12][file:13][file:41]

- **VehiculosService (`vehiculos.ts`)**  
  Consume la API de vehículos para listar, crear o actualizar vehículos asociados al perfil.  
  Se usa, entre otros, en las páginas de selección de vehículo y en el mapa de vehículos activos. [file:13][file:20]

- **VehiculoSeleccionadoService (`vehiculo-seleccionado.ts`)**  
  Mantiene en memoria y como observable el vehículo actualmente seleccionado por el usuario.  
  Es utilizado por `Tab1Page`, `Tab2Page` y otras páginas que necesitan conocer el vehículo activo. [file:11][file:23]

- **RutasService (`rutas.ts`)**  
  Gestiona las rutas configuradas para un perfil: obtiene rutas desde la API y las expone a las páginas que las necesitan.  
  Es clave para `Tab2Page` (mostrar/dibujar una ruta específica) y `rutas.page.ts` (gestión de rutas). [file:11][file:22]

- **RecorridosService (`recorridos.ts`)**  
  Centraliza la lógica de recorridos: iniciar recorrido, registrar posiciones GPS, obtener posiciones, finalizar recorrido y listar recorridos por perfil.  
  Es utilizado tanto por `Tab2Page` (seguimiento en tiempo real) como por `MapaVehiculosPage` (carga de recorridos activos y sus posiciones). [file:11][file:13][file:25]

- **CallesService / similar (`calles.ts`)**  
  Gestiona datos de calles o capas adicionales (por ejemplo, shapes o geometrías) que pueden usarse para enriquecer los mapas o rutas. [file:21]

- **LoadDataService (`loaddata.ts`)**  
  Servicio auxiliar para cargar datos iniciales o sincronizar información desde la API (semillas, catálogos, etc.) según el diseño de tu backend. [file:27]
