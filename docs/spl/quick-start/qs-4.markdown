---
layout: docs
title:  Tips for creating Streams applications
description:
weight: 70
published: true
tag: spl-qs
navlevel: 2
prev:
  file: qs-3
  title:  FAQ
---


*This document is a work in progress. Feel free to add tips here*.

You have an idea of what your application should do, e.g. "Detect when a moving bus passes the known points of interest and send any alerts to the bus".

How should you design your application?

First, recall the Streams application pattern:

![](/streamsx.documentation/images/atom/jpg/pattern.jpg)

All Streams applications have this basic pattern: ingest the data, filter/discard any data that is uninteresting or erroneous. Next, analyze or process the data and act on the results.

Each stage in the diagram above is handled by one or more operators.


For example, an application called `BusAlerts` tracks public transit vehicles as the vehicles move around the city. The buses periodically report their location. When a bus is near an area with a point of interest (POI), the application detects this and sends an alert.

For example, if a bus comes within 1km of the Golden Gate Bridge in San Francisco, you want to display this message inside the bus: “Approaching Golden Gate Bridge, pedestrian bridge is closed.”

The [`BusAlerts` application](https://developer.ibm.com/streamsdev/docs/common-patterns-tracking-moving-objects-streams-part-2-geofencing/) might be broken down into these phases:

![](/streamsx.documentation/images/atom/jpg/phases2.jpg)

The bus data is *ingested*, buses near POIs are *detected*, and then the
*alert* is sent.

So, a good step is to try to break down your application into the various steps that will need to be carried out.

Build your application incrementally
------

To follow the above pattern, you should build your application incrementally.

Instead of adding all the operators at once, **the best way to create your application is to do so in stages, starting small and progressively adding complexity.**

-   Ingest the data and validate that the data you are working with is
    correct.

-   Then, add operator(s) to further refine the data, with validation again as the last step.

-   Add more complex analytics and logic

-   Incorporate reporting, alerts and visualization.

So let's start with the first step in your streaming application, which is acquiring data for processing.

Acquiring input data
--------

Since all Streams applications start with a data ingestion step, this is the first stage of your application.

If you do not yet have data to ingest, you can skip to the "generating sample data" section for a few tips on generating data for your application.

### Pick a source operator

{% include faq/answers/data-sources.html %}


### Define the incoming schema and use it with the source operator

1. Define the output schema that describes each incoming tuple:

    For example, a type for tuples representing information about a bus:
    
     `type Bus = rstring id, rstring name, int32 id, timestamp last_seen, float32 latitude, float32 longitude;`

2. Data conversion

    If your data is in a different format, such as JSON or XML string, or a binary blob, it will need to be converted to Streams tuples.

    - JSON data: Use the [JSONToTuple`](https://github.com/IBMStreams/streamsx.json/wiki/JSONToTuple-Operator) operator to convert the JSON string to SPL tuples.

    - XML: Use the `XMLParse` operator to convert XML text to tuples.

    - Binary data: Parse operator converts data from `blob` type.
  
    - Apache Avro: **AvroToJSON** operator in the [Avro toolkit](https://github.com/IBMStreams/streamsx.avro)


3. Verify the data is correct. Create a small application that ingests the data and then prints it to console with a `Custom` operator.

    Example 1: no parsing needed

    //change this to match the tuples you expect

    type RawDataType = int32 id, rstring name, rstring timestamp;

        composite MyApp {

        graph

        stream<RawDataType> DataFromXYZ = XYZSource() {

        }

        () as DataPrinter = Custom(DataFromXYZ as port0) {

            logic

            onTuple port0: {

              printStringLn("New Tuple : " + ((rstring)(port0)));

            }
            }

The `DataPrinter` operator will almost always be more or less the same as shown above.

If you are not using the Streaming Analytics service and you have access to the local filesystem, you could also write the incoming data to a file using a **FileSink** and verify the output file's contents.


### Generating data

Sometimes you have to generate data to test your application.


-   Use a **Beacon** operator to [generate data](https://github.com/IBMStreams/samples/blob/main/Examples-for-beginners/003_sink_at_work/sample/sink_at_work.spl#L16)

-   More complex samples can be generated using a `Custom` operator. This example will create a stream called `SampleData` and will run until the application is terminated.
    
    	stream<rstring id, float64 temp>  SampleData = Custom()
		{
			logic
				onProcess :
				{
					while (! isShutdown()) { //run indefinitely
						// Randomly generate entities in Hong Kong
						float64 maxTemp = 50.0;
						float64 temperature = (random() * maxTemp);
                        float64 range = 100.0;
						float64 randomNum = random();
	                    rstring user_id = "User_" + ((rstring)((int32)(randomNum * range))) ;
						// submit tuple
						submit({ id = user_id, 
                                temp = temperature}, SampleData) ;
						block(0.1) ; //wait 0.1 seconds before submitting a new tuple
					}
				}

		}


## Where to find examples

-   Samples for most toolkits are included in the toolkit repository in
    the samples folder.

-   You can also search the [Streams Samples catalog](https://ibmstreams.github.io/samples) for examples. Click
    download zip to download the sample that you can import into Streams
    Studio, Atom or VSCode.

-   [Streamsdev](https://developer.ibm.com/streamsdev) also has articles and tutorials, search there.
