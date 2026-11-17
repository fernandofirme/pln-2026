# RAG de normativa de tránsito argentina

Sistema de preguntas y respuestas sobre la Ley Nacional de Tránsito 24.449, su Decreto reglamentario 779/1995, el Decreto 242/2022 (scoring) y el anexo nexo 2 del Decreto 779/1995 (régimen de infracciones, multas y puntos del scoring). Primero recupera fragmentos de los documentos y después genera la respuesta con un modelo local (Ollama). La interfaz Gradio muestra por separado la **respuesta** y los **fragmentos que recibió el modelo**, con archivo, página, posición dentro de la página y distancia.

**Trabajo Integrador 2 · Laboratorio de PLN · IFTS N.º 24** · Entrega individual.

## 1. Problema y destinatario

Este sistema permite que una persona que desea conocer aspectos básicos de la Ley de Tránsito en la República Argentina consulte distintos aspectos como requisitos de licencia, reglas de circulación y puntos que se restan por una falta en la Ley 24.449, el Decreto 779/1995, el Anexo 2 del Decreto 779/1995 y el Decreto 242/2022 para encontrar, además de la respuesta, qué norma respalda a cada una de ellas.

**Qué puede resolver:** 
Preguntas del tipo 
- "¿Cuantos puntos de scoring se reducen de un conductor por pasar un semáforo con luz roja?", o
- "¿Quién tiene prioridad de paso en las rotondas?"

**Qué no puede resolver:** 
Valores actuales en pesos de las multas, solo su equivalentes en U.F. (Unidades Fijas), distancias específicas de estacionamiento respecto al cordón.
Normativas provinciales, o actualizaciones posteriores a la fecha de acceso de cada norma.
Asesoramiento legal o disposiciones locales como domicilios de destacamentos oficiales.

## 2. Corpus

Los documentos están en la carpeta `datos/`, versionados en este repositorio.

| Archivo en `datos/` | Nombre original descargado | Organismo | Procedencia (URL) | Versión / fecha de acceso | Condiciones de uso | Páginas |
| --- | --- | --- | --- | --- | --- | --- |
| `ley_24449_transito.pdf` | Ley Nacional de Transito N 24449.pdf | Honorable Congreso de la Nación | https://www.argentina.gob.ar/normativa/nacional/norma-818/actualizacion | Texto actualizado. Acceso: 11/11/2026 | Creative Commons BY 4.0 según los Términos y condiciones de Argentina.gob.ar; copia sin modificaciones del texto, guardada como PDF desde el navegador | 69 |
| `decreto_779_1995_reglamentario.pdf` | Decreto Reglamentario 779_1995 - TRANSITO Y SEGURIDAD VIAL _ Argentina.gob.ar.pdf | Poder Ejecutivo Nacional | https://www.argentina.gob.ar/normativa/nacional/norma-30389/actualizacion | Texto actualizado. Acceso: 11/11/2026 | Creative Commons BY 4.0 según los Términos y condiciones de Argentina.gob.ar; copia sin modificaciones del texto, guardada como PDF desde el navegador | 5 |
| `anexo2_decreto_779_1995.pdf` | ANEXO 2 - Decreto Reglamentario 779_1995 - TRANSITO Y SEGURIDAD VIAL _ Argentina.gob.ar.pdf | Poder Ejecutivo Nacional | https://www.boletinoficial.gob.ar/detalleAviso/primera/261911/20220506?anexos=1 (publicación oficial del Decreto 242/2022, que sustituyó el Anexo 2). El PDF se guardó desde Argentina.gob.ar; la dirección exacta ya no está disponible al 16/11/2026 | Texto actualizado. Acceso: 13/11/2026 |  Texto normativo de publicación oficial. El PDF se guardó desde Argentina.gob.ar, cuyos términos indican CC BY 4.0; copia sin modificaciones del texto, guardada como PDF desde el navegador | 16 |
| `decreto_242_2022_scoring.pdf` | Decreto Reglamentario 242_2022 - LEY DE TRANSITO- Scoring_ Argentina.gob.ar.pdf | Poder Ejecutivo Nacional | https://www.boletinoficial.gob.ar/detalleAviso/primera/261911/20220506 (publicación oficial, B.O. 06/05/2022). El PDF se guardó desde https://www.argentina.gob.ar/normativa/nacional/decreto-242-2022-364297, que ya no está disponible. | Texto actualizado. Acceso: 11/11/2026 | Texto normativo de publicación oficial. El PDF se guardó desde Argentina.gob.ar, cuyos términos indican CC BY 4.0; copia sin modificaciones del texto, guardada como PDF desde el navegador | 4 |

### Notas sobre el corpus 

* **Texto extraíble:** los cuatro PDF tienen capa de texto (ninguna página salió vacía; el mínimo fue de 132 caracteres) y no se necesitó OCR. 
Páginas comparadas con el texto extraído:
- ley_24449_transito.pdf: pág. 68. El contenido es correcto, pero el título «Antecedentes normativos» sale al final del texto.
- decreto_779_1995_reglamentario.pdf: pág. 2. El contenido es correcto
- decreto_242_2022_scoring.pdf: pág. 2. El contenido es correcto
- anexo2_decreto_779_1995.pdf: pág. 1. El contenido es correcto, pero el encabezado "Sanción:
20-11-1995
Publicada en el Boletín Oficial" sale al final del texto.

* **Qué contiene cada PDF:**
  * `ley_24449_transito.pdf` (69 págs.): texto de la Ley 24.449 actualizado, con notas de modificación y enlaces a otras normas. Las páginas finales traen los «Antecedentes normativos».
  * `decreto_779_1995_reglamentario.pdf` (5 págs.): cuerpo del decreto que aprueba la reglamentación de la Ley (artículos 1° a 7°) y el menú de anexos de la web como texto. No incluye el contenido del Anexo 1 ni de los demás anexos.
  * `anexo2_decreto_779_1995.pdf` (16 págs.): régimen de infracciones, con las multas en UF y los puntos que se restan de la licencia por cada falta.
  * `decreto_242_2022_scoring.pdf` (4 págs.): Decreto 242/2022, que modifica los Decretos 779/1995 y 437/2011 según su encabezado.
* **Versiones:** 
- `ley_24449_transito.pdf`: texto actualizado
- `decreto_779_1995_reglamentario.pdf`: texto actualizado
- `anexo2_decreto_779_1995.pdf`: texto actualizado
- `decreto_242_2022_scoring.pdf`: texto actualizado

* **Numeración de páginas:** las citas del sistema usan la página del archivo PDF contada desde 1 (`metadata["page"] + 1`), que puede no coincidir con la numeración impresa en el documento.

## 3. Arquitectura

```text
datos/*.pdf (versionados en GitHub)
  → cargar_documentos(): PyPDFLoader (un Document por página, con archivo y página)
  → preparar_indice(): RecursiveCharacterTextSplitter (add_start_index) → embeddings bge-m3 (Ollama) → Chroma
pregunta → consultar(): embedding con el mismo modelo → similarity_search_with_score (k)
  → contexto "[n] archivo, página" → ChatPromptTemplate → granite4:latest (Ollama)
  → (respuesta, fragmentos) → interfaz Gradio
```

Todo el flujo está en `app.py`.

| Componente | Herramienta | Versión |
| --- | --- | --- |
| Python | 3.12 | 3.12.14 |
| Carga de PDF | `PyPDFLoader` (`langchain-community`, `pypdf`) | langchain-community 0.4.2, pypdf 7.1.0 |
| Fragmentación | `RecursiveCharacterTextSplitter` (`langchain-text-splitters`) | langchain-text-splitters 1.1.3 |
| Embeddings | `bge-m3` vía `OllamaEmbeddings` (`langchain-ollama`) | bge-m3:latest (ID 790764642607), langchain-ollama 1.1.0 |
| Índice vectorial | `Chroma` (`langchain-chroma`) | langchain-chroma 1.1.0 (con chromadb 1.5.9) |
| Generación | `granite4:latest` vía `ChatOllama` | granite4:latest (ID 4235724a127c) |
| Prompt | `ChatPromptTemplate` (`langchain-core`) | langchain-core 1.8.0 |
| Interfaz | `gradio` | gradio 6.32.0 |


## 4. Instalación local

**Requisitos:** Python 3.12, [uv](https://docs.astral.sh/uv/) y [Ollama](https://ollama.com/) instalado y en ejecución.

Estructura esperada del proyecto:

```text
.
├── app.py
├── exploracion_y_pruebas.ipynb
├── requirements.txt
├── README.md
├── .gitignore
└── datos/
    ├── ley_24449_transito.pdf
    ├── decreto_779_1995_reglamentario.pdf
    ├── anexo2_decreto_779_1995.pdf
    └── decreto_242_2022_scoring.pdf
```

Desde la carpeta del proyecto:

1. Crear el entorno e instalar las dependencias:

   ```bash
   uv venv --python 3.12
   uv pip install -r requirements.txt
   ```

2. Descargar los modelos de Ollama:

   ```bash
   ollama pull granite4:latest
   ollama pull bge-m3
   ```

3. Activar el entorno:

   ```bash
   # Windows (PowerShell)
   .venv\Scripts\activate

   # macOS / Linux
   source .venv/bin/activate
   ```

   Si PowerShell bloquea la activación: `Set-ExecutionPolicy -Scope Process -ExecutionPolicy RemoteSigned` y repetir.

4. Ejecutar la aplicación:

   ```bash
   python app.py
   ```

   La **primera ejecución** crea el índice a partir de los PDF de `datos/` y puede tardar: en el equipo donde se probó (94 páginas, 280 fragmentos) demoró unos 9 minutos hasta mostrar la dirección local. Después queda guardado y las siguientes ejecuciones lo reutilizan. Gradio abre por defecto en `http://127.0.0.1:7860`. Cuando la consola muestra esa dirección, la aplicación está lista: la terminal queda ocupada mientras esté abierta y se cierra con `Ctrl+C`.

**Si algo falla**

* Error de conexión con Ollama: comprobar que esté en ejecución y que `ollama list` muestre `granite4:latest` y `bge-m3`.
* `FileNotFoundError` al indexar: los nombres de los archivos de `datos/` deben coincidir exactamente con la lista `ARCHIVOS` de `app.py`.

**Configuración opcional (`.env`):** `app.py` usa `granite4:latest` y `bge-m3` por defecto. Para usar otros modelos se puede crear un `.env` en la raíz con `OLLAMA_MODEL=...` y `OLLAMA_EMBEDDING_MODEL=...`. El `.env` no se publica.

## 5. Reindexación

* El índice se guarda en `chroma_db_bge_m3/`, carpeta generada que no se publica.
* El nombre de la colección incluye el modelo de embeddings y el corte (por ejemplo `transito_bge_m3_1000_200`). Si se cambian `TAMANO_FRAGMENTO`, `SOLAPAMIENTO` o el modelo de embeddings, se crea una colección nueva sin borrar nada. Cambiar `K` no requiere reindexar.
* Si se **modifican o reemplazan los PDF**, o se **agregan o quitan archivos de la lista `ARCHIVOS`**, hay que reconstruir el índice, porque `app.py` no reindexa por sí solo una colección que ya tiene fragmentos:

  ```bash
  python app.py --reindexar
  ```

  Ese comando borra solo la colección de esta aplicación y la crea de nuevo a partir de `datos/`. Si hiciera falta empezar de cero, también se puede cerrar la aplicación y borrar la carpeta `chroma_db_bge_m3/`.

* No se mezclan vectores de modelos distintos: la consulta usa el mismo modelo que el índice.
* No se publican `.venv/`, `.env` ni `chroma_db_bge_m3/` (están en `.gitignore`).

## 6. Pruebas

Las pruebas se ejecutan con `exploracion_y_pruebas.ipynb`, que importa `app.py` y usa sus mismas funciones, de modo que prueba el sistema real. El cuaderno también contiene la revisión de la extracción y la comparación de cortes citadas en las secciones 2 y 7.

Las preguntas se escribieron antes de evaluar el sistema; la fuente esperada (artículo y página) se precisó al verificarla contra los PDF después de la corrida; la tabla sale de una corrida completa del cuaderno.

| Caso | Pregunta | Fuente esperada | Fragmento recuperado | Respuesta del modelo | Qué observé |
| --- | --- | --- | --- | --- | --- |
| Respondible 1 | Quién tiene prioridad en las rotondas? | Ley 24.449, Artículo 43 (inciso e), página 31. Establece que tiene prioridad de paso el que circula por la rotonda sobre el que intenta ingresar, y debe ceder el paso el que egresa. | ley_24449_transito.pdf p. 29; ley_24449_transito.pdf p. 31; ley_24449_transito.pdf p. 30 | En las rotondas, la prioridad de paso es para el que circula por ella sobre el que intenta ingresar, debiendo ceder la zona central al que egresa, salvo señalización en contrario. ([2]) | Correcto. El fragmento esperado llegó en 2.º lugar y la respuesta lo cita bien ([2]) |
| Respondible 2 | Cuándo entró en vigencia la ley? | decreto_779_1995_reglamentario.pdf, página 4 | decreto_779_1995_reglamentario.pdf p. 4; ley_24449_transito.pdf p. 68; ley_24449_transito.pdf p. 69 | La Ley N° 24.449 y su reglamentación entraron en vigencia el 1° de Diciembre de 1995 ([1], [2]). | Correcto. El fragmento esperado llegó en 1.º lugar y la respuesta coincide con el art. 2°. Cita también [2] (art. 94 de la Ley), que no da la fecha. |
| Respondible 3 | Sobre quién se aplica el descuento de puntos del scoring en una infracción? | decreto_242_2022_scoring.pdf, art. 2° (página 4): la deducción de puntos se aplica cuando el conductor profesional es también titular registral del dominio | decreto_242_2022_scoring.pdf p. 3; ley_24449_transito.pdf p. 61; anexo2_decreto_779_1995.pdf p. 10 | Sobre quienes son sancionados con multas y descuentos de puntos en la Licencia Nacional de Conducir por infracciones a la Ley de Tránsito se aplica el descuento de puntos del scoring en una infracción. ([1], [2], [3]) | Falla. El fragmento esperado (art. 2°, pág. 4) no llegó: se recuperó la pág. 3 del Decreto 242 y otras dos páginas que no responden. La respuesta es circular: no da el dato y cita los tres fragmentos. Pasos: recuperación y generación. |
| Respondible 4 | Cual es la cantidad de UF a descontar por cruzar cuando el semaforo está en rojo? | anexo2_decreto_779_1995.pdf, art. 57 (página 8): 5 puntos por cruzar con el semáforo en rojo | anexo2_decreto_779_1995.pdf p. 8; anexo2_decreto_779_1995.pdf p. 8; anexo2_decreto_779_1995.pdf p. 8 | La cantidad de UF a descontar por cruzar cuando el semáforo está en rojo es de CINCO (5) puntos. [2] | Correcto en el dato (5 puntos). La pregunta estaba mal formulada: pide UF, pero lo que se descuenta son puntos. El modelo respondió con los puntos, aunque repitió la palabra «UF». Citó [2] en lugar de [1], que es donde está el art. 57. |
| Sin respuesta 1 | ¿Cuánto cuesta renovar la licencia de conducir? | No figura en el corpus | anexo2_decreto_779_1995.pdf p. 1; anexo2_decreto_779_1995.pdf p. 2; anexo2_decreto_779_1995.pdf p. 4 | No encuentro esa información en los documentos. | Correcto. Se abstuvo con la frase exacta; los fragmentos recuperados no contienen el dato |
| Sin respuesta 2 | ¿Qué porcentaje de descuento se aplica por pago voluntario de infracciones en la provincia de Córdoba? | No figura en el corpus | ley_24449_transito.pdf p. 63; ley_24449_transito.pdf p. 63; ley_24449_transito.pdf p. 61 | No encuentro esa información en los documentos. | Correcto. Se abstuvo con la frase exacta; los fragmentos recuperados no contienen el dato |


### Limitación observada

En una prueba exploratoria y que no forma parte de la tabla: se preguntó: ¿cuál es el valor en pesos de una UF hoy? 
Qué pasó: el sistema no se abstuvo y presentó la definición de la UF (art. 84) como si fuera el valor, citando tres fragmentos de los que solo uno la respalda. 
Paso: generación; la recuperación fue correcta, porque ningún fragmento trae un valor en pesos. 
Qué cambiaría: reforzar la instrucción de abstención en el prompt para el caso en que el contexto menciona el tema pero no da el dato exacto, o agregar un umbral de distancia, y repetir las pruebas.

Este conjunto de pruebas muestra el funcionamiento del recorrido; no mide la precisión general del sistema.

## 7. Decisiones y límites

* **Fragmentación:** `RecursiveCharacterTextSplitter` con chunk_size = 1000 y chunk_overlap = 200, medidos en caracteres. Se compararon dos cortes sobre el corpus completo, con la misma pregunta e índices temporales en memoria: 500/100 (560 fragmentos) y 1000/200 (280 fragmentos). Pregunta: «En una encrucijada de calles, ¿qué vehículo tiene prioridad de paso?». En ambos cortes el primer fragmento es el comienzo del art. 41 de la Ley (pág. 29), con la regla de la derecha. Con 500/100 ese fragmento termina en la excepción c), y los fragmentos 2 y 3 son del art. 44 (semáforos) y de la regla de rotondas (art. 43, inciso e), sin las demás excepciones. Con 1000/200 el fragmento 2 es la pág. 30, que trae las excepciones finales del art. 41 (inciso g) y el orden entre ellas. Se eligió 1000/200 porque el art. 41 enumera las excepciones a lo largo de varios incisos y el corte breve deja afuera parte de ellas. Además, 500/100 duplica los fragmentos y, por eso, la indexación demora más, y con k = 3 le da al modelo la mitad de evidencia. La comparación se hizo con una sola pregunta: muestra un caso y no mide la calidad general.
* **Recuperación (`k`):** `k = 3` fragmentos por consulta.
Se probó k = 1, 3 y 5 con la pregunta del Decreto 242 (caso 3). Con k = 5 el fragmento esperado (art. 2°, pág. 4) tampoco llegó: los fragmentos 4 y 5 fueron del Anexo 2 (págs. 10 y 9) y no responden la pregunta. Subir k no resolvió la falla, que es de recuperación, y solo suma texto irrelevante, por eso se mantuvo k = 3. Posible causa: la pregunta no usa los términos del artículo (conductor profesional, titular registral)
* **Embeddings:** `bge-m3` en Ollama, tanto para indexar como para consultar.
* **Prompt:** el generador (`granite4:latest`, `temperature=0`) debe responder solo con el contexto, indicar la referencia `[n]` y responder exactamente «No encuentro esa información en los documentos.» cuando el dato no esté. `app.py` además reemplaza una respuesta vacía o que sea solo una referencia, y quita las citas de una abstención.
* **Fuentes:** la interfaz muestra los fragmentos que entraron al contexto, con archivo, página, posición inicial del fragmento dentro de la página (en caracteres) y distancia. La distancia sirve para ordenar, no es una probabilidad. Que un fragmento aparezca no prueba que el modelo lo haya interpretado bien.
* **Abstención:** el sistema la solicita en el prompt, pero no está garantizada: Chroma siempre devuelve vecinos, aunque ninguno responda la pregunta. Pruebas sin respuesta de la tabla:
- ¿Cuánto cuesta renovar la licencia de conducir? → Se abstuvo
- ¿Qué porcentaje de descuento se aplica por pago voluntario de infracciones en la provincia de Córdoba? → Se abstuvo
En pruebas exploratorias previas, el sistema  no se abstuvo con una pregunta directa sobre el valor en pesos de la UF (ver «Limitación observada», sección 6).

**Límites conocidos**

* La carga genera un `Document` por página, por lo que ningún fragmento cruza un salto de página. Una condición que continúa en la página siguiente queda partida.
* El texto indexado es el extraído del PDF, sin limpieza: encabezados, pies, notas y enlaces que contenga el PDF también se indexan. 
* Extracción: PyPDFLoader conserva el contenido, pero algunos bloques salen en otro orden que el visual. En "anexo2_decreto_779_1995" los datos de cabecera (fecha de sanción) aparecen al final de la página, y en la pág. 68 de la Ley el título «Antecedentes normativos» queda al final. En las páginas de artículos revisadas el orden es correcto.
* Enlaces: los PDF tienen hipervínculos funcionales a otras normas, pero la extracción toma solo el texto visible (por ejemplo, una ruta relativa), no el destino. Las normas enlazadas, como las leyes modificatorias, no están en el corpus.
* Notas dentro del texto: las notas de modificación y de veto, con sus enlaces, se indexan junto a los artículos y ocupan espacio en los fragmentos.
* Corpus: el PDF del Decreto 779/1995 trae el cuerpo del decreto y el menú de anexos de la web como texto, sin el Anexo 1 (reglamentación general) ni los demás anexos. Las preguntas que dependan de ese detalle no tienen respuesta en el corpus. Tampoco incluye normativa provincial ni de la Ciudad de Buenos Aires.
* Citas: el modelo puede citar fragmentos que no respaldan lo que afirma (por ejemplo, citó los tres fragmentos cuando solo uno servía). Cada cita debe contrastarse con el fragmento y con el documento original.
* Rendimiento: en el equipo de prueba, cada consulta tarda entre 40 y 65 segundos, y la primera indexación unos 9 minutos.

## 8. Uso de IA

* **Para qué se usó:** Entender la consigna y los cuadernos 04 y 07, generar `app.py` según lo evaluado en esos cuadernos, apoyo a la generación del cuaderno `exploracion_y_pruebas.ipynb` y `app.py`, y redactar borradores de `requirements.txt` y de este README.
* **En qué momentos:** etapas de programación de app.py, apoyo en confección y evaluación de resultados de pruebas.
* **Modelos:** claude.ai, chatGPT, Gemini.
* **Qué verifiqué personalmente:** selección de elementos del corpus, flujo de pasos para la resolución de trabajo práctico, comparación de páginas de los PDF con el texto extraído, ejecución desde una carpeta nueva siguiendo este README, revisión de las respuestas de las pruebas contra los PDF originales.
