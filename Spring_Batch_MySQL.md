# 什么是SpringBatch?
一个轻量级的、全面批处理框架，是一个强大，高效处理大数据的批处理应用程序。

# SpringBatch特性？
Spring Batch提供可重用的功能,在处理大量的记录是至关重要的,包括日志记录、跟踪、事务管理、作业处理统计,工作重新启动,跳过和资源管理。它还提供了更先进的技术服务和功能,将使非常大容量,通过优化和高性能的批处理作业分区技术。简单和复杂、高容量的批处理作业可以利用框架高度可伸缩的方式来处理重要的信息。
1.Transaction management
2.Chunk based processing
3.Declarative I/O
4.Start/Stop/Restart
5.Retry/Skip
6.Web based administration interface (Spring Batch Admin)

# 对账系统使用SpringBatch目的
使用批处理程序来操作动态上GB的数据很可能会拖死整个系统，但现在我们可以通过Spring Batch将其拆解为多个小块（chunk）.Spring框架中的Spring Batch模块，是专门设计了用于对各种类型文件进行批处理的工程。

# SpringBatch原理
Spring Batch 有很多组成部分,我们先来看批量作业中的核心部分。 可以将一个作业分成以下3个步骤:

1.读取数据
2.对数据进行各种处理
3.对数据进行写操作

例如, 打开一个CSV格式的数据文件,对文件中的数据执行某种处理,然后将数据写入数据库。 在Spring Batch中, 需要配置一个 reader 来读取文件中的数据(每次一行), 然后将数据传递给 processor 进行处理, 处理完成之后会将结果收集并分组为 “块 chunks” , 然后把这些记录发送给 writer ,在这里是插入到数据库中。 如图1所示。

Spring Batch批处理的基本逻辑 图1 Spring Batch批处理的基本逻辑

Spring Batch 提供了常见输入源的 reader 实现, 极大地简化了批处理过程. 例如 CSV文件, XML文件、数据库、文件中的JSON记录,甚至是 JMS; 同样也实现了对应的 writer。 如有需要,创建自定义的 reader 和 writer 也很简单。

下载本教程的源代码: SpringBatch-CSV演示代码

首先,让我们配置一个 file reader 来读取 CSV文件,将其内容映射到一个对象中,并将生成的对象插入数据库中。
