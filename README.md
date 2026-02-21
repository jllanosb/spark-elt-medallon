# 🏆 Spark ETL - Arquitectura Medallón

Proyecto educativo de ingeniería de datos que implementa un pipeline ETL con Apache Spark siguiendo el patrón de arquitectura Medallón (Bronze → Silver → Gold), adaptado a capas: Workload → Landing → Curated → Functional.

# 📋 Tabla de Contenidos
- 🎯 ¿Qué es este proyecto?
- 🏗️ Arquitectura Medallón Explicada
- 📁 Estructura del Repositorio
- ⚙️ Tecnologías Utilizadas
- 🚀 Guía de Ejecución Paso a Paso
- 🔍 Detalle de Cada Capa
- 📊 Esquema de Datos
- 💡 Mejores Prácticas Implementadas
- 🔧 Solución de Problemas Comunes
- 📚 Recursos de Aprendizaje

# 🎯 ¿Qué es este proyecto?
Este repositorio es una implementación didáctica de un pipeline de datos empresarial usando Apache Spark y Hadoop Ecosystem. Su objetivo principal es:
- ✅ Enseñar los fundamentos de la arquitectura Medallón en entornos on-premise
- ✅ Demonstrar buenas prácticas de ingesta, transformación y calidad de datos
- ✅ Proveer código reutilizable para procesos ETL escalables
- ✅ Facilitar el aprendizaje de Spark SQL, Hive y formatos columnares  

💡 Caso de uso: Procesamiento de transacciones comerciales con entidades PERSONA, EMPRESA y TRANSACCION, aplicando reglas de calidad y enriquecimiento progresivo.

# 🏗️ Arquitectura Medallón Explicada
La arquitectura Medallón organiza los datos en capas de refinamiento progresivo, mejorando la calidad y utilidad en cada etapa:
```table
┌─────────────────────────────────────────────────────────────────────────┐
│                           FLUJO DE DATOS                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  📥 FUENTES → 🥉 WORKLOAD → 🥈 LANDING → 🥇 CURATED → ⚡ FUNCTIONAL    
│              (Bronze)      (Silver)     (Gold)      (Analytics)         │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```
🔹 Capa 1: WORKLOAD (Bronze - Datos Crudos)
```table
Característica      Descripción
-----------------------------------------------------
Formato             TEXTFILE con delimitador `
Encoding            ISO-8859-1 (soporte legacy)
Propósito           Ingesta fiel de fuentes originales
Validación          Mínima (solo estructura)
```
🔹 Capa 2: LANDING (Silver - Datos Estandarizados)
```table
Característica      Descripción
------------------------------------------------------
Formato             AVRO con compresión Snappy 
Schema              Definido en archivos .avsc
Propósito           Estructura consistente + metadatos
Particionamiento    Por fecha en tablas transaccionales
```
🔹 Capa 3: CURATED (Gold - Datos Limpios)
```table
Característica      Descripción
--------------------------------------------------
Formato             Parquet con Snappy
Calidad             Reglas de validación aplicadas
Tipado              Conversión explícita de tipos
Propósito           Datos confiables para análisis
```
🔹 Capa 4: FUNCTIONAL (Analytics - Datos Enriquecidos)
```table
Característica      Descripción
--------------------------------------------------------
Formato             Parquet optimizado
Transformación      JOINs para enriquecimiento semántico
Optimización        Broadcast joins para tablas pequeñas
Propósito           Listo para dashboards y ML
```
📚 La arquitectura Medallión es ampliamente adoptada en plataformas como Databricks y Azure Synapse para organizar data lakes de forma escalable.

# 📁 Estructura del Repositorio
```text
spark-elt-medallon/
│
├── 📁 dataset/                   # Datos fuente de ejemplo
│   ├── empresa.data              # Catálogo de empresas (pipe-delimited)
│   ├── persona.data              # Registro de personas
│   └── transacciones.data        # Movimientos comerciales
│
├── 📁 schema/                    # Esquemas Avro para validación
│   ├── empresa.avsc              # Schema: id, nombre
│   ├── persona.avsc              # Schema: id, nombre, contacto, etc.
│   └── transaccion.avsc          # Schema: monto, fecha, relaciones
│
├── 📁 procesos/                  # Scripts PySpark del pipeline
│   ├── poblar_capa_workload.py   # ▶️ Ingesta inicial (CSV → Hive TEXTFILE)
│   ├── poblar_capa_landing.py    # ▶️ Estandarización (→ Avro + partición)
│   ├── poblar_capa_curated.py    # ▶️ Limpieza y validación de calidad
│   └── poblar_capa_functional.py # ▶️ Enriquecimiento con JOINs
│
├── 📄 instrucciones.txt          # Guía rápida de comandos de ejecución
└── 📄 README.md                  # ¡Este archivo! Documentación didáctica
```
# ⚙️ Tecnologías Utilizadas
```table
Tecnología          Versión         Propósito
------------------------------------------------------------------------
Apache Spark        3.5.0           Motor de procesamiento distribuido
Apache Hive         3.x             Metastore y consulta SQL sobre HDFS
Hadoop HDFS         3.x             Almacenamiento distribuido
Apache YARN         3.x             Gestor de recursos del cluster
Formato Avro        1.11+           Serialización con esquema evolutivo
Formato Parquet     1.12+           Almacenamiento columnar optimizado
Compresión Snappy   1.1+            Balance velocidad/tamaño en datos
```
🔗 Estas herramientas son estándar en ecosistemas de Big Data on-premise y en la nube

# 🚀 Guía de Ejecución Paso a Paso

🔹 Prerrequisitos
```text
# Cluster Hadoop con servicios activos:
✅ HDFS en ejecución
✅ YARN Resource Manager
✅ Hive Metastore + HiveServer2
✅ Spark instalado y configurado con Hive
✅ Acceso SSH al nodo edge con usuario `hadoop`
```
🔹 Paso 1: Iniciar servicios (si es necesario)
# Desde instrucciones.txt
```bash
start-dfs.sh
start-yarn.sh
hive --service metastore &
sleep 10
hive --service hiveserver2 &
```
🔹 Paso 2: Cargar datos fuente a HDFS
# Crear directorio y subir archivos .data
```bash
hdfs dfs -mkdir -p /user/hadoop/dataset
hdfs dfs -put /home/hadoop/spark-elt-medallon/dataset/* /user/hadoop/dataset/
hdfs dfs -ls /user/hadoop/dataset  # Verificar carga
```
🔹 Paso 3: Ejecutar cada capa del pipeline
🥉 Capa WORKLOAD (Ingesta)
```pyspark
spark-submit \
  --master yarn \
  --deploy-mode cluster \
  --conf spark.sql.warehouse.dir=/user/hadoop/warehouse \
  --conf spark.serializer=org.apache.spark.serializer.KryoSerializer \
  /home/hadoop/spark-elt-medallon/procesos/poblar_capa_workload.py \
  --env TopicosB \
  --username hadoop \
  --base_path /user \
  --local_data_path /user/hadoop/dataset
```
🥈 Capa LANDING (Estandarización Avro)

- Primero subir esquemas Avro
```bash
hdfs dfs -mkdir -p /user/hadoop/datalake/schema/TOPICOSB_LANDING/
hdfs dfs -put -f /home/hadoop/spark-elt-medallon/schema/*.avsc /user/hadoop/datalake/schema/TOPICOSB_LANDING/
hdfs dfs -ls /user/hadoop/datalake/schema/TOPICOSB_LANDING/
```
- Ejecutar proceso
```pyspark
spark-submit \
  --master yarn \
  --deploy-mode client \
  --conf spark.sql.warehouse.dir=/user/hadoop/warehouse \
  --conf spark.sql.avro.compression.codec=snappy \
  --packages org.apache.spark:spark-avro_2.12:3.5.0 \
  /home/hadoop/spark-elt-medallon/procesos/poblar_capa_landing.py \
  --env TopicosB \
  --username hadoop \
  --base_path /user \
  --schema_path /user/hadoop/datalake/schema \
  --source_db topicosb_workload
```
🥇 Capa CURATED (Calidad y Limpieza)
```pyspark
spark-submit \
  --master yarn \
  --deploy-mode client \
  --conf spark.sql.warehouse.dir=/user/hadoop/warehouse \
  --conf spark.sql.parquet.compression.codec=snappy \
  --conf spark.dynamicAllocation.enabled=true \
  --conf spark.executor.instances=10 \
  --conf spark.executor.memory=4g \
  --conf spark.driver.memory=2g \
  /home/hadoop/spark-elt-medallon/procesos/poblar_capa_curated.py \
  --env TopicosB \
  --username hadoop \
  --base_path /user \
  --source_db landing \
  --enable-validation  # ← Activa filtros de calidad
```
⚡ Capa FUNCTIONAL (Enriquecimiento)
```pyspark
spark-submit \
  --master yarn \
  --deploy-mode client \
  --conf spark.sql.warehouse.dir=/user/hadoop/warehouse \
  --conf spark.yarn.queue=default \
  --conf spark.sql.parquet.compression.codec=snappy \
  --conf spark.dynamicAllocation.enabled=false \
  /home/hadoop/spark-elt-medallon/procesos/poblar_capa_functional.py \
  --env TopicosB \
  --username hadoop \
  --base_path /user \
  --source_db curated \
  --num-executors 8 \
  --executor-memory 2g \
  --executor-cores 2 \
  --enable-broadcast  # ← Optimiza JOINs con tablas pequeñas
```
🔹 Paso 4: Detener servicios (opcional)
```bash
stop-yarn.sh
stop-dfs.sh
pkill -f HiveServer2
pkill -f HiveMetaStore
```
📊 Esquema de Datos
Entidad: PERSONA
```table
Campo       Tipo Original       Tipo Final      Regla de Calidad
---------------------------------------------------------------------------
ID          String              String          NOT NULL
NOMBRE      String              String          -
EDAD        String              Integer         BETWEEN 1 AND 99
SALARIO     String              Double          BETWEEN 0.01 AND 9999999.99
ID_EMPRESA  String              String          NOT NULL
```
Entidad: TRANSACCION_ENRIQUECIDA (Functional)
```table
Campo               Origen              Transformación
--------------------------------------------------------------------
ID_PERSONA          TRANSACCION         Clave de join
NOMBRE_PERSONA      PERSONA.NOMBRE      Enriquecimiento semántico
EDAD_PERSONA        PERSONA.EDAD        Conversión + validación
TRABAJO_PERSONA     EMPRESA.NOMBRE      JOIN con empresa empleadora
MONTO_TRANSACCION   TRANSACCION.MONTO   Conversión a Double
EMPRESA_TRANSACCION EMPRESA.NOMBRE      JOIN con empresa receptora
FECHA_TRANSACCION   TRANSACCION.FECHA   Columna de partición
```
# 💡 Mejores Prácticas Implementadas
- ✅ Esquemas explícitos: Evita inferencia automática y garantiza consistencia
- ✅ Validación progresiva: Reglas de calidad aplicadas en capa Curated
- ✅ Particionamiento inteligente: Por fecha en tablas transaccionales para consultas eficientes
- ✅ Compresión Snappy: Balance óptimo entre velocidad y almacenamiento 
- ✅ Broadcast joins: Optimización automática para tablas de dimensión pequeñas
- ✅ Logging estructurado: Mensajes claros para monitoreo y debugging
- ✅ Parámetros configurables: --env, --enable-validation, --enable-broadcast para flexibilidad
- ✅ Limpieza de recursos: spark.stop() y eliminación de vistas temporales  

 Estas prácticas siguen recomendaciones de Databricks y Microsoft para pipelines productivos

# 🔎 Comandos de diagnóstico útiles
```bash
# Verificar archivos en HDFS
hdfs dfs -ls /user/hadoop/datalake/TOPICOSB_LANDING/

# Consultar metadatos de tabla Hive
hive -e "DESCRIBE FORMATTED topicosb_landing.persona;"
```
```sql
# Contar registros por partición
spark.sql("SELECT FECHA_TRANSACCION, COUNT(*) FROM topicosb_functional.transaccion_enriquecida GROUP BY FECHA_TRANSACCION").show()
```
```bash
# Monitorear aplicación Spark en YARN
yarn application -list | grep "Proceso_Carga"
```
# 🤝 Contribuciones
Este proyecto está diseñado para fines educativos. ¡Las contribuciones son bienvenidas!

✅ Ideas para mejorar:
- [ ] Agregar tests unitarios con pytest y chispa
- [ ] Implementar lineage de datos con OpenLineage
- [ ] Añadir dashboard de monitoreo con Prometheus/Grafana
- [ ] Soporte para Delta Lake como formato unificado
- [ ] Docker-compose para entorno de desarrollo local

🏷️ Licencia: MIT - Libre uso para fines educativos y de investigación

👨‍💻 Autor: @jllanosb

📅 Última actualización: Febrero 2026

Contexto: Desarrollado con enfoque en formación en ingeniería de datos en entornos on-premise

# ✨ "La calidad de los datos no es un paso, es un viaje a través de capas de refinamiento" ✨