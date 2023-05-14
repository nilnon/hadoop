# hadoop-mapreduce源码阅读

## 工程结构

branch-3.3.5

```
├── bin  // 包含MapReduce相关的可执行文件和脚本
│   ├── mapred
│   ├── mapred.cmd
│   ├── mapred-config.cmd
│   ├── mapred-config.sh
│   └── mr-jobhistory-daemon.sh
├── conf // 包含MapReduce的配置文件，如mapred-site.xml和环境配置脚本
│   ├── configuration.xsl
│   ├── mapred-env.cmd
│   ├── mapred-env.sh
│   ├── mapred-queues.xml.template
│   └── mapred-site.xml
├── dev-support // 包含开发支持文件，如FindBugs配置和jdiff工具
│   ├── findbugs-exclude.xml
│   └── jdiff
├── hadoop-mapreduce-client // 包含MapReduce客户端模块的源码和构建文件
│   ├── hadoop-mapreduce-client-app // 负责与YARN资源管理器通信，提交和监控作业
│   ├── hadoop-mapreduce-client-common // 定义了作业配置、计数器、文件输入输出等通用功能
│   ├── hadoop-mapreduce-client-core // 实现了作业的提交、调度、任务分配和执行等核心功能
│   ├── hadoop-mapreduce-client-hs // 用于存储和提供MapReduce作业的历史信息
│   ├── hadoop-mapreduce-client-hs-plugins // 提供了用于扩展和定制作业历史服务器的插件机制
│   ├── hadoop-mapreduce-client-jobclient // 负责与YARN资源管理器通信，提交和监控作业
│   ├── hadoop-mapreduce-client-nativetask // 提供了一种利用本机库加速任务执行的机制
│   ├── hadoop-mapreduce-client-shuffle // 实现了Mapper输出的合并和排序操作
│   ├── hadoop-mapreduce-client-uploader // 用于将作业相关的文件上传到HDFS或其他存储系统
│   ├── pom.xml
│   └── target
├── hadoop-mapreduce-examples // 包含一些MapReduce示例代码
│   ├── dev-support
│   ├── pom.xml
│   ├── src
│   └── target
├── lib // 包含一些额外的依赖库
│   └── jdiff
├── pom.xml
├── shellprofile.d //  包含一些用于设置环境的脚本文件
│   └── hadoop-mapreduce.sh

```

## hadoop-mapreduce-client-common
```
├── java
│   └── org
│       └── apache
│           └── hadoop
│               ├── mapred
│               │   ├── LocalClientProtocolProvider.java // 提供本地客户端协议的实现
│               │   ├── LocalDistributedCacheManager.java // 本地分布式缓存管理器，用于管理分布式缓存的相关操作
│               │   ├── LocalJobRunner.java // 本地作业运行器，用于在本地模式下运行MapReduce作业
│               │   └── LocalJobRunnerMetrics.java //  本地作业运行器的度量信息
│               ├── mapreduce
│               │   ├── TypeConverter.java // 类型转换器，用于在mapred和mapreduce之间进行类型的转换
│               │   └── v2
│               │       ├── api // 包含与MapReduce V2版本相关的API接口
│               │       │   ├── HSAdminProtocol.java
│               │       │   ├── HSAdminRefreshProtocol.java
│               │       │   ├── HSAdminRefreshProtocolPB.java
│               │       │   ├── HSClientProtocol.java
│               │       │   ├── HSClientProtocolPB.java
│               │       │   ├── impl
│               │       │   ├── MRClientProtocol.java
│               │       │   ├── MRClientProtocolPB.java
│               │       │   ├── MRDelegationTokenIdentifier.java
│               │       │   ├── package-info.java
│               │       │   ├── protocolrecords
│               │       │   └── records
│               │       ├── jobhistory // 与作业历史相关的实用类和配置
│               │       │   ├── FileNameIndexUtils.java
│               │       │   ├── JHAdminConfig.java
│               │       │   ├── JobHistoryUtils.java
│               │       │   ├── JobIndexInfo.java
│               │       │   └── package-info.java
│               │       ├── security // 与MapReduce安全性相关的客户端和令牌管理类
│               │       │   ├── client
│               │       │   └── MRDelegationTokenRenewer.java
│               │       └── util // 包含一些实用工具类，用于处理本地资源、构建应用程序、构建MR配置等
│               │           ├── LocalResourceBuilder.java
│               │           ├── MRApps.java
│               │           ├── MRBuilderUtils.java
│               │           ├── MRProtoUtils.java
│               │           ├── MRWebAppUtil.java
│               │           └── package-info.java
│               └── yarn
│                   └── proto
│                       └── HSClientProtocol.java // 基于YARN的历史服务器客户端协议的Protocol Buffer定义
├── proto // 包含一些协议定义文件，用于生成Java代码
│   ├── HSAdminRefreshProtocol.proto
│   ├── MRClientProtocol.proto
│   ├── mr_protos.proto
│   └── mr_service_protos.proto
└── resources
    └── META-INF
        └── services // 包含一些服务提供者的配置文件，用于指定实现特定接口的类
            ├── org.apache.hadoop.mapreduce.protocol.ClientProtocolProvider
            ├── org.apache.hadoop.security.SecurityInfo
            ├── org.apache.hadoop.security.token.TokenIdentifier
            └── org.apache.hadoop.security.token.TokenRenewer

```

### mapred和mapreduce的区别

- mapred是Hadoop早期版本（1.x版本）中的编程模型和框架，它基于经典的MapReduce思想，将数据处理任务划分为Map和Reduce两个阶段。在mapred中，用户需要实现Mapper和Reducer接口来定义数据处理逻辑，并使用JobConf来配置作业。mapred提供了分布式数据处理的能力，可以在大规模集群上执行并行处理。

- mapreduce是Hadoop的新一代编程模型和框架，它在1.x版本的mapred基础上进行了重构和改进。mapreduce的核心概念与mapred类似，仍然包括Map和Reduce两个阶段，但引入了更灵活的API设计和更高级的特性。在mapreduce中，用户可以使用更简单的Mapper和Reducer接口，同时还引入了Combiner、Partitioner、InputFormat、OutputFormat等组件，以增加灵活性和扩展性。此外，mapreduce还支持更多的数据处理模式，如Map-Only、Map-Side Join等。

## hadoop-mapreduce-client-core

### mapreduce核心功能

- Map和Reduce任务执行：该模块包含了执行Map和Reduce任务的相关类和接口。它定义了Mapper和Reducer接口，用于用户自定义的Map和Reduce任务逻辑。还包含了MapContext和ReduceContext等上下文对象，用于任务执行过程中的数据交互和状态管理。

- 分区和排序：该模块提供了分区和排序的相关功能。分区（Partitioning）指的是将Map输出的键值对划分到不同的Reduce任务中，以便进行并行处理。排序（Sorting）指的是对Map输出的键值对进行排序，以满足Reduce任务的输入要求。该模块定义了Partitioner接口和SortComparator接口，用于用户自定义的分区和排序逻辑。

- 输入和输出格式：该模块定义了输入和输出的格式和处理。它包含了InputFormat接口和OutputFormat接口，用于定义MapReduce作业的输入和输出格式。Hadoop提供了多种默认的输入和输出格式，如TextInputFormat、KeyValueTextInputFormat、TextOutputFormat等，同时也支持用户自定义的输入和输出格式。

- 计数器（Counters）：该模块提供了计数器的功能，用于收集和跟踪作业的统计信息。计数器可以用来统计特定事件的发生次数或记录某个阶段的进度等。通过使用Counters类和相关的API，用户可以在作业中定义和更新计数器。

- 任务调度和执行：该模块包含了任务调度和执行的相关类和接口。它定义了TaskScheduler接口和TaskAttemptListener接口，用于任务的调度和执行管理。Hadoop提供了多种默认的任务调度器，如FifoScheduler、CapacityScheduler等，用户也可以自定义任务调度器。

- 任务跟踪和监控：该模块提供了任务跟踪和监控的功能。它定义了JobHistoryEventHandler接口和相关的类，用于收集和处理作业历史数据。通过任务跟踪和监控功能，用户可以获取作业的执行信息、状态和性能指标等。

### mapreduce核心类

- Mapper：用于将输入键/值对映射到一组中间键/值对。
- Reducer：用于将中间键/值对按键进行分组，并对每个组的值进行归约操作，生成最终的输出键/值对。
- InputFormat：用于指定输入数据的格式和如何划分输入数据的类。它定义了读取输入数据的方式，并将其划分为多个InputSplit供Mapper使用。
- OutputFormat：用于指定输出数据的格式和输出位置的类。它定义了将Reducer的输出写入到指定位置的方式。
- Job：表示一个完整的MapReduce作业，它包含了作业的配置信息，包括输入路径、输出路径、Mapper和Reducer的类等。
- JobContext：表示作业的上下文信息，提供了访问作业配置、输入输出路径等信息的方法。
- Task：表示一个MapReduce任务，可以是Mapper任务或Reducer任务。
- TaskAttemptContext：表示任务尝试的上下文信息，提供了访问任务尝试的配置、状态等信息的方法。
- InputSplit：表示输入数据的划分单元，它将输入数据划分为多个片段（split），每个片段由一个Mapper处理。
- Writable：可序列化的数据类型接口，用于在MapReduce作业中传递数据。

这些类是Hadoop MapReduce框架中的核心组件，用于实现MapReduce作业的输入、输出、映射、归约等功能。通过这些类的组合和扩展，可以实现复杂的数据处理任务。



###  Mapper

Hadoop MapReduce Mapper的设计思路是基于"分而治之"（Divide and Conquer）和"移动计算而非数据"（Move computation, not data）的原则

- 分而治之： MapReduce任务将输入数据划分为多个数据块，每个数据块由一个Mapper处理。这种分而治之的思想使得任务可以并行执行，充分利用集群中的计算资源。每个Mapper仅处理自己分配到的数据块，独立地执行自定义的逻辑处理。
- 移动计算而非数据： 在MapReduce模型中，数据存储在分布式文件系统（如HDFS）中，并且在集群中进行移动和处理。Mapper的设计思路是将计算任务移动到存储数据的节点，而不是将数据传输到计算节点。这样可以减少数据的传输开销，提高任务的执行效率。
- 逻辑处理与数据局部性： Mapper的设计充分考虑了数据局部性原则。输入数据划分为数据块后，每个Mapper在本地执行逻辑处理，利用存储在本地的数据块进行计算。这样可以最大程度地减少数据的网络传输，提高处理速度。
- 并行执行和容错性： MapReduce任务以并行的方式执行多个Mapper任务，每个任务都独立处理自己的数据块。这种并行执行能力使得任务可以在分布式环境下快速处理大规模数据。同时，Hadoop MapReduce提供了容错机制，确保即使某个Mapper失败，任务可以继续执行。

```Java
Maps input key/value pairs to a set of intermediate key/value pairs.
将输入键值对映射到一组中间键值对

Maps are the individual tasks which transform input records into a intermediate records. The transformed intermediate records need not be of the same type as the input records. A given input pair may map to zero or many output pairs.
映射是单个任务将输入记录转换为中间记录。转换后的中间记录不需要与输入记录属于同一类型。给定的输入对可以映射到零个或多个输出对。

The Hadoop Map-Reduce framework spawns one map task for each InputSplit generated by the InputFormat for the job. Mapper implementations can access the Configuration for the job via the JobContext.getConfiguration().
Hadoop Map-Reduce 框架为作业的 InputFormat 生成的每个 InputSplit 生成一个映射任务。 映射器实现可以通过 JobContext.getConfiguration() 访问作业的配置

The framework first calls setup(Mapper.Context), followed by map(Object, Object, Mapper.Context) for each key/value pair in the InputSplit. Finally cleanup(Mapper.Context) is called.
该框架首先调用 setup(Mapper.Context)，然后为 InputSplit 中的每个键/值对调用 map(Object, Object, Mapper.Context)。 最后调用 cleanup(Mapper.Context)

All intermediate values associated with a given output key are subsequently grouped by the framework, and passed to a Reducer to determine the final output. Users can control the sorting and grouping by specifying two key RawComparator classes.
与给定输出键关联的所有中间值随后由框架分组，并传递给 Reducer 以确定最终输出。 用户可以通过指定两个关键的 RawComparator 类来控制排序和分组

The Mapper outputs are partitioned per Reducer. Users can control which keys (and hence records) go to which Reducer by implementing a custom Partitioner.
Mapper 输出按 Reducer 进行分区。 用户可以通过实现自定义分区程序来控制哪些键（以及记录）进入哪个 Reducer

Users can optionally specify a combiner, via Job.setCombinerClass(Class), to perform local aggregation of the intermediate outputs, which helps to cut down the amount of data transferred from the Mapper to the Reducer.
用户可以通过 Job.setCombinerClass(Class) 选择性地指定一个组合器来执行中间输出的本地聚合，这有助于减少从 Mapper 传输到 Reducer 的数据量

Applications can specify if and how the intermediate outputs are to be compressed and which CompressionCodecs are to be used via the Configuration.
应用程序可以指定是否以及如何压缩中间输出，以及要通过配置使用哪些 CompressionCodec

If the job has zero reduces then the output of the Mapper is directly written to the OutputFormat without sorting by keys.
如果作业有零减少，则 Mapper 的输出将直接写入 OutputFormat，而不按键排序

@InterfaceAudience.Public
@InterfaceStability.Stable
public class Mapper<KEYIN, VALUEIN, KEYOUT, VALUEOUT> {

  /**
   * The <code>Context</code> passed on to the {@link Mapper} implementations.
   */
  public abstract class Context
    implements MapContext<KEYIN,VALUEIN,KEYOUT,VALUEOUT> {
  }
  
  /**
   * Called once at the beginning of the task.
   */
  protected void setup(Context context
                       ) throws IOException, InterruptedException {
    // NOTHING
  }

  /**
   * Called once for each key/value pair in the input split. Most applications
   * should override this, but the default is the identity function.
   */
  @SuppressWarnings("unchecked")
  protected void map(KEYIN key, VALUEIN value, 
                     Context context) throws IOException, InterruptedException {
    context.write((KEYOUT) key, (VALUEOUT) value);
  }

  /**
   * Called once at the end of the task.
   */
  protected void cleanup(Context context
                         ) throws IOException, InterruptedException {
    // NOTHING
  }
  
  /**
   * Expert users can override this method for more complete control over the
   * execution of the Mapper.
   * @param context
   * @throws IOException
   */
  public void run(Context context) throws IOException, InterruptedException {
    setup(context);
    try {
      while (context.nextKeyValue()) {
        map(context.getCurrentKey(), context.getCurrentValue(), context);
      }
    } finally {
      cleanup(context);
    }
  }
}
```

总结：Mapper负责将输入数据映射为中间结果，并通过一系列阶段处理和准备数据，以供Reducer进行进一步处理和最终输出。

- 映射功能： Mapper将输入键/值对映射为一组中间键/值对。每个输入记录可以转换为零个或多个输出记录。
- 任务执行流程： 每个InputSplit生成一个映射任务。首先调用setup(Mapper.Context)方法，然后对于InputSplit中的每个键/值对，调用map(Object, Object, Mapper.Context)方法进行映射处理。最后调用cleanup(Mapper.Context)方法。
- 中间输出和分组： 框架根据键对中间值进行分组，并将其传递给Reducer以进行最终输出。用户可以通过自定义RawComparator类来控制排序和分组。
- 分区和Reducer： Mapper输出按照Reducer进行分区。用户可以实现自定义分区程序来控制哪些键进入哪个Reducer。
- 本地聚合和组合器： 用户可以选择性地指定一个组合器（Combiner）来执行中间输出的本地聚合，减少数据传输到Reducer的量。
- 压缩和配置： 应用程序可以指定是否以及如何压缩中间输出，并通过配置选择使用哪种CompressionCodec。
- 无减少情况： 如果作业没有Reducer，则Mapper的输出将直接写入OutputFormat，而不进行键排序。

###  Reducer

```Java
Reduces a set of intermediate values which share a key to a smaller set of values.
Reducer是将一组共享键的中间值减少为一组较小的值的过程。

Reducer implementations can access the Configuration for the job via the JobContext.getConfiguration() method.
Reducer的实现可以通过JobContext.getConfiguration()方法访问作业的配置。

Reducer has 3 primary phases:
Reducer主要分为3个阶段：

1. Shuffle
The Reducer copies the sorted output from each Mapper using HTTP across the network.
Reducer使用HTTP通过网络从每个Mapper复制排序后的输出。

2. Sort
The framework merge sorts Reducer inputs by keys (since different Mappers may have output the same key).
The shuffle and sort phases occur simultaneously i.e. while outputs are being fetched they are merged.
框架通过键对Reducer输入进行合并排序（因为不同的Mapper可能输出相同的键）。
洗牌和排序阶段同时进行，即在获取输出的同时进行合并。

SecondarySort
To achieve a secondary sort on the values returned by the value iterator, the application should extend the key with the secondary key and define a grouping comparator. The keys will be sorted using the entire key, but will be grouped using the grouping comparator to decide which keys and values are sent in the same call to reduce.The grouping comparator is specified via Job.setGroupingComparatorClass(Class). The sort order is controlled by Job.setSortComparatorClass(Class).
为了对值迭代器返回的值进行二次排序，应用程序应该通过扩展键来添加二级键，并定义一个分组比较器。键将使用完整的键进行排序，但使用分组比较器进行分组，以确定哪些键和值在同一次调用reduce中发送。分组比较器通过Job.setGroupingComparatorClass(Class)进行指定。排序顺序由Job.setSortComparatorClass(Class)控制。

For example, say that you want to find duplicate web pages and tag them all with the url of the "best" known example. You would set up the job like:

- Map Input Key: url
- Map Input Value: document
- Map Output Key: document checksum, url pagerank
- Map Output Value: url
- Partitioner: by checksum
- OutputKeyComparator: by checksum and then decreasing pagerank
- OutputValueGroupingComparator: by checksum

3. Reduce
In this phase the reduce(Object, Iterable, Reducer.Context) method is called for each <key, (collection of values)> in the sorted inputs.
The output of the reduce task is typically written to a RecordWriter via Reducer.Context.write(Object, Object).
在此阶段，reduce(Object, Iterable, Reducer.Context)方法针对排序后的输入中的每个<键，（值的集合）>调用。
reduce任务的输出通常通过Reducer.Context.write(Object, Object)写入到RecordWriter中。

@Checkpointable
@InterfaceAudience.Public
@InterfaceStability.Stable
public class Reducer<KEYIN,VALUEIN,KEYOUT,VALUEOUT> {

  /**
   * The <code>Context</code> passed on to the {@link Reducer} implementations.
   */
  public abstract class Context 
    implements ReduceContext<KEYIN,VALUEIN,KEYOUT,VALUEOUT> {
  }

  /**
   * Called once at the start of the task.
   */
  protected void setup(Context context
                       ) throws IOException, InterruptedException {
    // NOTHING
  }

  /**
   * This method is called once for each key. Most applications will define
   * their reduce class by overriding this method. The default implementation
   * is an identity function.
   */
  @SuppressWarnings("unchecked")
  protected void reduce(KEYIN key, Iterable<VALUEIN> values, Context context
                        ) throws IOException, InterruptedException {
    for(VALUEIN value: values) {
      context.write((KEYOUT) key, (VALUEOUT) value);
    }
  }

  /**
   * Called once at the end of the task.
   */
  protected void cleanup(Context context
                         ) throws IOException, InterruptedException {
    // NOTHING
  }

  /**
   * Advanced application writers can use the 
   * {@link #run(org.apache.hadoop.mapreduce.Reducer.Context)} method to
   * control how the reduce task works.
   */
  public void run(Context context) throws IOException, InterruptedException {
    setup(context);
    try {
      while (context.nextKey()) {
        reduce(context.getCurrentKey(), context.getValues(), context);
        // If a back up store is used, reset it
        Iterator<VALUEIN> iter = context.getValues().iterator();
        if(iter instanceof ReduceContext.ValueIterator) {
          ((ReduceContext.ValueIterator<VALUEIN>)iter).resetBackupStore();        
        }
      }
    } finally {
      cleanup(context);
    }
  }
}
```

总结：

Reducer是MapReduce框架中的一个重要组件，用于将一组共享键的中间值减少为一组较小的值。它通过洗牌和排序阶段获取来自不同Mapper的输出，并在Reduce阶段对排序后的输入进行处理。通过实现Reducer的reduce方法，可以对相同键的值进行合并、计算和聚合。Reducer还支持二次排序和自定义分组逻辑，以控制输出结果的排序和分组方式。最终，Reducer将结果写入到指定的输出位置，供后续处理和分析使用。

## hadoop-mapreduce-client-app
## hadoop-mapreduce-client-jobclient
## hadoop-mapreduce-client-hs
## hadoop-mapreduce-client-shuffle
