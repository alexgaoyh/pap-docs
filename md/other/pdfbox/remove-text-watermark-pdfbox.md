# [杂谈] pdfbox 2.0.28 文字水印移除操作(Acrobat/WPS)

## 介绍

&ensp;&ensp;近期在调研 OCR 相关的技术选型，在实际的应用场景中，发现如果传入 OCR 的图片信息包含水印的话，会对识别结果造成影响，故需要优先对数据进行预处理。

## 思路

&ensp;&ensp;源数据信息是 pdf 类型的文件，将其转为 png 图像后传入 ocr 进行处理，所有可以在任意一个阶段进行处理，经过对比选择在 pdf 阶段进行水印移除操作。

&ensp;&ensp;原因在于源数据pdf文件是文字版本的（非影印版、扫描版），程序可以对 pdf 文件进行读取，之后进行水印移除操作。

## 代码

&ensp;&ensp;使用java语言，引用 pdfbox 2.0.28 版本进行水印移除操作。

&ensp;&ensp;本文采用的pdf文件中的水印，是通过 Adobe Acrobat Pro DC 或者 WPS PDF 这两种软件进行的水印添加，同其他介绍不一致的地方在于，通过这两种软件添加的水印，在使用 pdfbox 进行操作的时候，需要采用如下的方式。

&ensp;&ensp;主要是获取 pdf 中的所有元素，并且通过判断元素的类型，如果发现是水印相关的，移除当前元素，之后将处理过的所有元素复制到一个新的 pdf 文件中。

```java
import org.apache.pdfbox.contentstream.operator.Operator;
import org.apache.pdfbox.cos.COSName;
import org.apache.pdfbox.pdfparser.PDFStreamParser;
import org.apache.pdfbox.pdfwriter.ContentStreamWriter;
import org.apache.pdfbox.pdmodel.PDDocument;
import org.apache.pdfbox.pdmodel.PDPage;
import org.apache.pdfbox.pdmodel.common.PDStream;
import org.apache.pdfbox.pdmodel.font.PDFont;
import org.apache.pdfbox.pdmodel.font.PDType0Font;

import java.io.File;
import java.io.OutputStream;
import java.util.List;

/**
 * 使用 pdfbox 移除 pdf文件中的 文字水印
 * 思路： 获取到 pdf 中的所有元素， 之后判断 元素是否是水印相关(op.getName().equals("gs"))，如果是的话，将其删除(tokens.remove(tokens.size() - 1);)
 */
public class PdfBoxRemoveTextWatermark {

    /**
     * @param inputPdfFilePath  输入pdf文件路径       alexgaoyh.pdf
     * @param outputFdfFilePath 输出pdf文件路径       removed-text-watermark-alexgaoyh.pdf
     * @param fontPath          字体文件             simfang.ttf
     * @throws Exception
     */
    public static void removeTextWatermark(String inputPdfFilePath, String outputFdfFilePath, String fontPath) throws Exception {
        {
            File file = new File(inputPdfFilePath);
            PDDocument pd = PDDocument.load(file);
            // 需要的字体文件
            Map<COSName, PDFont> oldfont = new HashMap<COSName, PDFont>();
            PDType0Font targetfont = PDType0Font.load(pd, new File(fontPath));
            for (PDPage page : pd.getPages()) {
                PDFStreamParser pdfsp = new PDFStreamParser(page);
                pdfsp.parse();
                List<Object> tokens = pdfsp.getTokens();
                for (int j = 0; j < tokens.size(); j++) {
                    //创建一个object对象去接收标记
                    Object next = tokens.get(j);
                    if (next instanceof Operator) {
                        Operator op = (Operator) next;
                        if (op.getName().equals("gs")) {
                            tokens.remove(tokens.size() - 1);
                        }
                    }
                }
                PDStream updatedStream = new PDStream(pd);
                OutputStream out = updatedStream.createOutputStream();
                ContentStreamWriter tokenWriter = new ContentStreamWriter(out);
                tokenWriter.writeTokens(tokens);
                out.close();
                oldfont.forEach((k, v) -> {
                    page.getResources().put(k, targetfont);
                });
                page.setContents(updatedStream);
            }
            pd.save(outputFdfFilePath);
            pd.close();
        }
    }
}
```

## 总结

&ensp;&ensp;如上方法仅在 pdfbox 2.0.28 版本下进行了测试，其他版本下未进行测试，原因在于 pdfbox 每个版本下的方法变化较大。

&ensp;&ensp;当前方案采用的整体思路是获取原始 pdf 文件中的所有元素，根据元素的类型进行删除，之后将处理过的元素添加到一个新的 pdf 中（类似复制）。

&ensp;&ensp;需要注意在使用 Adobe Acrobat Pro DC 或者 WPS PDF 这两种软件添加水印的时候，使用的是 文字水印 的方法，而不是图片的方式。

## 参考

1. https://gitee.com/alexgaoyh/pap-base/blob/v1/src/test/java/com/pap/base/util/pdf/PdfBoxRemoveTextWatermark.java
2. https://github.com/Halfish/lstm-ctc-ocr/blob/master/fonts/simfang.ttf
3. https://blog.csdn.net/dgywj/article/details/111588270
4. https://blog.51cto.com/u_15127580/4107427

