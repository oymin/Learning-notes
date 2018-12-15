# Lambda 表达式

Lambda 表达式是一个匿名方法，将行为像数据一样进行传递

## Lambda 表达式的基本结构

```
(param1, param2, param3) -> {}
```

## Lambda 结构

- 一个 lambda 表达式可以有零个或多个参数

- 参数的类型既可以明确声明，也可以根据上下文来推断。例如：（int a）与（a）效果相同

- 所有参数需包含在原括号内，参数之间用逗号相隔，例例如：(a, b) 或 (int a, int b) 或 (String a, int b, ﬂoat c)

- 空圆括号代表参数集为空。例例如：() -> 42

- 当只有一个参数，且其类型可推导时，圆括号（）可省略。例如：a -> return a*a

-  Lambda 表达式的主体可包含零条或多条语句

- 如果 Lambda 表达式的主体只有一条语句，花括号{}可省略。匿名函数的返回类型与该主体表达式⼀致

- 如果 Lambda 表达式的主体包含一条以上语句，则表达式必须包含在花括号{}中（形成代码块）。匿名函数的返回类型与代码块的返回类型一致，若没有返回则为空

```
public class Test1 {
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8);

        // lambda 遍历集合
        list.forEach(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) {
                System.out.println(integer);
            }
        });

        list.forEach(integer -> { System.err.println(integer); });

        // method reference
        list.forEach(System.out::println);

    }
}
```
