## 朴素贝叶斯
    
    分类器。
    直接看代码，单元测试里面从数据层面介绍了朴素贝叶斯的原理。

```html
@Test
public void localBayesTest() throws Exception {
    try {
        final Classifier<String, String> bayes = new BayesClassifier<String, String>();

        bayes.learn("0", Stream.of(new String[]{"开心", "高兴", "快乐"}).collect(Collectors.toList()));
        bayes.learn("0", Stream.of(new String[]{"乐观", "舒心", "快乐"}).collect(Collectors.toList()));
        bayes.learn("0", Stream.of(new String[]{"惊喜", "高兴", "友谊"}).collect(Collectors.toList()));

        bayes.learn("1", Stream.of(new String[]{"汉奸", "吓人", "歹徒"}).collect(Collectors.toList()));
        bayes.learn("1", Stream.of(new String[]{"汉奸", "吓人", "独裁"}).collect(Collectors.toList()));

        String category = bayes.classify(Stream.of(new String[]{"开心", "惊喜", "歹徒"}).collect(Collectors.toList())).getCategory();
        System.out.println(category);

        // 共有11个词，其中 高兴、快乐、汉奸、吓人 各出现两次，友谊、惊喜、乐观、舒心、开心、歹徒、独裁、 各出现1词。
        // 两个分类

        // 开心在类别1中的概率为：（1 * 0.5 + 开心在所有类别中出现的次数1次 * (开心在类别1中出现的次数0 / 开心在所有类别中出现的次数1)） / (1 + 开心在所有类别中出现的次数1次) = （0.5 + 0） / 2 = 0.25
        // 惊喜在类别1中的概率为：（1 * 0.5 + 惊喜在所有类别中出现的次数1次 * (惊喜在类别1中出现的概率0 / 惊喜在所有类别中出现的次数1)） / (1 + 惊喜在所有类别中出现的次数1次) = （0.5 + 0） / 2 = 0.25
        // 歹徒在类别1中的概率为：（1 * 0.5 + 歹徒在所有类别中出现的次数1次 * (歹徒在类别1中出现的概率1 / 歹徒在所有类别中出现的次数1)） / (1 + 惊喜在所有类别中出现的次数1次) = （0.5 + 1） / 2 = 0.75
        // 0.25 * 0.25 * 0.75 = 0.046875
        // 最终验证的文章在类别1中的概率为 = （当前类别下有2篇文章 / 总共5篇验证集） * 0.046875 = 0.01875

        // 开心在类别0中的概率为：（1 * 0.5 + 开心在所有类别中出现的次数1次 * (开心在类别0中出现的次数1 / 开心在所有类别中出现的概率1)） / (1 + 开心在所有类别中出现的次数1次) = （0.5 + 1） / 2 = 0.75
        // 惊喜在类别0中的概率为：（1 * 0.5 + 惊喜在所有类别中出现的次数1次 * (惊喜在类别0中出现的概率1 / 惊喜在所有类别中出现的概率1)） / (1 + 惊喜在所有类别中出现的次数1次) = （0.5 + 1） / 2 = 0.75
        // 歹徒在类别0中的概率为：（1 * 0.5 + 歹徒在所有类别中出现的次数1次 * (歹徒在类别0中出现的概率0 / 歹徒在所有类别中出现的概率1)） / (1 + 惊喜在所有类别中出现的次数1次) = （0.5 + 1） / 2 = 0.25
        // 0.75 * 0.75 * 0.25 = 0.140625
        // 最终验证的文章在类别0中的概率为 = （当前类别下有3篇文章 / 总共5篇验证集） * 0.140625 = 0.084375

    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

```html
public interface IFeatureProbability<T, K> {

    public float featureProbability(T feature, K category);

}

```

```html
import java.util.*;

public abstract class Classifier<T, K> implements IFeatureProbability<T, K>, java.io.Serializable {

    private static final long serialVersionUID = 5504911666956811966L;

    private static final int INITIAL_CATEGORY_DICTIONARY_CAPACITY = 16;

    private static final int INITIAL_FEATURE_DICTIONARY_CAPACITY = 32;

    private int memoryCapacity = 1000;

    private Dictionary<K, Dictionary<T, Integer>> featureCountPerCategory;

    private Dictionary<T, Integer> totalFeatureCount;

    private Dictionary<K, Integer> totalCategoryCount;

    private Queue<Classification<T, K>> memoryQueue;

    public Classifier() {
        this.reset();
    }

    public void reset() {
        this.featureCountPerCategory = new Hashtable<K, Dictionary<T, Integer>>(Classifier.INITIAL_CATEGORY_DICTIONARY_CAPACITY);
        this.totalFeatureCount = new Hashtable<T, Integer>(Classifier.INITIAL_FEATURE_DICTIONARY_CAPACITY);
        this.totalCategoryCount = new Hashtable<K, Integer>(Classifier.INITIAL_CATEGORY_DICTIONARY_CAPACITY);
        this.memoryQueue = new LinkedList<Classification<T, K>>();
    }

    public Set<T> getFeatures() {
        return ((Hashtable<T, Integer>) this.totalFeatureCount).keySet();
    }

    public Set<K> getCategories() {
        return ((Hashtable<K, Integer>) this.totalCategoryCount).keySet();
    }

    public int getCategoriesTotal() {
        int toReturn = 0;
        for (Enumeration<Integer> e = this.totalCategoryCount.elements(); e.hasMoreElements(); ) {
            toReturn += e.nextElement();
        }
        return toReturn;
    }

    public int getMemoryCapacity() {
        return memoryCapacity;
    }

    public void setMemoryCapacity(int memoryCapacity) {
        for (int i = this.memoryCapacity; i > memoryCapacity; i--) {
            this.memoryQueue.poll();
        }
        this.memoryCapacity = memoryCapacity;
    }

    public void incrementFeature(T feature, K category) {
        Dictionary<T, Integer> features = this.featureCountPerCategory.get(category);
        if (features == null) {
            this.featureCountPerCategory.put(category, new Hashtable<T, Integer>(Classifier.INITIAL_FEATURE_DICTIONARY_CAPACITY));
            features = this.featureCountPerCategory.get(category);
        }
        Integer count = features.get(feature);
        if (count == null) {
            features.put(feature, 0);
            count = features.get(feature);
        }
        features.put(feature, ++count);

        Integer totalCount = this.totalFeatureCount.get(feature);
        if (totalCount == null) {
            this.totalFeatureCount.put(feature, 0);
            totalCount = this.totalFeatureCount.get(feature);
        }
        this.totalFeatureCount.put(feature, ++totalCount);
    }

    public void incrementCategory(K category) {
        Integer count = this.totalCategoryCount.get(category);
        if (count == null) {
            this.totalCategoryCount.put(category, 0);
            count = this.totalCategoryCount.get(category);
        }
        this.totalCategoryCount.put(category, ++count);
    }

    public void decrementFeature(T feature, K category) {
        Dictionary<T, Integer> features = this.featureCountPerCategory.get(category);
        if (features == null) {
            return;
        }
        Integer count = features.get(feature);
        if (count == null) {
            return;
        }
        if (count.intValue() == 1) {
            features.remove(feature);
            if (features.size() == 0) {
                this.featureCountPerCategory.remove(category);
            }
        } else {
            features.put(feature, --count);
        }

        Integer totalCount = this.totalFeatureCount.get(feature);
        if (totalCount == null) {
            return;
        }
        if (totalCount.intValue() == 1) {
            this.totalFeatureCount.remove(feature);
        } else {
            this.totalFeatureCount.put(feature, --totalCount);
        }
    }

    public void decrementCategory(K category) {
        Integer count = this.totalCategoryCount.get(category);
        if (count == null) {
            return;
        }
        if (count.intValue() == 1) {
            this.totalCategoryCount.remove(category);
        } else {
            this.totalCategoryCount.put(category, --count);
        }
    }

    public int getFeatureCount(T feature, K category) {
        Dictionary<T, Integer> features = this.featureCountPerCategory.get(category);
        if (features == null) return 0;
        Integer count = features.get(feature);
        return (count == null) ? 0 : count.intValue();
    }

    public int getFeatureCount(T feature) {
        Integer count = this.totalFeatureCount.get(feature);
        return (count == null) ? 0 : count.intValue();
    }

    public int getCategoryCount(K category) {
        Integer count = this.totalCategoryCount.get(category);
        return (count == null) ? 0 : count.intValue();
    }

    public float featureProbability(T feature, K category) {
        final float totalFeatureCount = this.getFeatureCount(feature);

        if (totalFeatureCount == 0) {
            return 0;
        } else {
            return this.getFeatureCount(feature, category) / (float) this.getFeatureCount(feature);
        }
    }

    public float featureWeighedAverage(T feature, K category) {
        return this.featureWeighedAverage(feature, category, null, 1.0f, 0.5f);
    }

    public float featureWeighedAverage(T feature, K category, IFeatureProbability<T, K> calculator) {
        return this.featureWeighedAverage(feature, category, calculator, 1.0f, 0.5f);
    }

    public float featureWeighedAverage(T feature, K category, IFeatureProbability<T, K> calculator, float weight) {
        return this.featureWeighedAverage(feature, category, calculator, weight, 0.5f);
    }

    public float featureWeighedAverage(T feature, K category, IFeatureProbability<T, K> calculator, float weight, float assumedProbability) {

        final float basicProbability = (calculator == null) ? this.featureProbability(feature, category) : calculator.featureProbability(feature, category);

        Integer totals = this.totalFeatureCount.get(feature);
        if (totals == null) totals = 0;
        return (weight * assumedProbability + totals * basicProbability) / (weight + totals);
    }

    public void learn(K category, Collection<T> features) {
        this.learn(new Classification<T, K>(features, category));
    }

    public void learn(Classification<T, K> classification) {

        for (T feature : classification.getFeatureset())
            this.incrementFeature(feature, classification.getCategory());
        this.incrementCategory(classification.getCategory());

        this.memoryQueue.offer(classification);
        if (this.memoryQueue.size() > this.memoryCapacity) {
            Classification<T, K> toForget = this.memoryQueue.remove();

            for (T feature : toForget.getFeatureset())
                this.decrementFeature(feature, toForget.getCategory());
            this.decrementCategory(toForget.getCategory());
        }
    }

    public abstract Classification<T, K> classify(Collection<T> features);

}

```

```html
import java.io.Serializable;
import java.util.Collection;

public class Classification<T, K> implements Serializable {

    private static final long serialVersionUID = -1210981535415341283L;

    private Collection<T> featureset;

    private K category;

    private float probability;

    public Classification(Collection<T> featureset, K category) {
        this(featureset, category, 1.0f);
    }

    public Classification(Collection<T> featureset, K category, float probability) {
        this.featureset = featureset;
        this.category = category;
        this.probability = probability;
    }

    public Collection<T> getFeatureset() {
        return featureset;
    }

    public float getProbability() {
        return this.probability;
    }

    public K getCategory() {
        return category;
    }

    @Override
    public String toString() {
        return "Classification [category=" + this.category + ", probability=" + this.probability + ", featureset=" + this.featureset + "]";
    }

}

```

```html
import java.util.Collection;
import java.util.Comparator;
import java.util.SortedSet;
import java.util.TreeSet;

public class BayesClassifier<T, K> extends Classifier<T, K> {

    private float featuresProbabilityProduct(Collection<T> features, K category) {
        float product = 1.0f;
        for (T feature : features)
            product *= this.featureWeighedAverage(feature, category);
        return product;
    }

    private float categoryProbability(Collection<T> features, K category) {
        return ((float) this.getCategoryCount(category) / (float) this.getCategoriesTotal()) * featuresProbabilityProduct(features, category);
    }

    private SortedSet<Classification<T, K>> categoryProbabilities(Collection<T> features) {

        SortedSet<Classification<T, K>> probabilities = new TreeSet<Classification<T, K>>(new Comparator<Classification<T, K>>() {
            public int compare(Classification<T, K> o1, Classification<T, K> o2) {
                int toReturn = Float.compare(o1.getProbability(), o2.getProbability());
                if ((toReturn == 0) && !o1.getCategory().equals(o2.getCategory())) toReturn = -1;
                return toReturn;
            }
        });

        for (K category : this.getCategories())
            probabilities.add(new Classification<T, K>(features, category, this.categoryProbability(features, category)));
        return probabilities;
    }

    @Override
    public Classification<T, K> classify(Collection<T> features) {
        SortedSet<Classification<T, K>> probabilites = this.categoryProbabilities(features);

        if (probabilites.size() > 0) {
            return probabilites.last();
        }
        return null;
    }

    public Collection<Classification<T, K>> classifyDetailed(Collection<T> features) {
        return this.categoryProbabilities(features);
    }

}

```
