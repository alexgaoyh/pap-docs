## 背景

&ensp;&ensp;《图像处理-Java-TIFF转换JPG》，之前在编写过关于[图像边缘检测-去黑边](https://pap-docs.pap.net.cn/#/md/algorithm/image/remove-black-border)、[图像边缘检测-自动纠偏](https://pap-docs.pap.net.cn/#/md/algorithm/image/auto-correction)、[图像处理-锐化](https://pap-docs.pap.net.cn/#/md/algorithm/image/sharpening-prewitt-overlay)、[图像处理-去噪/高斯模糊/套红](https://pap-docs.pap.net.cn/#/md/algorithm/image/image-denoise-gaussianBlur-red)、[图像处理-背景色平滑/反色](https://pap-docs.pap.net.cn/#/md/algorithm/image/image-backgroundSmooth-invert)、[图像处理-指定大小压缩](https://pap-docs.pap.net.cn/#/md/algorithm/image/image-compress-to-target-size)等一系列文章，接下来主要介绍TIFF格式的图像处理。

## 概述

&ensp;&ensp;使用JAVA语言实现，将给定的TIF格式的图像转换为JPG，其中TIF格式的图像有两种，一种是未经过压缩的，另一种是经过 LZW 压缩的。本文提供两个函数对其进行分别处理。

## 代码实现

```html
    // dependency com.twelvemonkeys.imageio.imageio-tiff.3.10.1  handle no-LZW compress TIF file
    public static void tiffToJpg(String tiffFilePath, String jpgFilePath) throws IOException, InterruptedException {
        ImageReader reader = ImageIO.getImageReadersByFormatName("TIF").next();
        ImageInputStream imageInputStream = new FileImageInputStream(new File(tiffFilePath));
        reader.setInput(imageInputStream);

        ImageWriter writer = ImageIO.getImageWritersByFormatName("JPEG").next();
        ImageOutputStream imageOutputStream = ImageIO.createImageOutputStream(new File(jpgFilePath));
        writer.setOutput(imageOutputStream);

        ImageWriteParam param = writer.getDefaultWriteParam();
        param.setCompressionMode(ImageWriteParam.MODE_EXPLICIT);
        param.setCompressionQuality(0.8f); // adjust the quality as needed

        int numImages = reader.getNumImages(true);

        int availableProcessors = Runtime.getRuntime().availableProcessors();
        ExecutorService executorService = Executors.newFixedThreadPool(availableProcessors);

        for (int i = 0; i < numImages; i++) {
            final int imageIndex = i;
            executorService.submit(() -> {
                try {
                    BufferedImage image = reader.read(imageIndex);
                    byte[] imageData = extractImageData(image);

                    synchronized (writer) {
                        writer.write(null, new javax.imageio.IIOImage(image, null, null), param);
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
            });
        }

        executorService.shutdown();
        executorService.awaitTermination(Long.MAX_VALUE, TimeUnit.NANOSECONDS);

        reader.dispose();
        writer.dispose();
        imageInputStream.close();
        imageOutputStream.close();
    }

    private static byte[] extractImageData(BufferedImage image) {
        Raster raster = image.getData();
        DataBufferByte dataBuffer = (DataBufferByte) raster.getDataBuffer();
        return dataBuffer.getData();
    }

    // dependency org.apache.commons.commons-imaging.1.0-alpha3  handle LZW compress TIF file
    public static void tiffToJpg(String inputFilePath, String outputFilePath) {
        File inputFile = new File(inputFilePath);
        File outputFile = new File(outputFilePath);
        
        try (FileInputStream inputStream = new FileInputStream(inputFile);
            FileOutputStream outputStream = new FileOutputStream(outputFile);
            ImageOutputStream imageOutputStream = ImageIO.createImageOutputStream(outputFile);
            ImageInputStream imageInputStream = ImageIO.createImageInputStream(inputFile)) {
        
            TiffImageParser parser = new TiffImageParser();
            BufferedImage image = Imaging.getBufferedImage(inputStream, null);
            
            ImageIO.write(image, "jpg", outputStream);
            
            ImageReader imageReader = ImageIO.getImageReaders(imageInputStream).next();
            imageReader.setInput(imageInputStream);
            IIOMetadata metadata = imageReader.getImageMetadata(0);
            ImageWriter writer = ImageIO.getImageWritersByFormatName("JPEG").next();
            writer.setOutput(imageOutputStream);
            ImageWriteParam param = writer.getDefaultWriteParam();
            param.setCompressionMode(ImageWriteParam.MODE_EXPLICIT);
            param.setCompressionQuality(1f);
            writer.write(metadata, new IIOImage(image, null, null), param);
        
        } catch (Exception e) {
        }
    }

```

## 参考
1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh
