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

## Lógica de controles — MVP v2 (DISEÑO CERRADO, listo para construir)

### Marco normativo de controles
Los controles usan la codificación y nomenclatura de ISO 27002:2022 (no MAGERIT).
Motivo: norma más moderna, internacionalmente reconocida, y útil para que los alumnos
trabajen con un marco normativo real de uso profesional.
En clase se compara con MAGERIT donde corresponde.

### Fórmula de riesgo residual — Factor de Cobertura (DECISIÓN CERRADA)

MAGERIT puro resta la efectividad de cada control directamente sobre probabilidad e impacto
por separado, usando una tabla de relevancia control-amenaza generada por macros VBA.
En esta app se simplifica con un Factor de Cobertura multiplicado sobre el riesgo inherente.

Estados de control y sus valores:
- Efectivo        → 0.20  (reduce el riesgo en un 80%)
- Parcial         → 0.50  (reduce el riesgo en un 50%)
- No implementado → 1.00  (sin reducción)

Fórmula:
  Factor_cobertura = promedio(valores de todos los controles de esa amenaza)
  Riesgo_residual  = Riesgo_inherente × Factor_cobertura

Excepción — Proveedor homologado:
  Si el control 5.19+5.20 está en estado "Efectivo" → Riesgo_residual = 0.10 (nivel Leve)
  para TODAS las amenazas del proveedor, independientemente del cálculo general.
  Motivo: la homologación del proveedor es un control integral que cubre todas sus amenazas.

Comparación con MAGERIT para usar en clase:
  "MAGERIT puro resta la efectividad de cada control sobre probabilidad e impacto por separado.
  Nosotros simplificamos a un factor promedio sobre el riesgo final. La dirección es la misma
  —más controles efectivos, menor riesgo residual— pero la magnitud no es exactamente igual.
  Esta simplificación es válida para tomar decisiones; en una auditoría real se usaría la
  metodología completa."

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

| Amenaza                              | Código ISO  | Nombre del control                          |
|-------------------------------------|-------------|---------------------------------------------|
| Error humano                        | 6.3         | Concienciación, educación y formación       |
| Error humano                        | 5.37        | Documentación de procedimientos operacionales|
| Fuga de información por el personal | 6.6         | Acuerdos de confidencialidad (NDA)          |
| Fuga de información por el personal | 8.12        | Prevención de fugas de datos (DLP)          |
| Ingeniería social (víctima)         | 6.3         | Concienciación, educación y formación       |
| Ingeniería social (víctima)         | 6.4         | Proceso disciplinario                       |
| Indisponibilidad del personal clave | 5.37        | Documentación de procedimientos operacionales|
| Indisponibilidad del personal clave | 5.30        | Preparación de TIC para continuidad         |
| Abuso de privilegios                | 8.2         | Gestión de privilegios de acceso            |
| Abuso de privilegios                | 5.3         | Segregación de tareas                       |
| Acción malintencionada interna      | 8.15        | Registro de eventos (logging)               |
| Acción malintencionada interna      | 5.3         | Segregación de tareas                       |

#### Proveedor — control único integral

| Código ISO  | Nombre del control                                              | Efecto                                      |
|-------------|----------------------------------------------------------------|---------------------------------------------|
| 5.19 + 5.20 | Seguridad en relaciones con proveedores + Acuerdos de seguridad| Si Efectivo → Riesgo residual = Leve (0.10) |
|             | (homologación: SLA + NDA + requisitos de seguridad documentados)| para TODAS las amenazas del proveedor       |

### Lo que hay que construir en v2
- Pantalla 2 (Amenazas): agregar sección de controles por amenaza dentro del acordeón
  - Controles pre-cargados según amenaza y tipo de activo
  - Cada control con selector de estado: Efectivo / Parcial / No implementado
- Pantalla 3 (Resumen): mostrar columnas adicionales
  - Riesgo inherente (ya existe)
  - Controles (cantidad y estado resumido)
  - Riesgo residual (nuevo, calculado con Factor de Cobertura)
  - Mismo código de colores existente aplicado al riesgo residual

### Restricciones para v2
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