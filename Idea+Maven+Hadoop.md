# IDEA + Maven + Hadoop

### 1、新建Intellij下的Maven项目

点击File->New->Project，在弹出的对话框中选择Maven，JDK选择你自己安装的版本，点击Next

### 2、填写Maven的 GroupID 和 ArtifactId

你可以根据自己的项目随便填，点击Next，这样就新建好了一个空的项目

这里程序名填写WordCount,我们的程序是一个通用的网上的范例,用来计算文件中单词出现的次数。代码如下：

```
public class WordCount{
    /**
     * 建立Mapper类TokenizerMapper继承自泛型类Mapper
     * Mapper类:实现了Map功能基类
     * Mapper接口：
     * WritableComparable接口：实现WritableComparable的类可以相互比较。所有被用作key的类应该实现此接口。
     * Reporter 则可用于报告整个应用的运行进度，本例中未使用。
     *
     */
    public static class TokenizerMapper extends Mapper<Object, Text, Text, IntWritable>{
        /**
         * IntWritable, Text 均是 Hadoop 中实现的用于封装 Java 数据类型的类，这些类实现了WritableComparable接口，
         * 都能够被串行化从而便于在分布式环境中进行数据交换，你可以将它们分别视为int,String 的替代品。
         * 声明one常量和word用于存放单词的变量
         */
        private final static IntWritable one =new IntWritable(1);
        private Text word =new Text();
        /**
         * Mapper中的map方法：
         * void map(K1 key, V1 value, Context context)
         * 映射一个单个的输入k/v对到一个中间的k/v对
         * 输出对不需要和输入对是相同的类型，输入对可以映射到0个或多个输出对。
         * Context：收集Mapper输出的<k,v>对。
         * Context的write(k, v)方法:增加一个(k,v)对到context
         * 程序员主要编写Map和Reduce函数.这个Map函数使用StringTokenizer函数对字符串进行分隔,通过write方法把单词存入word中
         * write方法存入(单词,1)这样的二元组到context中
         */
        public void map(Object key, Text value, Context context) throws IOException, InterruptedException{
            StringTokenizer itr =new StringTokenizer(value.toString());
            while (itr.hasMoreTokens()){
                word.set(itr.nextToken());
                context.write(word, one);
            }
        }
    }

    public static class IntSumReducer extends Reducer<Text,IntWritable,Text,IntWritable> {
        private IntWritable result =new IntWritable();
        /**
         * Reducer类中的reduce方法：
         * void reduce(Text key, Iterable<IntWritable> values, Context context)
         * 中k/v来自于map函数中的context,可能经过了进一步处理(combiner),同样通过context输出
         */
        public void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
            int sum =0;
            for (IntWritable val : values) {
            sum += val.get();
            }
            result.set(sum);
            context.write(key, result);
        }
    }

    public static void main(String[] args) throws Exception {
        /**
         * Configuration：map/reduce的j配置类，向hadoop框架描述map-reduce执行的工作
         */
        Configuration conf =new Configuration();
        Job job =new Job(conf, "word count");    //设置一个用户定义的job名称
        job.setJarByClass(WordCount.class);
        job.setMapperClass(TokenizerMapper.class);    //为job设置Mapper类
        job.setCombinerClass(IntSumReducer.class);    //为job设置Combiner类
        job.setReducerClass(IntSumReducer.class);    //为job设置Reducer类
        job.setOutputKeyClass(Text.class);        //为job的输出数据设置Key类
        job.setOutputValueClass(IntWritable.class);    //为job输出设置value类
        FileInputFormat.addInputPath(job, new Path(args[0]));    //为job设置输入路径
        FileOutputFormat.setOutputPath(job, new Path(args[1]));//为job设置输出路径
        System.exit(job.waitForCompletion(true) ? 0 : 1);        //运行job
    }
}
```

### 3、设置程序的编译版本

打开Intellij的Preference偏好设置，定位到Build，Execution，Deployment->Compiler->Java Compiler，

将WordCount的Target bytecode version修改为你的jdk版本(我的是1.8)

### 4、配置依赖

编辑pom.xml进行配置

- 添加apache源

在`project`内尾部添加

```
<repositories>
    <repository>
        <id>apache</id>
        <url>http://maven.apache.org</url>
    </repository>
</repositories>
```

- 添加`hadoop`依赖

这里只需要用到基础依赖hadoop-core和hadoop-common；

(如果需要读写HDFS，则还需要依赖hadoop-hdfs和hadoop-client；如果需要读写HBase，则还需要依赖hbase-client

在`project`内部添加依赖

```
<dependencies>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-core</artifactId>
        <version>1.0.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-common</artifactId>
        <version>2.7.2</version>
    </dependency>
</dependencies>
```

修改pom.xml完成后，Intellij右上角会提示Maven projects need to be Imported，点击Import Changes以更新依赖,或者点击Enable Auto Import

最后,我的完整的pom.xml如下:

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.fun</groupId>
    <artifactId>hadoop</artifactId>
    <version>1.0-SNAPSHOT</version>

    <repositories>
        <repository>
            <id>apache</id>
            <url>http://maven.apache.org</url>
        </repository>
    </repositories>

    <dependencies>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-core</artifactId>
            <version>1.0.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>2.7.2</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <artifactId>maven-dependency-plugin</artifactId>
                <configuration>
                    <excludeTransitive>false</excludeTransitive>
                    <stripVersion>true</stripVersion>
                    <outputDirectory>./lib</outputDirectory>
                </configuration>

            </plugin>
        </plugins>
    </build>
</project>
```

### 6、配置输入和输出结果文件夹

- 添加和src目录同级的input文件夹到项目中，我的输入文件源如下:

```
dfdfadgdgag
aadads
fudflcl
cckcer
fadf
dfdfadgdgag
fudflcl
fuck
fuck
fuckfuck
haha
aaa
```
- 配置运行参数

在Intellij菜单栏中选择Run->Edit Configurations，在弹出来的对话框中点击+，新建一个Application配置。配置Main class为WordCount（可以点击右边的...选择）,Program arguments为`input/` `output/`，即输入路径为刚才创建的input文件夹，输出为output

注意：`由于Hadoop的设定，下次运行时务必删除output文件夹！`

- 运行程序,结果如下

```
aaa 1
aadads 1
cckcer 1
dfdfadgdgag 2
fadf 1
fuck 2
fuckfuck 1
fudflcl 2
haha 1
```
---
### 注意
#### windows下：Failed to set permissions of path: \tmp\ \.staging to 0700
这是个官方BUG，是Windows下文件权限问题，在Linux下可以正常运行，不存在这样的问题。

解决方法是修改`/hadoop1.0.0/src/core/org/apache/hadoopo/fs/FileUtil.java` 里面的checkReturnValue，注释掉即可

```
private static void checkReturnValue(boolean rv, File p,  
                                       FsPermission permission  
                                       ) throws IOException {  
    /** 
if (!rv) { 
throw new IOException("Failed to set permissions of path: " + p + 
" to " + 
String.format("%04o", permission.toShort())); 
} 
**/  
  }
```

