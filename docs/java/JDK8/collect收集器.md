## collect 收集器

`Collector` 接口作为 collect 方法的参数

## Collector 接口

Collector 是一个接口，它是一个可变的汇聚操作，将输入元素累计到一个可变的结果容器中；它会在所有元素都处理完毕后，将积累的结果转换为一个最终的表示（这是一个可选操作），它支持串行与并行两种方式执行。

为了确保串行与并行操作结果的等价性，Collector 函数需要满足两个条件：identity（同一性）与associativty（结核性）。

> **可变的汇聚操作包括：**

- 将元素累积到集合 `Collection` 当中；

- 使用 `StringBuilder` 拼接字符串；

- 计算元素的汇总信息，如 sum、min、max 或 average；

- 计算“数据透视表”摘要，如“卖方最大价值交易”等;

## Collectors

**Collectors** 类提供了许多常见的可变汇聚的实现，Collectors 本身实际上是一个工厂

> **Collectors 指定4个函数，这些函数一起将条目累积到可变结果容器中，并可选地执行对结果进行最后的转换。它们是:**

- **supplier :** 创建一个新的结果容器

- **accumulator :** 将新的数据元素合并到结果容器中

- **combiner :** 将两个结果容器组合成一个

- **finisher :** 对容器执行可选的最终转换
