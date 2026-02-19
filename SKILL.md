---
name: ataopro-alibaba-java-coding-guidelines
description: 阿里巴巴Java开发手册（阿里开发规范），用于代码规范检查。当用户需要检查Java代码是否符合阿里开发规范时调用。
author: ataopro
version: 1.0.0
date: 2026-02-20
copyright: Copyright © 2026 ataopro. All rights reserved.
---

# 阿里巴巴Java开发手册

本技能基于阿里巴巴Java开发手册（简称：阿里开发规范），用于代码规范检查和指导开发。

## 触发场景

当用户表达以下意图时，此技能将被激活：

1. **代码规范检查**
   - "检查代码规范"
   - "帮我检查Java代码"
   - "代码审查"
   - "查看代码是否符合阿里规范"
   - "检查代码是否有问题"

2. **指定开发规范**
   - "使用阿里开发规范"
   - "用阿里巴巴规范开发"
   - "按阿里开发手册来"
   - "遵循阿里开发规范"
   - "阿里开发规范"

3. **优化建议**
   - "优化Java代码"
   - "代码改进建议"
   - "如何写出规范的Java代码"

## 核心规则速查

### 1. 命名风格

| 规则 | 正例 | 反例 |
|------|------|------|
| 类名 | `UserDO`, `CacheServiceImpl` | `userDo`, `User_DO` |
| 方法名/变量 | `getUserById`, `localValue` | `get_user_by_id` |
| 常量 | `MAX_STOCK_COUNT` | `MAX_COUNT`, `maxStockCount` |
| 布尔变量 | `deleted`, `active` (不加is前缀) | `isDeleted` (POJO中) |

### 2. 常量定义

- **禁止魔法值**：不允许未经定义的常量直接出现
  ```java
  // 反例
  String key = "Id#taobao_" + tradeId;
  
  // 正例
  final String KEY_PREFIX = "Id#taobao_";
  String key = KEY_PREFIX + tradeId;
  ```

- **Long类型赋值使用大写L**：`Long a = 2L;` (不是2l)

### 3. OOP规约

- **避免空指针**：使用常量或确定有值的对象调用equals
  ```java
  // 正例
  "test".equals(object);
  Objects.equals(object, "test");
  
  // 反例
  object.equals("test");
  ```

- **包装类比较**：使用equals而非==
  ```java
  // 正例
  Integer a = 127;
  Integer b = 127;
  a.equals(b);  // true
  
  // 反例 -128~127范围外
  Integer a = 128;
  Integer b = 128;
  a == b;  // false
  ```

- **POJO类**：所有属性使用包装类型，不设默认值

- **访问控制从严**：
  - 工具类不允许有public构造方法
  - 成员变量仅本类使用用private
  - static成员变量考虑是否为final

### 4. 集合处理

- **hashCode与equals**：重写equals必须重写hashCode

- **subList不可强转**：
  ```java
  // 反例
  ArrayList list = subList; // ClassCastException
  
  // 正例
  List list = subList;
  ```

- **Arrays.asList()**：返回的列表不可修改
  ```java
  String[] str = {"a", "b"};
  List list = Arrays.asList(str);
  list.add("c"); // UnsupportedOperationException
  ```

- **foreach中禁止remove**：
  ```java
  // 反例
  for (String item : list) {
      list.remove(item); // ConcurrentModificationException
  }
  
  // 正例
  Iterator<String> it = list.iterator();
  while (it.hasNext()) {
      if (condition) {
          it.remove();
      }
  }
  ```

- **Map遍历**：使用entrySet而非keySet

### 5. 并发处理

- **线程池创建**：
  ```java
  // 反例 - 可能导致OOM
  ExecutorService executor = Executors.newFixedThreadPool(10);
  
  // 正例
  ExecutorService executor = new ThreadPoolExecutor(
      10, 50, 60L, TimeUnit.SECONDS,
      new LinkedBlockingQueue<>(100)
  );
  ```

- **SimpleDateFormat线程不安全**：
  ```java
  // 方案1: 使用 private static final ThreadLocal<DateFormat>ThreadLocal
  df = 
      ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));
  
  // 方案2: JDK8+ 使用DateTimeFormatter
  private static final DateTimeFormatter DFT = 
      DateTimeFormatter.ofPattern("yyyy-MM-dd");
  ```

- **Random多线程竞争**：使用ThreadLocalRandom

### 6. 控制语句

- **switch必须包含default**

- **if/while等必须使用大括号**
  ```java
  // 反例
  if (condition) statement;
  
  // 正例
  if (condition) {
      statement;
  }
  ```

- **避免多层嵌套**：超过3层用卫语句/策略模式

### 7. 注释规范

- **类/方法注释**：使用Javadoc `/** */` 格式
- **TODO标记**：`// TODO(作者: 日期) 说明`
- **FIXME标记**：`// FIXME(作者: 日期) 说明`

### 8. MySQL规约

- **主键设计**：自增为主键，分布式用UUID/分布式ID
- **索引命名**：`idx_字段名`、`uniq_字段名`
- **TEXT字段**：单独表或前缀索引
- **count(*) vs count(列)**：count(*)包含NULL，count(列)不包含

### 9. 异常处理

- **异常捕获**：不要捕获Throwable/Exception(除非特定需求)
- **finally块**：确保资源关闭，使用try-with-resources
- **异常链**：保留原异常信息

### 10. 日志规约

- **日志级别**：
  - ERROR: 影响功能的异常
  - WARN: 不影响功能但需注意
  - INFO: 关键业务节点
  - DEBUG: 调试信息

- **日志内容**：包含上下文信息，避免`userId + " login"`

## 快速检查清单

代码提交前检查：

- [ ] 命名符合规范（类名、方法名、常量等）
- [ ] 无魔法值
- [ ] equals/hashCode配对重写
- [ ] 集合操作无ConcurrentModificationException风险
- [ ] 线程安全（ThreadLocal、线程池）
- [ ] 无空指针风险（equals顺序、参数校验）
- [ ] switch有default
- [ ] if/while有花括号
- [ ] 注释完整（Javadoc）

## 相关资源

- 阿里巴巴Java开发手册PDF下载
- IDEA插件：Alibaba Java Coding Guidelines
- Maven插件：SpotBugs + CheckStyle

---

## 更新日志

### v1.0.0 (2026-02-20)
- 初始版本
- 基于阿里巴巴Java开发手册整理
- 包含10大章节核心规范

---

**版权声明**: 本技能内容整理自阿里巴巴Java开发手册，仅供学习参考使用。
**作者**: ataopro
**版本**: 1.0.0
