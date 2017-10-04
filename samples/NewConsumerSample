%spark

import org.apache.spark.streaming._
import org.apache.spark.streaming.StreamingContext._
import org.apache.kafka.clients.consumer.ConsumerRecord
import org.apache.kafka.common.serialization.StringDeserializer
import org.apache.spark.streaming.kafka010._
import org.apache.spark.streaming.kafka010.LocationStrategies.PreferConsistent
import org.apache.spark.streaming.kafka010.ConsumerStrategies.Subscribe

/** Flag to detect whether new context was created or not */
var newContextCreated = false      

/** Batch insterval for StreamingContex */
val batchIntervalSeconds = 1       

/** Creates the StreamingContext */
def creatingFunc(): StreamingContext = {
  val ssc = new StreamingContext(sc, Seconds(batchIntervalSeconds))
  println("Creating function called to create new StreamingContext")
  newContextCreated = true  
  ssc
}

/** Connection params */
val kafkaParams = Map[String, Object]( // Create a StreamingContext
  "bootstrap.servers" -> "127.0.0.1:9092",
  "key.deserializer" -> classOf[StringDeserializer],
  "value.deserializer" -> classOf[StringDeserializer],
  "group.id" -> "test-consumer-group",
  "auto.offset.reset" -> "earliest",
  "enable.auto.commit" -> (false: java.lang.Boolean)
)

/** Stop any existing StreamingContext  */
if (true) {	
  StreamingContext.getActive.foreach { _.stop(stopSparkContext = false) }
} 

/** Get or create a streaming context */
val ssc = StreamingContext.getActiveOrCreate(creatingFunc)
if (newContextCreated) {
  println("New context created from currently defined creating function") 
} else {
  println("Existing context running or recovered from checkpoint, may not be running currently defined creating function")
}

/** Array of Kafka topics */
val topics = Array("kafka_main_topic")

/** Stream creation */
val stream = KafkaUtils.createDirectStream[String, String](
  ssc,
  PreferConsistent,
  Subscribe[String, String](topics, kafkaParams)
)

/** Parsing values */
val values = stream.map{ record => record.value}
values.print()

/** Start StreamingContext */
ssc.start()

/** Stops StreamingContext at an specific time interval */
ssc.awaitTerminationOrTimeout(batchIntervalSeconds * 20 * 1000)