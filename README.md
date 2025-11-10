# Diagrama-de-Estado
# Diagramas de Estado: Sistema de Farmacia

Aquí tienes el código PlantUML para cada uno de los tres diagramas de estado que discutimos.

Puedes copiar y pegar cada bloque de código por separado en un editor de PlantUML (como el sitio web oficial, VScode con la extensión de PlantUML, etc.) para generar las imágenes. GitHub también puede renderizar estos diagramas automáticamente si están en un bloque de código `plantuml`.

---

## 1. Receta Médica (Flujo de Validación y Dispensación)

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
