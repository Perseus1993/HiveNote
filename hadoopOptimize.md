# Hadoop优化

#### HDFS小文件影响
影响namenode寿命，因为文件元数据存储在namenode内存

影响计算引擎的任务数量，因为每个小文件都会生成一个map任务

#### 数据输入小文件处理
合并小文件：对小文件进行归档，自定义inputformat将小文件存储成sequenceFile文件

采用combineFileInputFormat作为输入 参考资料： https://blog.csdn.net/wawmg/article/details/17095125

开启JVM重用 https://blog.csdn.net/javastart/article/details/76724271

#### map阶段
增大环形缓冲区

增大环形缓冲区溢写比例

减少溢写文件的merge次数

如果可行，采用combiner提前合并减小IO

#### reduce阶段
合理设置map和reduce数

设置map和reduce共存，map运行到一段时间后让reduce也进行工作，减少等待时间

避免使用reduce

增加reduce去map取数据的并行数

增大reduce的内存

#### IO传输
采用数据压缩，安装Snappy和LZOP压缩

使用sequencefile二进制文件

####压缩
压缩格式	Hadoop自带？   	算法	文件扩展名	支持切分	换成压缩格式后，原来的程序是否需要修改

DEFLATE	  是，直接使用	DEFLATE	 .deflate	  否	         和文本处理一样，不需要修改

Gzip	    是，直接使用	DEFLATE	 .gz	      否	         和文本处理一样，不需要修改

bzip2	    是，直接使用	bzip2	   .bz2	      是	         和文本处理一样，不需要修改

LZO	      否，需要安装	LZO	     .lzo	      是	         需要建索引，还需要指定输入格式

Snappy	  否，需要安装	Snappy	  .snappy	  否	         和文本处理一样，不需要修改

####切片

1）简单地按照文件的内容长度进行切片

2）切片大小，默认等于Block大小

3）切片时不考虑数据集整体，而是逐个针对每一个文件单独切片

提示：切片大小公式：max(0,min(Long_max,blockSize))
