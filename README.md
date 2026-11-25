# INGENIERIA-DE-SOLUCIONES-CON-INTELIGENCIA-ARTIFICIAL_evaluacion3

# Observabilidad y Seguridad en Agente de Soporte Estudiantil (v1 vs v2)

**Asignatura:** Ingeniería de Soluciones con IA (ISY1010)  
**Institución:** Duoc UC (Caso Inventado: Instituto Técnico Valle Central)  
**Integrante:** Nicolas San Martin  
## Descripción del Proyecto
Este repositorio contiene la instrumentación, auditoría de seguridad y rediseño arquitectónico de un agente conversacional de soporte estudiantil desarrollado con **LangChain** y **OpenAI (`gpt-4o-mini` vía OpenRouter)**.

El proyecto evalúa una **Versión 1 (Línea Base)** en producción para detectar cuellos de botella de latencia y vulnerabilidades críticas (OWASP Top 10 for LLM Applications y cumplimiento de la Ley N° 21.719 de Protección de Datos Personales en Chile), validando con evidencia empírica las mejoras aplicadas en una **Versión 2 (Rediseño Determinista)**

##Arquitectura y Mejoras Implementadas
### Versión 1 (Línea Base)
* Agente ReAct con 4 herramientas (`consultar_situacion_academica`, `consultar_notas`, `emitir_certificado`, `anular_matricula`).
* Telemetría en dos niveles correlacionados por `trace_id`:
  * `eventos_v1.log`: Logs estructurados en JSON.
  * `trazas_v1.jsonl`: Trazabilidad granular de spans (LLM y herramientas) mediante `BaseCallbackHandler`.
* **Hallazgos detectados:** Latencia p95 elevada (4.42 s), exposición de 35 RUT en texto plano en logs, vulnerabilidad IDOR por recepción de `estudiante_id` como parámetro libre y capacidad de anulación directa de matrícula sin supervisión humana.
### Versión 2 (Rediseño Seguro y Resiliente)
* **Privacidad (Ley N° 21.719):** Filtro de enmascaramiento (`FiltroPII`) mediante expresiones regulares para ocultar RUT y correos electrónicos antes de enviar datos al LLM o escribir en logs.
* **Control de Acceso por Sesión (OWASP LLM06):** Eliminación del parámetro `estudiante_id` en las herramientas; el identificador se obtiene directamente del contexto autenticado (`contextvars`).
* **Supervisión Humana (*Human-in-the-Loop*):** Reemplazo de `anular_matricula` por `derivar_a_coordinacion` mediante generación de tickets.
* **Defensa contra Prompt Injection (OWASP LLM01):** Sanitización de registros de base de datos (`limpiar_dato`), delimitación de contexto con etiquetas XML (`<sesion>`) y salida estructurada estricta con **Pydantic**.
* **Optimización de Latencia:** Precarga de sesión, caché en memoria con TTL de 60 segundos y *timeouts* (1.6 s) con reintentos automáticos.

## Resultados Comparativos (Batch de 16 escenarios x 2 reps + Red Team)
| Métrica Evaluada | Versión 1 (Base) | Versión 2 (Rediseño) | Variación |
| :--- | :---: | :---: | :---: |
| **Precisión Global** | 88 % | 78 % | -10 % |
| **Consistencia** | 100 % | 81 % | -19 % |
| **Tasa de Error (Herramientas / JSON)** | 6 % | 0 % | -6 % |
| **Latencia p50 (Mediana)** | 2.19 s | 1.78 s | -18.7 % |
| **Latencia p95 (Cola)** | 4.42 s | 3.60 s | -18.6 % |
| **Llamadas LLM por solicitud** | 1.56 | 1.28 | -17.9 % |
| **Tokens promedio por solicitud** | 816 | 798 | -2.2 % |
| **Ataques Red Team exitosos** | 2 de 10 | 0 de 10 | **Mitigado (100%)** |
| **Apariciones de PII (RUT) en logs** | 35 | 0 | **Mitigado (100%)** |

## Cómo ejecutar el proyecto
1. Abrir el archivo `.ipynb` en **Google Colab**.
2. Configurar el secreto `OPENROUTER_API_KEY` en el panel de **Secrets** (ícono de llave en el menú izquierdo de Colab).
3. Ejecutar todas las celdas secuencialmente (`Entorno de ejecución -> Ejecutar todas`).
