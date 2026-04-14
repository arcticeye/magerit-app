# DECISIONS.md — Aplicación MAGERIT v3

## Contexto del proyecto
- **Autor**: Matías Daniel Adés, consultor senior de ciberseguridad y GRC
- **Objetivo**: Construir portfolio práctico demostrable en automatización con IA aplicada a ciberseguridad y GRC
- **Uso inmediato**: Clase práctica en diplomatura de ciberseguridad sobre gestión de riesgos
- **Stack general del portfolio**: n8n + LLMs + agentes + MCP
- **Restricciones**: No reinventarse como AI Engineer. Aplicar IA siempre en dominios ya dominados (GRC, riesgo, seguridad). Cada proyecto debe ser defendible en entrevista.

## Estado actual
MVP v1 completado: análisis de riesgo inherente.

## Qué se construyó
La aplicación es una herramienta web educativa para análisis de riesgos basada en la metodología MAGERIT v3. Permite a los usuarios crear activos de tres tipos (Sistema Informático, Proveedor, Personas), configurar amenazas pre-cargadas para cada activo con probabilidades y degradaciones editables, y calcular riesgos inherentes por dimensión (Confidencialidad, Integridad, Disponibilidad). La interfaz consta de tres pantallas navegables: Activos (para gestión completa de activos con ABM), Amenazas (para configuración de amenazas por activo en formato de acordeones), y Resumen (tabla de riesgos con colores según nivel). Los cálculos se basan en fórmulas específicas: Impacto = Valor_activo × Degradación, Riesgo_dimensión = Impacto × Probabilidad, Riesgo_amenaza = max(Riesgo_C, Riesgo_I, Riesgo_D). La aplicación es completamente offline, responsive y en español, diseñada para proyección en clases de ciberseguridad.

## Stack técnico
- **Frontend**: HTML5, CSS3, JavaScript vanilla (sin frameworks ni dependencias externas).
- **Ejecución**: Archivo único `index.html` que funciona abriéndolo directamente en cualquier navegador moderno sin necesidad de servidor.
- **Persistencia**: Datos en memoria del navegador (sin almacenamiento local ni backend).
- **Estilo**: CSS inline para simplicidad y portabilidad.

## Decisiones de diseño tomadas
- **Tipos de activo**: Se implementaron 3 tipos (Sistema Informático, Proveedor, Personas) basados en MAGERIT v3, cubriendo activos tecnológicos, proveedores críticos y personal clave. Motivo: Simplificar para uso educativo sin abarcar todos los tipos posibles de MAGERIT.
- **Escalas de valoración**: Escala única cualitativa-numérica (Muy Alta=1.0, Alta=0.8, Media=0.6, Baja=0.3, Muy Baja=0.1, N/A=0) para activos, probabilidades y degradaciones. Motivo: Consistencia y facilidad de cálculo, alineado con MAGERIT pero simplificado para la aplicación.
- **Fórmulas de cálculo**: Impacto_dim = Valor_activo_dim × Degradación_dim, Riesgo_dim = Impacto_dim × Probabilidad, Riesgo_amenaza = max(Riesgo_C, Riesgo_I, Riesgo_D). Niveles de riesgo: 0.00-0.19=Leve (verde), 0.20-0.39=Medio (amarillo), 0.40-0.59=Moderado (naranja), 0.60-0.79=Alto (rojo), 0.80-1.00=Crítico (rojo oscuro). Motivo: Implementación directa de MAGERIT v3 para riesgo inherente, con colores para visualización clara.
- **Estructura de pantallas**: 3 pantallas secuenciales (Activos → Amenazas → Resumen) con navegación superior. Pantalla 1: Lista de activos con ABM (agregar/editar/eliminar), Pantalla 2: Amenazas expandibles por activo, Pantalla 3: Tabla de resumen. Motivo: Flujo lógico y educativo, permitiendo iteración sin perder datos.
- **Amenazas pre-cargadas**: Mínimo 5 por tipo, basadas en catálogo MAGERIT v3, editables. Motivo: Proporcionar base realista sin requerir input manual completo.
- **Diseño**: Profesional, sobrio, responsive, en español, legible a distancia. Header azul oscuro (#1a365d). Motivo: Apto para aulas, sin distracciones visuales.
- **ABM de activos**: Editar pre-carga todos los campos, eliminar con confirmación. Motivo: Usabilidad completa para gestión de datos.
- **Amenazas como acordeones**: Íconos ▶/▼, hover gris claro. Motivo: Claridad visual y espacio eficiente.

## Qué quedó fuera (próximas iteraciones)
- MVP v2: controles de seguridad + riesgo actual/residual
- MVP v3: riesgo objetivo + plan de tratamiento del riesgo (PTR)
- Futuro: logging, trazabilidad, modo auditor, integración con IA (LLM)

## Catálogo implementado
### Tipos de activo
- **Sistema Informático**: Cubre HW, SW, datos, red como un todo. Dimensiones: Confidencialidad (C), Integridad (I), Disponibilidad (D).
- **Proveedor**: Terceros prestadores de servicios críticos. Dimensiones: C, I, D.
- **Personas**: Equipos internos, roles críticos. Dimensiones: C, I, D.

### Amenazas por tipo
#### Sistema Informático
1. Acceso no autorizado | Prob: Alta (0.8) | C: Alta (0.8), I: Media (0.6), D: Baja (0.3)
2. Malware / Ransomware | Prob: Alta (0.8) | C: Media (0.6), I: Alta (0.8), D: Alta (0.8)
3. Fallo de hardware o software | Prob: Media (0.6) | C: N/A (0), I: Alta (0.8), D: Alta (0.8)
4. Error de configuración | Prob: Alta (0.8) | C: Media (0.6), I: Alta (0.8), D: Media (0.6)
5. Denegación de servicio (DoS) | Prob: Media (0.6) | C: N/A (0), I: N/A (0), D: Muy Alta (1.0)
6. Fuga de información | Prob: Media (0.6) | C: Muy Alta (1.0), I: N/A (0), D: N/A (0)
7. Ingeniería social / Phishing | Prob: Alta (0.8) | C: Alta (0.8), I: Media (0.6), D: Baja (0.3)

#### Proveedor
1. Interrupción del servicio del proveedor | Prob: Media (0.6) | C: N/A (0), I: N/A (0), D: Muy Alta (1.0)
2. Incumplimiento contractual | Prob: Baja (0.3) | C: Alta (0.8), I: Alta (0.8), D: Media (0.6)
3. Brecha de seguridad en proveedor | Prob: Media (0.6) | C: Muy Alta (1.0), I: Alta (0.8), D: Media (0.6)
4. Acceso indebido a datos por parte del proveedor | Prob: Baja (0.3) | C: Muy Alta (1.0), I: Media (0.6), D: N/A (0)
5. Falta de continuidad del proveedor | Prob: Baja (0.3) | C: N/A (0), I: N/A (0), D: Alta (0.8)
6. Dependencia tecnológica (lock-in) | Prob: Media (0.6) | C: N/A (0), I: Media (0.6), D: Alta (0.8)

#### Personas
1. Error humano | Prob: Alta (0.8) | C: Media (0.6), I: Alta (0.8), D: Media (0.6)
2. Fuga de información por el personal | Prob: Media (0.6) | C: Muy Alta (1.0), I: N/A (0), D: N/A (0)
3. Ingeniería social (víctima) | Prob: Alta (0.8) | C: Alta (0.8), I: Media (0.6), D: N/A (0)
4. Indisponibilidad del personal clave | Prob: Media (0.6) | C: N/A (0), I: N/A (0), D: Alta (0.8)
5. Abuso de privilegios | Prob: Media (0.6) | C: Alta (0.8), I: Alta (0.8), D: Media (0.6)
6. Acción malintencionada interna | Prob: Baja (0.3) | C: Muy Alta (1.0), I: Alta (0.8), D: Alta (0.8)

### Escalas
- **Valoración de activos, probabilidades y degradaciones**: Muy Alta = 1.0, Alta = 0.8, Media = 0.6, Baja = 0.3, Muy Baja = 0.1, N/A = 0
- **Niveles de riesgo**: 0.00-0.19 = Leve (verde), 0.20-0.39 = Medio (amarillo), 0.40-0.59 = Moderado (naranja), 0.60-0.79 = Alto (rojo), 0.80-1.00 = Crítico (rojo oscuro)

## Cómo arrancar una sesión nueva

### Mensaje de inicio para C-laude.ai
Copiar y pegar esto al abrir una conversación nueva en el proyecto:

---
Continuamos con MVP v2 de la aplicación MAGERIT. Adjunto el DECISIONS.md con el estado actual.

Metodología de trabajo:
- Este chat es para diseño y decisiones. Claude Code en VS Code es para construir.
- Yo no toco código. Vos me generás los prompts exactos para pasarle a Claude Code.
- Antes de generar cualquier prompt, conversamos el diseño hasta cerrarlo completamente.

Para arrancar: leé el DECISIONS.md, confirmame que entendiste el estado actual, y luego preguntame lo que necesites saber sobre la v2 antes de proponer nada.
---

### Setup técnico
- Archivo: `index.html` en `C:\Users\arcticeye\magerit-app\`
- Abrir en navegador: doble click sobre el archivo
- Editor: VS Code con extensión Claude Code (modo Agent)
- No requiere servidor ni dependencias

### Archivos del proyecto
- `index.html` — aplicación completa
- `DECISIONS.md` — este archivo, adjuntarlo en cada sesión nueva</content>
<parameter name="filePath">c:\Users\arcticeye\magerit-app\DECISIONS.md