## 背景

&ensp;&ensp;《Excel数据转换为一对多的工具类》在ERP类的软件中，会存在大量的Excel导入导出功能，本文提供一种工具，将Excel中多个Sheet中的数据，转换为一对多的关系，并且支持自定义关联的关联关系设定。

## 示例

&ensp;&ensp; Excel中SheetName=='订单头'的数据

|  订单编号   |        收货人         | 收货电话 | 收货地址 |
|:-----------:|:-----------------------:|:------------------------:|:------------------------:|
|      PAP20240101101010      |           ALEXGAOYH            | 13800138000 | pap.net.cn |
|      PAP20240101101011      |      ALEXGAOYH       | 13800138000 | 河南省 |

&ensp;&ensp; Excel中SheetName=='订单明细'的数据

|订单编号|收货人|收货电话|收货地址|SKU编号|数量|
|:----|:----|:----|:----|:----|:----|
|PAP20240101101010|ALEXGAOYH|13800138000|pap.net.cn|SKU000001|1|
|PAP20240101101010|ALEXGAOYH|13800138000|pap.net.cn|SKU000002|2|
|PAP20240101101010|ALEXGAOYH|13800138000|pap.net.cn|SKU000003|3|
|PAP20240101101011|ALEXGAOYH|13800138000|河南省|SKU000004|4|
|PAP20240101101011|ALEXGAOYH|13800138000|河南省|SKU000005|5|
|PAP20240101101011|ALEXGAOYH|13800138000|河南省|SKU000006|6|
|PAP20240101101011|ALEXGAOYH|13800138000|河南省|SKU000007|7|

&ensp;&ensp; 转换为一对多的关系，出参结构为：
```html
[
	{
		订单编号=PAP20240101101010, 收货人=ALEXGAOYH, 收货电话=13800138000, 
                收货地址=pap.net.cn, cn_net_pap_index=1, 
		cn_net_pap_child= [
			{订单编号=PAP20240101101010, 收货人=ALEXGAOYH, 收货电话=13800138000, 收货地址=pap.net.cn, SKU编号=SKU000001, 数量=1, cn_net_pap_index=1}, 
			{订单编号=PAP20240101101010, 收货人=ALEXGAOYH, 收货电话=13800138000, 收货地址=pap.net.cn, SKU编号=SKU000002, 数量=2, cn_net_pap_index=2}, 
			{订单编号=PAP20240101101010, 收货人=ALEXGAOYH, 收货电话=13800138000, 收货地址=pap.net.cn, SKU编号=SKU000003, 数量=3, cn_net_pap_index=3}
		]
	},
	{
		订单编号=PAP20240101101011, 收货人=ALEXGAOYH, 收货电话=13800138000, 
                收货地址=河南省, cn_net_pap_index=2, 
		cn_net_pap_child= [
			{订单编号=PAP20240101101011, 收货人=ALEXGAOYH, 收货电话=13800138000, 收货地址=河南省, SKU编号=SKU000004, 数量=4, cn_net_pap_index=4}, 
			{订单编号=PAP20240101101011, 收货人=ALEXGAOYH, 收货电话=13800138000, 收货地址=河南省, SKU编号=SKU000005, 数量=5, cn_net_pap_index=5}, 
			{订单编号=PAP20240101101011, 收货人=ALEXGAOYH, 收货电话=13800138000, 收货地址=河南省, SKU编号=SKU000006, 数量=6, cn_net_pap_index=6}, 
			{订单编号=PAP20240101101011, 收货人=ALEXGAOYH, 收货电话=13800138000, 收货地址=河南省, SKU编号=SKU000007, 数量=7, cn_net_pap_index=7}
		]
	}
]
```

## 调用方法

&ensp;&ensp; 此方法分别传入 head-excel、detail-excel对应的文件绝对路径和sheetName，之后设定两个sheet对应的字段关联关系，本例是将 '订单编号-收货人-收货电话' 一起当做两个sheet的业务关联字段，之后设置 顺序号和一对多的字段名称。顺序号是为了如果导入失败，友好的展示出来数据在文件中的第几行，一对多字段是为了防止字段重复。

```java
public class ExcelUtilTest {
    @Test
    public void extract() {
        String excelAbsolutePath = "test.xlsx";

        List<CompareDTO> compareDTOS = new ArrayList<>();
        compareDTOS.add(new CompareDTO("订单编号", "订单编号"));
        compareDTOS.add(new CompareDTO("收货人", "收货人"));
        compareDTOS.add(new CompareDTO("收货电话", "收货电话"));

        List<Map<String, Object>> oneToManyRowList = ExcelUtil.getOneToManyRowList(excelAbsolutePath, "订单头",
                excelAbsolutePath, "订单明细",
                compareDTOS,
                "cn_net_pap_index",
                "cn_net_pap_child");
    }
}
```

## 参考
1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh
