# PetFlow IA — Consultas y Reclamos

Automatización desarrollada en **Make** para recibir consultas de clientes por Gmail, registrarlas en Airtable, clasificarlas con IA y decidir si pueden responderse automáticamente o si requieren aprobación humana (**HITL — Human in the Loop**).

## Objetivo

Automatizar la atención inicial de consultas y reclamos, manteniendo control humano cuando la respuesta involucra situaciones sensibles, por ejemplo:

- devoluciones de dinero;
- reclamos;
- baja confianza de la IA;
- excepciones económicas;
- casos críticos o que requieren revisión.

## Tecnologías utilizadas

- **Make** — orquestación de los escenarios.
- **Gmail** — recepción y respuesta de correos.
- **Airtable** — almacenamiento de clientes, tickets, aprobaciones y errores.
- **OpenAI** — clasificación y generación de propuestas de respuesta.
- **JSON** — intercambio estructurado de datos entre módulos.

## Arquitectura general

```text
Gmail
  ↓
Airtable — Cliente / Ticket
  ↓
OpenAI
  ↓
Parse JSON
  ↓
Airtable — Actualización del ticket
  ↓
Router
  ├── Respuesta automática
  │      ↓
  │    Gmail
  │      ↓
  │    Airtable
  │
  └── Requiere aprobación humana
         ↓
       Airtable — Aprobaciones
         ↓
       Gmail — APROBAR / RECHAZAR
         ↓
       Webhook HITL
         ↓
       Router
         ├── Aprobar → Gmail → Airtable
         └── Rechazar → Airtable
```

## Escenario 1 — Recepción y Procesamiento

**Nombre:** `PetFlow IA - Recepción y Procesamiento`

Flujo principal:

1. Gmail detecta un nuevo correo.
2. Airtable crea o actualiza el cliente.
3. Se crea un ticket.
4. OpenAI analiza el mensaje.
5. La respuesta de IA se procesa como JSON.
6. Airtable actualiza el ticket con:
   - categoría;
   - prioridad;
   - sentimiento;
   - confianza;
   - resumen;
   - respuesta propuesta;
   - necesidad de aprobación.
7. Un Router decide:
   - **respuesta automática**, o
   - **aprobación humana**.
8. Si existe un error en OpenAI, se registra en Airtable y se detiene la ejecución controladamente.

## IA y salida estructurada

La IA devuelve información en formato JSON con campos como:

```json
{
  "categoria": "producto_stock",
  "prioridad": "media",
  "sentimiento": "neutral",
  "confianza": 0.92,
  "resumen": "Consulta por precio de una correa",
  "requiere_aprobacion": false,
  "motivo_aprobacion": "",
  "respuesta_propuesta": "Gracias por contactarnos..."
}
```

### Reglas principales

Se solicita aprobación humana cuando:

- la confianza es menor a 0.80;
- existe una devolución de dinero;
- el caso es crítico;
- hay una excepción económica;
- existe un reclamo sensible;
- la IA no dispone de información suficiente.

## Escenario 2 — Aprobación HITL

**Nombre:** `PetFlow IA - Aprobación HITL`

Cuando un ticket requiere revisión:

1. se registra una aprobación pendiente en Airtable;
2. el aprobador recibe un correo;
3. el correo contiene las opciones **APROBAR** y **RECHAZAR**;
4. la decisión llega a Make mediante un Webhook;
5. un Router procesa la decisión.

### Si se aprueba

- se actualiza el ticket;
- se envía la respuesta propuesta al cliente;
- el estado final queda como **Respuesta enviada**.

### Si se rechaza

- no se envía la respuesta al cliente;
- el ticket queda como **Rechazado por humano**.

## Manejo de errores

El escenario incluye un Error Handler sobre el módulo de OpenAI.

Ante una falla:

1. se crea un registro en la tabla **Errores** de Airtable;
2. se guarda información del escenario, módulo, tipo de error y descripción;
3. se utiliza **Commit** para finalizar la ejecución de forma controlada.

Esto permite trazabilidad y evita que un error quede oculto.

## Pruebas realizadas

| Prueba | Resultado |
|---|---|
| Consulta simple con respuesta automática | ✅ Aprobada |
| Reclamo que requiere aprobación humana | ✅ Aprobada |
| HITL — decisión APROBAR | ✅ Aprobada |
| HITL — decisión RECHAZAR | ✅ Aprobada |
| Registro de error en Airtable | ✅ Aprobada |

## Evidencias

Las capturas incluidas en la entrega muestran:

- respuesta automática en Gmail;
- correo de aprobación HITL;
- respuesta enviada luego de aprobación humana;
- ejecución de la rama de rechazo;
- estado **Rechazado por humano** en Airtable;
- Error Handler de Make;
- tabla de errores de Airtable.

## Blueprints

La entrega incluye los Blueprints exportados desde Make:

- `PetFlow IA - Recepción y Procesamiento`
- `PetFlow IA - Aprobación HITL`

Los Blueprints permiten reconstruir la arquitectura del escenario, pero las conexiones y credenciales deben configurarse nuevamente en cada entorno.

## Seguridad

Para una implementación real se recomienda:

- no publicar API Keys ni credenciales;
- mantener las conexiones administradas desde Make;
- regenerar Webhooks expuestos durante pruebas;
- limitar el acceso a Airtable;
- revisar permisos de Gmail;
- utilizar variables o conexiones seguras para secretos;
- evitar incluir datos sensibles de clientes en repositorios públicos.

## Estructura sugerida del repositorio

```text
PetFlow-IA/
│
├── README.md
├── blueprints/
│   ├── PetFlow_Recepcion_Procesamiento.json
│   └── PetFlow_Aprobacion_HITL.json
│
├── docs/
│   └── PetFlow_IA_Entrega_Final.docx
│
└── evidencias/
    ├── 01_Respuesta_Automatica.png
    ├── 02_HITL_Solicitud_Aprobacion.png
    ├── 03_HITL_Aprobado.png
    ├── 04_HITL_Rechazado_Make.png
    ├── 05_HITL_Rechazado_Airtable.png
    ├── 06_Manejo_Errores.png
    └── 07_Tabla_Errores_Airtable.png
```

## Resultado

El prototipo demuestra un flujo de atención automatizada con IA que combina:

- automatización;
- almacenamiento estructurado;
- clasificación inteligente;
- respuestas automáticas;
- supervisión humana;
- trazabilidad;
- manejo de errores.

  ## Video demostrativo

[Ver demostración completa de PetFlow IA](https://drive.google.com/file/d/1n2U8Jjw55NyR5iCLEeLWRacmW7ynfVuC/view?usp=drive_link)

El diseño permite automatizar tareas repetitivas sin eliminar el control humano en decisiones sensibles.

---

**Proyecto:** PetFlow IA — Consultas y Reclamos  
**Estado:** Entrega final / prototipo funcional
