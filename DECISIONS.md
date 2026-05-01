# MAGERIT App — Documento de proyecto

## Qué es esto
Aplicación web educativa de análisis de riesgos basada en metodología MAGERIT v3
e ISO/IEC 27002:2022. Destinada a clases prácticas en diplomatura de ciberseguridad.
Autor: Matías Daniel Adés, consultor senior de ciberseguridad y GRC.

## Stack técnico
- Un único archivo index.html (HTML + CSS + JavaScript vanilla)
- Sin frameworks, sin dependencias, sin servidor
- Se abre directamente en el navegador

## Estado actual — MVP v3 completado
Título de la app: "Análisis y Evaluación de Riesgos"
Subtítulo: "Aplicación Educativa que integra MAGERIT v3 e ISO/IEC 27002:2022"

Flujo de 4 pantallas:
1. Activos: ABM completo (agregar, editar, eliminar) con valores C, I, D.
   Cada dimensión tiene ícono (?) con tooltip explicativo sobre qué implica valorarla.
2. Amenazas: pre-cargadas por tipo de activo, editables en acordeón,
   con probabilidad, degradación y controles ISO 27002:2022 por amenaza.
   Cada control tiene ícono (?) con descripción extraída de ISO 27002:2022.
   Al cambiar el estado de un control, se replica automáticamente en todas
   las amenazas donde aparece el mismo código ISO (excepto Proveedor).
   Botón: "Calcular Riesgo Residual".
3. Resumen: tabla con riesgo inherente (C, I, D + final), riesgo residual,
   tratamiento y riesgo objetivo — todos con código de colores por nivel.
   Botón: "Planificar Tratamiento" (habilita pantalla 4; muestra alerta si
   todos los riesgos son Leve).
4. Plan de Tratamiento (PTR): lista amenazas con riesgo residual > Leve,
   ordenadas de mayor a menor. Para cada una el analista elige tipo de
   tratamiento y completa responsable, descripción y fecha objetivo.
   Botón: "Calcular Riesgo Objetivo" (actualiza columnas en Resumen y navega allí).

## Lógica de negocio implementada

### Activos
Tipos de activo (3):
- Sistema Informático (cubre HW + SW + datos + red como un todo)
- Proveedor
- Personas

Escala única para valoración de activos, probabilidad y degradación:
- Muy Alta=1.0, Alta=0.8, Media=0.6, Baja=0.3, Muy Baja=0.1, N/A=0

### Riesgo Inherente
Fórmulas:
- Impacto_dim = Valor_activo_dim × Degradación_dim
- Riesgo_dim = Impacto_dim × Probabilidad
- Riesgo_inherente = max(Riesgo_C, Riesgo_I, Riesgo_D)

Niveles de riesgo (aplica a inherente, residual y objetivo):
- 0.00–0.19 = Leve (verde)
- 0.20–0.39 = Medio (amarillo)
- 0.40–0.59 = Moderado (naranja)
- 0.60–0.79 = Alto (rojo)
- 0.80–1.00 = Crítico (rojo oscuro)

Reglas:
- Amenazas con checkbox "Incluir en el análisis" desmarcado no aparecen en el resumen ni en el PTR.

### Controles (ISO 27002:2022)
Marco normativo: ISO 27002:2022 (no MAGERIT puro).
Motivo: norma moderna, internacionalmente reconocida, útil para que los alumnos
trabajen con un marco normativo real de uso profesional.

Controles pre-cargados por amenaza y tipo de activo según catálogo definido.
Cada control tiene código ISO, nombre, estado de implementación y tooltip
con descripción del propósito del control extraída de ISO 27002:2022.

Estados de control:
- Efectivo        → valor 0.20 (reduce el riesgo en un 80%)
- Parcial         → valor 0.50 (reduce el riesgo en un 50%)
- No implementado → valor 1.00 (sin reducción)

Nota: Para activos tipo Proveedor el control es binario (Efectivo / No implementado).
No existe estado Parcial para proveedores — la homologación es una decisión formal.

Propagación de estados: al cambiar el estado de un control en cualquier amenaza,
el nuevo estado se propaga automáticamente a todos los controles con el mismo
código ISO en todas las amenazas de todos los activos no-Proveedor.
Los controles de Proveedor (5.19+5.20) quedan excluidos de la propagación.

### Riesgo Residual — Factor de Cobertura
Fórmula general:
  Factor_cobertura = promedio(valores de todos los controles de esa amenaza)
  Riesgo_residual  = Riesgo_inherente × Factor_cobertura

Ejemplo verificado:
  Activo SI: C=0.8, I=0.6, D=0.3 / Amenaza acceso no autorizado: Prob=0.8, DegC=0.8, DegI=0.6, DegD=0.3
  Riesgo inherente = max(0.51, 0.29, 0.07) = 0.51
  Controles: 5.15 Efectivo(0.20) + 8.5 Parcial(0.50) → Factor = 0.35
  Riesgo residual = 0.51 × 0.35 = 0.18 ✓

Excepción — Proveedor homologado:
  Si el control 5.19+5.20 se marca como "Efectivo" en cualquier amenaza del proveedor:
  - Aparece un confirm() explicando que la homologación es un control integral
  - Si el usuario acepta: todos los controles de TODAS las amenazas del proveedor
    pasan a Efectivo y el riesgo residual = 0.10 (Leve) para todas
  - Si el usuario cancela: no se hace ningún cambio
  La misma lógica simétrica aplica al cambiar a "No implementado":
  - Confirm() advierte que se quitará la homologación de todas las amenazas
  - Si acepta: todos los controles vuelven a No implementado

Comparación con MAGERIT para usar en clase:
  "MAGERIT puro resta la efectividad de cada control sobre probabilidad e impacto
  por separado, usando una tabla de relevancia generada por macros VBA. Nosotros
  simplificamos a un factor promedio sobre el riesgo final. La dirección es la misma
  —más controles efectivos, menor riesgo residual— pero la magnitud no es exactamente
  igual. Esta simplificación es válida para tomar decisiones; en una auditoría real
  se usaría la metodología completa."

### Plan de Tratamiento del Riesgo (PTR)

Solo aparecen amenazas con riesgo residual > 0.19 (superiores a Leve), ordenadas
de mayor a menor. Los riesgos Leves no requieren PTR formal.

Tipos de tratamiento y efecto en el Riesgo Objetivo:

| Tratamiento | Riesgo Objetivo | Lógica |
|-------------|----------------|--------|
| Mitigar     | Recalculado según controles comprometidos | El analista tilda controles pendientes (No implementado / Parcial); se simulan como Efectivos |
| Transferir  | Riesgo Residual × 0.50 | Se asume seguro/tercerización que cubre el 50% del impacto |
| Evitar      | 0.00 | Se elimina la actividad/condición que genera el riesgo |
| Aceptar     | Igual al Riesgo Residual | Se asume conscientemente sin acción adicional |

Campos por amenaza: Responsable, Descripción del plan, Fecha objetivo.

Flujo de navegación del PTR:
- "Calcular Riesgo Residual" en Amenazas → genera tabla en Resumen y habilita botón "Planificar Tratamiento".
- Si ya existe un PTR y se vuelve a calcular el Riesgo Residual: confirm() advierte que el PTR se borrará.
- "Planificar Tratamiento" en Resumen → habilita pestaña "Plan de Tratamiento" y navega allí.
- "Calcular Riesgo Objetivo" en PTR → actualiza columnas Tratamiento y Riesgo Objetivo en Resumen y navega allí.

### Catálogo de controles ISO 27002:2022 — mapeo amenaza → controles

#### Sistema Informático

| Amenaza                        | Código ISO  | Nombre del control                          |
|-------------------------------|-------------|---------------------------------------------|
| Acceso no autorizado          | 5.15        | Control de acceso                           |
| Acceso no autorizado          | 8.5         | Autenticación segura                        |
| Malware / Ransomware          | 8.7         | Controles contra el código malicioso        |
| Malware / Ransomware          | 8.13        | Copias de seguridad de la información       |
| Fallo de hardware o software  | 8.13        | Copias de seguridad de la información       |
| Fallo de hardware o software  | 8.14        | Redundancia de recursos de tratamiento      |
| Error de configuración        | 8.9         | Gestión de la configuración                 |
| Error de configuración        | 8.8         | Gestión de vulnerabilidades técnicas        |
| Denegación de servicio (DoS)  | 8.6         | Gestión de capacidades                      |
| Denegación de servicio (DoS)  | 8.20        | Controles de red                            |
| Fuga de información           | 8.12        | Prevención de fugas de datos (DLP)          |
| Fuga de información           | 8.24        | Uso de criptografía                         |
| Ingeniería social / Phishing  | 6.3         | Concienciación, educación y formación       |
| Ingeniería social / Phishing  | 8.23        | Filtrado web                                |

#### Personas

| Amenaza                              | Código ISO  | Nombre del control                           |
|-------------------------------------|-------------|----------------------------------------------|
| Error humano                        | 6.3         | Concienciación, educación y formación        |
| Error humano                        | 5.37        | Documentación de procedimientos operacionales|
| Fuga de información por el personal | 6.6         | Acuerdos de confidencialidad (NDA)           |
| Fuga de información por el personal | 8.12        | Prevención de fugas de datos (DLP)           |
| Ingeniería social (víctima)         | 6.3         | Concienciación, educación y formación        |
| Ingeniería social (víctima)         | 6.4         | Proceso disciplinario                        |
| Indisponibilidad del personal clave | 5.37        | Documentación de procedimientos operacionales|
| Indisponibilidad del personal clave | 5.30        | Preparación de TIC para continuidad          |
| Abuso de privilegios                | 8.2         | Gestión de privilegios de acceso             |
| Abuso de privilegios                | 5.3         | Segregación de tareas                        |
| Acción malintencionada interna      | 8.15        | Registro de eventos (logging)                |
| Acción malintencionada interna      | 5.3         | Segregación de tareas                        |

#### Proveedor — control único integral

| Código ISO  | Nombre del control                                               | Efecto                                       |
|-------------|------------------------------------------------------------------|----------------------------------------------|
| 5.19 + 5.20 | Seguridad en relaciones con proveedores + Acuerdos de seguridad  | Si Efectivo → Riesgo residual = 0.10 (Leve)  |
|             | (homologación: SLA + NDA + requisitos de seguridad documentados) | para TODAS las amenazas del proveedor        |

## Decisiones de diseño — desviaciones de MAGERIT puro

1. ACTIVOS SIMPLIFICADOS: En lugar de los tipos granulares de MAGERIT (HW, SW, COM, D, L, P),
   se usan 3 tipos operativos: Sistema Informático, Proveedor, Personas.
   Sistema Informático agrupa internamente HW + SW + datos + red sin modelar dependencias.
   Motivo: simplicidad educativa y proximidad a la práctica corporativa real.

2. RIESGO COLAPSADO: MAGERIT calcula riesgo por dimensión de forma separada.
   En esta app se muestra el desglose (R. Inherente C, I, D) pero el Riesgo Inherente final
   es max(Riesgo_C, Riesgo_I, Riesgo_D) como criterio conservador.
   Motivo: facilitar la toma de decisiones y la comunicación en clase.

3. SIN DEPENDENCIAS DE ACTIVOS: MAGERIT modela árboles de dependencia entre activos.
   Esta app no implementa dependencias.
   Motivo: complejidad innecesaria para el MVP educativo.

4. CONTROLES ISO 27002:2022 EN LUGAR DE SALVAGUARDAS MAGERIT: Se usa ISO 27002:2022
   como marco de controles por ser más moderno y reconocido internacionalmente.
   El mapeo amenaza→control es decisión del analista (no existe en MAGERIT Libro II).
   Motivo: valor educativo y relevancia profesional.

5. RIESGO RESIDUAL POR FACTOR DE COBERTURA: En lugar de la sustracción por dimensión
   de MAGERIT puro, se usa un factor multiplicador promedio sobre el riesgo inherente final.
   Motivo: implementable sin macros VBA, educativamente equivalente en dirección.

6. PTR SIMPLIFICADO: El PTR no incluye campos de coste, beneficio, prioridad ni seguimiento.
   Solo captura tipo de tratamiento, responsable, descripción y fecha objetivo.
   El Riesgo Objetivo se calcula automáticamente según el tipo de tratamiento.
   Motivo: foco en el proceso de decisión, no en la gestión operativa del plan.

7. TRANSFERIR = REDUCCIÓN DEL 50%: Se asume que transferir el riesgo (ej: seguro)
   reduce el impacto en un 50%. Valor fijo, sin modelar la cobertura real del seguro.
   Motivo: simplificación pedagógica para ilustrar el concepto.

## Valores pre-cargados de amenazas — justificación

Los valores de probabilidad y degradación pre-cargados en la app son estimaciones
iniciales basadas en la frecuencia e impacto típico de cada amenaza según el
catálogo de MAGERIT v3 Libro II. No son valores normativos fijos — el analista
los ajusta según el contexto real de cada organización.
Para usar en clase: "Los valores pre-cargados son un punto de partida razonable.
Vuestro trabajo como analistas es ajustarlos a la realidad de cada organización."

## Próximas iteraciones
- MVP v4 (futuro): logging, trazabilidad, modo auditor, integración con IA (LLM)

## Instrucciones para Claude Code
- Leer index.html completo antes de cualquier cambio
- Nunca reescribir el archivo completo — hacer cambios quirúrgicos
- Confirmar el plan antes de ejecutar si el cambio afecta más de una pantalla
- Mantener todo en un único archivo index.html

## Flujo de trabajo
- Claude.ai: diseño y decisiones. Claude Code en terminal o VS Code: construcción.
- Claude Code se usa vía extensión VS Code con modelo Sonnet 4.5.
  Si la extensión no funciona, usar terminal con: npx claude
- Antes de cada sesión de Claude Code: hacer git commit como punto de restauración
- Después de cada sesión de Claude Code: hacer git commit + git push origin main

## Cómo arrancar una sesión nueva en Claude.ai
1. Adjuntar este archivo (DECISIONS.md) al inicio de la conversación
2. Mensaje: "Adjunto DECISIONS.md. Leelo y confirmame que entendiste el estado actual antes de continuar."
3. Esperar confirmación antes de continuar
