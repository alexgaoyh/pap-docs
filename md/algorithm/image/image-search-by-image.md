## 背景

&ensp;&ensp;《图像处理-Java-以图搜图》，之前在编写过关于[图像边缘检测-去黑边](https://pap-docs.pap.net.cn/#/md/algorithm/image/remove-black-border)、[图像边缘检测-自动纠偏](https://pap-docs.pap.net.cn/#/md/algorithm/image/auto-correction)、[图像处理-锐化](https://pap-docs.pap.net.cn/#/md/algorithm/image/sharpening-prewitt-overlay)、[图像处理-去噪/高斯模糊/套红](https://pap-docs.pap.net.cn/#/md/algorithm/image/image-denoise-gaussianBlur-red)、[图像处理-背景色平滑/反色](https://pap-docs.pap.net.cn/#/md/algorithm/image/image-backgroundSmooth-invert)、[图像处理-指定大小压缩](https://pap-docs.pap.net.cn/#/md/algorithm/image/image-compress-to-target-size)、[图像处理-Java-字深字浅](https://pap-docs.pap.net.cn/#/md/algorithm/image/image-fontweight-deep-shallow)等一系列文章，接下来主要介绍基于opencv+lucene的以图搜图功能。

## 概述

&ensp;&ensp;使用JAVA语言实现，实现思路是使用opencv获得图像的特征，之后将图像特征存入lucene后使用KnnVectorQuery进行搜索，从而达到以图搜图的效果。


## 实现步骤

1. 引入OpenCV后获得图像特征；
2. 处理图像特征，包括归一化等操作；
3. 归一化后的图像特征存入lucene中；
4. 使用 KnnVectorQuery 进行相关搜索


## 代码实现

```html
        <dependency>
            <groupId>org.opencv</groupId>
            <artifactId>opencv-401</artifactId>
            <version>401</version>
            <scope>system</scope>
            <systemPath>${pom.basedir}/libs/opencv-401.jar</systemPath>
        </dependency>

        <dependency>
            <groupId>org.apache.lucene</groupId>
            <artifactId>lucene-core</artifactId>
            <version>9.10.0</version>
        </dependency>
```

```html
    /**
     * 图像特征
     *
     * @param imagePath
     * @return
     */
    public static byte[] matOfKeyPointImage(String imagePath) {
        Mat image = Imgcodecs.imread(imagePath);

        // Convert image to grayscale
        Mat grayImage = new Mat();
        Imgproc.cvtColor(image, grayImage, Imgproc.COLOR_BGR2GRAY);

        // Initialize ORB detector
        ORB detector = ORB.create();

        // Detect keypoints
        MatOfKeyPoint keypoints = new MatOfKeyPoint();
        detector.detect(grayImage, keypoints);

        // Compute descriptors
        Mat descriptors = new Mat();
        detector.compute(grayImage, keypoints, descriptors);

        // Convert descriptors to byte array
        MatOfByte matOfByte = new MatOfByte();
        descriptors.convertTo(matOfByte, CvType.CV_8U);

        // Convert MatOfByte to byte array
        byte[] descriptorsData = new byte[(int) (matOfByte.total() * matOfByte.channels())];
        matOfByte.get(0, 0, descriptorsData);

        return descriptorsData;
    }

    @Test
    public void testQuery() throws IOException {
        // 第一个存储的向量，用来后续的搜索
        float[] firstVector = null;

        try (MMapDirectory dir = new MMapDirectory(indexPath)) {
            try (IndexWriter writer = new IndexWriter(dir, new IndexWriterConfig())) {
                List<String> imageAbsPathList = new ArrayList<>();
                imageAbsPathList.add("pap-1.jpg");
                imageAbsPathList.add("pap-2.jpg");
                imageAbsPathList.add("pap-3.jpg");
                for(String imageAbsPath : imageAbsPathList) {
                    byte[] bytes = OpenCVUtils.matOfKeyPointImage(imageAbsPath);
                    // TODO 这里不合适，后续需要再次调整，由于lucene的限制，这里限制了特征向量的长度。 这里主要是把 byte[] -> float[], 并且只取前面1024个特征。
                    float[] floats = OpenCVUtils.normalize(OpenCVUtils.convertArray(OpenCVUtils.byteArrayToFloatList(bytes), 1024));
                    if(firstVector == null) {
                        firstVector = floats;
                    }
                    Document doc = new Document();
                    doc.add(new StoredField("id", imageAbsPath));
                    doc.add(new KnnVectorField("field", floats));
                    writer.addDocument(doc);
                }
            }
            System.out.println();
            try (IndexReader reader = DirectoryReader.open(dir)) {
                IndexSearcher searcher = new IndexSearcher(reader);

                TopDocs results = searcher.search(new KnnVectorQuery("field", firstVector, 3), 10);
                System.out.println("Hits: " + results.totalHits);
                for (ScoreDoc sdoc : results.scoreDocs) {
                    Document doc = reader.document(sdoc.doc);
                    StoredField idField = (StoredField) doc.getField("id");
                    System.out.println("Found: " + idField.toString() + " = " + String.format("%.1f", sdoc.score));
                }
            }
        }

    }
```

## 参考
1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/pap4j-boot3
