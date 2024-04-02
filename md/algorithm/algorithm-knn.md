## KNN
    
    分类器。
    直接看代码，未封装，可以直接从代码层面Debug。

```html
package com.pap.base.util.knn;

import java.util.Arrays;

/**
 *
 */
public class KNNTest {


    private static double[][] DATA={
            { 5.1, 3.5, 1.4, 0.2, 0 },
            { 4.9, 3.0, 1.4, 0.2, 0 }, { 4.7, 3.2, 1.3, 0.2, 0 },
            { 4.6, 3.1, 1.5, 0.2, 0 }, { 5.0, 3.6, 1.4, 0.2, 0 },
            { 7.0, 3.2, 4.7, 1.4, 1 }, { 6.4, 3.2, 4.5, 1.5, 1 },
            //{ 6.9, 3.1, 4.9, 1.5, 1 }, //测试数据
            { 5.5, 2.3, 4.0, 1.3, 1 },
            { 6.5, 2.8, 4.6, 1.5, 1 }, { 5.7, 2.8, 4.5, 1.3, 1 },
            { 6.5, 3.0, 5.8, 2.2, 2 }, { 7.6, 3.0, 6.6, 2.1, 2 },
            { 4.9, 2.5, 4.5, 1.7, 2 }, { 7.3, 2.9, 6.3, 1.8, 2 },
            { 6.7, 2.5, 5.8, 1.8, 2 }, { 6.9, 3.1, 5.1, 2.3, 2 }
    };
    private static int K = 6;
    private static int CLASSFIY=3;

    // @Test
    public void test() {
        // 待求解数组
        double distince[] = {6.9, 3.1, 4.9, 1.5, 1};

        KNNTest knn = new KNNTest();
        //求出求解的分类与二维数组间元素的临近距离
        double[] questionDistinces = new double[DATA.length];
        for(int i=0;i<DATA.length;i++){
            double[] item = DATA[i];
            questionDistinces[i] = knn.distince(item, distince);
        }
        System.out.println("临近距离集合："+Arrays.toString(questionDistinces));
        int nearest[] = knn.paraseKDistince(questionDistinces, K);
        System.out.println("K 个最临近距离下标集合："+Arrays.toString(nearest));

        System.out.println("{ 6.9, 3.1, 4.9, 1.5, x }的 x 位置求解为："+knn.vote(nearest));
    }

    //计算临近距离[除开求解分类]
    public double distince(double []paraFirstData,double []paraSecondData){
        double tempDistince = 0;
        if((paraFirstData!=null && paraSecondData!=null) && paraFirstData.length==paraSecondData.length){
            for(int i=0;i<paraFirstData.length-1;i++){
                tempDistince += Math.abs(paraFirstData[i] - paraSecondData[i]);
            }
        }else{
            System.out.println("firstData 与 secondData 数据结构不一致");
        }
        return tempDistince;
    }

    //对临近距离排序,从小到大[这里采用冒泡排序]
    public double[] sortDistinceArray(double []paraDistinceArray){
        if(paraDistinceArray!=null && paraDistinceArray.length>0){
            for(int i=0;i<paraDistinceArray.length;i++){
                for(int j=i+1;j<paraDistinceArray.length;j++){
                    if(paraDistinceArray[i]>paraDistinceArray[j]){
                        double temp = paraDistinceArray[i];
                        paraDistinceArray[i] = paraDistinceArray[j];
                        paraDistinceArray[j] = temp;
                    }
                }
            }
        }
        return paraDistinceArray;
    }

    //获取临近值数组中，从近到远获取k个值为新数组
    public double[] paraseKDistince(double[] sortedDistinceArray,String sortTypeStr,int k){
        double[] kDistince = new double[k];
        if("asc".equals(sortTypeStr)){
            for(int i=0;i<k;i++){
                kDistince[i] = sortedDistinceArray[i];
            }
        }
        if("des".equals(sortTypeStr)){
            for(int i=0;i<k;i++){
                kDistince[i] = sortedDistinceArray[sortedDistinceArray.length-i-1];
            }
        }

        return kDistince;
    }

    //获取临近距离中的K的距离的下标数组
    public int[] paraseKDistince(double[] distinceArray,int k){
        double[] tempDistince = new double[k+2];
        int[] tempNearest = new int[k+2];
        //初始化两个数组
        tempDistince[0] = Double.MIN_VALUE;
        for(int i=1;i<k+2;i++){
            tempDistince[i] = Double.MAX_VALUE;
            tempNearest[i] = -1;
        }
        //准备筛选临近距离
        for(int i=0;i<distinceArray.length;i++){
            for(int j=k;j>=0;j--){
                if(distinceArray[i]<tempDistince[j]){
                    tempDistince[j+1] = tempDistince[j];
                    tempNearest[j+1] = tempNearest[j];
                }else{
                    tempDistince[j+1] = distinceArray[i];
                    tempNearest[j+1] = i;
                    break;
                }
            }
        }
        int[] returnNearests = new int[k];
        for (int i = 0; i < k; i++) {
            returnNearests[i] = tempNearest[i + 1];
        }
        return returnNearests;
    }

    //得到角标对应的分类
    public int getClasssify(int index){
        return (int)DATA[index][4];
    }

    //对分类进行投票得到结果[得到分类票数最多的分类]
    public int vote(int[] nearestIndex){
        int[] votes = new int[CLASSFIY];
        for(int i=0;i<nearestIndex.length;i++){
            votes[getClasssify(nearestIndex[i])]++;
        }
        System.out.println("分类投票数集合："+ Arrays.toString(votes));
        int tempMajority = -1;
        int tempMaximalVotes = -1;
        for (int i = 0; i < votes.length; i++) {
            if (votes[i] > tempMaximalVotes) {
                tempMaximalVotes = votes[i];
                tempMajority = i;
            }
        }
        System.out.println("投票数最高："+tempMaximalVotes+",分类是："+tempMajority);
        return tempMajority;
    }

}

```

## 参考

1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/

