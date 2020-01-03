# HBase MR
可以通过HBase的java API来实现操作Map MapReduce

#官方案例

####案例1
先导入环境变量
```shell
$ export HBASE_HOME=/opt/module/hbase
$ export HADOOP_HOME=/opt/module/hadoop-2.7.2
```
配置下hadoop-env.sh
`export HADOOP_CLASSPATH=$HADOOP_CLASSPATH:/opt/module/hbase/lib/*`

来到HBase目录，运行语句来判断 student有多少行

`/opt/module/hadoop-2.7.2/bin/yarn jar lib/hbase-server-1.3.1.jar rowcounter student`
####案例2 使用MR把本地数据导入HBase
创建一个tsv文件 fruit.tsv
`1001	Apple	Red
1002	Pear	Yellow
1003	Pineapple	Yellow`

把文件上传到HDFS根目录

创建HBase表
`create 'fruit', 'info'`

执行MR 到 HBase的fruit表中
```
/opt/module/hadoop-2.7.2/bin/yarn jar lib/hbase-server-1.3.1.jar importtsv \
-Dimporttsv.columns=HBASE_ROW_KEY,info:name,info:color fruit \
hdfs://hadoop102:9000/fruit.tsv
```
#自定义HBase-MapReduce1

需求：将本地表fruit的一部分数据通过MR传到fruit_mr表中
```java
public class ReadFruitMapper extends TableMapper<ImmutableBytesWritable, Put> {

	@Override
	protected void map(ImmutableBytesWritable key, Result value, Context context)
	throws IOException, InterruptedException {
	//将fruit的name和color提取出来，相当于将每一行数据读取出来放入到Put对象中。
		Put put = new Put(key.get());
		//遍历添加column行
		for(Cell cell: value.rawCells()){
			//添加/克隆列族:info
			if("info".equals(Bytes.toString(CellUtil.cloneFamily(cell)))){
				//添加/克隆列：name
				if("name".equals(Bytes.toString(CellUtil.cloneQualifier(cell)))){
					//将该列cell加入到put对象中
					put.add(cell);
					//添加/克隆列:color
				}else if("color".equals(Bytes.toString(CellUtil.cloneQualifier(cell)))){
					//向该列cell加入到put对象中
					put.add(cell);
				}
			}
		}
		//将从fruit读取到的每行数据写入到context中作为map的输出
		context.write(key, put);
	}
}
```
```java
public class FruitDriver implements Tool {
    private Configuration configuration = null;
    public int run(String[] args) throws Exception {
        //获取job
        Job job = Job.getInstance(configuration);
        //设置驱动类路径
        job.setJarByClass(FruitDriver.class);
        //设置mapper
        job.setMapperClass(FruitMapper.class);
        job.setMapOutputKeyClass(LongWritable.class);
        job.setOutputValueClass(Text.class);
        //设置reducer
        TableMapReduceUtil.initTableReducerJob(args[1], FruitReducer.class, job);
        //设置输出kv
        //输出路径
        FileInputFormat.setInputPaths(job, new Path(args[0]));
        //提交
        Boolean result = job.waitForCompletion(true);
        return result ?0:1;
    }

    public void setConf(Configuration conf) {
        configuration = conf;

    }

    public Configuration getConf() {
        return configuration;
    }

    public static void main(String[] args) {
        try{
            Configuration configuration = new Configuration();
            int run = ToolRunner.run(configuration, new FruitDriver(), args);
            System.exit(run);
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}
```
```java
public class FruitReducer extends TableReducer<LongWritable, Text, NullWritable> {
    @Override
    protected void reduce(LongWritable key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
        for(Text value : values){
            String[] fields = value.toString().split("\t");

            Put put =new Put(Bytes.toBytes(fields[0]));
            //两个列
            put.addColumn(Bytes.toBytes("info"), Bytes.toBytes("name"), Bytes.toBytes(fields[1]));
            put.addColumn(Bytes.toBytes("info"), Bytes.toBytes("color"), Bytes.toBytes(fields[2]));
            context.write(NullWritable.get(), put);
        }
    }
}
```

#自定义HBase-MapReduce2
需求：通过MR将fruit的一部分数据通过MR传到fruit2表中
```java
public class Fruit2Mapper extends TableMapper<ImmutableBytesWritable, Put> {
    @Override
    protected void map(ImmutableBytesWritable key, Result value, Context context) throws IOException, InterruptedException {
        Put put = new Put(key.get());

        for(Cell cell : value.rawCells()){
            //判断当前cell是否是name列
            if("name".equals(Bytes.toString(CellUtil.cloneQualifier(cell)))){
                put.add(cell);
            }
        }
        context.write(key, put);
    }
}

```

```java
public class Fruit2Reducer extends TableReducer<ImmutableBytesWritable, Put, NullWritable> {
    @Override
    protected void reduce(ImmutableBytesWritable key, Iterable<Put> values, Context context) throws IOException, InterruptedException {

        for(Put put : values){
            context.write(NullWritable.get(), put);

        }
    }
}
```

```java
public class Fruit2driver implements Tool {
    private Configuration configuration= null;

    public int run(String[] args) throws Exception {
        Job job = Job.getInstance();

        job.setJarByClass(Fruit2driver.class);

        //设置mapper输出kv类型
        TableMapReduceUtil.initTableMapperJob(args[0],
                new Scan(),
                Fruit2Mapper.class,
                ImmutableBytesWritable.class,
                Put.class,
                job
                );
        //设置reducer和输出的表
        TableMapReduceUtil.initTableReducerJob(args[1],
                FruitReducer.class,
                job
        );

        boolean result = job.waitForCompletion(true);


        return result? 0 : 1;
    }

    public void setConf(Configuration conf) {
        configuration = conf;
    }

    public Configuration getConf() {
        return null;
    }

    public static void main(String[] args) {
        try{
            Configuration configuration = new Configuration();
            ToolRunner.run(configuration, new Fruit2driver(), args);
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}

```
