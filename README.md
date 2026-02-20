# 🏆 Spark ETL - Arquitectura Medallón

Proyecto educativo de ingeniería de datos que implementa un pipeline ETL con Apache Spark siguiendo el patrón de arquitectura Medallón (Bronze → Silver → Gold), adaptado a capas: Workload → Landing → Curated → Functional.

# 📋 Tabla de Contenidos
🎯 ¿Qué es este proyecto?
🏗️ Arquitectura Medallón Explicada
📁 Estructura del Repositorio
⚙️ Tecnologías Utilizadas
🚀 Guía de Ejecución Paso a Paso
🔍 Detalle de Cada Capa
📊 Esquema de Datos
💡 Mejores Prácticas Implementadas
🔧 Solución de Problemas Comunes
📚 Recursos de Aprendizaje

# 🎯 ¿Qué es este proyecto?
Este repositorio es una implementación didáctica de un pipeline de datos empresarial usando Apache Spark y Hadoop Ecosystem. Su objetivo principal es:
✅ Enseñar los fundamentos de la arquitectura Medallón en entornos on-premise
✅ Demonstrar buenas prácticas de ingesta, transformación y calidad de datos
✅ Proveer código reutilizable para procesos ETL escalables
✅ Facilitar el aprendizaje de Spark SQL, Hive y formatos columnares  

💡 Caso de uso: Procesamiento de transacciones comerciales con entidades PERSONA, EMPRESA y TRANSACCION, aplicando reglas de calidad y enriquecimiento progresivo.

# 🏗️ Arquitectura Medallón Explicada
La arquitectura Medallón organiza los datos en capas de refinamiento progresivo, mejorando la calidad y utilidad en cada etapa:

┌─────────────────────────────────────────────────────┐
│                    FLUJO DE DATOS                    │
├─────────────────────────────────────────────────────┤
│                                                      │
│  📥 FUENTES → 🥉 WORKLOAD → 🥈 LANDING → 🥇 CURATED → ⚡ FUNCTIONAL
│              (Bronze)      (Silver)     (Gold)      (Analytics)
│                                                      │
└─────────────────────────────────────────────────────┘

🔹 Capa 1: WORKLOAD (Bronze - Datos Crudos)

Característica      Descripción
-----------------------------------------------
Formato             TEXTFILE con delimitador `
Encoding            ISO-8859-1 (soporte legacy)
Propósito           Ingesta fiel de fuentes originales
Validación          Mínima (solo estructura)

🔹 Capa 2: LANDING (Silver - Datos Estandarizados)

Característica      Descripción
-----------------------------------------------
Formato             AVRO con compresión Snappy 
Schema              Definido en archivos .avsc
Propósito           Estructura consistente + metadatos
Particionamiento    Por fecha en tablas transaccionales

🔹 Capa 3: CURATED (Gold - Datos Limpios)

Característica      Descripción
-----------------------------------------
Formato             Parquet con Snappy
Calidad             Reglas de validación aplicadas
Tipado              Conversión explícita de tipos
Propósito           Datos confiables para análisis

🔹 Capa 4: FUNCTIONAL (Analytics - Datos Enriquecidos)

Característica      Descripción
Formato             Parquet optimizado
Transformación      JOINs para enriquecimiento semántico
Optimización        Broadcast joins para tablas pequeñas
Propósito           Listo para dashboards y ML

📚 La arquitectura Medallión es ampliamente adoptada en plataformas como Databricks y Azure Synapse para organizar data lakes de forma escalable.

# 📁 Estructura del Repositorio

spark-elt-medallon/
│
├── 📁 dataset/                    # Datos fuente de ejemplo
│   ├── empresa.data              # Catálogo de empresas (pipe-delimited)
│   ├── persona.data              # Registro de personas
│   └── transacciones.data        # Movimientos comerciales
│
├── 📁 schema/                     # Esquemas Avro para validación
│   ├── empresa.avsc              # Schema: id, nombre
│   ├── persona.avsc              # Schema: id, nombre, contacto, etc.
│   └── transaccion.avsc          # Schema: monto, fecha, relaciones
│
├── 📁 procesos/                   # Scripts PySpark del pipeline
│   ├── poblar_capa_workload.py   # ▶️ Ingesta inicial (CSV → Hive TEXTFILE)
│   ├── poblar_capa_landing.py    # ▶️ Estandarización (→ Avro + partición)
│   ├── poblar_capa_curated.py    # ▶️ Limpieza y validación de calidad
│   └── poblar_capa_functional.py # ▶️ Enriquecimiento con JOINs
│
├── 📄 instrucciones.txt          # Guía rápida de comandos de ejecución
└── 📄 README.md                  # ¡Este archivo! Documentación didáctica

# ⚙️ Tecnologías Utilizadas

Tecnología          Versión         Propósito
------------------------------------------------------------------------
Apache Spark        3.5.0           Motor de procesamiento distribuido
Apache Hive         3.x             Metastore y consulta SQL sobre HDFS
Hadoop HDFS         3.x             Almacenamiento distribuido
Apache YARN         3.x             Gestor de recursos del cluster
Formato Avro        1.11+           Serialización con esquema evolutivo
Formato Parquet     1.12+           Almacenamiento columnar optimizado
Compresión Snappy   1.1+            Balance velocidad/tamaño en datos

🔗 Estas herramientas son estándar en ecosistemas de Big Data on-premise y en la nube

# 🚀 Guía de Ejecución Paso a Paso




    🏷️ Licencia: MIT - Libre uso para fines educativos y de investigación
    👨‍💻 Autor: @jllanosb

    📅 Última actualización: Febrero 2026
    🇵🇪 Contexto: Desarrollado con enfoque en formación en ingeniería de datos en entornos on-premise

✨ "La calidad de los datos no es un paso, es un viaje a través de capas de refinamiento" ✨