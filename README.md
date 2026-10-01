# 🛡️ SecureReport

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
