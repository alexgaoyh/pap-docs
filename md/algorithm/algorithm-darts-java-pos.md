## 背景
&ensp;&ensp;在 NLP 世界中，中英文环境的很大一个差别就是中文不存在英文中的空格，所以在实际的应用中往往需要提前内置大量的字典（词），用这些已经被定义好的字典（词）来进行文本的快速分词，本文介绍的双数组字典树就是其中的一种解决方案。

## 相关技术
&ensp;&ensp;双数组字典树(Double Array Trie)改进自 Trie，但是与 HaNLP 中的 AhoCorasickDoubleArrayTrie 又略有不足。

1. Trie
   1. Trie 树是搜索树的一种，它在本质上是一个确定的有限状态自动机，每个结点代表一个状态，根据输入变量的不同，进行状态转移。
   2. 为了减少 Trie 树结构的空间浪费，同时保证 Trie[/size]树查询的效率，有研究者提出了用两个数组来表示 Trie 树，也就是双数组 Trie 树(Double-Array Trie)
   3. darts-java 是对 Taku Kudo 的 C++ 版 Double Array Trie 的 Java 移植，只有一个 Java 文件。
   4. darts-java 在进行查找的过程中，提供的方法是查询共同前缀的，如果想要在一个大文本中查询到所有的字典，时间复杂度较高（可以对比 AhoCorasickDoubleArrayTrie）。
2. HaNLP AhoCorasickDoubleArrayTrie
   1. 使用 Double Array Trie 实现了一个性能极高的 Aho Corasick 自动机。
   2. 在 HaNLP 中广泛使用，比如分词任务、词性标注。
   3. DoubleArrayTrie和AhoCorasickDoubleArrayTrie的实用性对比


## [基于 darts-java 的改进](https://gitee.com/alexgaoyh/darts-java/commit/f13008157657d730b23c8f362b6b179745e583b1)
&ensp;&ensp;在使用 darts-java 的过程中，发现它只存储了字典字符串，没有存储其他的比如词性的信息，期望在进行查找的过程中，不仅仅能够查找出来字典，还能够查询出来额外的数据。
&ensp;&ensp;这样改进的好处在于在获得分词的时候，同样能够获取到提前内置进来的字典的其他属性值，详见下面代码示例。

```html
public static void main(String[] args) {
    List<ValueAttrDTO> valueAttrDTOs = new ArrayList<ValueAttrDTO>();
    // 词性： 形容词 用 a 表示， a对应的ascii码是 97
    valueAttrDTOs.add(new ValueAttrDTO("一举", 97));
    // 词性： 成语 用 i 表示， i对应的ascii码是 105
    valueAttrDTOs.add(new ValueAttrDTO("一举一动", 105));
    valueAttrDTOs.add(new ValueAttrDTO("一举成名", 105));
    valueAttrDTOs.add(new ValueAttrDTO("一举成名天下知", 105));
    // 词性： 形容词 用 a 表示， a对应的ascii码是 97
    valueAttrDTOs.add(new ValueAttrDTO("万能", 97));
    // 词性： 名词 用 n 表示， n对应的ascii码是 110
    valueAttrDTOs.add(new ValueAttrDTO("万能胶", 110));
    DoubleArrayTrie doubleArrayTrie = new DoubleArrayTrie();
    doubleArrayTrie.build(valueAttrDTOs);
    // doubleArrayTrie.dump();
    List<Double> doubles = doubleArrayTrie.commonPrefixSearch("一举成名天下知的是一个成语");
    if(doubles != null && doubles.size() > 0) {
        for(double _double : doubles) {
            int intPart = (int)_double;
            System.out.println("word : " + valueAttrDTOs.get(intPart).getValue() +
                    " ;pos : " + valueAttrDTOs.get(intPart).getPos());
        }
    }
}

// 返回值分别输出了前缀的词语word，并且包含所属词性pos对应的ASCII码。
word : 一举 ;pos : 97
word : 一举成名 ;pos : 105
word : 一举成名天下知 ;pos : 105
```

&ensp;&ensp;主要的改进点在于，按照 darts-java 的代码逻辑，如果 Base 数组对应的值是一个负数，则说明是一个结尾。这里将数组的类型由 int 改为 double，前面的 int 逻辑保持不变，后面增加的小数位包含所属的词性信息（本例使用小数点后四位小数存储词性信息）。如果还有额外的信息需要存储，同样可以按照这种方法进行处理。

```java
public class DoubleArrayTrie {
    private final static int BUF_SIZE = 16384;
    private final static int UNIT_SIZE = 8; // size of int + int

    private static class Node {
        int code;
        int depth;
        int left;
        int right;

        int pos;
    };

    private double check[];
    private double base[];

    private boolean used[];
    private int size;
    private int allocSize;
    private List<String> key;
    private List<ValueAttrDTO> valueAttrDTOs;
    private int keySize;
    private int length[];
    private int value[];
    private int progress;
    private int nextCheckPos;
    // boolean no_delete_;
    int error_;

    // int (*progressfunc_) (size_t, size_t);

    // inline _resize expanded
    private int resize(int newSize) {
        double[] base2 = new double[newSize];
        double[] check2 = new double[newSize];
        boolean used2[] = new boolean[newSize];
        if (allocSize > 0) {
            System.arraycopy(base, 0, base2, 0, allocSize);
            System.arraycopy(check, 0, check2, 0, allocSize);
            System.arraycopy(used2, 0, used2, 0, allocSize);
        }

        base = base2;
        check = check2;
        used = used2;

        return allocSize = newSize;
    }

    private int fetch(Node parent, List<Node> siblings) {
        if (error_ < 0)
            return 0;

        int prev = 0;

        for (int i = parent.left; i < parent.right; i++) {
            if ((length != null ? length[i] : key.get(i).length()) < parent.depth)
                continue;

            String tmp = key.get(i);

            int cur = 0;
            if ((length != null ? length[i] : tmp.length()) != parent.depth)
                cur = (int) tmp.charAt(parent.depth) + 1;

            if (prev > cur) {
                error_ = -3;
                return 0;
            }

            if (cur != prev || siblings.size() == 0) {
                Node tmp_node = new Node();
                tmp_node.depth = parent.depth + 1;
                tmp_node.code = cur;
                tmp_node.left = i;
                if (siblings.size() != 0)
                    siblings.get(siblings.size() - 1).right = i;
                tmp_node.pos = valueAttrDTOs.get(i).pos;

                siblings.add(tmp_node);
            }

            prev = cur;
        }

        if (siblings.size() != 0)
            siblings.get(siblings.size() - 1).right = parent.right;

        return siblings.size();
    }

    private int insert(List<Node> siblings) {
        if (error_ < 0)
            return 0;

        int begin = 0;
        int pos = ((siblings.get(0).code + 1 > nextCheckPos) ? siblings.get(0).code + 1
                : nextCheckPos) - 1;
        int nonzero_num = 0;
        int first = 0;

        if (allocSize <= pos)
            resize(pos + 1);

        outer: while (true) {
            pos++;

            if (allocSize <= pos)
                resize(pos + 1);

            if (check[pos] != 0) {
                nonzero_num++;
                continue;
            } else if (first == 0) {
                nextCheckPos = pos;
                first = 1;
            }

            begin = pos - siblings.get(0).code;
            if (allocSize <= (begin + siblings.get(siblings.size() - 1).code)) {
                // progress can be zero
                double l = (1.05 > 1.0 * keySize / (progress + 1)) ? 1.05 : 1.0
                        * keySize / (progress + 1);
                resize((int) (allocSize * l));
            }

            if (used[begin])
                continue;

            for (int i = 1; i < siblings.size(); i++)
                if (check[begin + siblings.get(i).code] != 0)
                    continue outer;

            break;
        }

        // -- Simple heuristics --
        // if the percentage of non-empty contents in check between the
        // index
        // 'next_check_pos' and 'check' is greater than some constant value
        // (e.g. 0.9),
        // new 'next_check_pos' index is written by 'check'.
        if (1.0 * nonzero_num / (pos - nextCheckPos + 1) >= 0.95)
            nextCheckPos = pos;

        used[begin] = true;
        size = (size > begin + siblings.get(siblings.size() - 1).code + 1) ? size
                : begin + siblings.get(siblings.size() - 1).code + 1;

        for (int i = 0; i < siblings.size(); i++)
            check[begin + siblings.get(i).code] = begin;

        for (int i = 0; i < siblings.size(); i++) {
            List<Node> new_siblings = new ArrayList<Node>();

            if (fetch(siblings.get(i), new_siblings) == 0) {
                base[begin + siblings.get(i).code] = (value != null) ? (-value[siblings
                        .get(i).left] - 1) : (-siblings.get(i).left - 1) + (-0.0001 * siblings.get(i).pos);

                if (value != null && (-value[siblings.get(i).left] - 1) >= 0) {
                    error_ = -2;
                    return 0;
                }

                progress++;
                // if (progress_func_) (*progress_func_) (progress,
                // keySize);
            } else {
                int h = insert(new_siblings);
                base[begin + siblings.get(i).code] = h;
            }
        }
        return begin;
    }

    public DoubleArrayTrie() {
        check = null;
        base = null;
        used = null;
        size = 0;
        allocSize = 0;
        // no_delete_ = false;
        error_ = 0;
    }

    // no deconstructor

    // set_result omitted
    // the search methods returns (the list of) the value(s) instead
    // of (the list of) the pair(s) of value(s) and length(s)

    // set_array omitted
    // array omitted

    void clear() {
        // if (! no_delete_)
        check = null;
        base = null;
        used = null;
        allocSize = 0;
        size = 0;
        // no_delete_ = false;
    }

    public int getUnitSize() {
        return UNIT_SIZE;
    }

    public int getSize() {
        return size;
    }

    public int getTotalSize() {
        return size * UNIT_SIZE;
    }

    public int getNonzeroSize() {
        int result = 0;
        for (int i = 0; i < size; i++)
            if (check[i] != 0)
                result++;
        return result;
    }

    public int build(List<ValueAttrDTO> _valueAttrDTOS) {
        return build(_valueAttrDTOS.stream().map(ValueAttrDTO::getValue).collect(Collectors.toList()), null, null, _valueAttrDTOS.size(), _valueAttrDTOS);
    }

    public int build(List<String> _key, int _length[], int _value[],
                     int _keySize, List<ValueAttrDTO> _valueAttrDTOS) {
        if (_keySize > _key.size() || _key == null)
            return 0;

        // progress_func_ = progress_func;
        key = _key;
        length = _length;
        keySize = _keySize;
        value = _value;
        valueAttrDTOs = _valueAttrDTOS;
        progress = 0;

        resize(65536 * 32);

        base[0] = 1;
        nextCheckPos = 0;

        Node root_node = new Node();
        root_node.left = 0;
        root_node.right = keySize;
        root_node.depth = 0;

        List<Node> siblings = new ArrayList<Node>();
        fetch(root_node, siblings);
        insert(siblings);

        // size += (1 << 8 * 2) + 1; // ???
        // if (size >= allocSize) resize (size);

        used = null;
        key = null;

        return error_;
    }

    public void open(String fileName) throws IOException {
        File file = new File(fileName);
        size = (int) file.length() / UNIT_SIZE;
        check = new double[size];
        base = new double[size];

        DataInputStream is = null;
        try {
            is = new DataInputStream(new BufferedInputStream(
                    new FileInputStream(file), BUF_SIZE));
            for (int i = 0; i < size; i++) {
                base[i] = is.readInt();
                check[i] = is.readInt();
            }
        } finally {
            if (is != null)
                is.close();
        }
    }

    public void save(String fileName) throws IOException {
        DataOutputStream out = null;
        try {
            out = new DataOutputStream(new BufferedOutputStream(
                    new FileOutputStream(fileName)));
            for (int i = 0; i < size; i++) {
                out.writeDouble(base[i]);
                out.writeDouble(check[i]);
            }
            out.close();
        } finally {
            if (out != null)
                out.close();
        }
    }

    public double exactMatchSearch(String key) {
        return exactMatchSearch(key, 0, 0, 0);
    }

    public double exactMatchSearch(String key, int pos, int len, int nodePos) {
        if (len <= 0)
            len = key.length();
        if (nodePos <= 0)
            nodePos = 0;

        double result = -1;

        char[] keyChars = key.toCharArray();

        double b = base[nodePos];
        double p;

        for (int i = pos; i < len; i++) {
            p = b + (double) (keyChars[i]) + 1;
            if (b == check[(int)p])
                b = base[(int)p];
            else
                return result;
        }

        p = b;
        double n = base[(int)p];
        if (b == check[(int)p] && n < 0) {
            result = -n - 1;
        }
        return result;
    }

    public List<Double> commonPrefixSearch(String key) {
        return commonPrefixSearch(key, 0, 0, 0);
    }

    public List<Double> commonPrefixSearch(String key, int pos, int len,
                                           int nodePos) {
        if (len <= 0)
            len = key.length();
        if (nodePos <= 0)
            nodePos = 0;

        List<Double> result = new ArrayList<Double>();

        char[] keyChars = key.toCharArray();

        double b = base[nodePos];
        double n;
        double p;

        for (int i = pos; i < len; i++) {
            p = b;
            n = base[(int)p];

            if (b == check[(int)p] && n < 0) {
                result.add(-n - 1);
            }

            p = b + (int) (keyChars[i]) + 1;
            if (b == check[(int)p])
                b = base[(int)p];
            else
                return result;
        }

        p = b;
        n = base[(int)p];

        if (b == check[(int)p] && n < 0) {
            result.add(-n - 1);
        }

        return result;
    }

    // debug
    public void dump() {
        for (int i = 0; i < size; i++) {
            if(base[i] != 0 &&  check[i] != 0) {
                System.err.println("i: " + i + " [" + base[i] + ", " + check[i]
                        + "]");
            }
        }
    }

    private static class ValueAttrDTO {
        String value;
        int pos;

        public ValueAttrDTO(String value, int pos) {
            this.value = value;
            this.pos = pos;
        }

        public void setValue(String value) {
            this.value = value;
        }

        public void setPos(int pos) {
            this.pos = pos;
        }

        public String getValue() {
            return value;
        }

        public int getPos() {
            return pos;
        }
    };
}

```
