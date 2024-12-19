# QuickStart-JMH

JMH（Java Microbenchmark Harness）是用于编写Java微基准测试的一套工具API。以下是对它的概述：

### 1. 用途

主要用于精准地测量Java代码片段的性能，帮助开发者确定哪段代码执行效率更高、资源消耗更合理等，比如对比不同算法实现或者相似功能的不同代码写法在时间、内存使用等方面的性能差异。

### 2. 特点

- **精准可靠**：它在进行性能测试时，会尽力排除诸如JVM预热、垃圾回收等外部因素对测试结果的干扰，尽可能给出准确反映代码本身性能表现的结果。例如，它能控制测试的迭代次数、预热轮次等，让测试环境相对稳定且可重复。
- **易于使用**：通过简单的注解和规范的代码结构就能搭建起有效的基准测试用例。像使用`@Benchmark`注解标记要测试的方法，用`@Warmup`注解设置预热相关参数，`@Measurement`注解来配置正式测量的轮次、时间等参数，开发者可以很便捷地按照自己的需求去定制化测试。

### 3. 运行机制

- **编译阶段**：JMH提供的注解等信息会被相关的编译插件处理，为后续的性能测试做好准备工作，比如生成合适的字节码结构等。
- **运行阶段**：它会按照预设的配置，先执行预热阶段，让JVM对相关代码进行优化加载等操作，之后进入正式的测量阶段，收集诸如方法执行时间、吞吐量等性能指标数据，并最终呈现出测试报告，方便开发者查看和分析。

### 4. 应用场景

- **算法优化对比**：当有多种算法实现同一个功能，比如排序算法有冒泡排序、快速排序等不同实现时，可以用JMH准确地测试出它们在不同数据规模下的性能表现，进而选择更优的算法应用到实际项目中。
- **代码优化验证**：对一些业务逻辑代码进行重构或者优化后，使用JMH来验证优化后的代码是否在性能上确实有所提升，像优化了某个频繁调用的方法内的循环逻辑，就可以通过JMH测试看看执行时间是否缩短了。

总之，JMH为Java开发者在进行性能相关的分析和优化时提供了一个很实用、可靠的测试框架。

## 示例、测试各种序列化反序列化工具的速度

要测试序列化速度，就要关注序列化的耗时，所以这里关注的是每次序列化和反序列的耗时。

创建一个 User 类：

```java
public class User implements Cloneable {

    private String name;
    private int age;
    private String address;
    private String phone;
    private String email;
    private String qq;
    private String wechat;
    private String weibo;
    private String github;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public String getPhone() {
        return phone;
    }

    public void setPhone(String phone) {
        this.phone = phone;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public String getQq() {
        return qq;
    }

    public void setQq(String qq) {
        this.qq = qq;
    }

    public String getWechat() {
        return wechat;
    }

    public void setWechat(String wechat) {
        this.wechat = wechat;
    }

    public String getWeibo() {
        return weibo;
    }

    public void setWeibo(String weibo) {
        this.weibo = weibo;
    }

    public String getGithub() {
        return github;
    }

    public void setGithub(String github) {
        this.github = github;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```

创建 MapStruct 的 Mapper

```java
import org.mapstruct.Mapper;
import org.mapstruct.factory.Mappers;

/**
 * @author WuQinglong
 * @since 2023/11/7 9:49 AM
 */
@Mapper
public interface UserMapper {

    UserMapper INSTANCE = Mappers.getMapper(UserMapper.class);

    User copy(User user);

}
```

创建基准测试类

```java
import cn.hutool.core.date.DatePattern;
import cn.hutool.json.JSONUtil;
import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.google.gson.Gson;
import org.openjdk.jmh.annotations.Benchmark;
import org.openjdk.jmh.annotations.Mode;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;
import org.openjdk.jmh.runner.options.TimeValue;
import org.springframework.beans.BeanUtils;

import java.text.SimpleDateFormat;
import java.util.concurrent.TimeUnit;

/**
 * @author WuQinglong
 * @date 2024/12/5 11:38
 */
public class JsonMain {

    private static final ObjectMapper objectMapper = new ObjectMapper()
        .disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES)
        .setSerializationInclusion(JsonInclude.Include.NON_NULL)
        .setDateFormat(new SimpleDateFormat(DatePattern.NORM_DATETIME_PATTERN));

    private static final Gson gson = new Gson();

    public static void main(String[] args) throws Exception {
        Options opt = new OptionsBuilder()
            .include(JsonMain.class.getSimpleName())
            .mode(Mode.AverageTime) // 比较执行的平均耗时
            .warmupIterations(5) // 预热执行 5 次（默认值）
            .warmupTime(TimeValue.seconds(10)) // 每次预热 10 秒（默认值）
            .measurementIterations(5) // 执行 5 次基准测试（默认值）
            .measurementTime(TimeValue.seconds(10)) // 每次基准测试执行 10 秒（默认值）
            .forks(1) // fork 1个进程进行测试，默认 5个
            .timeUnit(TimeUnit.MICROSECONDS) // 结果值的单位，默认是秒
            .build();
        new Runner(opt).run();
    }

    @Benchmark
    public Object hutool() {
        User User = create();
        String s = JSONUtil.toJsonStr(User);
        return JSONUtil.toBean(s, User.class);
    }

    @Benchmark
    public Object beanUtil() throws Exception {
        User User = create();
        User ret = new User();
        BeanUtils.copyProperties(User, ret);
        return ret;
    }

    @Benchmark
    public Object jackson() throws Exception {
        User User = create();

        String s = objectMapper.writeValueAsString(User);
        return objectMapper.readValue(s, User.class);
    }

    @Benchmark
    public Object gson() throws Exception {
        User User = create();
        String s = gson.toJson(User);
        return gson.fromJson(s, User.class);
    }

    @Benchmark
    public Object cloneFun() throws Exception {
        User User = create();
        return User.clone();
    }

    @Benchmark
    public Object mapstruct() throws Exception {
        User User = create();
        return UserMapper.INSTANCE.copy(User);
    }

    private User create() {
        User user = new User();
        user.setName("John Doe");
        user.setAge(30);
        user.setAddress("123 Main St");
        user.setPhone("555-1234");
        user.setEmail("john.doe@example.com");
        user.setQq("123456789");
        user.setWechat("johndoe");
        user.setWeibo("johndoe");
        user.setGithub("johndoe");
        return user;
    }

}
```

截取一个基准测试的输出结果如下

```java
# JMH version: 1.35
# VM version: JDK 1.8.0_432, OpenJDK 64-Bit Server VM, 25.432-b07
# VM invoker: /Users/didi/Library/Java/JavaVirtualMachines/liberica-full-1.8.0_432/jre/bin/java
# VM options: -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=58492:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8
# Blackhole mode: full + dont-inline hint (auto-detected, use -Djmh.blackhole.autoDetect=false to disable)
# Warmup: 5 iterations, 10 s each
# Measurement: 5 iterations, 10 s each
# Timeout: 10 min per iteration
# Threads: 1 thread, will synchronize iterations
# Benchmark mode: Average time, time/op
# Benchmark: org.example.jmh.JsonMain.beanUtil

# Run progress: 0.00% complete, ETA 00:10:00
# Fork: 1 of 1
# Warmup Iteration   1: 1.376 us/op
# Warmup Iteration   2: 1.343 us/op
# Warmup Iteration   3: 1.330 us/op
# Warmup Iteration   4: 1.365 us/op
# Warmup Iteration   5: 1.342 us/op
Iteration   1: 1.645 us/op
Iteration   2: 2.060 us/op
Iteration   3: 1.392 us/op
Iteration   4: 1.313 us/op
Iteration   5: 1.586 us/op

Result "org.example.jmh.JsonMain.beanUtil":
  1.599 ±(99.9%) 1.122 us/op [Average]
  (min, avg, max) = (1.313, 1.599, 2.060), stdev = 0.291
  CI (99.9%): [0.477, 2.721] (assumes normal distribution)
```

第一部分输出了基准测试的一些配置，第二部分是基准测试的过程，第三部分是基准测试的结果。

以下是对上述 JMH（Java Microbenchmark Harness）基准测试结果数据的详细解释：

### 1. “1.599 ±(99.9%) 1.122 us/op [Average]”部分

- **“1.599”**：这是所测试方法（在这个例子里是 `org.example.jmh.JsonMain.beanUtil` 这个方法）每次操作（operation，即每次调用该方法）的平均执行时间，单位是微秒（`us`）。也就是说，平均来看，每次调用这个方法大概需要花费1.599微秒的时间来完成执行。
- **“±(99.9%) 1.122”**：这里表示的是误差范围。在统计测量中，由于存在各种不确定性因素，测量结果往往带有一定的误差。“±(99.9%)” 说明这个误差范围是按照99.9%的置信水平来计算的，而“1.122”就是对应的误差数值（单位同样是微秒/操作）。这意味着在99.9%的置信度下，真实的平均执行时间有较大概率落在 “1.599 - 1.122” 到 “1.599 + 1.122” 这个区间内，也就是 [0.477, 2.721]（后面的置信区间部分会再详细说明这个区间的意义）。

### 2. “(min, avg, max) = (1.313, 1.599, 2.060), stdev = 0.291”部分

- **“(min, avg, max)”**：展示了在所有测试样本中，该方法单次执行时间的最小值（`min`）、平均值（`avg`）和最大值（`max`）。在这里，最小值是1.313微秒，表示该方法在某次执行时达到的最快速度；平均值就是前面提到的1.599微秒；最大值是2.060微秒，意味着在某些情况下，该方法的执行可能会慢到花费2.060微秒这么长的时间，这体现了方法执行时间的波动范围。
- **“stdev = 0.291”**：“stdev”指的是标准差（Standard Deviation），它是用来衡量一组数据离散程度的统计量。标准差为0.291说明各个测试样本中该方法执行时间相对平均值的离散情况，标准差越大，表示数据越分散，即方法执行时间的波动越大；在这里，标准差0.291表明执行时间存在一定的波动，但不算特别大。

### 3. “CI (99.9%): [0.477, 2.721] (assumes normal distribution)”部分

- **“CI (99.9%)”**：“CI”是置信区间（Confidence Interval）的缩写，99.9%表示置信水平。置信区间是在一定置信水平下，估计总体参数所在的区间范围。简单来说，基于这次的测试数据以及假定数据符合正态分布（“assumes normal distribution” 所提示的），我们有99.9%的把握认为，所测试方法真正的平均执行时间是落在 [0.477, 2.721] 这个区间内的。也就是说，虽然我们通过测试得到的平均执行时间是1.599微秒，但考虑到测量的不确定性等因素，真实的平均执行时间大概率会在这个计算出来的区间之中。

## 各种参数

### warmupMode 预热模式

取值有以下三种，默认使用 INDI，即：

```java
/**
 * 为每个基准测试做单独的热身
 */
INDI(false, true),

/**
 * 在任何基准测试开始之前进行批量预热。
 */
BULK(true, false),

/**
 * 在任何基准测试开始之前进行批量预热，然后为每个基准测试进行单独的预热。
 */
BULK_INDI(true, true)
```
