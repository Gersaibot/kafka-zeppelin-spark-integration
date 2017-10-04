# kafka-zeppelin-spark-integration

El presente es un proyecto que cubrirá el manejo de mensajes recibidos en un tópico de Kafka a través de Zeppelin mediante el intérprete de Spark.

## Introducción

**Apache Kafka** es una proyecto open source escrito en Scala y desarrollado por la Apache Software Fundation. Kafka implementa un modelo de publicacion y subscripción para la intermediación de mensajes por medio de canales o "tópicos".

**Apache Zeppelin** es un proyecto en incubación desarrollado por la Apache Software Fundation que consiste en una implementación de la idea/concepto de _web noteboook_, el cual fue introducido por primera vez po **IPython**. Zeppelin esta enfocado en el desarrollo de procesos analíticos e interactivos de datos mediante tecnologías y lenguajes como Shell, Spark (Scala), Hive, Elasticsearch, R y demás.

Este proyecto puede considerarse como una continuación de mi anterior repositorio [mosquitto-kafka-integration](https://github.com/Gersaibot/mosquitto-kafka-integration), el cual cubre la instalación de un broker **MQTT Mosquitto** y su integración con **Apache Kafka**.

Es decir, mediante el siguiente conjunto de instrucciones se podrá construir el siguiente flujo de eventos:
1. Recepción de mensaje en tópico de Mosquitto
2. Publicación automatica de mensaje en tópico de Kafka a través de un conector
3. Consulta de mensajes recibidos en tópico de Kafka a través de Zeppelin mediante el intérprete de Spark y librerías adicionales. 

## Pre-requisitos

* Java (1.7+)
* Apacha Kafka v0.11.0.* compilado en Scala 2.11

## Versiones

* Apache Zookeeper v0.7.3 (sólo intérprete de Spark)
* Apache Spark v2.2.0 - Scala 2.11 y pre-compilado para Apache Hadoop 2.7+

## Instalación

En primer lugar se debe desacargar Apache Zeppelin desde el [sitio web oficial](https://zeppelin.apache.org/download.html) del proyecto. Para este proyecto solo será necesario el intérprete de Spark.

Una vez descargado, es necesario descomprimir el archivo. De ahora en adelante nos referiremos a la ruta de descompresión de Zeppelin como `$ZEPPELIN_HOME` (ruta raíz de Zeppelin).

Bajo el mismo orden de ideas se debe descargar,  desde su sitio web official, el proyecto Apache Spark.

La ruta de descompresión de Spark sera referida de ahora en adelante como `$SPARK_HOME`.

Además, es necesario descargar los binarios de la librería [spark-core_2.11-1.5.2](https://book2s.com/java/jar/s/spark-core-2-11/download-spark-core_2.11-1.5.2.jar.html), la cual resolvera las dependencias no incluidas en Spark 2.2.0 y necesarias para las librerías adicionales de Spark que se configurarán posteriormente. Esta librería debe ser agregada a las librerías de Spark:

```bash
cp spark-core_2.11-1.5.2.jar $SPARK_HOME/jars/
```

## Configuración

De no exister, es necesario crear el archivo `zeppelin-env.sh`, en el cual configuraremos la variable `SPARK_HOME` de forma que apunte al directorio $SPARK_HOME:

```bash
cp $ZEPPELIN_HOME/conf/zeppelin-env.sh.template $ZEPPELIN_HOME/conf/zeppelin-env.sh
vi $ZEPPELIN_HOME/conf/zeppelin-env.sh
		export SPARK_HOME=$SPARK_HOME
```
_Ej:_
```bash
export SPARK_HOME=/home/gabriel/Workspace/spark-2.2.0-bin-hadoop2.7
```

Procedemos a iconfigurar el interprete de Spark. Para ello iniciamos el servicio de Zeppelin y accedemos a la dirección http://localhost:8080/ - automatic! :

```bash
./$ZEPPELIN_HOME/bin/zeppelin-daemon.sh
```

Nos dirigimos al menú de los interpretes y agregamos las siguientes dependencias al interprete de Spark:

- org.apache.spark:spark-streaming-kafka_2.11:1.6.3
- org.apache.kafka:kafka_2.11:0.11.0.1
- org.apache.kafka:kafka-clients:0.11.0.1
- org.apache.spark:spark-streaming_2.11:2.2.0
- org.apache.spark:spark-core_2.11:2.2.0
- org.apache.spark:spark-sql_2.11:2.2.0
- org.apache.spark:spark-streaming-kafka-0-10_2.11:2.2.0 (para el nuevo API de Kafka)

Las versiones de las librerias son susceptibles a las versiones de las herramientas en uso. Si se trabajan con versiones distintas a las indicadas anteriormente, es necesario verificar que las versiones este en concordancia.

Zeppelin instalara las dependencias desde los repositorios de maven. De existir un error, Zeppelin no tardará en notificar cuales librerías casan conflictos.

De no existir ningun error, podemos crear un Notebook que haga uso del intérprete de Spark y ejecutar las siguientes líneas de código para verificar que el intérprete de Spark funciona correcamente:

```scala
%spark
print(sc.version)
```

## Despliegue de Notebooks

Podemos crear Notebooks y probar los ejemplos publicadoes en la carpeta samples. Cada uno realiza un consulta al tópico de Kafka en cuestión a través de dos conjuntos de librerias distintas. Estes conjuntos operan respectivamente con el API de consulta actual de Kafka y el API de consulta antiguo. Por ende, existen ligeras diferencias entre los métodos de consulta al tópico de Kafka y los parámetros de configuración de la conexión. Entre los parámetros mas relevantes podemos señalar los siguientes:

- **metadata.broker.list / bootstrap.servers**: Lista de hosts expuestos por Kafka. 
- **group.id**: Valor que señala el grupo de consumidores al pertenece el consumidor actual.
- **auto.offset.reset**: Valor que determina el último commit desde el cual se comenzará a leer los mensajes de Kafka.

En la [documentación oficial](https://kafka.apache.org/documentation/) de Apache Kafka existe información más detallada de los parámetros que pueden configurarse y los APIs de consutla (actual y antiguo). 

Una vez ejecutado cualquiera de los ejemplos, podremos ver los mensajes publicados en el tópico de Kafka en tiempo real mientras el notebook se encuentre en ejecución.

Si se realizó algun tipo de integración entre Kafka y Mosquitto (como la explicada en el repositorio [mosquitto-kafka-integration](https://github.com/Gersaibot/mosquitto-kafka-integration)), podremos visualizar los mensajes publicados en el tópico de Mosquitto tiempo real. 

## Referencias

- [Apache Kafka](https://kafka.apache.org/)
- [Apache Zeppelin](https://zeppelin.apache.org/)
- [Apache Spark](https://spark.apache.org/)






