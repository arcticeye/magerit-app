# MAGERIT App — Documento de proyecto

## Qué es esto
Aplicación web educativa de análisis de riesgos basada en metodología MAGERIT v3.
Destinada a clases prácticas en diplomatura de ciberseguridad.
Autor: Matías Daniel Adés, consultor senior de ciberseguridad y GRC.

## Stack técnico
- Un único archivo index.html (HTML + CSS + JavaScript vanilla)
- Sin frameworks, sin dependencias, sin servidor
- Se abre directamente en el navegador

## Estado actual — MVP v1 completado
Funcionalidad implementada: análisis de riesgo inherente.

Flujo de 3 pantallas:
1. Activos: ABM completo (agregar, editar, eliminar)
2. Amenazas: pre-cargadas por tipo de activo, editables, formato acordeón
3. Resumen: tabla con riesgos calculados y colores por nivel

## Lógica de negocio implementada

Tipos de activo (3):
- Sistema Informático (cubre HW + SW + datos + red como un todo)
- Proveedor
- Personas

Escala única para valoración de activos, probabilidad y degradación:
- Muy Alta=1.0, Alta=0.8, Media=0.6, Baja=0.3, Muy Baja=0.1, N/A=0

Fórmulas:
- Impacto_dim = Valor_activo_dim × Degradación_dim
- Riesgo_dim = Impacto_dim × Probabilidad
- Riesgo_amenaza = max(Riesgo_C, Riesgo_I, Riesgo_D)

Niveles de riesgo:
- 0.00–0.19 = Leve (verde)
- 0.20–0.39 = Medio (amarillo)
- 0.40–0.59 = Moderado (naranja)
- 0.60–0.79 = Alto (rojo)
- 0.80–1.00 = Crítico (rojo oscuro)

Reglas:
- Amenazas marcadas "No aplica" no aparecen en el resumen

## Decisiones de diseño — desviaciones de MAGERIT puro

1. ACTIVOS SIMPLIFICADOS: En lugar de los tipos granulares de MAGERIT (HW, SW, COM, D, L, P),
   se usan 3 tipos operativos: Sistema Informático, Proveedor, Personas.
   Sistema Informático agrupa internamente HW + SW + datos + red sin modelar dependencias.
   Motivo: simplicidad educativa y proximidad a la práctica corporativa real.

2. RIESGO COLAPSADO: MAGERIT calcula riesgo por dimensión de forma separada.
   En esta app se muestra el desglose (Riesgo C, Riesgo I, Riesgo D) pero el Riesgo Final
   es max(Riesgo_C, Riesgo_I, Riesgo_D) como criterio conservador.
   Motivo: facilitar la toma de decisiones y la comunicación en clase.

3. SIN DEPENDENCIAS DE ACTIVOS: MAGERIT modela árboles de dependencia entre activos.
   Esta app no implementa dependencias.
   Motivo: complejidad innecesaria para el MVP educativo.

## Lógica de controles — MVP v2 (a construir)

Asociación: Control → Tipos de activo donde aplica → Amenazas que mitiga.
Basado en catálogo MAGERIT v3.

Cada control tendrá:
- Nombre y descripción
- Tipos de activo donde aplica
- Amenazas que mitiga (por nombre)
- Estado de implementación: Efectivo / Parcial / No implementado

Fórmula de riesgo actual (residual):
- A definir en la próxima sesión de diseño en Claude.ai

Lo que hay que construir en v2:
- Sección de controles por amenaza dentro de Pantalla 2
- Controles pre-cargados según amenaza y tipo de activo, editables en su estado
- Resumen actualizado: riesgo inherente + riesgo actual lado a lado

Restricciones para v2:
- Mantener archivo único index.html
- No romper funcionalidad existente de v1
- Misma escala visual y criterios de color

## Próximas iteraciones
- MVP v3: riesgo objetivo + plan de tratamiento del riesgo (PTR)
- Futuro: logging, trazabilidad, modo auditor, integración con IA (LLM)

## Instrucciones para Claude Code
- Leer index.html completo antes de cualquier cambio
- Nunca reescribir el archivo completo — hacer cambios quirúrgicos
- Confirmar el plan antes de ejecutar si el cambio afecta más de una pantalla
- Mantener todo en un único archivo index.html

## Flujo de trabajo
- Claude.ai: diseño y decisiones. Claude Code en VS Code: construcción.
- Antes de cada sesión de Claude Code: hacer git commit como punto de restauración
- Después de cada sesión de Claude Code: hacer git commit con descripción de lo construido

## Cómo arrancar una sesión nueva en Claude.ai
1. Adjuntar este archivo (DECISIONS.md) al inicio de la conversación
2. Mensaje: "Adjunto DECISIONS.md. Leelo y confirmame que entendiste el estado actual antes de continuar."
3. Esperar confirmación antes de continuar