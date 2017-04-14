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

1. 读取数据
2. 对数据进行各种处理
3. 对数据进行写操作

例如, 打开一个CSV格式的数据文件,对文件中的数据执行某种处理,然后将数据写入数据库。 在Spring Batch中, 需要配置一个 reader 来读取文件中的数据(每次一行), 然后将数据传递给 processor 进行处理, 处理完成之后会将结果收集并分组为 “块 chunks” , 然后把这些记录发送给 writer ,在这里是插入到数据库中。 如图1所示。

![Spring Batch批处理的基本逻辑](./fig1-basicl-ogic.png)
**图1 Spring Batch批处理的基本逻辑**


Spring Batch 提供了常见输入源的 reader 实现, 极大地简化了批处理过程. 例如 CSV文件, XML文件、数据库、文件中的JSON记录,甚至是 JMS; 同样也实现了对应的 writer。 如有需要,创建自定义的 reader 和 writer 也很简单。


首先,让我们配置一个 file reader 来读取 CSV文件,将其内容映射到一个对象中,并将生成的对象插入数据库中。
## 读取并处理CVS文件 ##


Spring Batch 的内置 reader,  `org.springframework.batch.item.file.FlatFileItemReader`,用来将文件解析为多个独立的行。 需要纯文本文件的引用,文件开头要忽略的行数(比如标题,表头等信息), 以及将单行转换为对象的 `line mapper`. 行映射器需要一个分割字符串的分词器(line tokenizer),用来将一行拆分成多个组成字段, 还需要一个 field set mapper ,根据字段值构建对象。  `FlatFileItemReader` 的配置如下所示:

> **清单1 一个Spring Batch 配置文件**

<bean id="hfReader" class="org.springframework.batch.item.file.FlatFileItemReader" scope="step">
        <property name="resource" value="file:#{jobParameters['hfInputFile']}" />
        <!-- Skip the first line of the file because this is the header that defines the fields -->
        <property name="linesToSkip" value="0" />
        <!-- Defines how we map lines to objects -->
        <property name="lineMapper">
            <bean class="org.springframework.batch.item.file.mapping.DefaultLineMapper">
                <!-- The lineTokenizer divides individual lines up into units of work -->
                <property name="lineTokenizer">
                    <bean class="org.springframework.batch.item.file.transform.DelimitedLineTokenizer">
                    	<property name="delimiter" value="|"/>
                        <!-- Names of the CSV columns -->
                        <property name="names" value="payNumber,tradeDate,bizType,poundage,tradeAmount,rmb,date,status,text" />
                    </bean>
                </property>
                <!-- The fieldSetMapper maps a line in the file to a Product object -->
                <property name="fieldSetMapper">
                	<bean class="com.miz.recon.reader.HfFieldSetMapper"/>
                </property>
            </bean>
        </property>
    </bean>
    
    ![FlatFileItemReader组件](./fig2-FlatFileItemReader.png)
**图2 FlatFileItemReader的组件**


**Resources:**  `resource` 属性指定了要读取的文件。 注释部分的 resource 使用了文件的相对路径,也就是批处理作业工作目录下的 `sample.csv` 。 有趣的是使用了Job参数 `InputFile` : 使用*job parameters* 则允许在运行时才根据需要决定相关参数。 在使用 import 文件的情况下, 在运行时才决定使用哪个参数比起在编译时就固定要灵活好用很多。 (要一遍又一遍,五六七八遍导入同一个文件是相当无聊的!)


**Lines to skip:**  属性`linesToSkip` 告诉 file reader 有多少标题行需要跳过。 通常CSV文件的第一行包含标题信息,如列名称,所以本例中让 reader 跳过文件的第一行。


**Line mapper:**   `lineMapper` 负责将一行记录转换为一个对象。 依赖两个组件:

- `LineTokenizer` 指定了如何将一行拆分为多个字段。 本例中列出了CSV文件中各列的列名。
- `fieldSetMapper` 根据字段值构造一个对象。 示例中构建了一个  `Product` 对象, 属性包括  id, name, description, 以及 quantity 。



请注意,虽然Spring Batch提供了基础框架, 但我们仍需要设置字段映射的逻辑。 清单2显示了 *Product* 对象的源码,也就是我们准备构建的对象。
