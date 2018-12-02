# The essentials of custom coding in stream flows

By Boris Melamed, Watson Studio development, Nov 2018

**Audience**: data scientists, data engineers.

##Introduction

Data professionals can use stream flows to quickly put together realtime streaming applications without having to write flow *topology* code. Users design flows by using a web-based drag-and-drop UI, and by specifying parameter values. As for the business logic of data processing, some of it can also be implemented by using specialized operators, with no coding. We are continuously adding more out-of-the-box coverage, but there might always be cases where custom coding is required.

This article clarifies when to use which type of custom code operator, and recommends best practices. For more details about stream flows, see [streams flow documentation](https://dataplatform.cloud.ibm.com/docs/content/streaming-pipelines/overview-streaming-pipelines.html). &nbsp;&nbsp;&nbsp;&nbsp;

(Note: Engineers who want to code streams on a more direct level can look at [Streams Studio](https://www.ibm.com/support/knowledgecenter/SSCRJU_4.3.0/com.ibm.streams.welcome.doc/doc/studio.html), [Lightweight Streams IDEs](https://developer.ibm.com/streamsdev/docs/develop-run-streams-applications-using-atom-visual-studio-code/), and [Topology SDKs in Java or Python](http://ibmstreams.github.io/streamsx.topology/doc.html).)

So when can you rely on specialized streams flow operators like Filter, and when is it best to write your own code, and how?

## Specialized operators or custom code?

When constructing Streams Flows, you can drag and drop various operators from the palette onto the canvas. 

----------------------------------------------------------
#### As a rule: **If there is a specialized operator that fits the bill, *use it!***
----------------------------------------------------------
### Some specialized operators

- Sources (ingestion): IBM Event Streams, IBM WatsonIoT, MQTT.
- Targets (sinks): IBM Cloudant, IBM Event Streams, IBM Cloud Object Storage, Redis.
- Processing and analytics: Filter, Aggregation (count, sum, average, min, max, standard deviation).

<div style="page-break-after: always;"></div>

Here is a simple flow example that filters data using the specialized Filter operator:

![image-20181105132503051](./assets/image-20181105132503051.png)

But what if there is no specialized operator for what you are building?

### Cases for custom code

- Connect to sources and targets for which there is no specialized operator in the palette.
  Examples: MongoDB, Cassandra.
- Specific business logic data transformations and calculations, beyond what is covered by Filter and the available Aggregation functions.
- Parse, convert, or format data according to a format that is not supported in specialized operators. 
  Examples: 
  - Parse dates and times that arrive as strings in non-ISO-8601 format.
  - Parse data that's arriving in Avro format.
- Pre- and postprocess data for model scoring.
- Advanced data extraction using regular expressions and other types of custom parsing.
- Easily generate sample data, such as for incremental development or demos.

## Cloud functions or (Python) code operators?

These are the operators for inserting custom code:

| Operator       | Located in these palette sections of the canvas: | Official documentation                                       |
| -------------- | ------------------------------------------------ | ------------------------------------------------------------ |
| Cloud Function | "Targets", "Processing"                          | [Here](https://dataplatform.cloud.ibm.com/docs/content/streaming-pipelines/cloud_functions.html) |
| Code           | "Source", "Processing"                           | [Here](https://dataplatform.cloud.ibm.com/docs/content/streaming-pipelines/code.html) |

<div style="page-break-after: always;"></div>

So which one is best to use? It depends. This table points out the differences to help you pick the right approach for each use case.

### Comparison table

| Aspect                                          | Cloud Function                                               | Code                                                         |
| ----------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Available as source?                            | Not available as source.                                     | Yes. Can be:<br />- sync, such as streaming DB table data<br />- async, such as websocket subscriber<br />**(Advantage)**<br /><br />**Reminder**: Whenever there is a specialized source operator such as the Event Streams operator that fits your requirements, use it rather than writing custom code. |
| Available as target?                            | Yes                                                          | Yes, as a processor with Debug target operator as output.    |
| Available as processor?                         | Yes                                                          | Yes                                                          |
| Programming language and deployment environment | Any language and environment of your choice, since you can deploy cloud functions in your own docker image on IBM Cloud. <br />**(Advantage)** | Language: Python 3.5.<br />Environment: Your Streaming Analytics Instance.<br />You can extend your environment by [adding Python packages](https://dataplatform.cloud.ibm.com/docs/content/streaming-pipelines/installing_Python_libs.html). |
| Latency, performance                            | High invocation latency                                      | Low invocation latency <br />**(Advantage)**                 |
| In-memory, low-latency state across invocations | No in-memory state across invocations. State can be kept in a persistent store, such as Redis. Loading this state every time can further add to invocation latency. | Yes <br />**(Advantage)**                                    |
| Billing cost overhead                           | Invocation billing cost may accumulate.                      | There is no additional billing cost for every invocation.<br />**(Advantage)** |

<div style="page-break-after: always;"></div>

## Develop Code operators effectively

For enhanced productivity, it can help to combine built-in coding support with external tools.

#### Built-in support

The built-in coding support includes:

- Python syntax highlighting and validation, as you type. 
- Logging user messages and raising user errors. At runtime, they all conveniently appear as notifications in the streams flow UI, on the Metrics page. You can also [download the user log](https://dataplatform.cloud.ibm.com/docs/content/streaming-pipelines/downloading_logs.html?context=analytics) which contains these user messages and errors.
- As with all streams flow operators, at runtime, you can view sampled data and throughput rate at the inputs and outputs of code operators.

#### Utilizing external tools

For support that complements built-in coding assistance:

1. Use an external tool for authoring your code, such as:

   a. your favorite **editor** or **IDE**, with support for:

      - auto-complete, refactor, auto-layout, find/replace, ...
      - test/debug your business logic
      - version control integration

   b. or a **notebook**, where you can quickly run and test your code.

2. Use the developed code and:
   - copy it into the code operator editor, arranging it appropriately inside the applicable callback functions (`init`, `process`, `produce`) ,
   - or turn it into an **external code package**, to use from within your code operator.
     [Here](https://dataplatform.cloud.ibm.com/docs/content/streaming-pipelines/installing_Python_libs.html) is how to install Python packages and use them in Streams Flows code operators.

**Important**: In all these cases, keep in mind that in stream flows, this code will be run in a **Python 3.5** interpreter.

#### Known issues

When writing custom code, take into account limitations of streams flow development:

- The debug facilities and runtime customization are limited.
- There is no built-in version control.

For coding streams on a more direct level with full development support, have a look at [Streams Studio](https://www.ibm.com/support/knowledgecenter/SSCRJU_4.3.0/com.ibm.streams.welcome.doc/doc/studio.html), [Lightweight Streams IDEs](https://developer.ibm.com/streamsdev/docs/develop-run-streams-applications-using-atom-visual-studio-code/), and [Topology SDKs in Java or Python](http://ibmstreams.github.io/streamsx.topology/doc.html).

<div style="page-break-after: always;"></div>

## Conclusion

This article can help data professionals quickly build realtime streaming applications by designing stream flows, with no or minimal custom coding. 

We are giving the following recommendations regarding custom code:

- Where available, use specialized operators rather than writing custom code.
- There are typical use cases that require writing custom code.
- Based on the provided comparison table, pick between a Code operator and a Cloud Function operator, if you need to write custom code.
- Consider productivity-enhancing practices when authoring code in Code operators.

 Let us know about functional gaps that you would like to be covered by specialized operators.

