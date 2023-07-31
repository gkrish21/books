# Designing Data Intensive Applications

## Chapter 1. Reliable, Scalable, and Maintainable Applications


**Reliability**
  : The system should continue to work correctly (performing the correct function at the desired level of performance) even in the face of adversity (hardware or software faults, and even human error).
roughly, “continuing to work correctly, even when things go wrong.”


**Scalability**
  : As the system grows (in data volume, traffic volume, or complexity), there should be reasonable ways of dealing with that growth. Measuring load & performance. (Latency percentiles, throughput)

**Maintainability**
  : Over time, many different people will work on the system (engineering and operations, both maintaining current behavior and adapting the system to new use cases), and they should all be able to work on it productively. Operability simplicity & evolvability.



Note that a fault is not the same as a failure. A *fault* is usually defined as one component of the system deviating from its spec, whereas a *failure* is when the system as a whole stops providing the required service to the user. It is impossible to reduce the probability of a fault to zero; therefore it is usually best to design fault-tolerance mechanisms that prevent faults from causing failures.


### Describing Performance

In a batch processing system such as Hadoop, we usually care about *throughput*—the number of records we can process per second, or the total time it takes to run a job on a dataset of a certain size. In online systems, what’s usually more important is the service’s response time—that is, the time between a client sending a request and receiving a response.



**LATENCY AND RESPONSE TIME**

*Latency* and *response time* are often used synonymously, but they are not the same. The response time is what the client sees: besides the actual time to process the request (the service time), it includes network delays and queueing delays. Latency is the duration that a request is waiting to be handled—during which it is latent, awaiting service

Usually it is better to use percentiles. If you take your list of response times and sort it from fastest to slowest, then the median is the halfway point: for example, if your median response time is 200 ms, that means half your requests return in less than 200 ms, and half your requests take longer than that.

This makes the median a good metric if you want to know how long users typically have to wait: half of user requests are served in less than the median response time, and the other half take longer than the median. The median is also known as the 50th percentile, and sometimes abbreviated as p50. Note that the median refers to a single request; if the user makes several requests (over the course of a session, or because several resources are included in a single page), the probability that at least one of them is slower than the median is much greater than 50%.

In order to figure out how bad your outliers are, you can look at higher percentiles: the 95th, 99th, and 99.9th percentiles are common (abbreviated p95, p99, and p999). They are the response time thresholds at which 95%, 99%, or 99.9% of requests are faster than that particular threshold. For example, if the 95th percentile response time is 1.5 seconds, that means 95 out of 100 requests take less than 1.5 seconds, and 5 out of 100 requests take 1.5 seconds or more.

High percentiles of response times, also known as tail latencies, are important because they directly affect users’ experience of the service.


## Chapter 2. Data Models and Query Languages

In this chapter we will look at a range of general-purpose data models for data storage and querying. In particular, we will compare the relational model, the document model, and a few graph-based data models. We will also look at various query languages and compare their use cases.

### Relational Model Versus Document Model

The roots of relational databases lie in business data processing, which was performed on mainframe computers in the 1960s and ’70s. The use cases appear mundane from today’s perspective: typically transaction processing (entering sales or banking transactions, airline reservations, stock-keeping in warehouses) and batch processing (customer invoicing, payroll, reporting).

**Not Only SQL**

There are several driving forces behind the adoption of NoSQL databases, including:

- A need for greater scalability than relational databases can easily achieve, including very large datasets or very high write throughput

- A widespread preference for free and open source software over commercial database products

- Specialized query operations that are not well supported by the relational model

- Frustration with the restrictiveness of relational schemas, and a desire for a more dynamic and expressive data model.


Schema-on-read is similar to dynamic (runtime) type checking in programming languages, whereas schema-on-write is similar to static (compile-time) type checking. 


### Query Languages for Data

SQL is a declarative query language, whereas IMS and CODASYL queried the database using imperative code. What does that mean?

An **imperative language** tells the computer to perform certain operations in a certain order. You can imagine stepping through the code line by line, evaluating conditions, updating variables, and deciding whether to go around the loop one more time.

In a **declarative query language**, like SQL or relational algebra, you just specify the pattern of the data you want—what conditions the results must meet, and how you want the data to be transformed (e.g., sorted, grouped, and aggregated)—but not how to achieve that goal. It is up to the database system’s query optimizer to decide which indexes and which join methods to use, and in which order to execute various parts of the query.

**MapReduce** is neither a declarative query language nor a fully imperative query API, but somewhere in between: the logic of the query is expressed with snippets of code, which are called repeatedly by the processing framework. It is based on the map (also known as collect) and reduce (also known as fold or inject) functions that exist in many functional programming languages.

The map and reduce functions are somewhat restricted in what they are allowed to do. They must be pure functions, which means they only use the data that is passed to them as input, they cannot perform additional database queries, and they must not have any side effects. 

### Graph-Like Data Models

A graph consists of two kinds of objects: vertices (also known as nodes or entities) and edges (also known as relationships or arcs).

*Social graphs*
: Vertices are people, and edges indicate which people know each other.

*The web graph*
: Vertices are web pages, and edges indicate HTML links to other pages.

*Road or rail networks*
: Vertices are junctions, and edges represent the roads or railway lines between them.


## Chapter 3. Storage and Retrieval

first we’ll start this chapter by talking about storage engines that are used in the kinds of databases that you’re probably familiar with: traditional relational databases, and also most so-called NoSQL databases. We will examine two families of storage engines: *log-structured storage engines*, and *page-oriented storage engines* such as B-trees.


