## 创建和使用解析器

```java
String elStr = "xxx";

ExpressionParser parser = new SpelExpressionParser();
Expression expression = parser.parseExpression(elStr);
String result = (String) expression.getValue();
```

---
## 字面量

```java
String elStr = "'Hello World'";     // 字符串字面量

String elStr = "6.0221415E+23";     // 数字字面量
String elStr = "0x7FFFFFFF";
String elStr = "9527";

String elStr = "true";              // 布尔值字面量

String elStr = "null";              // null 对象字面量
```

---
## 集合

```java
String elStr = "{1, 2, 3, 4}";      // 表示 List 和 Set 对象

String elStr = "{name:'Nikola', dob:'10-July-1856'}")
                                    // 表示 Map 对象

String elStr = "new int[4]"         // 表示数组对象
String elStr = "new int[]{1,2,3}";
```

---
## 访问对象属性和集合单元

```java
String elStr = "dept1.deptno";      // 访问对象属性

String elStr = "arr[3]";            // 访问 数组 和 list 单元
String elStr = "list[3]";

String elStr = "Officers['president']";
                                    // 访问 Map 键值对
```


---
## 方法

方法和Java语法一样。

```java
// string literal, evaluates to "bc"
String c = parser.parseExpression("'abc'.substring(2, 3)").getValue(String.class);
```

---
## 运算符

表达式中支持各种运算符，运算规则和Java规则类似。唯一需要注意的是空值的处理，假设有非空值val，那么下面的表达式恒为真：val > null。这一点需要注意。

```java
// evaluates to true
boolean trueValue = parser.parseExpression("2 == 2").getValue(Boolean.class);

// evaluates to false
boolean falseValue = parser.parseExpression("2 < -5.0").getValue(Boolean.class);

// evaluates to true
boolean trueValue = parser.parseExpression("'black' < 'block'").getValue(Boolean.class);
```

逻辑运算符可以使用 and、or 和 ! 。

---

## 类型

特殊的T运算符可以获取表达式对象的类型。

```java
Class dateClass = parser.parseExpression("T(java.util.Date)").getValue(Class.class);

Class stringClass = parser.parseExpression("T(String)").getValue(Class.class);

boolean trueValue = parser.parseExpression(
        "T(java.math.RoundingMode).CEILING < T(java.math.RoundingMode).FLOOR")
        .getValue(Boolean.class);
```

---
## 构造器

在表达式中，使用new关键字来调用构造器。

```java
Inventor einstein = p.parseExpression(
        "new org.spring.samples.spel.inventor.Inventor('Albert Einstein', 'German')")
        .getValue(Inventor.class);
```

---
## 变量

在表达式上下文中，我们可以设置新变量。然后在表达式中使用#变量名访问变量。

```java
Inventor tesla = new Inventor("Nikola Tesla", "Serbian");
StandardEvaluationContext context = new StandardEvaluationContext(tesla);
context.setVariable("newName", "Mike Tesla");

parser.parseExpression("Name = #newName").getValue(context);

System.out.println(tesla.getName()) // "Mike Tesla"
```

---
## #this和#root

`#this` 和 `#root` 代表了表达式上下文的对象，#root就是当前的表达式上下文对象，#this则根据当前求值环境的不同而变化。下面的例子中，#this即每次循环的值。

```java
// create an array of integers
List<Integer> primes = new ArrayList<Integer>();
primes.addAll(Arrays.asList(2,3,5,7,11,13,17));

// create parser and set variable 'primes' as the array of integers
ExpressionParser parser = new SpelExpressionParser();
StandardEvaluationContext context = new StandardEvaluationContext();
context.setVariable("primes",primes);

// all prime numbers > 10 from the list (using selection ?{...})
// evaluates to [11, 13, 17]
List<Integer> primesGreaterThanTen = (List<Integer>) parser.parseExpression(
        "#primes.?[#this>10]").getValue(context);
```

---
## Bean 引用

这是Spring表达式独有的功能，我们可以在表达式中引用配置文件定义的其他Bean，这需要语法@Bean名称。

```java
ExpressionParser parser = new SpelExpressionParser();
StandardEvaluationContext context = new StandardEvaluationContext();
context.setBeanResolver(new MyBeanResolver());

// This will end up calling resolve(context,"foo") on MyBeanResolver during evaluation
Object bean = parser.parseExpression("@foo").getValue(context);
如果需要获取Bean工厂本身而不是它构造的Bean，可以使用&Bean名称。

Object bean = parser.parseExpression("&foo").getValue(context);
```

---
## 三元运算符

和Java的三元运算符类似。

```java
String falseString = parser.parseExpression(
        "false ? 'trueExp' : 'falseExp'").getValue(String.class);
```

---
## Elvis 运算符

在一些编程语言中（比如C#、Kotlin等）提供该功能，语法是?:。意义是当某变量不为空的时候使用该变量，当该变量为空的时候使用指定的默认值。

```java
ExpressionParser parser = new SpelExpressionParser();

String name = parser.parseExpression("name?:'Unknown'").getValue(String.class);

System.out.println(name); // 'Unknown'
```

---
## 安全导航运算符

这是来自Groovy的一个功能，语法是?.，当然有些语言也提供了这个功能。当我们对对象的某个属性求值时，如果该对象本身为空，就会抛出空指针异常，如果使用安全导航运算符，空对象的属性就会简单的返回空。

```java
city = parser.parseExpression("PlaceOfBirth?.City").getValue(context, String.class);

System.out.println(city); // null - does not throw NullPointerException!!!
```

---
## 集合选择

这有点类似Java 8的Filter流方法，作用是选择或者说是过滤，语法是集合对象.?[选择表达式]，Spring会迭代集合对象的每一个元素，并使用选择表达式判断该元素是否满足条件，最后返回由满足条件的元素组成的新集合。下面的例子就返回了值大于27的新Map。

```java
Map newMap = parser.parseExpression("map.?[value<27]").getValue();
```

---
## 集合投影

这类似Java 8的Map流方法或者SQL语言的选择语句，作用是将一个集合中所有元素的某属性抽取出来，组成一个新集合。语法是![投影表达式]。下面的例子选出了由Member的placeOfBirth的city属性组成的新集合。

```java
List placesOfBirth = (List)parser.parseExpression("Members.![placeOfBirth.city]");
```

---
## 表达式模板

表达式模板使用#{}定义，它允许我们混合多种结果。下面就是一个例子，首先Spring会先对模板中的表达式求值，在这里是返回一个随机值，然后将结果和外部的表达式组合起来。最终的结果就向下面这样了。

```java
String randomPhrase = parser.parseExpression(
        "random number is #{T(java.lang.Math).random()}",
        new TemplateParserContext()).getValue(String.class);
// 结果是 "random number is 0.7038186818312008"
```

如果表达式只是一个简单的表达式，就不需要使用模板。只有表达式有很多表达式组成时才需要。
