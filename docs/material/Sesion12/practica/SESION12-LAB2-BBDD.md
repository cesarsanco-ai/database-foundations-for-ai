{
  "nbformat": 4,
  "nbformat_minor": 0,
  "metadata": {
    "colab": {
      "provenance": [],
      "include_colab_link": true,
      "name": "Semana12_NB2_RAG_Pinecone_LangChain.ipynb"
    },
    "kernelspec": {
      "display_name": "Python 3",
      "name": "python3"
    },
    "language_info": {
      "name": "python",
      "version": "3.10.12"
    }
  },
  "cells": [
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "a1b2c3d4e5f6"
      },
      "source": [
        "# **Semana 12: RAG con Pinecone y LangChain (NB2 - Práctico/Ejercicios)**\n",
        "\n",
        "## **Propósito de la Sesión**\n",
        "Construir un pipeline completo de Retrieval Augmented Generation (RAG) utilizando Pinecone como base de datos vectorial en la nube, LangChain como orquestador, y un modelo de lenguaje (simulado con OpenAI o Hugging Face). Aprenderemos a crear índices, cargar documentos, dividirlos en fragmentos, generar embeddings y realizar preguntas sobre el contenido."
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "b2c3d4e5f6a7"
      },
      "source": [
        "### **Objetivos de Aprendizaje**\n",
        "Al finalizar este notebook, el estudiante será capaz de:\n",
        "1. **Crear** un índice gratuito en Pinecone.\n",
        "2. **Cargar documentos** (PDF o texto) y dividirlos en chunks.\n",
        "3. **Generar embeddings** de los chunks usando sentence-transformers.\n",
        "4. **Almacenar** los vectores en Pinecone con sus metadatos.\n",
        "5. **Realizar búsquedas** semánticas sobre los documentos.\n",
        "6. **Construir un pipeline RAG** que recupere contexto y genere respuestas.\n",
        "7. **Comparar** ChromaDB (local) vs Pinecone (nube) en términos de uso y limitaciones."
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "c3d4e5f6a7b8"
      },
      "source": [
        "## **Configuración Inicial**\n",
        "\n",
        "Instalaremos las librerías necesarias: pinecone-client, langchain, sentence-transformers, y otras utilidades."
      ]
    },
    {
      "cell_type": "code",
      "execution_count": null,
      "metadata": {
        "id": "d4e5f6a7b8c9"
      },
      "outputs": [],
      "source": [
        "# Instalación de librerías\n",
        "!pip install pinecone-client langchain langchain-community sentence-transformers pypdf tiktoken --quiet\n",
        "\n",
        "# Importaciones\n",
        "import pinecone\n",
        "from langchain.document_loaders import PyPDFLoader, TextLoader\n",
        "from langchain.text_splitter import RecursiveCharacterTextSplitter\n",
        "from langchain.embeddings import HuggingFaceEmbeddings\n",
        "from langchain.vectorstores import Pinecone as LangchainPinecone\n",
        "from langchain.llms import OpenAI\n",
        "from langchain.chains import RetrievalQA\n",
        "from langchain.prompts import PromptTemplate\n",
        "import os\n",
        "import time\n",
        "import pandas as pd\n",
        "import matplotlib.pyplot as plt\n",
        "import seaborn as sns\n",
        "from getpass import getpass\n",
        "\n",
        "# Configuración de visualización\n",
        "sns.set_style(\"whitegrid\")\n",
        "pd.set_option('display.max_colwidth', 200)\n",
        "\n",
        "print(\"Librerías instaladas e importadas correctamente.\")"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "e5f6a7b8c9d0"
      },
      "source": [
        "---\n",
        "## **Parte 1: Creación de Índice Gratuito en Pinecone**\n",
        "\n",
        "### **¿Qué es Pinecone?**\n",
        "\n",
        "Pinecone es una base de datos vectorial gestionada en la nube. Ofrece un plan gratuito con 1 índice y hasta 100k vectores de 384 dimensiones. Es ideal para prototipos y proyectos pequeños.\n",
        "\n",
        "### **Pasos para crear un índice:**\n",
        "\n",
        "1. **Registro**: Ve a [https://www.pinecone.io/](https://www.pinecone.io/) y crea una cuenta gratuita.\n",
        "2. **Obtener API Key**: En el dashboard, copia tu API Key.\n",
        "3. **Crear índice**:\n",
        "   *   Haz clic en \"Create Index\".\n",
        "   *   Nombre: `rag-curso` (o el que prefieras).\n",
        "   *   Dimensiones: `384` (para all-MiniLM-L6-v2).\n",
        "   *   Métrica: `cosine` (similitud de coseno).\n",
        "   *   Tipo de pod: `Starter` (gratuito).\n",
        "   *   Haz clic en \"Create Index\".\n",
        "\n",
        "✅ ¡Ya tienes tu índice vectorial en la nube!"
      ]
    },
    {
      "cell_type": "code",
      "execution_count": null,
      "metadata": {
        "id": "f6a7b8c9d0e1"
      },
      "outputs": [],
      "source": [
        "# Configurar credenciales de Pinecone\n",
        "print(\"--- CONEXIÓN A PINECONE ---\")\n",
        "PINECONE_API_KEY = getpass(\"Ingresa tu API Key de Pinecone: \")\n",
        "PINECONE_ENV = input(\"Ingresa tu entorno (ej. gcp-starter): \")\n",
        "\n",
        "# Inicializar conexión\n",
        "pinecone.init(api_key=PINECONE_API_KEY, environment=PINECONE_ENV)\n",
        "\n",
        "# Verificar índices existentes\n",
        "print(\"\\nÍndices disponibles:\")\n",
        "print(pinecone.list_indexes())\n",
        "\n",
        "# Nombre del índice (debe coincidir con el que creaste)\n",
        "index_name = \"rag-curso\"  # Cambia si usaste otro nombre\n",
        "\n",
        "# Conectar al índice\n",
        "if index_name in pinecone.list_indexes():\n",
        "    index = pinecone.Index(index_name)\n",
        "    print(f\"✅ Conectado al índice '{index_name}'\")\n",
        "    \n",
        "    # Mostrar estadísticas del índice\n",
        "    stats = index.describe_index_stats()\n",
        "    print(f\"Estadísticas del índice: {stats}\")\n",
        "else:\n",
        "    print(f\"❌ El índice '{index_name}' no existe. Créalo primero en la consola de Pinecone.\")"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "a7b8c9d0e1f2"
      },
      "source": [
        "---\n",
        "## **Parte 2: Carga y Preparación de Documentos**\n",
        "\n",
        "Trabajaremos con un documento de ejemplo. Usaremos un artículo de Wikipedia sobre Inteligencia Artificial (descargado como PDF) o un texto de muestra.\n",
        "\n",
        "### **2.1. Descargar documento de ejemplo**"
      ]
    },
    {
      "cell_type": "code",
      "execution_count": null,
      "metadata": {
        "id": "b8c9d0e1f2a3"
      },
      "outputs": [],
      "source": [
        "# Descargar un documento de ejemplo (artículo sobre IA de Wikipedia)\n",
        "!wget -O ia_article.pdf https://upload.wikimedia.org/wikipedia/commons/thumb/0/0b/Artificial_intelligence_%28AI%29.pdf\n",
        "\n",
        "# Alternativa: crear un archivo de texto con contenido relevante\n",
        "texto_ejemplo = \"\"\"\n",
        "# Inteligencia Artificial\n",
        "\n",
        "La inteligencia artificial (IA) es la simulación de procesos de inteligencia humana por parte de máquinas, especialmente sistemas informáticos. Estos procesos incluyen el aprendizaje (la adquisición de información y reglas para usar la información), el razonamiento (usar reglas para llegar a conclusiones aproximadas o definitivas) y la autocorrección.\n",
        "\n",
        "## Aprendizaje Automático\n",
        "El aprendizaje automático (machine learning) es una rama de la IA que permite a las máquinas aprender de datos sin ser explícitamente programadas. Los algoritmos de aprendizaje automático construyen un modelo basado en datos de entrenamiento para hacer predicciones o decisiones.\n",
        "\n",
        "## Aprendizaje Profundo\n",
        "El aprendizaje profundo (deep learning) es un subcampo del machine learning basado en redes neuronales artificiales con múltiples capas. Ha permitido avances significativos en visión por computadora, procesamiento de lenguaje natural y otras áreas.\n",
        "\n",
        "## Procesamiento de Lenguaje Natural\n",
        "El procesamiento de lenguaje natural (PLN) es el campo de la IA que se ocupa de la interacción entre computadoras y humanos mediante el lenguaje natural. Aplicaciones incluyen traducción automática, análisis de sentimientos y chatbots.\n",
        "\n",
        "## Aplicaciones de la IA\n",
        "La IA se aplica en diversos sectores: salud (diagnóstico de enfermedades), finanzas (detección de fraude), transporte (vehículos autónomos), educación (tutores inteligentes) y muchos más.\n",
        "\"\"\"\n",
        "\n",
        "with open('ia_introduccion.txt', 'w') as f:\n",
        "    f.write(texto_ejemplo)\n",
        "\n",
        "print(\"Documento de ejemplo creado: ia_introduccion.txt\")"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "c9d0e1f2a3b4"
      },
      "source": [
        "### **2.2. Cargar y dividir documentos en chunks**\n",
        "\n",
        "Usaremos LangChain para cargar el documento y dividirlo en fragmentos (chunks) de tamaño adecuado."
      ]
    },
    {
      "cell_type": "code",
      "execution_count": null,
      "metadata": {
        "id": "d0e1f2a3b4c5"
      },
      "outputs": [],
      "source": [
        "# Cargar documento de texto\n",
        "loader = TextLoader('ia_introduccion.txt')\n",
        "documentos = loader.load()\n",
        "\n",
        "# Configurar divisor de texto\n",
        "text_splitter = RecursiveCharacterTextSplitter(\n",
        "    chunk_size=200,        # Tamaño de cada fragmento (en caracteres)\n",
        "    chunk_overlap=50,      # Superposición entre fragmentos\n",
        "    length_function=len,\n",
        "    separators=[\"\\n\\n\", \"\\n\", \" \", \"\"]\n",
        ")\n",
        "\n",
        "# Dividir documentos\n",
        "chunks = text_splitter.split_documents(documentos)\n",
        "print(f\"Documento original dividido en {len(chunks)} fragmentos.\")\n",
        "\n",
        "# Mostrar algunos chunks\n",
        "print(\"\\nEjemplos de chunks:\")\n",
        "for i, chunk in enumerate(chunks[:3]):\n",
        "    print(f\"\\n--- Chunk {i+1} ---\")\n",
        "    print(chunk.page_content)\n",
        "    print(f\"Metadatos: {chunk.metadata}\")"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "e1f2a3b4c5d6"
      },
      "source": [
        "---\n",
        "## **Parte 3: Generación de Embeddings y Subida a Pinecone**\n",
        "\n",
        "Utilizaremos `sentence-transformers` a través de LangChain para generar embeddings de cada chunk y subirlos a Pinecone."
      ]
    },
    {
      "cell_type": "code",
      "execution_count": null,
      "metadata": {
        "id": "f2a3b4c5d6e7"
      },
      "outputs": [],
      "source": [
        "# Configurar embeddings con Hugging Face\n",
        "embeddings = HuggingFaceEmbeddings(\n",
        "    model_name=\"all-MiniLM-L6-v2\",\n",
        "    model_kwargs={'device': 'cpu'},\n",
        "    encode_kwargs={'normalize_embeddings': True}\n",
        ")\n",
        "\n",
        "print(f\"Modelo de embeddings cargado. Dimensión: 384\")"
      ]
    },
    {
      "cell_type": "code",
      "execution_count": null,
      "metadata": {
        "id": "a3b4c5d6e7f8"
      },
      "outputs": [],
      "source": [
        "# Subir documentos a Pinecone usando LangChain\n",
        "print(\"Subiendo documentos a Pinecone...\")\n",
        "start_time = time.time()\n",
        "\n",
        "vectorstore = LangchainPinecone.from_documents(\n",
        "    documents=chunks,\n",
        "    embedding=embeddings,\n",
        "    index_name=index_name\n",
        ")\n",
        "\n",
        "upload_time = time.time() - start_time\n",
        "print(f\"✅ Subida completada en {upload_time:.2f} segundos.\")\n",
        "\n",
        "# Verificar estadísticas del índice después de la subida\n",
        "stats = index.describe_index_stats()\n",
        "print(f\"\\nEstadísticas actualizadas:\")\n",
        "print(f\"  Total de vectores: {stats['total_vector_count']}\")\n",
        "print(f\"  Dimensiones: {stats['dimension']}\")"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "b4c5d6e7f8a9"
      },
      "source": [
        "---\n",
        "## **Parte 4: Búsqueda Semántica en Pinecone**\n",
        "\n",
        "Antes de construir el RAG completo, probemos la búsqueda semántica directamente."
      ]
    },
    {
      "cell_type": "code",
      "execution_count": null,
      "metadata": {
        "id": "c5d6e7f8a9b0"
      },
      "outputs": [],
      "source": [
        "def buscar_semantico(query, k=3):\n",
        "    \"\"\"Busca fragmentos similares a la consulta\"\"\"\n",
        "    print(f\"\\n🔍 Consulta: '{query}'\")\n",
        "    \n",
        "    # Generar embedding de la consulta\n",
        "    query_emb = embeddings.embed_query(query)\n",
        "    \n",
        "    # Buscar en Pinecone\n",
        "    results = index.query(\n",
        "        vector=query_emb,\n",
        "        top_k=k,\n",
        "        include_metadata=True\n",
        "    )\n",
        "    \n",
        "    print(f\"\\nResultados:\")\n",
        "    for i, match in enumerate(results['matches']):\n",
        "        print(f\"\\n  {i+1}. [Score: {match['score']:.4f}]\")\n",
        "        print(f\"     {match['metadata']['text'][:150]}...\")\n",
        "    \n",
        "    return results\n",
        "\n",
        "# Probar búsquedas\n",
        "buscar_semantico(\"¿Qué es el deep learning?\")\n",
        "buscar_semantico(\"Aplicaciones de la IA en salud\")\n",
        "buscar_semantico(\"Cómo funciona el procesamiento de lenguaje natural\")"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "d6e7f8a9b0c1"
      },
      "source": [
        "---\n",
        "## **Parte 5: Pipeline RAG con LangChain**\n",
        "\n",
        "Ahora construiremos el sistema completo de preguntas y respuestas. Para el LLM, usaremos OpenAI (requiere API key) o una alternativa gratuita como Hugging Face Hub.\n",
        "\n",
        "### **5.1. Configurar el LLM (OpenAI)**"
      ]
    },
    {
      "cell_type": "code",
      "execution_count": null,
      "metadata": {
        "id": "e7f8a9b0c1d2"
      },
      "outputs": [],
      "source": [
        "# Configurar OpenAI (requiere API key)\n",
        "OPENAI_API_KEY = getpass(\"Ingresa tu API Key de OpenAI (opcional): \")\n",
        "\n",
        "if OPENAI_API_KEY:\n",
        "    os.environ[\"OPENAI_API_KEY\"] = OPENAI_API_KEY\n",
        "    from langchain.llms import OpenAI\n",
        "    llm = OpenAI(temperature=0, model_name=\"gpt-3.5-turbo-instruct\")\n",
        "    print(\"✅ LLM de OpenAI configurado.\")\n",
        "else:\n",
        "    print(\"⚠️  No se proporcionó API key. Usaremos una simulación.\")\n",
        "    # Simulación simple para demostración\n",
        "    from langchain.llms import FakeListLLM\n",
        "    responses = [\"La inteligencia artificial es un campo que...\", \n",
        "                 \"El deep learning utiliza redes neuronales...\",\n",
        "                 \"Las aplicaciones de IA incluyen salud y finanzas...\"]\n",
        "    llm = FakeListLLM(responses=responses)\n",
        "    print(\"✅ Usando LLM simulado.\")"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "f8a9b0c1d2e3"
      },
      "source": [
        "### **5.2. Crear el chain de RetrievalQA**"
      ]
    },
    {
      "cell_type": "code",
      "execution_count": null,
      "metadata": {
        "id": "a9b0c1d2e3f4"
      },
      "outputs": [],
      "source": [
        "# Crear el retriever desde Pinecone\n",
        "retriever = vectorstore.as_retriever(\n",
        "    search_kwargs={\"k\": 3}  # Número de fragmentos a recuperar\n",
        ")\n",
        "\n",
        "# Crear prompt personalizado\n",
        "prompt_template = \"\"\"Responde la pregunta basándote ÚNICAMENTE en el siguiente contexto. Si no encuentras la respuesta en el contexto, di \"No tengo suficiente información para responder.\"\n",
        "\n",
        "Contexto:\n",
        "{context}\n",
        "\n",
        "Pregunta: {question}\n",
        "Respuesta:\"\"\"\n",
        "\n",
        "PROMPT = PromptTemplate(\n",
        "    template=prompt_template,\n",
        "    input_variables=[\"context\", \"question\"]\n",
        ")\n",
        "\n",
        "# Crear cadena de QA\n",
        "qa_chain = RetrievalQA.from_chain_type(\n",
        "    llm=llm,\n",
        "    chain_type=\"stuff\",  # \"stuff\", \"map_reduce\", \"refine\", \"map_rerank\"\n",
        "    retriever=retriever,\n",
        "    chain_type_kwargs={\"prompt\": PROMPT},\n",
        "    return_source_documents=True\n",
        ")\n",
        "\n",
        "print(\"✅ Cadena RAG configurada.\")"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "b0c1d2e3f4a5"
      },
      "source": [
        "### **5.3. Hacer preguntas al sistema RAG**"
      ]
    },
    {
      "cell_type": "code",
      "execution_count": null,
      "metadata": {
        "id": "c1d2e3f4a5b6"
      },
      "outputs": [],
      "source": [
        "def preguntar(question):\n",
        "    \"\"\"Función para hacer preguntas al sistema RAG\"\"\"\n",
        "    print(\"\\n\" + \"=\"*70)\n",
        "    print(f\"❓ PREGUNTA: {question}\")\n",
        "    print(\"=\"*70)\n",
        "    \n",
        "    # Ejecutar la consulta\n",
        "    result = qa_chain({\"query\": question})\n",
        "    \n",
        "    print(\"\\n📝 RESPUESTA:\")\n",
        "    print(result['result'])\n",
        "    \n",
        "    print(\"\\n📚 FUENTES UTILIZADAS:\")\n",
        "    for i, doc in enumerate(result['source_documents']):\n",
        "        print(f\"  {i+1}. {doc.page_content[:100]}...\")\n",
        "    \n",
        "    return result\n",
        "\n",
        "# Probar con preguntas sobre el documento\n",
        "preguntar(\"¿Qué es el aprendizaje automático?\")\n",
        "preguntar(\"¿Cómo se relaciona el deep learning con la IA?\")\n",
        "preguntar(\"¿En qué sectores se aplica la IA?\")"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "d2e3f4a5b6c7"
      },
      "source": [
        "---\n",
        "## **Parte 6: Comparativa ChromaDB vs Pinecone**\n",
        "\n",
        "Ahora que hemos trabajado con Pinecone, comparemos con ChromaDB (visto en NB1) en varios aspectos."
      ]
    },
    {
      "cell_type": "code",
      "execution_count": null,
      "metadata": {
        "id": "e3f4a5b6c7d8"
      },
      "outputs": [],
      "source": [
        "comparativa = pd.DataFrame({\n",
        "    'Característica': ['Tipo', 'Despliegue', 'Costo', 'Límites', 'Facilidad de uso', 'Persistencia', 'Índices soportados', 'Integración con LangChain', 'Ideal para'],\n",
        "    'ChromaDB': ['Local (embebida)', 'Misma máquina', 'Gratuito', 'Memoria/disco local', 'Muy fácil', 'Sí (archivos)', 'HNSW (configurable)', 'Sí', 'Prototipos, desarrollo local'],\n",
        "    'Pinecone': ['Nube (gestionado)', 'SaaS', 'Gratuito limitado', '1 índice, 100k vectores', 'Fácil (API)', 'Sí (automática)', 'HNSW (optimizado)', 'Sí', 'Producción, escalabilidad']\n",
        "})\n",
        "\n",
        "print(\"--- COMPARATIVA CHROMADB VS PINECONE ---\")\n",
        "display(comparativa)\n",
        "\n",
        "print(\"\\n📌 Conclusiones:\")\n",
        "print(\"• ChromaDB es ideal para desarrollo local, pruebas y cuando los datos no deben salir de la máquina.\")\n",
        "print(\"• Pinecone es ideal para producción, cuando se necesita escalabilidad y no se quiere gestionar infraestructura.\")\n",
        "print(\"• Ambos se integran bien con LangChain, lo que permite cambiar entre ellos fácilmente.\")"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "f4a5b6c7d8e9"
      },
      "source": [
        "---\n",
        "## **Ejercicios Prácticos para el Estudiante**\n",
        "\n",
        "Ahora aplica lo aprendido con estos ejercicios."
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "a5b6c7d8e9f0"
      },
      "source": [
        "### **Ejercicio 1: Indexar un documento propio**\n",
        "\n",
        "Elige un documento (PDF o texto) de tu interés. Puede ser un artículo, un capítulo de libro, o incluso una página web descargada. Cárgalo, divídelo en chunks (tamaño 300, solapamiento 50) e índexalo en Pinecone. Luego, realiza al menos 3 preguntas sobre su contenido."
      ]
    },
    {
      "cell_type": "code",
      "execution_count": null,
      "metadata": {
        "id": "b6c7d8e9f0a1"
      },
      "outputs": [],
      "source": [
        "# --- INICIO DE TU CÓDIGO ---\n",
        "\n",
        "# 1. Cargar documento (puedes usar PyPDFLoader para PDF)\n",
        "# from langchain.document_loaders import PyPDFLoader\n",
        "# loader = PyPDFLoader(\"tu_archivo.pdf\")\n",
        "# ...\n",
        "\n",
        "# 2. Dividir en chunks\n",
        "# ...\n",
        "\n",
        "# 3. Indexar en Pinecone (crear nuevo índice o usar el mismo con namespace diferente)\n",
        "# ...\n",
        "\n",
        "# 4. Hacer preguntas\n",
        "# ...\n",
        "\n",
        "# --- FIN DE TU CÓDIGO ---"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "c7d8e9f0a1b2"
      },
      "source": [
        "### **Ejercicio 2: Experimentar con diferentes tamaños de chunk**\n",
        "\n",
        "Prueba al menos 3 tamaños de chunk diferentes (ej. 100, 300, 500) con el mismo documento. Para cada tamaño:\n",
        "1. Indexa los documentos en Pinecone (puedes usar namespaces diferentes: `chunk_100`, `chunk_300`, etc.)\n",
        "2. Haz las mismas 3 preguntas a cada configuración.\n",
        "3. Compara la calidad de las respuestas. ¿Qué tamaño funciona mejor? ¿Por qué?"
      ]
    },
    {
      "cell_type": "code",
      "execution_count": null,
      "metadata": {
        "id": "d8e9f0a1b2c3"
      },
      "outputs": [],
      "source": [
        "# --- INICIO DE TU CÓDIGO ---\n",
        "\n",
        "# Tamaños a probar\n",
        "chunk_sizes = [100, 300, 500]\n",
        "\n",
        "for size in chunk_sizes:\n",
        "    print(f\"\\n{'='*50}\")\n",
        "    print(f\"Probando chunk_size = {size}\")\n",
        "    print('='*50)\n",
        "    # ...\n",
        "\n",
        "# --- FIN DE TU CÓDIGO ---"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "e9f0a1b2c3d4"
      },
      "source": [
        "### **Ejercicio 3: Añadir metadatos a los documentos**\n",
        "\n",
        "Modifica el proceso de indexación para incluir metadatos adicionales en cada chunk, como:\n",
        "*   `fuente`: nombre del documento original.\n",
        "*   `página`: número de página (si aplica).\n",
        "*   `categoría`: tema del chunk (puedes asignarlo manualmente o inferirlo).\n",
        "\n",
        "Luego, realiza una búsqueda que filtre por metadatos (por ejemplo, buscar solo en chunks de una categoría específica)."
      ]
    },
    {
      "cell_type": "code",
      "execution_count": null,
      "metadata": {
        "id": "f0a1b2c3d4e5"
      },
      "outputs": [],
      "source": [
        "# --- INICIO DE TU CÓDIGO ---\n",
        "\n",
        "# Crear documentos con metadatos\n",
        "# ...\n",
        "\n",
        "# Indexar\n",
        "# ...\n",
        "\n",
        "# Buscar con filtro (Pinecone soporta filtros en metadata)\n",
        "# index.query(vector=query_emb, filter={\"categoria\": \"machine_learning\"}, top_k=3)\n",
        "\n",
        "# --- FIN DE TU CÓDIGO ---"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "a1b2c3d4e5f6"
      },
      "source": [
        "### **Ejercicio 4: Reflexión sobre RAG en producción**\n",
        "\n",
        "En una celda Markdown, responde:\n",
        "1. ¿Qué ventajas ofrece Pinecone sobre ChromaDB para un sistema en producción?\n",
        "2. ¿Qué consideraciones de costo y latencia deberías tener en cuenta?\n",
        "3. ¿Cómo manejarías la actualización de documentos (añadir, modificar, eliminar) en un sistema RAG basado en Pinecone?\n",
        "4. Investiga: ¿Qué otros servicios de bases de datos vectoriales existen en la nube (AWS, GCP, Azure)?"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "b2c3d4e5f6a7"
      },
      "source": [
        "---\n",
        "## **Conclusiones**\n",
        "\n",
        "En esta sesión práctica hemos:\n",
        "1. **Configurado** un índice gratuito en Pinecone.\n",
        "2. **Cargado y procesado** documentos, dividiéndolos en chunks.\n",
        "3. **Generado embeddings** y subido los vectores a Pinecone.\n",
        "4. **Construido** un pipeline RAG completo con LangChain.\n",
        "5. **Comparado** ChromaDB (local) vs Pinecone (nube).\n",
        "\n",
        "**Conceptos clave reforzados:**\n",
        "*   RAG combina recuperación (búsqueda semántica) y generación (LLM).\n",
        "*   Las bases de datos vectoriales en la nube como Pinecone ofrecen escalabilidad sin gestión de infraestructura.\n",
        "*   LangChain abstrae las complejidades de la integración entre componentes.\n",
        "*   La elección entre solución local y en la nube depende de requisitos de escala, costo y privacidad.\n",
        "\n",
        "¡Has construido tu primer sistema RAG funcional!"
      ]
    },
    {
      "cell_type": "code",
      "execution_count": null,
      "metadata": {
        "id": "c3d4e5f6a7b8"
      },
      "outputs": [],
      "source": [
        "# (Opcional) Limpiar recursos si es necesario\n",
        "# No eliminamos el índice para no perder el progreso, pero en producción podrías hacerlo.\n",
        "# pinecone.delete_index(index_name)\n",
        "print(\"\\nNota: El índice en Pinecone permanece para futuros usos.\")"
      ]
    }
  ]
}