# Distributed Realms

Laboratorio de sistemas distribuidos para explorar **consistencia, fallos parciales y recuperación**, construido con **Java 25, Spring Boot 4 y Floci** como emulador AWS local.

El dominio usa una recompensa de misión RPG como ejemplo: conceder experiencia y entregar un objeto mediante tres servicios independientes. El objetivo es comprender y demostrar garantías distribuidas; no construir un RPG funcional ni enseñar programación.

## Primer laboratorio

Una saga orquestada coordina una recompensa entre tres procesos:

| Servicio | Responsabilidad |
| --- | --- |
| Quest | Iniciar la recompensa y persistir el progreso de la saga |
| Hero | Conceder XP una sola vez por recompensa |
| Inventory | Entregar un objeto una sola vez por recompensa |

Los servicios intercambian comandos y eventos mediante **SQS Standard** y conservan su propio estado en **DynamoDB**. Ambos servicios AWS se ejecutarán localmente en Floci.

Si la XP ya fue concedida y la entrega falla, se conserva la XP y se recupera la entrega pendiente. Durante ese intervalo, los servicios pueden mostrar estados distintos sin que se haya violado la regla de negocio.

## Dos iteraciones comparables

### 1. Vulnerable, sin ser artificialmente ingenua

Incluye validación, idempotencia de negocio, transiciones condicionales y persistencia local atómica. Su vulnerabilidad es la escritura separada del cambio de negocio y la publicación de su resultado.

El experimento detiene Hero después de guardar XP y antes de publicar `ExperienceGranted` o confirmar el comando. Tras la reentrega, Hero reconoce la recompensa procesada y confirma el duplicado sin republicar el resultado: no duplica XP, pero la saga queda pendiente.

Esto permite estudiar por qué **idempotencia local y recuperación del flujo distribuido son garantías distintas**. La republicación de resultados ante duplicados también se discutirá como alternativa.

### 2. Recuperable

Añade outbox transaccional y publicación recuperable. Mantiene las reglas de negocio y controles de la primera iteración para aislar la mejora.

Los experimentos cubrirán:

- Caídas entre persistencia y publicación.
- Caídas después de publicar y antes de registrar el envío.
- Comandos y resultados duplicados, incluidos mensajes equivalentes con identificadores distintos.
- Transiciones de saga concurrentes.
- Dos publicadores seleccionando la misma entrada de outbox.
- Reinicios del coordinador e indisponibilidad de Inventory.

La meta es un único efecto de negocio pese a reentregas; no se promete entrega de transporte exactamente una vez.

## Método de aprendizaje

Cada experimento tendrá una predicción, puntos de interrupción controlados, evidencia observable y criterios de aceptación. Se comprobarán XP, objetos, estado de saga y publicaciones pendientes.

Las versiones verificadas se conservarán mediante etiquetas Git previstas: `lab-01-vulnerable` y `lab-02-recoverable`. Estas etiquetas todavía no existen.

## Alcance y siguientes temas

El primer laboratorio usa una misión ya cumplida, XP fija e inventario con capacidad suficiente. No incluye combate, clases, niveles ni interfaz de juego.

Se explorarán después compensaciones, timeouts, coreografía, Event Sourcing, CQRS, concurrencia bajo carga y distribución con EventBridge y SNS.

## Estado del proyecto

**Fase de diseño.** El repositorio contiene la especificación; aún no hay servicios implementados, infraestructura ejecutable ni experimentos verificados. La compatibilidad de dependencias y operaciones requeridas en Floci se comprobará antes de implementar.

Consulta [la especificación del laboratorio y sus diagramas C4](distributed_realms_plan.md) para conocer los contratos, garantías, experimentos y arquitectura acordados.
