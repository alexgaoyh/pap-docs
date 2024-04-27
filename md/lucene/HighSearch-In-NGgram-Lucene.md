## 背景

&ensp;&ensp;《Lucene环境下基于NGram分词的中文高级检索》本文是在中文环境下，基于Lucene搜索引擎的NGram分词器，实现的高级检索功能，配合多种不同形式下的搜索条件进行检索。

&ensp;&ensp;当前高级检索的实现思路为：按照搜索条件的顺序，一个条件一个条件的添加搜索条件。比如： （A must B should C must D）代表 ((A与B)或C)与D)。

## 测试样本

```json
[
  {"title":  "番茄原产南美洲，学名番茄。。", "remark":  "用来进行额外的说明备注字段", "seqNo":  123456},
  {"title":  "番茄，即西红柿，石头里面。", "remark":  "https://pap-docs.pap.net.cn/ 开发测试的说明备注字段", "seqNo":  123457},
  {"title":  "番石榴", "remark":  "搜索一下：https://gitee.com/alexgaoyh", "seqNo":  123458}
]
```

## 搜索示例

&ensp;&ensp;接下来列举多种搜索条件，并且返回搜索条件下匹配的搜索结果和针对title字段的搜索结果高亮展示。


|                            搜索条件                             | 命中条数 | 
|:-----------------------------------------------------------:|:----:|
|            (remark 模糊 ‘榴’) must (remark 精确 ‘ao’)            |  1   |
|           (remark 模糊 ‘榴’) should (remark 精确 ‘ao’)           |  2   |
| (remark 模糊 ‘榴’) should (remark 精确 ‘ao’) must (title 精确 ‘石’) |  2   |
| (remark 模糊 ‘榴’) must (remark 精确 ‘ao’) must (title 精确 ‘石’) |  2   |
| (remark 模糊 ‘榴’) should (remark 精确 ‘ao’) should (title 精确 ‘石’) |  2   |


```html
    // relation-关系(与must / 或shoulud)， field-搜索字段， keyword-搜索关键词， match(精确term /模糊fuzzy)， boost 权重
    // 在页面交互的过程中，第一行的搜索条件是没有关系的，这里以第二行搜索条件的关系为准。
    public HighSearchDTO(String relation, String field, String keyword, String match, float boost) {
        this.relation = relation;
        this.field = field;
        this.keyword = keyword;
        this.match = match;
        this.boost = boost;
    }

    // 此条件说明 必须包含如下两个条件： tilte模糊搜索‘榴’   remark精确搜索‘ao’ 
    // 搜索结果
    // 命中一条记录： 
    //      1、番石<font color='red'>榴</font>
    highSearchDTOs.add(new HighSearchDTO("", "title", "榴", "fuzzy", 4.0f));
    highSearchDTOs.add(new HighSearchDTO("must", "remark", "ao", "term", 4.0f));


    // 此条件说明 或包含如下两个条件： tilte模糊搜索‘榴’ remark精确搜索‘ao’ 
    // 搜索结果
    // 命中两条记录： 
    //      1、番石<font color='red'>榴</font>     
    //      2、番茄，即西红柿，石头里面。
    highSearchDTOs.add(new HighSearchDTO("", "title", "榴", "fuzzy", 4.0f));
    highSearchDTOs.add(new HighSearchDTO("should", "remark", "ao", "term", 4.0f));


    // 此条件说明 精确包含如下两个条件 (模糊包含如下两个条件： tilte模糊搜索‘榴’ remark精确搜索‘ao’ ) title精确搜索‘石’ 
    // 搜索结果
    // 命中两条记录： 
    //      1、番<font color='red'>石</font><font color='red'>榴</font>   
    //      2、番茄，即西红柿，<font color='red'>石</font>头里面。
    highSearchDTOs.add(new HighSearchDTO("", "title", "榴", "fuzzy", 4.0f));
    highSearchDTOs.add(new HighSearchDTO("should", "remark", "ao", "term", 4.0f));
    highSearchDTOs.add(new HighSearchDTO("must", "title", "石", "term", 4.0f));


    // 此条件说明 精确包含如下两个条件 (精确包含如下两个条件： tilte模糊搜索‘榴’ remark精确搜索‘ao’ ) title精确搜索‘石’
    // 搜索结果
    // 命中一条记录： 
    //      1、番<font color='red'>石</font><font color='red'>榴</font>
    highSearchDTOs.add(new HighSearchDTO("", "title", "榴", "fuzzy", 4.0f));
    highSearchDTOs.add(new HighSearchDTO("must", "remark", "ao", "term", 4.0f));
    highSearchDTOs.add(new HighSearchDTO("must", "title", "石", "term", 4.0f));


    // 此条件说明 或包含如下两个条件 (或包含如下两个条件： tilte模糊搜索‘榴’ remark精确搜索‘ao’ ) title精确搜索‘石’
    // 搜索结果
    // 命中两条记录： 
    //      1、番石<font color='red'>榴</font>      
    //      2、番茄，即<font color='red'>西</font>红柿，石头里面。
    highSearchDTOs.add(new HighSearchDTO("", "title", "榴", "fuzzy", 4.0f));
    highSearchDTOs.add(new HighSearchDTO("should", "remark", "ao", "term", 4.0f));
    highSearchDTOs.add(new HighSearchDTO("should", "title", "西", "term", 4.0f));
    

```

## 实现

&ensp;&ensp;列出如何将HighSearchDTO对象集合转换为Lucen的Query搜索对象.

```html
    public static BooleanQuery.Builder convert(List<HighSearchDTO> paramsList, Analyzer analyzer) throws Exception {

        BooleanQuery.Builder bq = new BooleanQuery.Builder();

        if(paramsList.size() > 1) {
            for(int idx = 0; idx < paramsList.size() - 1; idx++) {
                if(idx == 0) {
                    bq = merge(paramsList.get(idx), paramsList.get(idx + 1), analyzer);
                } else {
                    bq = merge(bq, paramsList.get(idx + 1), analyzer);
                }
            }
        } else {
            HighSearchDTO left = paramsList.get(0);
            if(left.getMatch().equals("term")) {
                QueryParser queryParser = new QueryParser(left.getField(), analyzer);
                queryParser.setDefaultOperator(QueryParser.Operator.AND);
                Query queryLeft = queryParser.parse(left.getKeyword());
                bq.add(queryLeft, BooleanClause.Occur.MUST);
            } else if(left.getMatch().equals("fuzzy")) {
                QueryParser queryParser = new QueryParser(left.getField(), analyzer);
                Query queryLeft = queryParser.parse(left.getKeyword());
                bq.add(queryLeft, BooleanClause.Occur.MUST);
            }
        }

        return bq;
    }

    public static BooleanQuery.Builder merge(HighSearchDTO left, HighSearchDTO right, Analyzer analyzer) throws Exception {
        BooleanQuery.Builder bq = new BooleanQuery.Builder();

        Query queryLeft = null;
        if(left.getMatch().equals("term")) {
            QueryParser queryParser = new QueryParser(left.getField(), analyzer);
            queryParser.setDefaultOperator(QueryParser.Operator.AND);
            queryLeft = queryParser.parse(left.getKeyword());
        } else if(left.getMatch().equals("fuzzy")) {
            QueryParser queryParser = new QueryParser(left.getField(), analyzer);
            queryLeft = queryParser.parse(left.getKeyword());
        }

        Query queryRight = null;
        if(right.getMatch().equals("term")) {
            QueryParser queryParser = new QueryParser(right.getField(), analyzer);
            queryParser.setDefaultOperator(QueryParser.Operator.AND);
            queryRight = queryParser.parse(right.getKeyword());
        } else if(right.getMatch().equals("fuzzy")) {
            QueryParser queryParser = new QueryParser(right.getField(), analyzer);
            queryRight = queryParser.parse(right.getKeyword());
        }

        if(right.getRelation().equals("must")) {
            bq.add(queryLeft, BooleanClause.Occur.MUST);
            bq.add(queryRight, BooleanClause.Occur.MUST);
        } else {
            bq.add(queryLeft, BooleanClause.Occur.SHOULD);
            bq.add(queryRight, BooleanClause.Occur.SHOULD);
            bq.setMinimumNumberShouldMatch(1);
        }

        return bq;
    }

    public static BooleanQuery.Builder merge(BooleanQuery.Builder leftQuery, HighSearchDTO right, Analyzer analyzer) throws Exception {

        BooleanQuery.Builder bq = new BooleanQuery.Builder();

        Query queryRight = null;
        if(right.getMatch().equals("term")) {
            QueryParser queryParser = new QueryParser(right.getField(), analyzer);
            queryParser.setDefaultOperator(QueryParser.Operator.AND);
            queryRight = queryParser.parse(right.getKeyword());
        } else if(right.getMatch().equals("fuzzy")) {
            QueryParser queryParser = new QueryParser(right.getField(), analyzer);
            queryRight = queryParser.parse(right.getKeyword());
        }

        if(right.getRelation().equals("must")) {
            bq.add(leftQuery.build(), BooleanClause.Occur.MUST);
            bq.add(queryRight, BooleanClause.Occur.MUST);
        } else {
            bq.add(leftQuery.build(), BooleanClause.Occur.SHOULD);
            bq.add(queryRight, BooleanClause.Occur.SHOULD);
            bq.setMinimumNumberShouldMatch(1);
        }

        return bq;
    }
```

## 参考
1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh
