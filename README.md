<details>
<summary><h2><b> 💡Idea Proyecto </b></h2></summary>

# 🛡️ R3CON

### Análisis continuo de seguridad para pymes

SecureReport es un proyecto de plataforma que transforma el análisis técnico de una infraestructura en **hallazgos comprensibles, informes periódicos y alertas accionables**.

La propuesta consiste en desplegar dos contenedores Podman en un entorno autorizado por el cliente. Estos identifican activos y servicios, realizan comprobaciones de seguridad y envían los resultados a una plataforma central. El cliente puede consultar la evolución de su infraestructura desde un portal privado, descargar informes PDF y recibir avisos relevantes por Telegram.

> **Estado del proyecto:** fase de diseño. La arquitectura, las funcionalidades y los precios descritos son propuestas pendientes de implementación y validación.

## 🎯 Objetivo

Ayudar a pymes y startups sin un equipo especializado de ciberseguridad a responder tres preguntas:

- ¿Qué dispositivos y servicios tenemos en nuestra infraestructura?
- ¿Qué posibles riesgos se han detectado y con qué grado de confianza?
- ¿Qué deberíamos corregir primero?

SecureReport se plantea como un servicio de **análisis continuo de exposición y vulnerabilidades**. No sustituye una auditoría manual completa, un pentest profesional ni una certificación de cumplimiento.

## 🧭 Funcionamiento previsto

1. **Alta del cliente:** la organización accede al portal y define los activos, redes, interfaces y horarios autorizados.
2. **Despliegue de agentes:** instala dos contenedores Podman con permisos ajustados a las funciones que realizará cada uno.
3. **Recopilación y análisis:** los agentes descubren servicios y ejecutan comprobaciones no destructivas dentro del alcance acordado.
4. **Procesamiento central:** la API recibe resultados estructurados; el backend los valida, correlaciona y prioriza.
5. **Entrega de resultados:** el cliente consulta el dashboard, recibe informes PDF y obtiene avisos por Telegram cuando corresponde.
6. **Seguimiento:** el portal permite revisar hallazgos, registrar avances y comunicarse con el equipo.

## 🏗️ Arquitectura propuesta

```text
                 INFRAESTRUCTURA DEL CLIENTE
           ┌──────────────────────────────────────┐
           │ Contenedor 1: red y tráfico           │
           │ Nmap · TShark / dumpcap               │
           ├──────────────────────────────────────┤
           │ Contenedor 2: análisis de seguridad   │
           │ Comprobaciones · evidencia · hallazgos│
           └──────────────────┬───────────────────┘
                              │
                      Comunicación protegida
                              │
                              ▼
                  ┌─────────────────────────┐
                  │ API central · FastAPI   │
                  └────────────┬────────────┘
                               │
                     PostgreSQL + workers
                               │
                ┌──────────────┼──────────────┐
                ▼              ▼              ▼
          Portal privado   Informes PDF    n8n + Telegram
          y mensajería     e histórico     y apoyo de IA
```

| Componente | Responsabilidad | Tecnologías propuestas |
|---|---|---|
| **Contenedor 1 · Sensor de red** | Descubrimiento de activos, puertos y servicios; obtención opcional de indicadores de tráfico. | Podman, Nmap, TShark/dumpcap |
| **Contenedor 2 · Analizador** | Comprobaciones de seguridad no destructivas y preparación de hallazgos con evidencia. | Podman, Python y módulos de análisis |
| **API y base de datos** | Recepción de resultados, autenticación, separación de clientes e histórico. | FastAPI, PostgreSQL |
| **Procesamiento** | Correlación, deduplicación, priorización y generación de informes. | Workers y generador de PDF |
| **Portal y comunicaciones** | Dashboard, informes, mensajería del cliente y notificaciones. | Web, correo, n8n y Telegram |

## 🔎 Agentes de análisis

### Contenedor 1: red y tráfico

El primer contenedor utilizará **Nmap** para descubrir equipos, puertos y servicios dentro de los rangos autorizados. De forma opcional, podrá emplear **TShark o dumpcap** para obtener información de tráfico en interfaces concretas.

La captura de tráfico no se activará por defecto. Su alcance dependerá de la ubicación del agente, los permisos disponibles y la autorización del cliente. Instalar el contenedor en un servidor **no implica poder observar toda la red**.

### Contenedor 2: análisis de seguridad

El segundo contenedor trabajará sobre los activos y servicios detectados para ejecutar comprobaciones adicionales de bajo impacto. Cada resultado deberá incluir:

- Activo y servicio afectados.
- Regla aplicada o referencia técnica.
- Evidencia que respalda el hallazgo.
- Severidad y grado de confianza.
- Recomendación de corrección.
- Estado de revisión o remediación.

Una versión de software detectada puede sugerir un posible CVE, pero **no confirma por sí sola que el sistema sea vulnerable**. Los resultados dudosos se marcarán como pendientes de verificación.

## 🤖 IA y automatización con n8n

La IA se plantea como una herramienta de **apoyo al análisis**, no como autoridad final. Podrá ayudar a agrupar hallazgos repetidos, resumir evidencias y redactar recomendaciones claras para el cliente. Los resultados importantes o inciertos deberán poder revisarse antes de presentarse como confirmados.

**n8n** coordinará procesos como la generación periódica de informes, las solicitudes de revisión y el envío de avisos. La lógica principal de seguridad, la autenticación y el control de permisos permanecerán en el backend.

```text
Nuevo hallazgo
    → Validación y deduplicación
    → Priorización
    → Revisión si existe incertidumbre
    → Publicación en el portal
    → Aviso por Telegram, si procede
    → Inclusión en el informe periódico
```

## 📊 Portal, mensajería e informes

El proyecto contempla dos espacios web diferenciados:

- **Web pública:** presentación de la empresa, explicación del servicio, planes y contacto.
- **Portal privado:** acceso del cliente al dashboard, inventario, hallazgos, informes y mensajería con el equipo.

El **informe PDF periódico** incluirá un resumen ejecutivo, cambios desde el informe anterior, hallazgos priorizados, evidencia técnica, recomendaciones y estado de remediación.

Telegram servirá como **canal de aviso**, no como repositorio de información sensible. Por ejemplo, una notificación podrá indicar que se ha detectado un hallazgo de prioridad alta y dirigir al cliente al portal autenticado para consultar los detalles.

## 🔐 Seguridad y límites

El sistema debe construirse aplicando el principio de **mínimo privilegio** tanto a los agentes como a la plataforma central.

- Escanear únicamente redes y activos con autorización expresa.
- Definir horarios, límites de velocidad y exclusiones antes de ejecutar pruebas.
- No montar `/var/run/docker.sock` ni usar `--network host` por defecto.
- Limitar la captura a interfaces y permisos específicamente autorizados.
- Enviar preferentemente resultados estructurados y metadatos mínimos, no capturas completas de paquetes.
- Proteger la comunicación, las credenciales de los agentes y la separación de datos entre clientes.
- Mantener trazabilidad de escaneos, cambios y accesos.
- Definir políticas de conservación y tratamiento de los datos obtenidos.

> **Importante:** SecureReport no debe presentarse como una garantía de ausencia de vulnerabilidades. Los escaneos automatizados tienen límites y pueden generar falsos positivos o no detectar determinados problemas.

## 🚀 Roadmap

| Fase | Objetivo | Entrega |
|---|---|---|
| **1 · Base técnica** | Demostrar el flujo completo en un laboratorio propio. | Nmap → API → PostgreSQL |
| **2 · Primer producto** | Hacer consultables los resultados. | Portal privado, inventario e informe PDF |
| **3 · Análisis** | Ampliar las comprobaciones sin perder calidad. | Segundo contenedor, evidencia, confianza y deduplicación |
| **4 · Comunicación** | Automatizar el seguimiento del cliente. | Telegram, mensajería web y flujos n8n |
| **5 · Validación** | Evaluar utilidad y seguridad reales. | IA supervisada y pilotos con organizaciones autorizadas |

## 💼 Modelo de negocio propuesto

| Plan | Precio orientativo | Enfoque |
|---|---:|---|
| **Starter** | 49 €/mes | Organizaciones pequeñas que necesitan inventario e informes periódicos. |
| **Business** | 99 €/mes | Mayor cobertura, seguimiento más frecuente y alertas. |
| **Enterprise / MSP** | Desde 299 €/mes | Varias sedes u organizaciones e integraciones acordadas. |

Estos precios son **hipótesis**, no tarifas definitivas. Antes de comercializar el servicio será necesario validar la demanda, el coste de operación y soporte, los límites de cada plan y la calidad de los hallazgos.

## 📌 Estado actual

SecureReport se encuentra en **fase de definición**. El siguiente hito técnico es construir un laboratorio controlado y demostrar el recorrido completo:

```text
Escaneo autorizado → API → almacenamiento → hallazgo → informe
```

La prioridad inicial es obtener resultados fiables y explicables antes de añadir más herramientas o prometer una auditoría completa.
</details>


<details>
<summary><h2><b> 📅 Planificacion del proyecto </b></h2></summary>

# 🛡️ R3con — Planificación técnica del proyecto

> **R3con** es un prototipo funcional de reconocimiento y análisis continuo de seguridad para infraestructuras autorizadas. El proyecto se desarrolla como Trabajo Final de Ciclo de **ASIX, perfil de ciberseguridad**, por Raúl, Ricard y Jam.

**Estado:** fase de planificación y diseño de arquitectura.  
**Idioma:** español.  
**Entorno de demostración:** laboratorio aislado en VirtualBox/Proxmox.  
**Fecha objetivo de finalización:** antes de abril.

---

## 🎯 1. Objetivo del proyecto

R3con tiene como objetivo demostrar, en un laboratorio controlado, cómo una plataforma web puede:

- Descubrir equipos, puertos y servicios de una red autorizada mediante **Nmap**.
- Analizar indicadores y metadatos de red utilizando **TShark/Wireshark**, sin almacenar capturas PCAP completas.
- Detectar configuraciones o servicios que requieran revisión mediante reglas no destructivas.
- Centralizar resultados en una aplicación web con autenticación, roles y dashboard.
- Generar informes PDF automáticos diarios, semanales o bajo demanda.
- Enviar alertas críticas mediante un bot de Telegram.
- Automatizar tareas con **n8n**.
- Investigar e integrar **MicroCloud** como parte de la innovación y el aprendizaje de infraestructura privada.

> R3con no será una herramienta de explotación, ni realizará ataques contra redes externas. Todas las pruebas se limitarán a máquinas virtuales propias y a rangos de red explícitamente autorizados.

---

## ✅ 2. Alcance del MVP

### Funcionalidades obligatorias

| Funcionalidad | Descripción |
|---|---|
| Descubrimiento de red | Identificación de hosts, puertos y servicios con Nmap dentro de un CIDR autorizado |
| Análisis de tráfico | Obtención de metadatos y protocolos con TShark/dumpcap en el laboratorio |
| Reglas de seguridad | Detección de configuraciones o servicios que requieran revisión |
| Agentes Podman | Dos contenedores separados: sensor y analizador |
| Aplicación web | Login real, roles, dashboard, activos, hallazgos e informes |
| Base de datos | Persistencia de usuarios, escaneos, activos, hallazgos e informes en MariaDB/MySQL |
| PDF | Generación real de informes periódicos y bajo demanda |
| Telegram | Bot con alertas críticas y resumen de estado |
| MicroCloud | Prueba docente de despliegue y documentación de su integración |

### Mejoras futuras

- Asistente de IA dentro del portal.
- Explicaciones automáticas de hallazgos para usuarios no técnicos.
- Clasificación avanzada de riesgos mediante IA.
- Chat de soporte integrado.
- Integración con correo electrónico.
- MFA para usuarios administrativos.
- Más reglas y módulos de análisis.

---

## 🏗️ 3. Arquitectura general

```text
                           LABORATORIO R3CON
┌──────────────────────────────────────────────────────────────────────┐
│                      Red interna VirtualBox                          │
│                                                                      │
│  ┌───────────────────┐       ┌───────────────────────────────────┐  │
│  │ VM objetivo 1     │       │ VM objetivo 2                     │  │
│  │ Debian / Ubuntu   │       │ Kali / Ubuntu de práctica         │  │
│  │ SSH · HTTP · DNS  │       │ Servicios de laboratorio          │  │
│  └─────────▲─────────┘       └──────────────▲────────────────────┘  │
│            │                                │                       │
│            └─────────── Escaneo autorizado ─┘                       │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ VM R3con Agent                                                 │  │
│  │ Ubuntu Server + Podman                                         │  │
│  │                                                               │  │
│  │ ┌──────────────────────┐  ┌────────────────────────────────┐ │  │
│  │ │ r3con-sensor         │  │ r3con-analyzer                 │ │  │
│  │ │ Nmap                 │  │ TShark + reglas seguras        │ │  │
│  │ │ Hosts/puertos/serv.  │  │ Hallazgos JSON                 │ │  │
│  │ └──────────┬───────────┘  └──────────────┬─────────────────┘ │  │
│  └────────────┼─────────────────────────────┼───────────────────┘  │
└───────────────┼─────────────────────────────┼──────────────────────┘
                └───────────────┬─────────────┘
                                HTTPS
                                  │
                                  ▼
┌──────────────────────────────────────────────────────────────────────┐
│                          R3CON CORE                                  │
│                    Ubuntu Server / MicroCloud                        │
│                                                                      │
│ FastAPI ── MariaDB ── Worker ── PDF ── n8n ── Telegram              │
│      │                                                               │
│      ├── Dashboard web HTML/CSS/JavaScript                           │
│      ├── Login y control de roles                                    │
│      ├── Histórico de escaneos e informes                            │
│      └── Alertas críticas y resumen periódico                        │
└──────────────────────────────────────────────────────────────────────┘
```

### Flujo principal

```text
Política de alcance autorizada
        ↓
R3con Sensor y R3con Analyzer
        ↓
Resultados JSON estructurados
        ↓
API FastAPI mediante HTTPS
        ↓
MariaDB + procesamiento asíncrono
        ↓
Hallazgos priorizados, dashboard e informes PDF
        ↓
Alertas críticas por Telegram y automatizaciones n8n
```

---

## 🧩 4. Componentes principales

| Componente | Responsabilidad | Tecnologías propuestas |
|---|---|---|
| `r3con-sensor` | Descubre hosts, puertos y servicios en los rangos permitidos | Podman, Python, Nmap |
| `r3con-analyzer` | Ejecuta comprobaciones no destructivas, procesa metadatos de tráfico y crea hallazgos | Podman, Python, TShark/dumpcap, reglas YAML/JSON |
| R3con Core | Recibe resultados, valida datos, centraliza lógica y expone API | Python, FastAPI, Pydantic |
| Base de datos | Guarda clientes, usuarios, políticas, activos, escaneos y hallazgos | MariaDB/MySQL |
| Worker | Deduplica, prioriza, genera PDF y activa alertas | Python, tareas asíncronas |
| R3con Console | Portal con login, dashboard, informes y estado de remediación | HTML, CSS, JavaScript |
| Automatización | Programa procesos, notificaciones y flujos de soporte | n8n |
| Bot R3con Notify | Envía avisos críticos y resúmenes al grupo o usuario autorizado | Telegram Bot API |
| MicroCloud | Laboratorio de nube privada y orquestación de instancias | MicroCloud, LXD, MicroOVN, MicroCeph opcional |

---

## 🖥️ 5. Infraestructura del laboratorio

Se utilizará una red aislada de VirtualBox. Las máquinas objetivo no estarán conectadas a una red puente doméstica o del centro, evitando análisis fuera del laboratorio.

| Máquina virtual | Sistema operativo | CPU | RAM | Disco | Función |
|---|---|---:|---:|---:|---|
| `r3con-core` | Ubuntu Server 24.04 LTS | 2 vCPU | 4 GB | 40 GB | API, MariaDB, dashboard, worker, PDF y n8n |
| `r3con-agent` | Ubuntu Server 24.04 LTS | 2 vCPU | 2 GB | 25 GB | Podman, sensor y analizador |
| `target-web` | Ubuntu Server o Debian | 1–2 vCPU | 2 GB | 20 GB | HTTP/HTTPS, SSH y servicios de prueba |
| `target-services` | Debian, Ubuntu o Kali | 1–2 vCPU | 2 GB | 20 GB | DNS, SSH u otros servicios de laboratorio |

### Diseño de red

```text
Red NAT
└── Uso exclusivo de r3con-core:
    - Actualizaciones de software.
    - Comunicación con Telegram.
    - Uso opcional de una API de IA.

Red interna: r3con-lab
├── r3con-core
├── r3con-agent
├── target-web
└── target-services
```

> Si el equipo anfitrión no dispone de suficiente RAM, se reducirán temporalmente las VMs objetivo a 1 GB, o se levantarán únicamente las máquinas necesarias para cada prueba.

---

## ☁️ 6. Integración de MicroCloud

MicroCloud se incluirá como innovación técnica y entorno de aprendizaje, sin bloquear el funcionamiento del MVP.

### Fase A: MVP sin dependencia de MicroCloud

El sistema funcionará inicialmente con las VMs descritas anteriormente. Esta fase permite completar el flujo de valor antes de añadir complejidad adicional:

```text
Nmap → JSON → API → MariaDB → Dashboard → PDF → Telegram
```

### Fase B: demostración de MicroCloud

Se desplegará MicroCloud en una VM o en un grupo de VMs Ubuntu para probar la creación y gestión de instancias LXD. La plataforma R3con Core podrá desplegarse en una instancia separada.

```text
MicroCloud de laboratorio
│
├── Instancia LXD: r3con-core
│   ├── FastAPI
│   ├── MariaDB
│   ├── n8n
│   └── Portal R3con
│
├── Instancia LXD: r3con-monitoring
│   └── Logs, copias o pruebas de monitorización
│
└── Red virtual gestionada
    └── Aislamiento de servicios del laboratorio
```

Para un proyecto académico se puede usar un único nodo con fines de prueba. No se describirá como alta disponibilidad. Un clúster con tolerancia a fallos requiere más nodos y una configuración adicional de almacenamiento distribuido.

---

## 🐳 7. Agentes Podman

### Contenedor 1: `r3con-sensor`

**Responsabilidad:** descubrimiento de red dentro del alcance permitido.

Funciones:

- Ejecutar perfiles Nmap seguros sobre CIDR autorizados.
- Detectar hosts activos, puertos y servicios.
- Registrar versiones observables cuando estén disponibles.
- Convertir resultados de Nmap a JSON normalizado.
- Enviar resultados a la API mediante HTTPS.
- Ejecutarse manualmente o mediante programación diaria/semanal.

### Contenedor 2: `r3con-analyzer`

**Responsabilidad:** interpretar resultados y generar hallazgos técnicos.

Funciones:

- Consumir resultados normalizados del sensor.
- Aplicar reglas no destructivas para identificar riesgos potenciales.
- Usar TShark/dumpcap para generar metadatos de red cuando el laboratorio lo permita.
- Detectar ejemplos como SSH expuesto, Telnet habilitado, HTTP sin HTTPS, certificados próximos a caducar o servicios que requieren revisión.
- Crear resultados JSON con activo, evidencia, severidad, confianza y recomendación.

> No se almacenarán ficheros PCAP completos. Solo se conservarán metadatos y resultados necesarios para demostrar el análisis.

---

## 🔐 8. Control de alcance y seguridad

El sistema no permitirá introducir libremente comandos Nmap desde el dashboard. Cada agente ejecutará únicamente políticas de escaneo previamente configuradas.

Ejemplo de política:

```json
{
  "policy_id": "lab-policy-01",
  "allowed_cidrs": ["10.10.10.0/24"],
  "allowed_ports": "1-1024",
  "scan_profile": "safe",
  "max_hosts": 20,
  "excluded_hosts": ["10.10.10.1"],
  "schedule": "daily"
}
```

Antes de iniciar un análisis, el agente deberá:

1. Validar que el destino pertenece a un rango de red privado autorizado.
2. Comprobar que el CIDR coincide con la política recibida.
3. Rechazar direcciones IP públicas, dominios y argumentos arbitrarios.
4. Aplicar límites de hosts, puertos, velocidad y tiempo de ejecución.
5. Registrar toda ejecución y resultado.
6. Utilizar perfiles de escaneo predefinidos, como `safe`, en lugar de aceptar parámetros libres.

Medidas adicionales:

- Autenticación real con contraseñas hasheadas mediante Argon2id o bcrypt.
- JWT con caducidad y control de roles.
- HTTPS entre agentes y API.
- Credenciales de agente revocables.
- Validación de JSON y límite de tamaño de carga.
- Registro de auditoría de accesos, cambios de estado y descargas de informes.
- Separación lógica de datos por organización.
- Variables de entorno para secretos; nunca guardar credenciales en GitHub.
- MFA planteado como mejora futura después del MVP.

---

## 👥 9. Roles de usuario

| Rol | Permisos |
|---|---|
| Administrador R3con | Gestiona organizaciones, usuarios, políticas, agentes, reglas y auditoría global |
| Administrador cliente | Gestiona usuarios de su organización, consulta activos, informes y hallazgos; puede cambiar estados |
| Técnico cliente | Consulta activos y hallazgos de su organización; añade comentarios y marca estado, sin gestionar usuarios ni alcance |

### Estados de un hallazgo

```text
nuevo → revisado → resuelto
               └→ aceptado
```

- **Nuevo:** hallazgo recién detectado.
- **Revisado:** un usuario ha confirmado que conoce el hallazgo.
- **Resuelto:** se ha aplicado una corrección y se puede verificar posteriormente.
- **Aceptado:** el cliente acepta temporalmente el riesgo con una justificación.

---

## 🗄️ 10. Datos almacenados en la demostración

Durante la demo se almacenarán exclusivamente datos generados en el laboratorio:

- Direcciones IP privadas de las máquinas virtuales.
- Nombres de host y servicios instalados en las VMs de prueba.
- Puertos, protocolos y versiones detectadas.
- Resultados normalizados de Nmap.
- Metadatos de tráfico obtenidos con TShark.
- Hallazgos derivados de reglas internas.
- Usuarios de prueba y sus roles.
- Registros de acceso y acciones administrativas.
- Mensajes internos de soporte.
- Informes PDF generados.

No se almacenarán contraseñas reales, información de terceros, datos procedentes de Internet ni capturas PCAP completas.

---

## 🧠 11. Backend, frontend y lógica de negocio

### Stack tecnológico

| Capa | Tecnología | Uso |
|---|---|---|
| Backend | Python + FastAPI | API REST, autenticación, validación y lógica de negocio |
| Agentes | Python + Bash | Ejecución y normalización de Nmap/TShark |
| Frontend | HTML + CSS + JavaScript | Login, dashboard, tablas y vistas de informes |
| Base de datos | MariaDB/MySQL | Persistencia de usuarios, activos, hallazgos y PDFs |
| Informes | ReportLab o WeasyPrint | Generación de PDF |
| Automatización | n8n | Flujos, programación, soporte y notificaciones |
| Alertas | Telegram Bot API | Avisos críticos y resúmenes |
| Contenedores | Podman | Separación y despliegue de agentes |
| Infraestructura | VirtualBox, Proxmox y MicroCloud | Laboratorio y pruebas de nube privada |

### Flujo de negocio

```text
1. El administrador R3con crea la organización y usuarios de prueba.
2. Se define una política de alcance para el laboratorio.
3. Se registra el agente y se le asigna una identidad.
4. El agente ejecuta Nmap y comprobaciones permitidas.
5. Los resultados se normalizan y se envían mediante HTTPS.
6. FastAPI valida los datos y los guarda en MariaDB.
7. El worker deduplica y prioriza hallazgos.
8. El dashboard muestra activos, hallazgos y estados.
9. El sistema genera informes PDF periódicos o bajo demanda.
10. Si existe un hallazgo crítico, el bot de Telegram envía una alerta.
```

### Priorización de riesgos

La prioridad no dependerá únicamente de una severidad técnica. Se utilizará una puntuación interna orientativa:

\[
Riesgo = Severidad \times Exposición \times Criticidad\ del\ activo \times Confianza
\]

| Factor | Valores iniciales |
|---|---|
| Severidad | Bajo, medio, alto, crítico |
| Exposición | Interno, accesible desde varias redes, expuesto en el laboratorio |
| Criticidad del activo | Laboratorio, servidor interno, servicio relevante |
| Confianza | Indicio, probable, verificado |

---

## 📦 12. Modelo de datos inicial

```text
organizations
├── users
├── agents
├── scan_policies
├── assets
│   ├── services
│   └── findings
├── scans
├── reports
├── notifications
├── support_messages
└── audit_logs
```

| Entidad | Información principal |
|---|---|
| `organizations` | Nombre, plan de demostración, estado y fecha de alta |
| `users` | Organización, email, hash de contraseña, rol y estado |
| `agents` | Organización, versión, estado, última conexión y credencial |
| `scan_policies` | CIDR permitido, horarios, perfiles, exclusiones y límites |
| `assets` | IP, hostname, tipo, criticidad y exposición |
| `services` | Activo, puerto, protocolo, producto y versión observada |
| `scans` | Agente, política, fechas, estado y errores |
| `findings` | Activo, regla, severidad, evidencia, confianza y estado |
| `reports` | Organización, periodo, archivo, hash y fecha de generación |
| `notifications` | Hallazgo, canal, destinatario y resultado de entrega |
| `support_messages` | Organización, autor, mensaje y estado |
| `audit_logs` | Actor, acción, recurso, fecha y resultado |

---

## 📡 13. Endpoints mínimos de la API

| Método | Ruta | Función |
|---|---|---|
| `POST` | `/api/v1/auth/login` | Inicio de sesión |
| `POST` | `/api/v1/auth/refresh` | Renovación controlada de sesión |
| `GET` | `/api/v1/dashboard` | Resumen de la organización autenticada |
| `GET` | `/api/v1/assets` | Activos detectados |
| `GET` | `/api/v1/findings` | Hallazgos filtrables |
| `PATCH` | `/api/v1/findings/{id}` | Cambiar estado o añadir gestión del hallazgo |
| `GET` | `/api/v1/reports` | Histórico de informes |
| `GET` | `/api/v1/reports/{id}/download` | Descarga protegida de PDF |
| `POST` | `/api/v1/agents/enroll` | Registro controlado de agente |
| `POST` | `/api/v1/agents/{id}/results` | Envío de resultados JSON |
| `POST` | `/api/v1/agents/{id}/heartbeat` | Estado y última conexión del agente |
| `POST` | `/api/v1/messages` | Mensajería de soporte |

---

## 🤖 14. Automatización, Telegram e IA

### Automatización con n8n

n8n tendrá un papel de apoyo, no reemplazará el backend:

- Programar informes diarios o semanales.
- Recibir eventos o webhooks del backend.
- Activar envío de alertas Telegram.
- Crear flujos de recordatorio de hallazgos pendientes.
- Conectar con un futuro asistente IA.
- Automatizar respuestas básicas de soporte, siempre con control humano.

### Alertas Telegram

El bot enviará:

- Alertas de hallazgos críticos.
- Avisos de agente desconectado.
- Resumen diario o semanal.
- Enlace al portal web autenticado.

No enviará datos técnicos sensibles completos ni credenciales dentro de Telegram.

### IA como mejora futura

La IA se añadirá solo después de completar el MVP. Las posibles funciones son:

- Explicar recomendaciones técnicas para usuarios no especializados.
- Resumir hallazgos de un informe.
- Clasificar propuestas de prioridad con revisión humana.
- Asistente de soporte dentro del portal.
- Analizar logs de forma asistida.

La IA no podrá ejecutar escaneos, modificar políticas ni confirmar automáticamente una vulnerabilidad.

---

## 👨‍💻 15. Reparto de tareas

| Integrante | Responsabilidad principal | Entregables previstos |
|---|---|---|
| **Raúl** | Sistemas, redes y documentación técnica | VMs, red VirtualBox/Proxmox, Podman, Nmap, TShark, despliegue, pruebas, diagramas y manual técnico |
| **Ricard** | Backend, base de datos y supervisión documental | FastAPI, MariaDB, API, autenticación, modelo de datos, PDF y revisión de memoria |
| **Jam** | Frontend, diseño y documentación visual | Login, dashboard, vistas de hallazgos e informes, manual de usuario y apoyo transversal |
| **Equipo completo** | Integración, automatización y defensa | Telegram, n8n, pruebas, evidencias, memoria, presentación y demostración final |

---

## 🗓️ 16. Plan de trabajo

| Fase | Objetivo | Resultado esperado |
|---|---|---|
| 1. Laboratorio | Crear VMs, red aislada y repositorio | Topología funcional y documentación de IPs |
| 2. Sensor | Implementar Nmap y JSON | Descubrimiento real de objetivos del laboratorio |
| 3. Backend | API, MariaDB, login y roles | Resultados almacenados y accesibles por API |
| 4. Dashboard | Portal, activos, hallazgos y estados | Aplicación web funcional |
| 5. Informes | PDF automático y descarga protegida | Informe real basado en datos del laboratorio |
| 6. Alertas | Telegram y n8n | Aviso crítico y resumen periódico |
| 7. MicroCloud | Prueba docente e integración opcional | Evidencia de nube privada/LXD documentada |
| 8. Mejoras | IA, MFA o funciones extra si hay tiempo | Funcionalidades opcionales sin afectar el MVP |
| 9. Entrega | Memoria, presentación y demo | Proyecto defendible y reproducible |

---

## 📌 17. Criterios de éxito

El prototipo se considerará funcional si permite demostrar de extremo a extremo:

```text
Red de laboratorio autorizada
        ↓
Escaneo Nmap desde R3con Sensor
        ↓
Resultados enviados a FastAPI
        ↓
Datos almacenados en MariaDB
        ↓
Hallazgo visible en el dashboard
        ↓
Informe PDF generado
        ↓
Alerta crítica recibida en Telegram
```

La prioridad será entregar este flujo de forma estable, documentada y reproducible antes de añadir IA, MFA, nuevas herramientas o características complejas.

---

## ⚠️ Nota sobre seguridad y ética

Todas las herramientas se usarán exclusivamente contra máquinas virtuales propias y redes creadas para el laboratorio. Cada exploración estará limitada mediante una política de alcance para impedir objetivos externos, argumentos arbitrarios o redes no autorizadas.
