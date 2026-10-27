# Agente Clínico de Apoyo para SAPU — EP2 ISY0101

**Asignatura:** ISY0101 Ingeniería de Soluciones con Inteligencia Artificial (Duoc UC)
**Integrantes:** Cristian Valenzuela · Martin Chavez · Demian Hurtubia
**Evaluación:** Parcial N°2 — Desarrollo de un agente funcional (continuación del caso de la EP1)

## 1. Descripción del caso

Agente de apoyo para el personal de urgencia de un **SAPU** (Servicio de Atención Primaria de Urgencia). Responde consultas en lenguaje natural sobre protocolos, vademécum y criterios de derivación, **citando documento y página**, sin inventar información y sin procesar datos identificables de pacientes (Ley N° 20.584). Es una herramienta de apoyo: **toda salida requiere validación del médico a cargo**.

## 2. Arquitectura general

```
                     ┌───────────────────────────┐
 Consulta  ────────► │ Guardrail (bloquea RUT)   │
                     └─────────────┬─────────────┘
                                   ▼
        ┌─────────────── Planificador LLM (IL2.3) ───────────────┐
        │  plan JSON → validador → priorizador → ejecutor        │
        │  reglas adaptativas (evento / fallo → re-planificar)   │
        └───────┬──────────────┬───────────────┬─────────────────┘
                ▼              ▼               ▼
        Herramientas de   Razonamiento     Escritura
          consulta        calcular_dosis   registrar_derivacion
        buscar_protocolo  (mg/kg + tope)   (ficha JSON)
        consultar_vademecum
                ▲
                │ embeddings (mistral-embed) + FAISS
        Base documental (guías, vademécum, protocolos)

   Memoria (IL2.2):  corto plazo = ventana de conversación (k=4)
                     largo plazo = bitácora del turno en FAISS persistido en disco
```

## 3. Contenido del repositorio

| Archivo | Indicador | Qué demuestra |
|---|---|---|
| `notebooks/agente_sapu_IL2_1.ipynb` | IL2.1 (IE1, IE2) | Agente LangChain con 4 herramientas: 2 de consulta, 1 de razonamiento, 1 de escritura |
| `notebooks/agente_sapu_IL2_2.ipynb` | IL2.2 (IE3, IE4) | Memoria de corto plazo, bitácora persistente y recuperación semántica de casos previos |
| `notebooks/agente_sapu_IL2_3.ipynb` | IL2.3 (IE5) | Planificación (plan → validación → prioridad → ejecución) y re-planificación adaptativa |
| `evidencia/` | — | Capturas de las pruebas ejecutadas |
| `requirements.txt` · `.env.example` | — | Dependencias y variables de entorno |

Cada notebook es **autónomo**: incluye su configuración, base documental y herramientas.

## 4. Cómo ejecutar

### Opción A — Google Colab (recomendada)
1. Abre el notebook en Colab.
2. En el panel 🔑 **Secrets**, crea `LLM_API_KEY` con tu clave de [Mistral](https://console.mistral.ai/) y actívalo para el notebook. (`LLM_BASE_URL` y `LLM_MODEL` son opcionales; tienen valor por defecto.)
3. Ejecuta todas las celdas en orden (**Entorno de ejecución → Ejecutar todas**). La primera celda instala las dependencias.

### Opción B — Local
```bash
git clone <URL_DEL_REPOSITORIO>
cd <carpeta>
python -m venv .venv && source .venv/bin/activate      # Windows: .venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env                                    # y completa LLM_API_KEY
jupyter notebook notebooks/
```

## 5. Pruebas incluidas

| Notebook | Prueba | Qué verifica |
|---|---|---|
| IL2.1 | P1–P4 | Consulta + cálculo de dosis; consulta + escritura de derivación; fármaco inexistente (no inventa); bloqueo de RUT |
| IL2.2 | P1–P5 | Seguimiento en corto plazo; nota y derivación guardadas; límite de la ventana; **cambio de turno** (recarga de memoria desde disco); bloqueo de RUT en memoria |
| IL2.3 | E1–E4 | Flujo normal; **empeoramiento del paciente** (re-planifica y prioriza derivación); información faltante (escala al médico); plan inválido rechazado |

Las capturas de cada prueba están en `evidencia/`.

## 6. Decisiones de diseño

> *Sección a completar por el equipo con sus propias justificaciones técnicas (por qué LangChain, FAISS, Mistral, la ventana k=4, etc.).*

## 7. Limitaciones conocidas

- Los documentos clínicos de los notebooks son de **DEMO** (valores ilustrativos). Para uso real deben reemplazarse por las guías y normas oficiales del MINSAL/SAPU.
- Las reglas de clasificación ESI del notebook IL2.3 son simplificadas (DEMO).
- El guardrail detecta **RUT**, pero no nombres escritos libremente; en un entorno real se requiere detección de nombres (NER).
- En Google Colab, las carpetas `memoria_turno/` y `derivaciones/` se pierden al reiniciar el entorno.
- No existe garantía absoluta de cero alucinaciones: el sistema las reduce con fuentes, restricciones y validación humana obligatoria.

## 8. Uso de IA

Se utilizó Claude (Anthropic) como apoyo para generar el código base de los notebooks y este README. El equipo revisó y validó el contenido. Las justificaciones técnicas y las reflexiones individuales son del equipo.

## 9. Referencias

- LangChain. (s.f.). *LangChain Documentation*. https://python.langchain.com/
- Mistral AI. (s.f.). *Mistral AI Documentation*. https://docs.mistral.ai/
- Johnson, J., Douze, M., & Jégou, H. (2019). Billion-scale similarity search with GPUs. *IEEE Transactions on Big Data*.
- Lewis, P., et al. (2020). Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks. *Advances in Neural Information Processing Systems, 33*.
- Ministerio de Salud de Chile (MINSAL). (s.f.). *Guías Clínicas y Normas Técnicas de Urgencia*.
- Ley N° 20.584. (2012). Regula los derechos y deberes de las personas en relación con acciones vinculadas a su atención en salud.
