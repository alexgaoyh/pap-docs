## 概述

&ensp;&ensp;《作业调度问题-遗传算法》本文提供一种采用遗传算法去解决作业调度的JAVA代码实现。

## 背景

&ensp;&ensp;假设目前有指定数量的不同作业，每个作业需要在不同的机器上执行不同的时间，那么在限制机器数的情况下，如何获得一个执行结果是本文代码编写的背景。

## 数据实例

&ensp;&ensp; 限制在3个作业，3台机器的情况下，同时约定不同作业下不同任务的执行机器和执行时长

| 作业    | 任务                                                                         | 
|-------|----------------------------------------------------------------------------|
| JOB-0 | 分别需要在机器 0 上执行 3 个单位的时间，在机器 1 上执行 2 个单位的时间，在机器 2 上执行 2 个单位的时间                                                               |
| JOB-1 | 分别需要在机器 0 上执行 2 个单位的时间，在机器 2 上执行 1 个单位的时间，在机器 1 上执行 4 个单位的时间。                                                                      | 
| JOB-2 | 分别需要在机器 1 上执行 4 个单位的时间，在机器 2 上执行 3 个单位的时间。                                                          | 


## 执行结果

&ensp;&ensp;总共需要11个单位的时间，可以在限制条件下执行完所有作业，并且不同作业的执行明细如下所示。

&ensp;&ensp;通过如下‘作业执行明细’可以看到，针对 machine1 这台机器，需要按照 job2.process0 -> job0.process1 -> job1.process2 的步骤进行执行。

| 作业执行明细                                                   |
|----------------------------------------------------------|
| job: 0, process: 0, machine: 0, startTime: 2, endTime: 5 |
| job: 0, process: 1, machine: 1, startTime: 5, endTime: 7 |
| job: 0, process: 2, machine: 2, startTime: 7, endTime: 9 |
| job: 1, process: 0, machine: 0, startTime: 0, endTime: 2 |
| job: 1, process: 1, machine: 2, startTime: 2, endTime: 3 |
| job: 1, process: 2, machine: 1, startTime: 7, endTime: 11 |
| job: 2, process: 0, machine: 1, startTime: 0, endTime: 4 |
| job: 2, process: 1, machine: 2, startTime: 4, endTime: 7 |


## 代码

```java
public class GeneticAlgorithm {
    private final int populationNumber = 60;
    private final double crossProbability = 0.95;
    private final double mutationProbability = 0.05;
    private int jobNumber;
    private int machineNumber;
    private int processNumber;
    private int chromosomeSize;
    private int[][] machineMatrix = new int[1024][1024];
    private int[][] timeMatrix = new int[1024][1024];
    private int[][] processMatrix = new int[1024][1024];
    private Set<Gene> geneSet = new HashSet<>();
    private Random random = new Random();

    public GeneticAlgorithm(int jobNumber, int machineNumber) {
        this.jobNumber = jobNumber;
        this.machineNumber = machineNumber;
        for (int[] matrix : this.machineMatrix) Arrays.fill(matrix, -1);
        for (int[] process : this.processMatrix) Arrays.fill(process, -1);
    }

    private List<Integer> makeList(int n) {
        List<Integer> result = new ArrayList<>();
        for (int i = 0; i < n; i++) result.add(i);
        return result;
    }

    private Integer[] filterArray(Integer[] arr, int filterVal) {
        List<Integer> result = new ArrayList<>();
        for (int i = 0; i < arr.length; i++) {
            if (arr[i] != filterVal) {
                result.add(arr[i]);
            }
        }
        return result.toArray(new Integer[0]);
    }

    public void initialPopulation() {
        for (int i = 0; i < populationNumber; i++) {
            Gene g = new Gene();
            int size = jobNumber * machineNumber;
            List<Integer> indexList = makeList(size);
            Integer[] chromosome = new Integer[size];
            Arrays.fill(chromosome, -1);
            for (int j = 0; j < jobNumber; j++) {
                for (int k = 0; k < machineNumber; k++) {
                    int index = random.nextInt(indexList.size());
                    int val = indexList.remove(index);
                    if (processMatrix[j][k] != -1) {
                        chromosome[val] = j;
                    }
                }
            }
            g.chromosome = filterArray(chromosome, -1);
            g.fitness = calculateFitness(g).fulfillTime;
            geneSet.add(g);
        }
    }

    public List<Integer> subArray(Integer[] arr, int start, int end) {
        List<Integer> list = new ArrayList<>();
        for (int i = start; i < end; i++) list.add(arr[i]);
        return list;
    }

    public Result calculateFitness(Gene g) {
        Result result = new Result();
        for (int i = 0; i < g.chromosome.length; i++) {
            int jobId = g.chromosome[i];
            int processId = result.processIds[jobId];
            int machineId = machineMatrix[jobId][processId];
            int time = timeMatrix[jobId][processId];
            result.processIds[jobId] += 1;
            result.startTime[jobId][processId] = processId == 0 ? result.machineWorkTime[machineId] :
                    Math.max(result.endTime[jobId][processId - 1], result.machineWorkTime[machineId]);
            result.machineWorkTime[machineId] = result.startTime[jobId][processId] + time;
            result.endTime[jobId][processId] = result.machineWorkTime[machineId];
            result.fulfillTime = Math.max(result.fulfillTime, result.machineWorkTime[machineId]);

        }
        return result;
    }

    private Gene crossGene(Gene g1, Gene g2) {
        List<Integer> indexList = makeList(chromosomeSize);
        int p1 = indexList.remove(random.nextInt(indexList.size()));
        int p2 = indexList.remove(random.nextInt(indexList.size()));

        int start = Math.min(p1, p2);
        int end = Math.max(p1, p2);

        List<Integer> proto = subArray(g1.chromosome, start, end + 1);
        List<Integer> t = new ArrayList<>();
        for (Integer c : g2.chromosome) t.add(c);
        for (Integer val : proto) {
            for (int i = 0; i < t.size(); i++) {
                if (val.equals(t.get(i))) {
                    t.remove(i);
                    break;
                }
            }
        }

        Gene child = new Gene();
        proto.addAll(t.subList(start, t.size()));
        List<Integer> temp = t.subList(0, start);
        temp.addAll(proto);
        child.chromosome = temp.toArray(new Integer[0]);
        child.fitness = (double) calculateFitness(child).fulfillTime;
        return child;
    }

    public Gene mutationGene(Gene gene, int n) {
        List<Integer> indexList = makeList(chromosomeSize);
        for (int i = 0; i < n; i++) {
            int a = indexList.remove(random.nextInt(indexList.size()));
            int b = indexList.remove(random.nextInt(indexList.size()));
            int t = gene.chromosome[a];
            gene.chromosome[a] = gene.chromosome[b];
            gene.chromosome[b] = t;
        }
        gene.fitness = calculateFitness(gene).fulfillTime;
        return gene;
    }
    
    public Gene selectGene(int n) {
        List<Integer> indexList = makeList(geneSet.size());
        Map<Integer, Boolean> map = new HashMap<>();
        for (int i = 0; i < n; i++) {
            map.put(indexList.remove(random.nextInt(indexList.size())), true);
        }
        Gene bestGene = new Gene(0xfffff);
        int i = 0;
        for (Gene gene : geneSet) {
            if (map.containsKey(i)) {
                if (bestGene.fitness > gene.fitness) {
                    bestGene = gene;
                }
            }
            i++;
        }
        return bestGene;
    }

    public Result run(List<List<Integer[]>> job) {
        int jobSize = job.size();

        for (int i = 0; i < jobSize; i++) {
            chromosomeSize += job.get(i).size();
            processNumber = Math.max(processNumber, job.get(i).size());
            for (int j = 0; j < job.get(i).size(); j++) {
                machineMatrix[i][j] = job.get(i).get(j)[0];
                timeMatrix[i][j] = job.get(i).get(j)[1];

            }
        }

        for (int i = 0; i < jobSize; i++) {
            for (int j = 0; j < processNumber; j++) {
                if (machineMatrix[i][j] != -1) {
                    processMatrix[i][machineMatrix[i][j]] = j;
                }
            }
        }
        initialPopulation();
        for (int i = 0; i < populationNumber; i++) {
            double p = (double) random.nextInt(100) / 100.0;
            if (p < mutationProbability) {
                int index = random.nextInt(geneSet.size());
                int k = 0;
                for (Gene gene : geneSet) {
                    if (k == index) {
                        mutationGene(gene);
                        break;
                    }
                    k++;
                }
            } else {
                Gene g1 = selectGene(), g2 = selectGene();
                Gene child1 = crossGene(g1, g2), child2 = crossGene(g2, g1);
                geneSet.add(child1);
                geneSet.add(child2);
            }
        }
        Gene bestGene = new Gene(0xffffff);
        for (Gene gene : geneSet) {
            if (bestGene.fitness > gene.fitness) {
                bestGene = gene;
            }
        }
        return calculateFitness(bestGene);
    }

    public Gene selectGene() {
        return selectGene(3);
    }

    public Gene mutationGene(Gene gene) {
        return mutationGene(gene, 2);
    }

    static public void main(String[] args) {
        List<List<Integer[]>> job = Arrays.asList(
                Arrays.asList(new Integer[]{0, 3}, new Integer[]{1, 2}, new Integer[]{2, 2}),
                Arrays.asList(new Integer[]{0, 2}, new Integer[]{2, 1}, new Integer[]{1, 4}),
                Arrays.asList(new Integer[]{1, 4}, new Integer[]{2, 3})
        );
        int n = 3, m = 3;
        GeneticAlgorithm ga = new GeneticAlgorithm(n, m);
        Result result = ga.run(job);
        int processNumber = ga.processNumber;

        int[][] machineMatrix = ga.machineMatrix;
        System.out.println(result.fulfillTime);

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < processNumber; j++) {
                if (machineMatrix[i][j] != -1) {
                    System.out.println(String.format("job: %d, process: %d, machine: %d, startTime: %d, endTime: %d",
                            i, j, machineMatrix[i][j], result.startTime[i][j], result.endTime[i][j]));
                }
            }
        }
    }
}
```

## 参考

1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/pap-base/blob/v1/src/test/java/com/pap/base/jsp/GeneticAlgorithm.java
