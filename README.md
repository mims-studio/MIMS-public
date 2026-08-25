# MIMS — un recepcionista que trabaja por WhatsApp y por teléfono

Los negocios pequeños pierden reservas por no coger el teléfono. Mientras cortas el
pelo o sacas platos no puedes atender, y la llamada perdida no vuelve.

MIMS es un asistente que contesta por ellos: coge la reserva, la mueve, la cancela y
avisa al dueño. Por WhatsApp y por voz. El negocio se da de alta solo —conversando
con un chat, no rellenando un formulario— y en minutos tiene su número, su asistente
entrenado con sus servicios y sus horarios, y su panel.

> Caso de estudio. El código es privado; aquí está la arquitectura y las decisiones.
> Producto en producción con clientes de pago.

---

## Lo difícil no es el chatbot

Un bot que contesta lo monta cualquiera en una tarde. Lo que cuesta es lo de debajo:
**dar de alta un negocio entero sin que nadie toque nada a mano.**

Cuando alguien paga, en los siguientes 60 segundos hay que:

```mermaid
flowchart LR
  A["Chat de alta<br/>(conversación, no formulario)"] --> B["Pago<br/>Stripe"]
  B --> C["Se crea el negocio<br/>servicios · horarios · equipo"]
  C --> D["Número de teléfono<br/>comprado y enrutado"]
  D --> E["WhatsApp Business<br/>verificado en Meta"]
  D --> F["Agente de voz<br/>que descuelga llamadas"]
  E --> G["Bandeja de atención<br/>Chatwoot"]
  G --> H["Panel del cliente<br/>+ bienvenida por WhatsApp"]
  F --> H
```

Siete sistemas externos, en cadena, cada uno con sus propios fallos. Y todo eso
mientras el cliente mira una pantalla que dice «activando tu alta».

**Ahí es donde está la ingeniería de verdad:** que un paso que falla no deje al
cliente pagado y a medias.

---

## Decisiones que sostienen eso

**Cada paso se puede repetir sin romper nada.** Reintentar un alta a medias no
duplica el negocio, ni compra otro número, ni crea un segundo agente. Suena obvio y
es la mitad del trabajo: sin idempotencia, cada fallo transitorio te deja basura que
alguien tiene que limpiar a mano.

**Los estados no mienten.** Una corrida a medias nunca queda como «aprobada»: pasa a
«aprovisionando» y, si falla, a «error» con el motivo. El estado en base de datos
siempre dice la verdad sobre lo que pasó, porque es lo único en lo que puedes
apoyarte cuando algo se rompe a las tres de la mañana.

**Frenos donde el error es irreversible.** Verificar un número en WhatsApp tiene un
solo intento: si lo gastas mal, el número queda inservible **para siempre**. Así que
esa llamada está detrás de un candado que solo se abre a mano. Un número quemado no
se recupera con un rollback.

**Y cuando el número no sirve, salta al siguiente.** Si el que le tocó no puede
completarse, el sistema lo aparta —para que no se lo lleve el siguiente cliente— y
le da otro del pool, recableando la voz y la mensajería de una pieza.

**Aislamiento entre clientes, comprobado.** Cada negocio ve solo lo suyo. No por
convención: hay pruebas que intentan cruzar los datos de dos negocios a propósito y
tienen que fallar.

**Las reglas de negocio viven en la base de datos.** Los huecos, los solapes, la
capacidad y las franjas se calculan en SQL, no en el código de la aplicación. Un
solo sitio que decide si una reserva cabe, compartido por WhatsApp, por la voz y por
el panel. Tres clientes distintos no pueden dar tres respuestas distintas a la misma
pregunta.

---

## Lo que cubre

| | |
|---|---|
| **Reservas** | crear, mover, cancelar y consultar, con capacidad y solapes reales |
| **Dos modelos** | por mesas (restaurantes) o por profesional (peluquerías, clínicas) |
| **22 sectores** | de restaurante a fisioterapia, cada uno con su lenguaje y su plantilla |
| **27 idiomas** | contesta a cada cliente en el suyo |
| **Voz** | descuelga el teléfono, conversa y reserva |
| **Panel** | calendario, clientes, conversaciones, equipo, facturación |
| **App móvil** | iOS y Android |
| **Roles** | dueño, encargado y empleado, con permisos por función |
| **Alta** | conversacional, con pago y aprovisionamiento automáticos |

---

## Con qué está hecho

**Producto** · TypeScript · Node.js · Next.js · React · Tailwind · React Native (Expo)

**Datos** · PostgreSQL con lógica en PL/pgSQL · migraciones versionadas · Drizzle ORM

**Infraestructura** · Docker y Docker Compose · Caddy · VPS propio · Vercel · despliegues
reproducibles desde el tronco

**Integraciones** · WhatsApp Cloud API (Meta) · Stripe · Chatwoot · Retell AI · LiveKit ·
Deepgram · proveedores SIP · Resend

**IA** · Gemini con tool-calling · prompts por sector · cadena de respaldo entre modelos
para que una caída del proveedor no deje al negocio mudo

**Calidad** · Vitest · pruebas de integración contra base real · un registro de hallazgos
donde cada bug queda escrito con su reproducción y su causa

---

## Números

| | |
|---|---|
| Migraciones de base de datos | **80** |
| Pruebas automáticas | **336** |
| Servicios externos integrados | **8** |
| Sectores soportados | **22** |
| Idiomas | **27** |

---

## Quién lo ha hecho

Andreu Martín — producto, arquitectura y desarrollo.

Disponible para proyectos como freelance: integraciones complejas, automatización de
procesos, SaaS multi-tenant y asistentes con IA que hacen cosas de verdad, no solo
conversar.

**[mims.studio](https://mims.studio)**
