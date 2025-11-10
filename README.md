# Análisis de Diagramas de Estado: Sistema de Farmacia

Este documento describe el análisis y diseño de los diagramas de estado para un sistema de farmacia, identificando los objetos clave y su ciclo de vida.

---

## 1. ¿Qué es un Diagrama de Estado?

Un diagrama de estado se utiliza para modelar el **comportamiento de un solo objeto** o sistema, mostrando cómo responde a diferentes eventos a lo largo de su vida.

Para ser funcional, debe tener:

* **Estado Inicial (⚫):** El punto de partida.
* **Estados:** Rectángulos con esquinas redondeadas que representan las situaciones del objeto (Ej: "Esperando", "Aprobado").
* **Transiciones:** Flechas que muestran el cambio de un estado a otro.
* **Eventos (Disparadores):** Texto sobre la flecha que indica *qué* causó la transición (Ej: "Presionar botón").
* **Estado Final (🎯):** El punto donde termina el ciclo de vida del objeto.

---

## 2. Diferencia Clave: Diagrama de Estado vs. Diagrama de Secuencia

Es fundamental no confundir estos dos diagramas.

* **Diagrama de Estado:** Se enfoca en **UN SOLO** objeto y sus **condiciones internas (estados)** a lo largo del tiempo.
    * *Responde a:* "¿En qué estados puede estar un `Pedido` y cómo pasa de 'Pendiente' a 'Pagado'?"

* **Diagrama de Secuencias:** Se enfoca en **MÚLTIPLES** objetos y cómo **interactúan** entre sí (qué mensajes se envían) en un orden cronológico.
    * *Responde a:* "¿Cómo interactúan el `Cliente`, el `SistemaDePago` y el `Pedido` para 'Pagar un Pedido'?"

| Característica | Diagrama de Estado | Diagrama de Secuencias |
| :--- | :--- | :--- |
| **Enfoque** | El ciclo de vida de **un** objeto. | La interacción entre **varios** objetos. |
| **Elemento principal** | Estados (condiciones) | Mensajes (comunicación) |
| **Propósito** | Modelar el **comportamiento** interno. | Modelar la **colaboración** entre partes. |

**Regla de Oro:** No se enfoca en "una sola acción", sino en el ciclo de vida de **"un solo objeto"**. Intentar modelar todo el sistema en un solo diagrama de estado lo volverá ilegible.

---

## 3. Objetos Principales para Modelar

Tras analizar el sistema de farmacia, se identificaron 3 objetos cuyo ciclo de vida es complejo y crítico para el negocio. Cada uno tendrá su propio diagrama de estado.

1.  **`Receta Médica`**: Para el flujo de validación y dispensación.
2.  **`LoteDeMedicamento`**: Para el flujo de inventario y seguridad.
3.  **`Pedido / Venta`**: Para el flujo de transacción comercial.

---

## 4. Consideraciones de Análisis

### ¿El cliente se toma en consideración?

No, no directamente. El `Cliente` es la *fuente* del evento en el mundo real, pero no es el *actor* que interactúa con el sistema.

* **Mundo Real:** El `Cliente` le dice al `Personal`: "Quiero cancelar mi pedido".
* **Interacción Humano-Sistema:** El `Personal` entra al sistema y presiona el botón "Anular Pedido".
* **Diagrama de Estado:** El diagrama del `Pedido` solo modela la transición desde `En Preparación` a `Cancelado`, disparada por el **evento `anularPedido`**.

El diagrama no se preocupa de *por qué* el personal disparó el evento, solo de que el evento ocurrió.

### ¿Son suficientes estos 3 diagramas?

Sí. Para una presentación y un análisis de alto nivel, es la estrategia correcta.

* **Evita el "Diagrama Monstruo":** En lugar de un diagrama ilegible, se presentan 3 diagramas claros y enfocados.
* **Demuestra Dominio:** Prueba que se entiende que el "Sistema de Farmacia" no es un objeto, sino un *contenedor* de objetos (`Receta`, `Lote`, `Pedido`) que sí tienen estados.
* **Nivel Lógico:** Estos diagramas representan un **Análisis de Nivel 1 (Lógico)**. Un análisis más profundo (Nivel 2) añadiría **Guardas `[condiciones]`** y **Acciones `/ acciones()`** a las transiciones, pero esto no es necesario para explicar el flujo conceptual.

---

## 5. Código PlantUML de los Diagramas

A continuación, se presenta el código PlantUML para cada uno de los objetos identificados.

### 1. Receta Médica (Flujo de Validación y Dispensación)

Este diagrama muestra cómo la receta es validada por el personal farmacéutico y luego surtida.

```plantuml
@startuml
title Diagrama de Estado: Receta Médica

' Configuración visual
skinparam state {
  BackgroundColor<<endState>> #FFB0B0
  BorderColor<<endState>> #D80000
  BackgroundColor<<archivedState>> #DDDDDD
}

' Estados
state "Nueva" as Nueva
state "En Verificación" as EnVerificacion
state "Aprobada" as Aprobada
state "Rechazada" as Rechazada
state "Surtida Parcialmente" as SurtidaParcial
state "Surtida Totalmente" as SurtidaTotal
state "Expirada" as Expirada
state "Archivada" as Archivada <<archivedState>>
state "Final" as Fin <<endState>>

' Flujo principal
[*] --> Nueva : registrarEnSistema

Nueva --> EnVerificacion : farmaceuticoValida
Nueva --> Expirada : expira

EnVerificacion --> Aprobada : aprobarReceta
EnVerificacion --> Rechazada : rechazarReceta

Aprobada --> SurtidaTotal : surtirTodo
Aprobada --> SurtidaParcial : surtirParcial
Aprobada --> Expirada : expira

SurtidaParcial --> SurtidaTotal : surtirRestante
SurtidaParcial --> Expirada : expira

' Estados finales
SurtidaTotal --> Archivada
Rechazada --> Archivada
Expirada --> Archivada

Archivada --> Fin
Fin --> [*]

@enduml








@startuml
title Diagrama de Estado: Lote de Medicamento

' Configuración visual
skinparam state {
  BackgroundColor #LightGreen
  BorderColor #339933
  BackgroundColor<<dangerState>> #FFC0C0
  BorderColor<<dangerState>> #D80000
  BackgroundColor<<finalState>> #EEEEEE
}

' Estados
state "Solicitado" as Solicitado
state "En Tránsito" as EnTransito
state "En Cuarentena" as EnCuarentena
state "Disponible" as Disponible
state "Próximo a Vencer" as ProximoAVencer
state "Vencido" as Vencido <<dangerState>>
state "Retirado" as Retirado <<dangerState>>
state "Agotado" as Agotado <<finalState>>

' Flujo principal
[*] --> Solicitado : generarOrdenDeCompra

Solicitado --> EnTransito : proveedorEnvia
EnTransito --> EnCuarentena : recibirEnAlmacen
EnCuarentena --> Disponible : aprobarCalidad
EnCuarentena --> Retirado : rechazarCalidad

Disponible --> Disponible : venderUnidad
Disponible --> ProximoAVencer : detectarFechaCorta
Disponible --> Vencido : cumplirFechaCaducidad
Disponible --> Retirado : emitirAlertaSanitaria
Disponible --> Agotado : ultimaUnidadVendida

ProximoAVencer --> Vencido : cumplirFechaCaducidad
ProximoAVencer --> Retirado : emitirAlertaSanitaria
ProximoAVencer --> Agotado : ultimaUnidadVendida

' Estados finales
Vencido --> [*]
Retirado --> [*]
Agotado --> [*]

@enduml







@startuml
title Diagrama de Estado: Pedido / Venta

' Configuración visual
skinparam state {
  BackgroundColor<<finalState>> #LightGreen
  BorderColor<<finalState>> #008000
  BackgroundColor<<cancelState>> #FFB0B0
  BorderColor<<cancelState>> #D80000
}

' Estados
state "En Creación" as EnCreacion
state "Pendiente de Pago" as PendientePago
state "Pagado" as Pagado
state "Pendiente de Receta" as PendienteReceta
state "En Preparación" as EnPreparacion
state "Listo para Entrega" as ListoEntrega
state "Completado" as Completado <<finalState>>
state "Cancelado" as Cancelado <<cancelState>>

' Pseudostado de decisión (Choice)
state choice <<choice>>

' Flujo principal
[*] --> EnCreacion : clienteIniciaPedido

EnCreacion --> PendientePago : finalizarCaptura
EnCreacion --> Cancelado : cancelar

PendientePago --> Pagado : pagoExitoso
PendientePago --> Cancelado : pagoRechazado
PendientePago --> Cancelado : tiempoAgotado

' Lógica de decisión después de pagar
Pagado --> choice : [evaluarItems]
choice --> PendienteReceta : [requiereReceta]
choice --> EnPreparacion : [!requiereReceta]

' Flujo de validación de receta
PendienteReceta --> EnPreparacion : recetaAprobada
PendienteReceta --> Cancelado : recetaRechazada

' Flujo de entrega
EnPreparacion --> ListoEntrega : terminarEmpaque
ListoEntrega --> Completado : entregarACliente

' Estados finales
Completado --> [*]
Cancelado --> [*]

@enduml









