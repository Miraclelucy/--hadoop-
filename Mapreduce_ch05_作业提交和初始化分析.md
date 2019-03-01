# 一、概述
作业提交过程中涉及三个组件：JobClient、JobTracker和TaskScheduler，他们的功能分别是准备运行环境、接收作业、以及初始化作业。


# 二、作业提交详细过程
JobClient将作业提交到JobTracker端之前，需要进行一些初始化的工作，包括：
  - 获取作业ID，
  - 创建HDFS目录，
  - 上传作业文件至HDFS，
  - 生成split文件等
这些工作由函数JobClient.submitJobInternal(job)实现。
1. 执行shell提交作业的命令

2. 作业文件上传
MapReduce作业文件的上传和下载都是由DistributeCache工具完成的。它是hadoop为方便用户进行应用程序开发而设计的数据分发工具。文件的分发对用户是透明的，

3. 生成InputSplit文件
用户提交MapReduce作业后，JobClient会调用InputFormat的getSplits方法生成InputSplit相关信息，
该信息包括两部分：
  - InputSplit元数据信息；(将被JobTracker使用，用以生成Task本地性相关的数据结构)
  - 原始InputSplit信息；(将被Map Task初始化时使用，用以获取自己要处理的数据)
这两部分信息分别被保存到目录${mapreduce.jobtracker.staging.root.dir}/${user}/.staging/${JobId}下的文件job.split和job.splitmetainfo中。
4. 作业提交到JobTracker
作业提交最终由JobClient.submitJob(job)方法提交到JobTracker端，
  - 为作业创建JobInProgress对象；
  - 检查用户是否具有指定队列的作业提交权限；
  - 检查作业配置的内存使用量是否合理；
  - 通知TaskSchduler初始化作业；

# 三、作业初始化详细过程
调度器调用JobTracker.initJob()函数对作业进行初始化。初始化的主要工作是构造Map Task和Reduce Task并对他们进行初始化。
Hadoop会将每个作业分解成4中类型的任务。分别是Setup Task、Map Task、Reduce Task、Cleanup Task。
1. Setup Task：
进行一些简单的作业初始化工作，比如将状态设置为setup。
2. Map Task：
Map阶段处理数据的任务。其数目及对应的处理数据的片数由应用程序中的InputFormat组件确定。
3. Reduce Task：
Reduce阶段处理数据的任务。其数目由用户通过参数mapred.reduce.tasks指定。考虑到Reduce Task能否运行依赖于Map Task的输出结果。因此hadoop刚开始只调用Map Task，直到Map Task完成的数目到达一定的比例（mapred.reduce.slowstart.completed.maps指定，默认是5%），才开始调用Reduce Task。
4. Cleanup Task：


# 四、DistributeCache原理分析
它主要的功能是将作业分发到各个TaskTracker上。具体流程分为以下4个步骤：

- 步骤1：用户提交作业后，DistributeCache将作业上传到HDFS中的固定目录中
- 步骤2：JobTracker端的任务调度器将作业对应的任务发派到各个TaskTracker上，
- 步骤3：任何一个TaskTracker收到该作业的第一个任务后，由DistributeCache自动将作业需要的文件缓存到本地目录下，然后开始该任务
- 步骤4：对于TaskTracker接下来收到的任务，DistributeCache不会再重复为其下载文件，而是直接运行。

TaskTracker文帝目录中，不同级别的文件放在不同的目录下：

DistributeCache中的文件或者目录并不是用完后就被清理的，而是由专门的线程根据文件的大小和文件/目录数目上限周期性的进行清理。
