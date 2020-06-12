## 一、前提

什么是流？在用JDK1.8的你肯定知道我就不多bb了。本篇主要介绍一下内容：

- 流的创建
- 流的操作
- 流的结束

听上去是不是很弟弟，别急，看完这篇，让你用流信手拈来。

#  二、流基本介绍

## 什么是流

- JDK8提供的对集合数据进行处理的一种方式
- 流中的元素是特定类型的对象，形成一个队列。 Java中的Stream**并不会存储元素**，而是按需计算
- 数据源流的来源。 可以是集合，数组，I/O channel， 产生器generator 等
- 有聚合操作 类似SQL语句一样的操作， 比如filter, map, reduce, find, match, sorted等

## 流的特点

- 一个流只能使用一次
- Pipelining: 中间操作都会返回流对象本身。 这样多个操作可以串联成一个管道， 如同流式风格（fluent style），这样做可以对操作进行优化， 比如延迟执行(laziness)和短路( short-circuiting)
- 内部迭代： 以前对集合遍历都是通过Iterator或者For-Each的方式, 显式的在集合外部进行迭代， 这叫做外部迭代， Stream提供了内部迭代的方式， 通过访问者模式(Visitor)实现

## 串行流和并行流

顾名思义，串行流是一个个的执行，相当于单线程，并行流是并行执行，相当于多线程。

- 创建串行流：stream()
- 创建并行流：parallelStream()

根据实际测试，在数据量比较大的时候，并行流的速度快串行流1.5到3倍，看实际需求有所不同。

# 三、正文

好，前面都是对流一点都不知道的人说的废话，下面进入正文

## 1.各个容器类实现Stream的方式

本节覆盖了日常能用到的容器、数组、IO创建流的方式：

```java
    @Test
    public void getStreamTest()
    {
        /**
         * {@code Collection}在JDK1.8版本实现了Stream()方法和parallelStream()方法，
         * 所有实现Collection的容器类通过调用stream()方法或者 parallelStream()
         */
        // list
        List<String> list = new ArrayList<>();
        list.add("a");
        list.add("b");
        list.add("c");
        list.stream();

        // Set
        Set<Integer> set = new HashSet<Integer>();
        set.add(1);
        set.add(2);
        set.add(3);
        set.stream();

        // ArrayList
        ArrayList<Integer> arrList = new ArrayList<>();
        arrList.add(1);
        arrList.add(2);
        arrList.stream();

        // Vector
        Vector<Integer> vector = new Vector<>();
        vector.add(1);
        vector.add(2);
        vector.stream();

        // Queue
        Queue<Integer> queue = new ArrayBlockingQueue<>(12);
        queue.add(1);
        queue.add(2);
        queue.stream();

        // Stack
        Stack<Integer> stack = new Stack<>();
        stack.add(1);
        stack.add(2);
        stack.stream();

        // Map 通过调用map的entrySet()方法，获取Set再调用stream()
        Map<Integer, String> map = new HashMap<>();
        map.put(1, "a");
        map.put(2, "b");
        map.entrySet().stream();

        /**
         * {@code Arrays}在JDK1.8中加入了stream系列方法， 数组可以调用Arrays.stream(T []
         * array)实现流，同时还有参数不同的stream函数用来截取数组
         */
        // 数组
        String[] arr = {"a", "b", "c"};
        Arrays.stream(arr);

        // 值，直接将值转变成流对象
        Stream.of("a", "b", "c");

        // 从IO获取流
        try (
            Stream<String> lines = Files.lines(Paths.get("文件路径名"), Charset.defaultCharset())
        )
        {
            // 可对lines做一些操作
        }
        catch (IOException e)
        {}
        // 拓展：无限流的创建
        // 迭代
        Stream.iterate(0, x -> x + 1).limit(10).forEach(System.out::println);
        // 随机生成
        Stream.generate(() -> Math.random());
    }
```

## 2.流的常用方法

```java
    @Test
    public void streamMethodTest()
    {
        /**
         * @decription 过滤器filter
         * @param 一个返回true或者false的lambda表达式
         * @return 筛选出返回结果为true的元素
         */
        Stream.of(1, 2, 3).filter(a ->
        {
            return a.equals(1);
        }).count();

        /**
         * @description 去重
         * @return the new stream
         */
        Stream.of(1, 2, 2, 3).distinct();

        /**
         * @decription 截取前n个元素
         * @return the new stream
         */
        Stream.of(1, 2, 3).limit(2);

        /**
         * @decription 跳过前n个元素
         * @return the new stream
         */

        Stream.of(1, 2, 3).skip(2);

        /**
         * @decription map() 一对一映射 输入一个元素，按照给定的规则处理该元素
         */
        Stream.of("a", "b", "c").map(a ->
        {
            return a.toUpperCase();
        }).map(String::toLowerCase);

        /**
         * @description flatMap() 将流扁平化，说人话就是将流中最基层的元素抽出来为单独的流，eg:
         *              List<List<Integer>>
         *              类型的，会把最底层的Integer抽取出来，和map()不同的是map()将List<Integer>视为一个流进行处理
         */
        // 写法1
        Stream.of(Arrays.asList(1, 2), Arrays.asList(3, 4), Arrays.asList(5, 6, 7)).flatMap(Collection::stream).map(a ->
        {
            // 此时 a就只是Integer了
            return a.byteValue();
        });
        // 写法2
        Stream.of(Arrays.asList(1, 2), Arrays.asList(3, 4), Arrays.asList(5, 6, 7)).flatMap(l ->
        {
            return l.stream();
        }).map(a ->
        {
            return a.byteValue();
        });

        /**
         * @description 是否匹配任意元素,有任意一个符合为true
         */
        Stream.of(1, 2, 3).anyMatch(a ->
        {
            return a.equals(1);
        });

        /**
         * @description 是否匹配所有元素
         */
        Stream.of(1, 2, 3).allMatch(a ->
        {
            return a.equals(1);
        });

        /**
         * @description 是否所有元素都不匹配
         */
        Stream.of(1, 2, 3).noneMatch(a ->
        {
            return a.equals(1);
        });

        /**
         * @description 从流中返回任意元素，返回结果放在Optional对象中(实测怎么感觉就是findFirst())
         */
        Stream.of(1, 2, 3).findAny();

        /**
         * @description 返回第一个元素，返回结果放在Optional对象中
         */
        Optional<Integer> res = Stream.of(Arrays.asList(1, 2), Arrays.asList(3, 4)).flatMap(Collection::stream)
                .findFirst();
        res.ifPresent(System.out::print);

        /**
         * @description reduce 自定义lambda表达式求和 以
         *              identity为初始值，对流中的所有元素执行lambda表达式求和
         */
        Stream.of(1, 2, 3).reduce(0, (a, b) ->
        {
            return a + b;
        });

    }
```

## 3.普通流到数值流的转换

```java
    @Test
    public void test3()
    {
        Stream.of(1, 2, 3).mapToDouble(Double::valueOf);
        Stream.of(1, 2, 3).mapToLong(Long::valueOf);
    }
```

## 4.Stream收集器—优雅的结束stream

```java
    @Test
    public void endStreamTest()
    {
        // 返回Map
        Stream.of("a", "b", "c").collect(Collectors.toMap(s -> s, s -> s));
        // 返回 List
        Stream.of("a", "b", "c").collect(Collectors.toList());
        // 返回Set
        Stream.of("a", "b", "c").collect(Collectors.toSet());

        // ========================stream收集器 =============================//
        /**
         * @description:求数量
         */
        // 写法1
        Stream.of(1, 2, 3).count();
        // 写法2
        Stream.of(1, 2, 3).collect(Collectors.counting());

        /**
         * @description:求最值,最大值最小值都类似，比较方法有comparingDouble、comparingInt、comparingLong、甚至自定义
         */
        // 写法1
        Optional<Integer> res1 = Stream.of(1, 2, 3, 3).max((x, y) ->
        {
            return (x < y) ? -1 : ((x == y) ? 0 : 1);
        });
        System.out.println(res1.get());
        // 写法2 依据数值流比较大小不需要参数
        OptionalInt res2 = Stream.of(1, 2, 3, 3).mapToInt(a ->
        {
            return a;
        }).max();
        System.out.println(res2.getAsInt());
        // 写法3 借助已有的Collectors.maxBy方法，没啥用，知道能这样写就好
        Optional<Integer> res3 = Stream.of(1, 2, 3, 3).collect(Collectors.maxBy(Comparator.comparingInt(a ->
        {
            return a;
        })));
        System.out.println(res3.get());

        /**
         * @description:求和、求平均值 方法也很多，类似求最值
         */

        // ========================stream连接字符串 =============================//
        /**
         * @description:连接字符串，默认中间无连接符，可以在joining方法中加入连接符 joining("_")
         */
        Stream.of("a", "b").collect(Collectors.joining());
        Stream.of("a", "b").reduce("", (a, b) ->
        {
            return a + b;
        });

        /**
         * @description:自定义规约操作
         * @param:1.初始值
         * @param:2.对流中的数据进行处理
         * @param:3.对处理后的元素进行自定义规约操作
         */
        Stream.of("a", "b", "c").collect(Collectors.reducing("begin:", a ->
        {
            return a;
        }, (a, b) ->
        {
            return a + b;
        }));

        // ========================stream连接字符串 =============================//
        /**
         * @description:将元素按指定规则分组
         */
        Map<String, List<String>> group1 = Stream.of("a", "b", "c").collect(Collectors.groupingBy(a ->
        {
            if ("a".equals(a))
            {
                return "isA";
            }
            else if ("b".equals(a))
            {
                return "isB";

            }
            else
            {
                return "isOthers";
            }
        }));

        // 可以在collect方法中添加参数，如再次分组或者统计每个分组的数量
        Map<String, Long> group2 = Stream.of("a", "b", "c").collect(Collectors.groupingBy((a) ->
        {
            if ("a".equals(a))
            {
                return "isA";
            }
            else if ("b".equals(a))
            {
                return "isB";

            }
            else
            {
                return "isOthers";
            }
        }, Collectors.counting()));
    }
```



# 结尾

[Java8StreamAPI.java](https://github.com/ChengYongchao/kevinProject/blob/master/src/cyc/java/stream/Java8StreamAPI.java)

看到这么多代码，有人可能要问号警告了，相信我，按代码敲一遍，从此流操作就是so easy！