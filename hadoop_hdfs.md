## 源代码学习之 —— HDFS

</br>

<font color="#6495ED" size="4dp">下面简单
了解以下HDFS</font>

* <font size="2dp">HDFS全称为分布式文件系统，具有高容错特点，它可以部署在廉价的通用硬件上，提供高吞吐率的数据访问，适合哪些需要处理那些<font color="#FF00FF">需要处理海量数据集的应用程序</font>。
* HDFS的主要特性如下：
  - 支持超大文件。超大文件指的主要时P、T级的数据，Hadoop文件系统能够支持这样海量级的数据。
  - 检测和快速应对硬件故障。在大量通用硬件平台构建集群时，故障，特别时硬件故障时常见的问题。一般的HDFS系统都是由数百台甚至上千台存储着数据文件的服务器组成，这么多的服务器意味着高故障率。所以说，HDFS的故障检测和自动恢复方面做的很优秀。
  - 流式数据访问。HDFS处理的数据模块都比较大，应用一次需要访问大量的数据。同时，这些应用一般时批量处理，而不是用户交互式处理。HDFS使应用程序能够以流式的形式访问数据集，<font color="#FF00FF">注重的使数据的吞吐量，而不是数据访问的速度。</font>
  - 简化的一致性模型。大部分的HDFS程序操作文件时需要一次写入，多次读取。在HDFS中，一个文件一旦经过创建、写入、关闭后，一般就不需要修改了。这样简单的一致性模型，有利于提高吞吐量的数据访问模型。
  - 低延迟数据访问。低延迟数据，就像和用户进行交互的应用，需要数据在毫秒或秒的范围内得到响应。由于Hadoop针对高数据吞吐做了优化，与此牺牲了获取数据的延迟。（低延迟访问可以考虑 HBase 或 Cassandra）
  - 大量的小文件。HDFS支持超大文件，是通过将数据分布在数据节点（DataNode），并将文件的元数据保存在名字节点（NameNode）上。<font color="#FF00FF">NameNode的内存大小，决定了HDFS文件系统可保存的文件数量</font>，虽然现在的系统内存都比较大，但大量的小文件还是会影响NameNode 的性能。
* HDFS体系结构
  - 为了支持流式数据访问和存储超大文件，HDFS引入了一些比较特殊的设计，在一个全配置的集群上，<font color=""#FF00FF"">“运行HDFS”</font>意味着网络分布的不同服务器上运行一些守护进程（daemon），这些进程有各自的特殊角色，并且相互配合，一起形成一个分布式文件系统。
  - HDFS采用了主从（Master/Slave）体系结构，NameNode、DataNode和客户端Client是HDFS中3个重要的角色。
  - 在一个HDFS中，有一个NameNode 和 一个 SecondaryNameNode，典型的集群有几十到几百个数据节点，规模大的系统可以有上千、甚至几千个数据节点；而客户端一般情况下，比数据节点的个数还多。NameNode 和 SecondaryNameNode、DataNode和 Client 的关系如下图所示：
  - ![Alt text](http://m.qpic.cn/psb?/V12dDOqO2iwHZM/RkjVkMWlsrZlGeNUSKNB6q00Vy2y*4J7Pk9.uyxgh5M!/b/dL4AAAAAAAAA&bo=qwMpAgAAAAADB6E!&rf=viewer_4 "optional title")
  - NameNode 可以看作是分布式文件系统中的管理者，它负责管理文件系统<font color="#FF00FF">命名空间、集群配置和数据块复制等</font>。
  - DataNode是文件存储的基本单元，它以数据块的形式保存了HDFS中文件的内容和数据块的数据校验信息。
  - Client和NameNode、DataNode通信，访问HDFS文件系统，操作文件。
    </font> 

</br></br>

<font color="#6495ED" size="3dp">HDFS中数据块的概念是什么？有哪些优势？</font>
</br>
![Alt text](http://a3.qpic.cn/psb?/V12dDOqO2iwHZM/zSm8FOETEvUthNzubb.QR5k*PG8h4VX9r*.JthJxJf8!/m/dLYAAAAAAAAAnull&bo=8ADwAAAAAAABByA!&rf=photolist&t=5"optional title")

<font size="2dp">
    HDFS数据块的概念是一个很大的单元，默认的HDFS数据块大小是64MB.和普通的文件系统类似，HDFS上的文件也进行分块，块作为单独的存储单元，是HDFS的文件存储处理的单元。
</font>
<font size="2dp">HDFS是针对大文件设计的分布式系统，使用数据块带来了很多好处，具体如下：</br>
      1）HDFS可以保存比存储节点单一磁盘大的文件。<br>文件块可以保存在不同的磁盘上。其实，在HDFS中，文件数据可以存放在集群上的任何一个磁盘上，不需要保存在同一个磁盘上，或同一个机器的不同磁盘上。<br>  2）简化了存储子系统。<br>  将管理“块”和管理“文件”的功能区分凯，简化了存储管理，也消除了分布式管理文件元数据的复杂性。<br>  3）方便容错，有利于数据复制。<br>  在HDFS中，为了应对损坏的块以及磁盘、机器故障，数据块会在不同的机器上进行复制<font color="#FF00FF">（一般副本数为3，即一份数据保存在3个不同的地方），如果一个数据块副本丢失或者损坏了，系统会在其他地方读取副本，</font>这个过程对用户来说是透明的，它实现了分布式系统中的位置透明性和故障透明性。同时，一个因损坏或机器故障而丢失的块会从其他地方复制到某一个政策运行的机器，以保证副本数目恢复到正常水平。该正常水平的副本数，也称副本系数。
</font>
<br><br><br>
<font color="#DAA520" size="3dp">HDFS中的节点详解</font><br>  
<img src="http://a3.qpic.cn/psb?/V12dDOqO2iwHZM/qhf*94ct2Zl5SAFj2gFXBfpXkFWQW2pX.Vbzs37.D7g!/m/dD4BAAAAAAAAnull&bo=SANBAgAAAAADByo!&rf=photolist&t=5" width="600px" height="400px"/>
    <br>
<font>**1、NameNode**</font><br>  
<font size="2dp">NameNode 是HDFS主从结构中主节点上运行的主要进程，它是HDFS的书记员，维护者整个文件系统的文件目录树，文件/目录的元信息和文件的数据块索引，即每个文件对应的数据块列表<font color="#FF00FF">（NameNode第一关系）。</font>这些信息以两种形式存储在本地文件系统中：一种是**命名空间镜像（File System Image，FSImage，也称文件系统镜像**，另一种是**命名空间镜像的日志（Edit Log）**。<br>  
**FSImage**保存着某一特定时刻HDFS的目录树、元信息和数据块索引等信息，后续对这些信息的改动，则保存在**EditLog**中，它们一起提供了一个完整的**NameNode第一关系**<br>  
同时，通过NameNode，客户端还可以了解到数据块所在的数据节点信息。<font color="#FF00FF">需要注意的是，NameNode与DataNode相关信息不保留在NameNode的本地系统中，也就是FSImage和EditLog中，NameNode每次启动时都会动态地重建这些信息，</font>这些信息就构成了**NameNode第二关系**。运行时，客户端通过NameNode获取上述信息，然后和DataNode进行交互，读写文件数据。<br>
&emsp;&emsp;另外，NameNode还能获取HDFS整体运行状态的一些信息，如系统的可用空间、已经使用的空间、各DataNode的当前状态等。
</font>
<br>

<font>**2、Secondary NameNode**</font><br>
<font size="2dp">&emsp;&emsp;Secondary NameNode 是用于定期合并命名空间镜像和镜像和镜像的编辑日志的辅助守护进程。和NameNode一样，每个集群都有一个Secondary NameNode，在大规模部署的条件下，一般第二名称节点也独自占用一台服务器。
  &emsp;&emsp;第二名词节点和名称节点的区别在于它不接收或记录HDFS的任务实时变化，而只是根据集群配置的时间间隔，不停地获取HDFS某一个时间点的命名空间镜像和镜像的编辑日志，合并得到一个新的命名空间镜像。该新镜像会上传到名称节点，替换原有的命名空间镜像，并清空上述日志。应该说第二名称节点（SNN）为NameNode的第一关系提供了一个简单的检查点（Checkpoint）机制，并避免出现编辑日志过大，导致名称节点启动时间过长的问题。
  &emsp;&emsp;简单来说就是，NameNode是HDFS集群中的单一故障点，通过Secondary NameNode 的 Checkpoint 可以减少停机的时间并减低NameNode元数据丢失的风险。但是，Secondary NameNode 不支持 NameNode 的故障自动恢复，NameNode 失效处理需要人工干预。
</font>
<br><br>
<font>**3、DataNode**</font><br>
<font size="2dp">&emsp;&emsp;HDFS集群上的从节点都会驻留一个数据节点的守护进程，来执行分布式文件系统中最忙碌的部分：<font color="#FF00FF">将HDFS数据块写到Linux本地文件系统的实际文件中，或者从这些实际文件读取数据块</font><br>
&emsp;&emsp;虽然HDFS是为大文件设计，但存放在HDFS上的文件和传统文件系统类似，也是将文件分块，然后进行存储。但和传统文件系统不同，在数据节点上，HDFS文件块（也就是数据块）以Linux文件系统上的普通文件进行保存。客户端继续宁文件内容操作时，先由NameNode告知客户端每个数据块驻留在哪个DataNode，然后客户端直接与DataNode守护进程进行通信，处理与数据块对应的本地文件。<font color="#FF00FF">同时，DataNode会和其他DataNode进行通信，复制数据块，保证数据的冗余性。
</font><br>
&emsp;&emsp;DataNode 作为从节点，会不断地向NameNode报告。初始化时，每个DataNode将当前存储的数据块告知NameNode。后续DataNode工作过程中，DataNode仍会不断地更新NameNode，为之提供本地修改的相关信息，并接受来自NameNode的指令，创建、移动或者删除本地磁盘上的数据块。<br>
&emsp;&emsp;下图说明了NameNode 与 DataNode 的角色<br>
<img src="http://a4.qpic.cn/psb?/V12dDOqO2iwHZM/H4LEouQuV3m8AeoQ6ib01iScsPLKNHX5X5q72XR2uVQ!/m/dL8AAAAAAAAAnull&bo=lwM9AZcDPQEDByI!&rf=photolist&t=5">
<br>
<font color="#9370D8">图中显示两个数据文件，它们都位于“/usr/bin”目录下。其中，“data1.txt” 文件有3个数据块，表示为 b1、b2 和 b3， “data1.txt”文件由数据块b4和b5组成。这两个文件的内容分散在几个DataNode上。示例中，每个数据块都有3个副本。如数据块1（属于“data1.txt”文件）的3个副本分布于DataNode1、DataNode2和DataNode3上，当这些数据节点中任意一个崩溃或者无法通过网络访问时，可以通过其他节点访问“data1.txt”文件。</font> <br>
</font><br><br>

<font>**4、Client**</font><br>
<font size="2dp">
  &emsp;&emsp;客户端是用户和HDFS进行交互的手段，HDFS提供了各种各样的客户端，包括命令行接口、Java API、 Thrift接口、C语言库、用户空间文件系统（Filesystem in Userspace, FUSE）等。<br>
  &emsp;&emsp;HDFS正是通过Java的客户端，屏蔽了访问HDFS的各种各样细节，用户通过标准的Hadoop文件接口，就可以访问复杂的HDFS，而不需要考虑与NameNode、DataNode等节点的交互细节，降低了Hadoop应用开发的难度，也证明了Hadoop抽象文件系统的适用性。<br>
  &emsp;&emsp;此外，HDFS的Thrift接口、C语言库和FUSE等模块，它们和HDFS的命令行一样，都是在Java API的基础上开发的，用于访问HDFS的接口。<br>
</font><br><br>

<font color="#C71585" size="4dp">HDFS源代码结构</font><br>
<font size="2dp">先来开一下HDFS的结构（HDFS的源代码都在 org.apache.hadoop.hdfs包下）</font><br>
<img src="http://a4.qpic.cn/psb?/V12dDOqO2iwHZM/vgLTIByZ*grKftt2m0DXiwhdBTpWif9PdxizukfrRec!/m/dL8AAAAAAAAAnull&bo=rgMvAq4DLwIDByI!&rf=photolist&t=5"><br><br>
<font>**1、基础包**</font><br>
<font size="2dp">&emsp;&emsp;基础包中包括工具和安全包。其中，<font color="#FF00FF">hdfs.util</font>中包含了一些HDFS实现需要的辅助数据结构；<font color="#FF00FF">hdfs.security.token.delegation</font>结合Hadoop的安全框架，提供了安全访问HDFS的机制。该安全特性最先是由Yahoo开发的，集成了企业广泛应用的Kerberos标准，使得用户可以在一个集群管理各类商业敏感数据。</font><br><br>
<font>**2、HDFS实体实现包**</font><br>
<font size="2dp">&emsp;&emsp;这是代码分析的重点，包含了7个包：<br>

&emsp;&emsp;<font size="3dp" color="#FF00FF">hdfs.protocol</font>提供了HDFS各个实体间通过IPC交互的接口。<br>
<br>
&emsp;&emsp;<font size="3dp" color="#FF00FF">hdfs.server.common</font>
包含了一些NameNode和DataNode共享的功能，如系统升级、存储空间信息等。<br>
<img src="http://m.qpic.cn/psb?/V12dDOqO2iwHZM/X74TqTs8*IZ3lq3Ux071vIRlL5x8csvInkQuw2zpNtw!/b/dMAAAAAAAAAA&bo=NwJgAgAAAAADB3U!&rf=viewer_4"><br>
&emsp;&emsp;<font size="3dp" color="#FF00FF">hdfs.server.namenode、 hdfs.server.datanode</font>和hdfs分别包含了NameNode、DataNode和Client 的实现。这些是HDFS代码分析的重点<br>
&emsp;&emsp;<font size="3dp" color="#FF00FF">hdfs.server.namenode.metrics 和 hdfs.server.datanode.metrics</font>实现了名称节点和数据节点上度量数据的手机功能。度量数据包括NameNode进程和DataNode上度量数据的收集功能。度量数据包括NameNode进程和DataNode进程上事件的计数，例如DataNode上就可以收集到写入字节数、被复制的块的数量等信息。
</font><br><br>
<font>**3、应用包**</font><br>
&emsp;&emsp;<font size="2dp">包括 <font color="#FF00FF" size="3dp">hdfs.tools</font>和<font size="3dp" color="#FF00FF">hdfs.server.balancer</font>，这两个包提供查询HDFS状态信息工具 dfsadmin、文件系统检查工具fsck和HDFS均衡器balancer（通过start-balancer.sh启动）的实现。</font><br><br>
<font>**4、WebHDFS相关包**</font><br>
&emsp;&emsp;<font size="2dp">包括<font size="3dp" color="#FF00FF">hdfs.web.resource、hdfs.server.namenode.web.resources、hdfs.server.datanode.web.resources、hdfs.web</font>共4个包<br>
&emsp;&emsp;WebHDFS是HDFS 1.0 中引入的新功能，它提供了一个完整的、通过HTTP访问HDFS的机制。对比只读的hdfs 文件系统，WebHDFS提供了HTTP上读写HDFS的能力，并在此基础上实现了访问HDFS的C客户端和用户空间文件系统（FUSE）。</font><br><br>

<font color="#C71585" size="4dp">基于远程过程调用的接口</font><br>
<font size="2dp">HDFS的体系结构包括了名称节点、数据节点和客户端3个主要角色，它们间有两种主要的通信接口：
&emsp;&emsp;Hadoop 远程过程调用接口。
&emsp;&emsp;基于 TCP 或 HTTP的流式接口。<br><br>
<font color=" #FF7F50"> 这些接口可以分为三大类。</font>
(1)客户端相关接口（定义在 org.apache.hadoop.hdfs.protocol包中）具体接口如下：<br>
&emsp;&emsp; <font>**ClientProtocol：**</font>客户端与NameNode。这是一个非常重要的接口，是HDFS客户访问文件系统的入口。客户端通过这个接口访问NameNode，操作文件或目录的元数据信息，读取文件也必须先访问NameNode，接下来再和DataNode进行交互，操作文件数据。另外，从NameNode能获取分布式文件系统的一些整体运行状态信息，也是通过这个接口进行的。<br>
&emsp;&emsp;<font>**ClientDatanodeProtocol：**</font>客户端与数据节点间的接口。用于客户端和数据节点进行交互，这个接口用得比较少，客户端和数据节点间的主要交互式通过流接口进行 读/写 文件数据的操作。错误发生时，客户端需要数据节点配合进行恢复，或者当客户端进行本地文件读优化时，需要通过IPC接口获取一些信息。<br><br>

(2)服务器间的接口（包括NameNode、DataNode间存在的IPC调用关系，其接口定义在org.apache.hadoop.hdfs.server.protocol包中）具体接口如下：<br>
&emsp;&emsp;<font>**DatanodeProtocol：**</font>数据节点与名称节点间的接口，这是另外一个重要的接口。在HDFS的主从（Master/Slave）体系结构中，数据节点作为从节点不断地通过这个接口向主节点名称节点报告一些信息，同步信息到名称节点；同时，该接口的一些方法，方法的返回值会带回名称节点指令，数据节点或移动、或删除、或恢复本地磁盘上的数据块，或者执行其他操作。<br>
&emsp;&emsp;<font>**InterDatanodeProtocol：**</font>数据节点与数据节点间的接口。数据节点进行通信，恢复数据块，保证数据的一致性。<br>
&emsp;&emsp;<font>**NamenodeProtocol：**</font>第二名称节点、HDFS均衡器与名称节点间的接口。第二名称节点会不停地获取名称节点上某一个时间点的命名空间镜像和镜像的变化日志，然后合并得到一个新的镜像，并将该结果发送回名称节点，再这个过程中，名称节点回通过这个接口，配合第二名称节点完成元数据的合并。该接口也为HDFS均衡器balancer的正常工作提供一些信息，具体就不展开了。<br><br>

(3)与安全相关的接口<br>
&emsp;&emsp;分别在 &emsp;org.apache.hadoop.security.authorize 包和 &emsp;org.apache.hadoop.security 包中，包括 <font size="3dp" color="#FF00FF">RefreshAuthorizationPolicyProtocol 和 RefreshUserMappingsProtocol</font>，名称节点实现了这两个接口。
</font><br><br>

<font size="4dp" color="#D2691E">与客户端相关的接口</font><br><br>
<font size="3dp">主要包括 ClientProtocol 和 ClientDatanodeProtocol</font><br>
<br>
<font>**1、ClientProtocol**</font>
<img src="http://a3.qpic.cn/psb?/V12dDOqO2iwHZM/m8LpSr5x0cZo5ez9F91zfBiZu0ec62s2FqHM*xOxM*Y!/m/dL4AAAAAAAAAnull&bo=BgTbAAYE2wADByI!&rf=photolist&t=5"><br><br>
&emsp;&emsp;<font size="2dp">Client与NameNode之间的通讯协议定义在 ClientProtocol接口，ClientProtocol是一个冗长的箭扣，一共包含34个方法。这些方法可以分成两组，一组与Hadoop 文件系统相关，另一组用于查询、设置HDFS的状态，多功能工具 dfsadmin的实现依赖这组接口。<br>
&emsp;&emsp;其中第一部分又可以细分为3类，(1)文件/目录元数据部分的操作、写文件相关的操作和读文件相关的操作。<br><br>
&emsp;&emsp;<font size="2dp" color="#FF8C00">ClientProtocol中包含的元数据操作接口的函数和Hadoop抽象文件系统中相应的方法有着对应关系，方法虽多，但却简单；客户端读文件时，NameNode配合的工作比较少，只涉及 getBlockLocations() 和 reportBadBlocks() 两个方法；最复杂的部分出项在写文件（包括创建文件），一共涉及8个ClientProtocol成员函数，包括：用于打开文件的 create() 和 append()、用于追加数据块的addBlock() 和 abandonBlock()、用于持久化文件信息的 fsync()、用于关闭文件的complete() 和与租约相关的 renewLease() 和 recoverLease()。</font><br><br>
&emsp;&emsp;对于大多数文件/目录操作，HDFS客户端提供的API和ClientProtocol中的方法基本是对应的，以修改文件/目录名为例，再Hadoop抽象文件系统中，它的定义是：<br><br>
<img src="http://a4.qpic.cn/psb?/V12dDOqO2iwHZM/AoWXqGDHJkSG25r167YbJlrtsCzn3pPtAsI*S3K0JTE!/m/dFMBAAAAAAAAnull&bo=QgQfAkIEHwIDCSw!&rf=photolist&t=5"><br><br>

&emsp;&emsp;大多数处理文件和目录相关事务的操作，ClientProtocol都提供了和Hadoop FileSystem 中对应API类似的方法，便于客户端的实现。但ClientProtocol中用于支持读写文件数据相关的方法，包括打开文件、输入/输出流和关闭文件等操作，酒和HDFS的特性、体系结构密切相关了。<br>
&emsp;&emsp;Hadoop抽象文件系统中提供了 create()方法，用于创建新文件并获得输出流；或者通过append()方法，再一个已有文件上获得输出流；而open()方法则用于打开文件并钩爪输入流。再ClientProtocol中，虽然也有同名的create()和appen()方法，但它们只是实现FileSystem中同名方法中的某一个步骤。<br><br>

&emsp;&emsp;<font size="3dp" color="#8B0000 ">**关于写数据需要的方法：**</font><br><br>
&emsp;&emsp;ClientProtocol.create()在HDFS目录树种创建一个新文件，它又两种形式，都需要大量的参数。按照函数注释的说明，create()方法会在HDFS目录树种创建一个新的空文件，其路径由参数src提供。注意，名称节点没有“当前路径”的概念，传入的路径参数必须是绝对路径。通过ClientProtocol.create()创建文件后，该文件对其他用户可见可读。create()方法的声明如下：<br><img src="http://m.qpic.cn/psb?/V12dDOqO2iwHZM/n.T08SYB8aPj0KmTfeet41EwLgkiLr9sE.stOCtVeRE!/b/dLYAAAAAAAAA&bo=NwRQATcEUAERBzA!&rf=viewer_4"><br>
&emsp;&emsp;从图中可以开出，create()方法除了路径src外，还需要6个参数，其中，masked携带了文件的权限，replication和blockSize则是被创建文件的数据块副本书和数据块大小，这些都是会持久化到命令空间镜像的文件元数据，特别是replication和blockSize，它们是HDFS特有的。标志位 createParent 用于控制是否覆盖已经存在的文件和是否（递归）创建不存在的父目录。<font color="#FF00FF">需要特别关注的参数是客户端名字clientName，名称节点需要通过clientName，维护和这个客户端关联的一些状态信息，后续还会继续讨论这些状态信息。</font>
&emsp;&emsp;与create()相比，ClientProtocol.append()就简单很多。追加操作不需要创建文件，也就是不需要携带大量创建文件时需要的元数据和标志位。这两个操作还有一个不同点，append()会返回一个LocatedBlock 对象。通过这个返回值，客户端获得追加的文件数据应该写往哪个数据块以及数据块的位置信息，接下来就可以联系DataNode，开始追加数据。如下图为append()：<br>
<img src="http://m.qpic.cn/psb?/V12dDOqO2iwHZM/X9iTYoMTILdSSJwE7JIq4QTkEgPrMnMSoL1lS6cZKP8!/b/dAUBAAAAAAAA&bo=OAS7ADgEuwARBzA!&rf=viewer_4"><br><br>
&emsp;&emsp;<font size="3dp" color="#8B0000">那么问题来了，通过create()创建文件之后，数据应该写到什么地方呢？</font><br>
&emsp;&emsp;ClientProtocol的成员函数addBlock() 用于为当前客户端打开的文件添加一个数据块。当用户创建一个新文件，或者写满一个数据块后，客户端就会调用这个方法，向名称节点申请一个新的数据块。<br>
&emsp;&emsp;addBlock()方法有两种形式，它们都有参数src和clientName，分别是对应的文件名和客户端名字。在各个远程调用之间，Hadoop IPC不会再服务器端保存会话信息，所以，调用addBlock()时，客户端还是需要提供上述上下文信息。<font color="#FF00FF">附加的excludedNodes参数可用于通知NameNode，在选择数据目标时派出某些DataNode。和append()方法一样，addBlock()返回一个LocatedBlock对象，提供了写数据时需要的相关信息。addBlock()方法有两种形式，</font>代码如下：<br>
<img src="http://m.qpic.cn/psb?/V12dDOqO2iwHZM/Nte2E7EVa9E7srVaDO5S96KIu*WL9En9LdtmH.YIAVs!/b/dDQBAAAAAAAA&bo=OAS7ADgEuwARBzA!&rf=viewer_4"><br><br>
&emsp;&emsp;<font size="2dp">**addBlock()方法是ClientProtocol接口种与数据块相关的第一个方法，其他数据块方法包括abandonBlock()、getBlockLocations()和reportBadBlocks()等。**</font><br></br>
<font>**这里重点说一下abandonBlock()方法**</font><br>
&emsp;&emsp;abandonBlock()方法用于放弃一个数据块。这并不是一个常见的操作，当客户端联系不上名称节点通过addBlock()返回的数据节点时，需要通过abandonBlock()方法，明确放弃这个数据块并重新申请新的数据块。下面是一个使用abandonBlock()方法的典型场景：<br>
&emsp;&emsp;假设客户端通过addBlock(),向NameNode申请了一个新数据块，同时，假设申请的文件副本数是1，这时，addBlock()返回的LocatedBlock对象会提供一个DataNode的地址，用于存放该数据块。如果addBlock()返回后，开始写数据前，对应的DataNode因为某种原因崩溃了，客户端联系不上这个节点，在这种情况下，客户端就需要通过abandonBlock()通知名称节点，放弃此数据块，如果在addBlock()返回后，开始写数据前，对应的数据节点因为某种原因崩溃了，客户端联系不上这个节点，在这种情况下，客户端就需要通过abandonBlock()通知NameNode，放弃此数据块，并再次调用addBlock()方法申请新的数据块。在申请新的数据块时，需要将原有DataNode的信息放入addBlock()的excludedNodes参数，保证NameNode不会再次选中并返回这个已经崩溃的节点。<br>
&emsp;&emsp;<font color=" #E9967A">注意：DataNode崩溃后，NameNode不会立即感知到变化，而是在一定的超时时间后，才能做出数据节点不存在的判断，从这个角度看，abandonedBlock()也是HDFS中一个很重要的方法。</font><br><br>
<img src="http://m.qpic.cn/psb?/V12dDOqO2iwHZM/NpEey3S8Bkjsng6cNFhi81kbWH2NaI4pqBKX4tBoaYE!/b/dMUAAAAAAAAA&bo=EATCAAAAAAARB.Y!&rf=viewer_4"><br><br>
<font>**如果客户端从崩溃中恢复，并试图继续未完成的写文件操作，这时候，recoverLease()就派上用场了**</font><br>
&emsp;&emsp;就如同方法名一般，recoverLease()，它是用于恢复租约得。ClientPrtocol.recoverLease()比renewLease()多一个参数，字符串src指定了需要进行租约恢复的文件路径，如果方法返回true，表明这个文件已经被成功关闭，客户端可以通过append()打开文件，继续写数据。<br>

</font><br><br>


