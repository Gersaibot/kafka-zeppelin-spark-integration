%spark
 
import _root_.kafka.serializer.DefaultDecoder
import _root_.kafka.serializer.StringDecoder
import org.apache.spark.streaming.kafka.KafkaUtils
import org.apache.spark.storage.StorageLevel
import org.apache.spark.streaming._

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
val kafkaConf = Map(
  "metadata.broker.list" -> "127.0.0.1:9092",
  "zookeeper.connect" -> "127.0.0.1:2181",
  "group.id" -> "test-consumer-group",
  "zookeeper.connection.timeout.ms" -> "6000",
  "auto.offset.reset" -> "smallest"
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


/**
 * Stream creation ans subscribing to topic 
 * and partition 1.
 */
val stream = KafkaUtils.createStream[Array[Byte], String, DefaultDecoder, StringDecoder](
  ssc,
  kafkaConf,
  Map("kafka_main_topic" -> 1),   // subscripe to topic and partition 1
  StorageLevel.MEMORY_ONLY
)

/** Parsing values */
val values = stream.map{ case(x, y) => y}
values.print()

/** Start StreamingContext */
ssc.start()

/** Stops StreamingContext at an specific time interval */
ssc.awaitTerminationOrTimeout(batchIntervalSeconds * 20 * 1000)