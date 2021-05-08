# Tổng quan về Apache Spark
## Tìm hiểu về Spark RDD
### 1.	RDD là gì?
RDD (Resilient Distributed Dataset) là một cấu trúc dữ liệu cơ bản và  trừu tượng trong Apache Spark và Spark Core. RDD là tập hợp các đối tượng phân tán không thay đổi, chịu được lỗi, có nghĩa là một khi tạo RDD thì không thể thay đổi được nó. Mỗi tập dữ liệu trong RDD được chia thành các phân vùng logic, có thể được tính toán trên node khác nhau của cụm. Ngoài ra, RDD trừu tượng hóa dữ liệu của việc phân vùng và việc phân phối dữ liệu được thiết kế để chạy tính toán song song trên  nhiều nút. Do đó không phải lo lắng về sự song song như Spark. Một RDD chỉ có thể có trong một SparkContext và RDD có tên và id duy nhất.
Những ưu điểm của RDD:
•	Xử lý trong bộ nhớ
•	Tính bất biến
•	Khả năng chịu lỗi
•	Lười tiến hóa
•	Tính phân vùng
•	Tính song song
Hạn chế:
•	Spark RDD không phù hợp cho các ứng dụng thực hiện cập nhật cho kho lưu trữ trạng thái như hệ thống lưu trữ cho ứng dụng web. Đối với các ứng dụng này, sẽ hiệu quả hơn nếu sử dụng các hệ thống thực hiện ghi nhật ký truyền thống và kiểm tra dữ liệu như cơ sở dữ liệu. Mục tiêu của RDD là cung cấp một mô hình lập trình hiệu quả cho phân tích hàng loạt và loại bỏ các ứng dụng không đồng bộ này.
### 2.	Tạo RDD
RDD được tạo chủ yếu theo hai cách, đầu tiên là song song hóa một tập hợp hiện có và thứ hai là tham chiếu tập dữ liệu trong hệ thống lưu trữ bên ngoài (HDFS, HDFS, S3, …)
Trước tiên phải khởi tạo SparkSession bằng cách sử dụng phương thức xây dựng mẫu được xác định trong lớp SparkSession. Trong khi khởi tạo, cần cung cấp master và tên ứng dụng.
<br>
**Code:**
<br>
val spark:SparkSession = SparkSession.builder()
      .master("local[1]")
      .appName("SparkByExamples.com")
      .getOrCreate()
<br> <br>
![1](https://user-images.githubusercontent.com/75170587/117544430-9a23b800-b04b-11eb-91eb-1fa317cedbee.PNG)
<br>
<br>
**Sử dụng sparkContext.parallelize()**
<br>
sparkContext.parallelize được sử dụng để song song hóa một tập hợp hiện có trong chương trình trình điều khiển. Đây là phương pháp cơ bản để tạo RDD và được sử dụng chủ yếu trong POC’s hoặc tạo mẫu và nó yêu cầu tất cả dữ liệu phải có trên chương trình trình điều khiển trước khi tạo RDD. Do đó nó không được sử dụng nhiều cho các ứng dụng sản xuất.
<br>
**Code:**
<br>
val dataSeq = Seq(("Java", 20000), ("Python", 100000), ("Scala", 3000))   
val rdd=spark.sparkContext.parallelize(dataSeq)
<br>
<br>
**Sử dụng sparkContext.textFile()**
<br>
Sử dụng phương thức textFile(), chúng ta có thể đọc tệp văn bản (.txt) vào RDD.
<br>
**Code:**
<br>
val rdd2 = spark.sparkContext.textFile("/path/textFile.txt")
<br>
<br>
**Sử dụng sparkContext. wholeTextFiles()**
<br>
Hàm wholeTextFiles() trả về một PairRDD với khóa là đường dẫn tệp và giá trị là nội dung tệp
<br>
**Code:**
<br>
val rdd3 = spark.sparkContext.wholeTextFiles("/path/textFile.txt")
<br>
<br>
**Sử dụng sparkContext.emptyRDD()**
<br>
Sử dụng phương thức emptyRDD() trên sparkContext, chúng ta có thể tạo một RDD không có dữ liệu. Phương pháp này tạo ra một RDD trống không có phân vùng.
<br>
**Code**
<br>
val rdd = spark.sparkContext.emptyRDD // creates EmptyRDD[0]
val rddString = spark.sparkContext.emptyRDD[String]
<br>
<br>
**Tạo một RDD trống với phân vùng**
<br>
**Code:**
<br>
val rdd2 = spark.sparkContext.parallelize(Seq.empty[String])
<br>
<br>
**RDD song song hóa và phân vùng lại**
<br>
Khi chúng ta sử dụng phương thức parallelize(), textFile() hoặc wholeTextFiles() của SparkContxt để khởi tạo RDD,
nó sẽ tự động chia dữ liệu thành các phân vùng dựa trên tính khả dụng của tài nguyên.
<br>
**getNumPartitions** - Trả về một số phân vùng mà tập dữ liệu được chia thành.
Bất kỳ phép transformation nào được áp dụng trên RDD đều thực hiện song song. Spark sẽ chạy một tác vụ cho mỗi phân vùng của cụm.
<br>
**Code:**
<br>
println("initial partition count:"+rdd.getNumPartitions)
<br>
**Set parallelize manually** - Chúng ta cũng có thể đặt một số phân vùng theo cách thủ công, tất cả những gì chúng ta cần là
chuyển một số phân vùng làm tham số thứ hai cho các hàm này,ví dụ như sparkContext.parallelize (dataSeq, 10)).
<br>
**Repalallize sử dụng repartition và coalesce** - - Spark cung cấp hai cách để phân vùng lại; đầu tiên sử dụng phương thức repartition() xáo trộn dữ liệu từ tất cả các nút còn được gọi là trộn đầy đủ và phương thức coalesce() thứ hai trộn dữ liệu từ các nút tối thiểu.
Hàm repartition() hoặc  coalesce() đều trả về một RDD mới.
<br>
**Code:**
<br>
val reparRdd = rdd.repartition(4)
println("re-partition count:"+reparRdd.getNumPartitions)
<br>
<br>
**RDD Operations**
<br>
**RDD transformations** - Transformations là hoạt động lười biếng, thay vì cập nhật một RDD, các phép toán này trả về một RDD khác.
<br>
**RDD actions**- các hoạt động kích hoạt tính toán và trả về giá trị RDD.
<br>
<br>
**Ví dụ RDD Transformations**
<br>
**Transformations** trên spark RDD trả về một RDD khác và các transformation là lười biếng nghĩa là chúng không thực thi cho đến khi bạn gọi một hành động trên RDD. Một số transformation trên RDD’s là flatMap, map, ReduceByKey, filter, sortByKey và trả về RDD mới thay vì cập nhật hiện tại.
<br><br>
![2](https://user-images.githubusercontent.com/75170587/117544973-f5ef4080-b04d-11eb-93fc-3730d14b6150.PNG)
<br><br>
Đầu tiên tạo một RDD bằng cách đọc một file text. Code:
<br>
val rdd:RDD[String] = spark.sparkContext.textFile("src/main/scala/test.txt")
<br>
**flatMap** - Phép biến đổi flatMap() làm phẳng RDD sau khi áp dụng hàm và trả về một RDD mới. Trong ví dụ dưới đây, đầu tiên, nó chia từng bản ghi theo khoảng trắng trong RDD và cuối cùng làm phẳng nó. Kết quả RDD bao gồm một từ duy nhất trên mỗi bản ghi.
<br>
**Code:**
<br>
val rdd2 = rdd.flatMap(f=>f.split(" "))
<br>
**map** - Phép biến đổi map() được sử dụng để áp dụng bất kỳ hoạt động phức tạp nào như thêm một cột, cập nhật một cột, đầu ra của các phép biến đổi map sẽ luôn có cùng số lượng bản ghi như đầu vào.
<br>
**filter** Phép biến đổi filter () được sử dụng để lọc các bản ghi trong RDD. Ví dụ lọc tất cả các từ bắt đầu bằng “a”. Code:
<br>
val rdd4 = rdd3.filter(a=> a._1.startsWith("a"))
<br>
**reduceByKey** - ReduceByKey () hợp nhất các giá trị cho mỗi khóa với hàm được chỉ định. Ví dụ, nó làm giảm chuỗi từ bằng cách áp dụng hàm sum trên giá trị. Kết quả của RDD chứa các từ duy nhất và số lượng của chúng.
<br>
**Code:**
<br>
val rdd5 = rdd4.reduceByKey(_ + _)
<br>
**sortByKey** - Phép biến đổi sortByKey () được sử dụng để sắp xếp các phần tử RDD trên khóa. Ví dụ, trước tiên, chuyển đổi RDD [(String, Int]) thành RDD [(Int, String]) bằng cách sử dụng phép biến đổi map và áp dụng sortByKey mà lý tưởng là sắp xếp trên một giá trị số nguyên. Và cuối cùng, câu lệnh foreach với println trả về tất cả các từ trong RDD và số lượng của chúng là cặp khóa-giá trị.
<br>
**Code:**
<br>
val rdd6 = rdd5.map(a=>(a._2,a._1)).sortByKey()
rdd6.foreach(println
<br><br>
**Các loại RDD**
<br>
PairRDDFunctions hoặc PairRDD - Pair RDD là một cặp khóa-giá trị. Đây là loại RDD chủ yếu được sử dụng.<br>
* ShuffledRDD
* DoubleRDD
* SequenceFileRDD
* HadoopRDD
* ParallelCollectionRDD
<br>
<br>
**Hoạt động Shuffle**
<br>
**Shuffle** - Xáo trộn là một cơ chế mà Spark sử dụng để phân phối lại dữ liệu giữa các trình thực thi khác nhau và thậm chí trên các máy. Xáo trộn kích hoạt khi thực hiện các hoạt động chuyển đổi nhất định như gropByKey (), ReduceByKey (), join () trên RDDS.
<br>
Spark Shuffle là một hoạt động tốn kém vì nó liên quan đến những điều sau:
*	Disk I/O
*	Liên quan đến tuần tự hóa dữ liệu và giải mã hóa
*	Network I/O
<br>
Khi tạo RDD, Spark không nhất thiết phải lưu trữ dữ liệu cho tất cả các khóa trong một phân vùng vì tại thời điểm tạo, không có cách nào chúng ta có thể đặt khóa cho tập dữ liệu. Do đó, khi chạy thao tác ReduceByKey () để tổng hợp dữ liệu trên các khóa, Spark sẽ thực hiện những việc sau, trước tiên cần chạy các tác vụ để thu thập tất cả dữ liệu từ tất cả các phân vùng và Spark RDD  kích hoạt xáo trộn và phân vùng lại cho một số hoạt động  như repartition () và Coalesce (), groupByKey (), ReduceByKey (), cogroup () và join ().
<br><br><br>
## Spark DataFrame
### 1.	Định nghĩa
DataFrame là một tập hợp phân phối dữ liệu được tổ chức thành các cột và được đặt tên. DataFrame tương đương với một bảng trong một cơ sở dữ liệu quan hệ hoặc một khung dữ liệu trong R / Python, nhưng tối ưu và phong phú hơn. DataFrames có thể được xây dựng từ một loạt các nguồn như các tập tin dữ liệu có cấu trúc, bảng trong Hive, cơ sở dữ liệu bên ngoài, hoặc RDDs hiện có.
<br>
Dưới đây là các tính năng đặc trưng của DataFrame:
<br>
*	Hỗ trợ các định dạng dữ liệu khác nhau (Avro, csv, tìm kiếm đàn hồi và Cassandra) và hệ thống lưu trữ (HDFS, bảng HIVE, mysql, v.v.).
*	Tối ưu hóa hiện đại và tạo mã thông qua trình tối ưu hóa Spark SQL.
*	Có thể dễ dàng tích hợp với tất cả các công cụ và khuôn khổ Big Data thông qua Spark-Core.
*	Cung cấp API cho Lập trình Python, Java, Scala và R.
<br>
### 2.	Khởi tạo DataFrame
Tạo DataFrame từ RDD
<br>
Tạo một RDD
<br>
![3](https://user-images.githubusercontent.com/75170587/117545349-92fea900-b04f-11eb-9eed-876e0c3eb36f.PNG)
<br>
Sử dụng hàm toDF()
<br>
![4](https://user-images.githubusercontent.com/75170587/117545368-ad388700-b04f-11eb-8ed1-bf2e414a1386.PNG)
<br>
Tạo DataFrame từ CSV
<br>
![1](https://user-images.githubusercontent.com/75170587/117545396-cccfaf80-b04f-11eb-9960-dd180c1f96af.PNG)
<br>
Tạo DataFrame từ List Collection 
<br>
![1](https://user-images.githubusercontent.com/75170587/117545437-f983c700-b04f-11eb-9903-869425fc43ee.PNG)
<br>
### 3.	Các chức năng với cột trong DataFrame
<br>
![1](https://user-images.githubusercontent.com/75170587/117545529-57b0aa00-b050-11eb-87ba-a28b690e44f1.PNG)
<br>
### 4.	Filter 
<br>
![1](https://user-images.githubusercontent.com/75170587/117545568-8cbcfc80-b050-11eb-8fdb-0ca8ebee92be.PNG)
<br>
Kết quả
<br>
![1](https://user-images.githubusercontent.com/75170587/117545593-a4948080-b050-11eb-94b4-a5bb83d0236e.PNG)
<br>
### 5.	Select
Bạn có thể chọn cột một hoặc bội số của DataFrame bằng cách chuyển tên cột bạn muốn chọn đến hàm
<br>
![1](https://user-images.githubusercontent.com/75170587/117545644-da396980-b050-11eb-94e2-8ffc71904ed6.PNG)
<br>
### 6.	Sort
Để sắp xếp DataFrame bằng cách tăng dần hoặc giảm dần thứ tự dựa trên một hoặc nhiều cột.
<br>
Data mẫu
<br>
![1](https://user-images.githubusercontent.com/75170587/117545671-f806ce80-b050-11eb-883e-a7767f18604f.PNG)
<br>
Syntax
<br>
![1](https://user-images.githubusercontent.com/75170587/117545703-12d94300-b051-11eb-90ba-566dd2a78f19.PNG)
<br><br>
## Tổng quan về Mapreduce
### MapReduce là gì?
MapReduce là mô hình được thiết kế độc quyền bởi Google, nó có khả năng lập trình xử lý các tập dữ liệu lớn song song và phân tán thuật toán trên 1 cụm máy tính. MapReduce trở thành một trong những thành ngữ tổng quát hóa trong thời gian gần đây.
<br>
MapReduce sẽ bao gồm những thủ tục sau: thủ tục 1 Map() và 1 Reduce(). Thủ tục Map() bao gồm lọc (filter) và phân loại (sort) trên dữ liệu khi thủ tục khi thủ tục Reduce() thực hiện quá trình tổng hợp dữ liệu. Đây là mô hình dựa vào các khái niệm biển đối của bản đồ và reduce những chức năng lập trình theo hướng chức năng. Thư viện của thủ tục Map() và Reduce() sẽ được viết bằng nhiều loại ngôn ngữ khác nhau. Thủ tục được cài đặt miễn phí và được sử dụng phổ biến nhất là là Apache Hadoop.
<br><br>
**Các hàm chính của MapReduce**
MapReduce có 2 hàm chính là Map() và Reduce(), đây là 2 hàm đã được định nghĩa bởi người dùng và nó cũng chính là 2 giai đoạn liên tiếp trong quá trình xử lý dữ liệu của MapReduce. Nhiệm vụ cụ thể của từng hàm như sau:
<br>
* Hàm Map(): có nhiệm vụ nhận Input cho các cặp giá trị/ khóa và output chính là tập những cặp giá trị/khóa trung gian. Sau đó, chỉ cần ghi xuống đĩa cứng và tiến hành thông báo cho các hàm Reduce() để trực tiếp nhận dữ liệu.
* Hàm Reduce(): có nhiệm vụ tiếp nhận từ khóa trung gian và những giá trị tương ứng với lượng từ khóa đó. Sau đó, tiến hành ghép chúng lại để có thể tạo thành một tập khóa khác nhau. Các cặp khóa/giá trị này thường sẽ thông qua một con trỏ vị trí để đưa vào các hàm reduce. Quá trình này sẽ giúp cho lập trình viên quản lý dễ dàng hơn một lượng danh sách cũng như phân bổ giá trị sao cho phù hợp nhất với bộ nhớ hệ thống.
* Ở giữa Map và Reduce thì còn 1 bước trung gian đó chính là Shuffle. Sau khi Map hoàn thành xong công việc của mình thì Shuffle sẽ làm nhiệm vụ chính là thu thập cũng như tổng hợp từ khóa/giá trị trung gian đã được map sinh ra trước đó rồi chuyển qua cho Reduce tiếp tục xử lý.
<br>
![1](https://user-images.githubusercontent.com/75170587/117545880-dbb76180-b051-11eb-87c0-d4f78909f7ed.PNG)
<br><br>
**Ưu điểm**
<br>
* MapReduce có khả năng xử lý dễ dàng mọi bài toán có lượng dữ liệu lớn nhờ khả năng tác vụ phân tích và tính toán phức tạp. Nó có thể xử lý nhanh chóng cho ra kết quả dễ dàng chỉ trong khoảng thời gian ngắn.
* Mapreduce có khả năng chạy song song trên các máy có sự phân tán khác nhau. Với khả năng hoạt động độc lập kết hợp phân tán, xử lý các lỗi kỹ thuật để mang lại nhiều hiệu quả cho toàn hệ thống.
* MapRedue có khả năng thực hiện trên nhiều nguồn ngôn ngữ lập trình khác nhau như: Java, C/ C++, Python, Perl, Ruby,… tương ứng với nó là những thư viện hỗ trợ.
* Mã độc trên internet ngày càng nhiều hơn nên việc xử lý những đoạn mã độc này cũng trở nên rất phức tạp và tốn kém nhiều thời gian. Chính vì vậy, các ứng dụng MapReduce dần hướng đến quan tâm nhiều hơn cho việc phát hiện các mã độc để có thể xử lý chúng. Nhờ vậy, hệ thống mới có thể vận hành trơn tru và được bảo mật nhất.
<br>
## Machine-Learning
### Sử dụng Spark cho Machine-Learning
Khi tạo mô hình học máy, khía cạnh quan trọng nhất để chuẩn bị mô hình là độ chính xác trong xử lý dữ liệu và tiết kiệm bộ nhớ máy tính. Nếu tập dữ liệu đã cho không phù hợp với bộ nhớ, thì phải sử dụng tính toán phân phối để tính toán một cụm có nhiều máy. Loại mô hình điện toán phân tán có sẵn cho đến bây giờ là HADOOP. Tuy nhiên, với SPARK, giờ đây bạn có thể xử lý dữ liệu từ các máy cục bộ độc lập và xây dựng mô hình dữ liệu với bộ dữ liệu đầu vào lớn hơn. Thông thường, các tập dữ liệu đầu vào này lớn hơn dung lượng bộ nhớ mà máy tính của bạn có. Đó là loại đàn hồi mà Apache Spark cung cấp. Do đó, Apache Spark được đặc trưng với Cơ sở hạ tầng đàn hồi. Điều này cho phép các nhà khoa học dữ liệu lặp lại các vấn đề dữ liệu nhanh hơn 100 lần so với HADOOP. Nó có 8000 nút tạo thành cụm lớn nhất thế giới được biết đến.
### Spark Mlib
Apache Spark cung cấp một API Học máy được gọi là MLlib . PySpark cũng có API học máy này bằng Python. Nó hỗ trợ các loại thuật toán và tiện ích học tập phổ biến, bao gồm phân loại, hồi quy, phân cụm, lọc cộng tác, giảm kích thước, các nguyên tắc tối ưu hóa cơ bản, như được nêu bên dưới:
<br>
**mllib.classification** - Gói spark.mllib hỗ trợ nhiều phương pháp khác nhau để phân loại nhị phân, phân loại đa lớp và phân tích hồi quy. Một số thuật toán phổ biến nhất trong phân loại là Rừng ngẫu nhiên, Vịnh Naive, Cây quyết định , v.v.
<br>
**mllib.clustering** - Clustering là một vấn đề học tập không có giám sát, theo đó bạn nhằm mục đích nhóm các tập con của các thực thể với nhau dựa trên một số khái niệm về sự giống nhau.
<br>
**mllib.fpm** - Đối sánh mẫu thường xuyên là khai thác các mục thường xuyên, tập phổ biến, chuỗi con hoặc các cấu trúc con khác thường nằm trong số các bước đầu tiên để phân tích một tập dữ liệu quy mô lớn. Đây đã là một chủ đề nghiên cứu tích cực trong việc khai thác dữ liệu trong nhiều năm.
<br>
**mllib.linalg** - Tiện ích MLlib cho đại số tuyến tính.
<br>
**mllib.recommendation** - Lọc cộng tác thường được sử dụng cho các hệ thống khuyến nghị. Các kỹ thuật này nhằm mục đích điền vào các mục còn thiếu của ma trận liên kết mục người dùng.
<br>
**spark.mllib** - Nó hiện hỗ trợ lọc cộng tác dựa trên mô hình, trong đó người dùng và sản phẩm được mô tả bằng một tập hợp nhỏ các yếu tố tiềm ẩn có thể được sử dụng để dự đoán các mục nhập bị thiếu. spark.mllib sử dụng thuật toán Bình phương tối thiểu xen kẽ (ALS) để tìm hiểu các yếu tố tiềm ẩn này.
<br>
**mllib.regression** - Hồi quy tuyến tính thuộc họ thuật toán hồi quy. Mục tiêu của hồi quy là tìm mối quan hệ và sự phụ thuộc giữa các biến. Giao diện làm việc với mô hình hồi quy tuyến tính và tóm tắt mô hình tương tự như trường hợp hồi quy logistic. Spark MLlib được tích hợp chặt chẽ trên Spark giúp giảm bớt sự phát triển của các thuật toán học máy quy mô lớn hiệu quả như thường là lặp đi lặp lại trong tự nhiên. Cộng đồng mã nguồn mở của Spark đã dẫn đến sự phát triển nhanh chóng và việc áp dụng Spark MLlib. Có hơn 200 cá nhân từ 75 tổ chức cung cấp khoảng hơn 2000 bản vá chỉ riêng cho MLlib.
<br>
### Example
**Machine Learning áp dụng thuật toán Naive Bayes cho dataset Irris**
<br>
Giai đoạn tiền xử lý
<br>
![1](https://user-images.githubusercontent.com/75170587/117546332-20dc9300-b054-11eb-9518-e2bc3964d867.PNG)
<br>
Giai đoạn xử lý dữ liệu
<br>
![1](https://user-images.githubusercontent.com/75170587/117546360-3ea9f800-b054-11eb-95de-5cf8dae5db6b.PNG)
<br>
Giai đoạn tạo model và predict
<br>
![1](https://user-images.githubusercontent.com/75170587/117546402-6f8a2d00-b054-11eb-9b88-4f6b442fd866.PNG)
<br>
Cuối cùng là đánh giá model
<br>
![1](https://user-images.githubusercontent.com/75170587/117546436-a102f880-b054-11eb-861d-130ca3d711b4.PNG)
<br>
