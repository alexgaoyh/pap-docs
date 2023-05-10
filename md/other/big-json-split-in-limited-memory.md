# [杂谈]大型JSON数据切分(Java Jackson)

## 介绍

&ensp;&ensp;最近遇到一个需求，期望将一个大型的json文件存储至Elasticsearch中，第一反应是期望将原始数据进行拆分，这样就可以在受限的内存空间中完成数据的导入。

## 思路

&ensp;&ensp;本文使用 JAVA 语言进行切分，并且使用 jackson 组件。
&ensp;&ensp;对 JSON 数据进行处理，首先想到的进行 反序列化 操作，但是这样的话，会将所有数据同时存储在内存中，在受限内存的环境下并不友好，故放弃此方法。
&ensp;&ensp;改为使用 JsonParser 这个更底层的解析类进行数据处理，在设置 -Xmx10M 参数的前提下，能够正常解析 180M 的一个 JSON 文件。

## 代码

&ensp;&ensp;分为四个例子，进行递进的理解


## 代码一：使用 JsonParser 解析JSON，根节点是对象。

```json
{
	"name": "alexgaoyh",
	"attrs": [{
			"identifier": "100001",
			"address": ""
		},
		{
			"identifier": "100002",
			"address": ""
		},
		{
			"identifier": "100003",
			"address": "",
			"personal": [{
				"remark": "",
				"name": "alexgaoyh"
			}]
		},
		{
			"identifier": "100004",
			"address": ""
		}
	]
}
```
```html
    public void processTopObject(String jsonFilePath) {
        File jsonFile = new File(jsonFilePath);
        JsonFactory jsonfactory = new JsonFactory();
        try {
            List<Map<String, Object>> allMapList = new ArrayList<>();
            Map<String, Object> currMap = new HashMap<>();
            JsonParser jsonParser = jsonfactory.createJsonParser(jsonFile);
            JsonToken jsonToken = jsonParser.nextToken();

            // String nameFieldText = null;

            while (jsonToken != JsonToken.END_OBJECT) {
                String fieldname = jsonParser.getCurrentName();
                if ("name".equals(fieldname)) {
                    jsonToken = jsonParser.nextToken();
                    // nameFieldText = jsonParser.getText();
                }
                if ("attrs".equals(fieldname)) {
                    jsonToken = jsonParser.nextToken();
                    if (jsonToken == JsonToken.START_ARRAY) {
                        jsonToken = jsonParser.nextToken();
                        while (jsonToken != JsonToken.END_ARRAY) {
                            String fieldnameDetail = jsonParser.getCurrentName();
                            if ("identifier".equals(fieldnameDetail)) {
                                jsonToken = jsonParser.nextToken();
                                // currMap.put("name", nameFieldText);
                                currMap.put(fieldnameDetail, jsonParser.getText());
                            }
                            // 无用的 personl 处理一下之后跳过
                            if ("personal".equals(fieldnameDetail)) {
                                jsonToken = jsonParser.nextToken();
                                while (jsonParser.nextToken() != JsonToken.END_ARRAY) {
                                }
                            }
                            if (jsonToken == JsonToken.END_OBJECT) {
                                allMapList.add(currMap);
                                currMap = new HashMap<>();
                            }
                            jsonToken = jsonParser.nextToken();
                        }
                    }
                }
                jsonToken = jsonParser.nextToken();
            }
            System.out.println("Total Records Found : " + allMapList.size());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

## 代码二：使用 JsonParser 解析JSON，根节点是数组

```json
[{
		"identifier": "100001",
		"address": ""
	},
	{
		"identifier": "100002",
		"address": ""
	},
	{
		"identifier": "100003",
		"address": "",
		"personal": [{
			"remark": "",
			"name": "alexgaoyh"
		}]
	},
	{
		"identifier": "100004",
		"address": ""
	}
]
```
```html
    public void processTopArray(String jsonFilePath) {
        File jsonFile = new File(jsonFilePath);
        JsonFactory jsonfactory = new JsonFactory();
        try {
            List<Map<String, String>> allMapList = new ArrayList<>();
            Map<String, String> currMap = new HashMap<>();
            JsonParser jsonParser = jsonfactory.createJsonParser(jsonFile);
            JsonToken jsonToken = jsonParser.nextToken();
            while (jsonToken != JsonToken.END_ARRAY) {
                String fieldname = jsonParser.getCurrentName();
                if ("identifier".equals(fieldname)) {
                    jsonToken = jsonParser.nextToken();
                    currMap.put(fieldname, jsonParser.getText());
                }
                // 无用的 personl 处理一下之后跳过
                if ("personal".equals(fieldname)) {
                    jsonToken = jsonParser.nextToken();
                    while (jsonParser.nextToken() != JsonToken.END_ARRAY) {
                    }
                }
                if (jsonToken == JsonToken.END_OBJECT) {
                    allMapList.add(currMap);
                    currMap = new HashMap<>();
                }
                jsonToken = jsonParser.nextToken();
            }
            System.out.println("Total Records Found : " + allMapList.size());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

## 代码三：使用 JsonParser 切分JSON，根节点是数组

&ensp;&ensp; 代码三使用的 json 数据同代码二一样，不同之处在于按照对象进行解析，便于后续的json结构切分。
```html
    public void processTopArray(String jsonFilePath) {
        File jsonFile = new File(jsonFilePath);
        try {
            ObjectMapper mapper = new ObjectMapper();
            JsonParser parser = mapper.getFactory().createParser(jsonFile);
            if(parser.nextToken() != JsonToken.START_ARRAY) {
                throw new IllegalStateException("Expected an array");
            }
            while(parser.nextToken() == JsonToken.START_OBJECT) {
                ObjectNode node = mapper.readTree(parser);
                System.out.println(node);
            }

            parser.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

## 代码四：使用 JsonParser 切分JSON，根节点是数组
&ensp;&ensp; [大小为180M的JSON文件](https://github.com/zemirco/sf-city-lots-json/blob/master/citylots.json)，使用互联网上的大型JSON文件，并对其进行解析，为切分做准备
&ensp;&ensp; 注意在进行解析的时候，增加了针对 BigDecimal 数据的处理，从而避免了精度丢失。
&ensp;&ensp; 按照代码逻辑，features 数组下的数据按照对象进行输出，从而就可以根据切分规则进行切分。
&ensp;&ensp; 比如定义一个 List<ObjectNode> tmpList = new ArrayList<>(); 将 ObjectNode node 对象添加进来，就可以理解为切分后的对象。
```html
    public void parseObject(String jsonFilePath) {
        File jsonFile = new File(jsonFilePath);
        JsonFactory jsonfactory = new JsonFactory();

        ObjectMapper mapper = new ObjectMapper();
        mapper.setNodeFactory(JsonNodeFactory.withExactBigDecimals(true));
        mapper.configure(DeserializationFeature.USE_BIG_DECIMAL_FOR_FLOATS, true);
        try {
            JsonParser jsonParser = jsonfactory.createJsonParser(jsonFile);
            JsonToken jsonToken = jsonParser.nextToken();

            while (jsonToken != JsonToken.END_OBJECT) {
                String fieldname = jsonParser.getCurrentName();
                if ("type".equals(fieldname)) {
                    jsonToken = jsonParser.nextToken();
                }
                if ("features".equals(fieldname)) {
                    jsonToken = jsonParser.nextToken();
                    if (jsonToken == JsonToken.START_ARRAY) {

                        while(jsonParser.nextToken() == JsonToken.START_OBJECT) {
                            ObjectNode node = mapper.readTree(jsonParser);
                            //System.out.println(node);
                        }
                    }
                }
                jsonToken = jsonParser.nextToken();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

## 总结

&ensp;&ensp;将大量数据切分为相同结果的多组小数据，从而可以进行并行处理；

## 参考

1. https://stackoverflow.com/questions/24835431/use-jackson-to-stream-parse-an-array-of-json-objects
2. https://blog.csdn.net/weixin_42338707/article/details/111296652
3. https://github.com/zemirco/sf-city-lots-json/blob/master/citylots.json
