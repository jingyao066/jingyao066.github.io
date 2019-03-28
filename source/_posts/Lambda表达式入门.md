---
title: Lambda表达式入门
tags: java
date: 2018-12-06 18:26:54
---

# 简介
其实Lambda表达式的本质只是一个"语法糖"，由编译器推断并帮你转换包装为常见的代码，因此可以使用更少的代码来实现同样的功能。
(建议不要乱用,因为这就和某些很高级的黑客写的代码一样,简洁,难懂,难以调试,维护人员想骂娘)
Lambda表达式是Java SE 8中一个重要的新特性。lambda表达式允许你通过表达式来代替功能接口。
lambda表达式就和方法一样,它提供了一个正常的参数列表和一个使用这些参数的主体(body,可以是一个表达式或一个代码块)。
Lambda表达式还增强了集合库。
Java SE 8添加了2个对集合数据进行批量操作的包: java.util.function 包以及 java.util.stream 包。
流(stream)就如同迭代器(iterator),但附加了许多额外的功能。
 总的来说,lambda表达式和 stream 是自Java语言添加泛型(Generics)和注解(annotation)以来最大的变化。
在本文中,我们将从简单到复杂的示例中见认识lambda表达式和stream的强悍。

# 环境准备
必须安装jdk1.8以上版本..

# Lambda表达式的语法
基本语法:
(parameters) -> expression
或
(parameters) ->{ statements; }

简单例子：
// 1. 不需要参数,返回值为 5
() -> 5

// 2. 接收一个参数(数字类型),返回其2倍的值
x -> 2 * x

// 3. 接受2个参数(数字),并返回他们的差值
(x, y) -> x – y

// 4. 接收2个int型整数,返回他们的和
(int x, int y) -> x + y

// 5. 接受一个 string 对象,并在控制台打印,不返回任何值(看起来像是返回void)
(String s) -> System.out.print(s)

# 基本的Lambda例子
假设有一个玩家List ,程序员可以使用 for 语句 ("for 循环")来遍历,在Java SE 8中可以转换为另一种形式:
```
String[] atp = {"小明","小红", "小黑", "小白","小兰", "小强","小绿", "小小"};
        List<String> players =  Arrays.asList(atp);

        // 以前的循环方式
//        for (String player : players) {
//            System.out.print(player + "; ");
//        }

        //使用lambda表达式以及函数操作(functional operation)
        //players.forEach((player) -> System.out.println(player + ";"));

        //在JAVA 8 中使用双冒号操作符(double colon operator)
        //players.forEach(System.out::println);
```

```
//下面是使用lambdas 来实现 Runnable接口 的示例:
        // 1.1使用匿名内部类
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("Hello world !");
            }
        }).start();

        // 1.2使用 lambda expression
        new Thread(() -> System.out.println("Hello world !")).start();

        // 2.1使用匿名内部类
        Runnable race1 = new Runnable() {
            @Override
            public void run() {
                System.out.println("Hello world !");
            }
        };

        // 2.2使用 lambda expression
        Runnable race2 = () -> System.out.println("Hello world !");

        // 直接调用 run 方法(没开新线程哦!)
        race1.run();
        race2.run();
```

# 使用Lambda对集合进行排序
在java中，Comparator 类被用来排序集合，
在下面的例子中,我们将根据球员的 name, surname, name 长度 以及最后一个字母。 和前面的示例一样,先使用匿名内部类来排序,然后再使用lambda表达式精简我们的代码。
在第一个例子中,我们将根据name来排序list。 使用旧的方式,代码如下所示:

String[] atp = {"小明","小红", "小黑", "小白","小兰", "小强","小绿", "小小"};
```
// 1.1 使用匿名内部类根据 name 排序 players 
Arrays.sort(players, new Comparator<String>() { 
    @Override 
    public int compare(String s1, String s2) { 
        return (s1.compareTo(s2));  
    } 
});
```
使用lambdas,可以通过下面的代码实现同样的功能:
```
// 1.2 使用 lambda expression 排序 players 
Comparator<String> sortByName = (String s1, String s2) -> (s1.compareTo(s2)); 
Arrays.sort(players, sortByName); 
  
// 1.3 也可以采用如下形式: 
Arrays.sort(players, (String s1, String s2) -> (s1.compareTo(s2)));
```

其他的排序如下所示。 和上面的示例一样,代码分别通过匿名内部类和一些lambda表达式来实现Comparator :
```
// 1.1 使用匿名内部类根据 surname 排序 players 
Arrays.sort(players, new Comparator<String>() { 
    @Override 
    public int compare(String s1, String s2) { 
        return (s1.substring(s1.indexOf(" ")).compareTo(s2.substring(s2.indexOf(" "))));  
    } 
}); 
  
// 1.2 使用 lambda expression 排序,根据 surname 
Comparator<String> sortBySurname = (String s1, String s2) -> 
    ( s1.substring(s1.indexOf(" ")).compareTo( s2.substring(s2.indexOf(" ")) ) ); 
Arrays.sort(players, sortBySurname); 
  
// 1.3 或者这样,怀疑原作者是不是想错了,括号好多... 
Arrays.sort(players, (String s1, String s2) -> 
      ( s1.substring(s1.indexOf(" ")).compareTo( s2.substring(s2.indexOf(" ")) ) ) 
    );
  
// 2.1 使用匿名内部类根据 name lenght 排序 players
Arrays.sort(players, new Comparator<String>() {
    @Override
    public int compare(String s1, String s2) {
        return (s1.length() - s2.length());  
    }
});
  
// 2.2 使用 lambda expression 排序,根据 name lenght
Comparator<String> sortByNameLenght = (String s1, String s2) -> (s1.length() - s2.length());
Arrays.sort(players, sortByNameLenght);
  
// 2.3 or this
Arrays.sort(players, (String s1, String s2) -> (s1.length() - s2.length()));
  
// 3.1 使用匿名内部类排序 players, 根据最后一个字母
Arrays.sort(players, new Comparator<String>() {
    @Override
    public int compare(String s1, String s2) {
        return (s1.charAt(s1.length() - 1) - s2.charAt(s2.length() - 1));  
    }
});
  
// 3.2 使用 lambda expression 排序,根据最后一个字母
Comparator<String> sortByLastLetter = 
    (String s1, String s2) ->
        (s1.charAt(s1.length() - 1) - s2.charAt(s2.length() - 1));  
Arrays.sort(players, sortByLastLetter);
  
// 3.3 or this
Arrays.sort(players, (String s1, String s2) -> (s1.charAt(s1.length() - 1) - s2.charAt(s2.length() - 1)));
```

就是这样,简洁又直观。 在下一节中我们将探索更多lambdas的能力,并将其与 stream 结合起来使用。

# 使用Lambdas和Streams
Stream是对集合的包装,通常和lambda一起使用。 
 使用lambdas可以支持许多操作,如 map, filter, limit, sorted, count, min, max, sum, collect 等等。
同样,Stream使用懒运算,他们并不会真正地读取所有数据,遇到像getFirst() 这样的方法就会结束链式语法。
在接下来的例子中,我们将探索lambdas和streams 能做什么。
我们创建了一个Person类并使用这个类来添加一些数据到list中,将用于进一步流操作。
Person 只是一个简单的POJO类:
```
public class Person {
  
private String firstName, lastName, job, gender;
private int salary, age;
  
public Person(String firstName, String lastName, String job,
                String gender, int age, int salary)       {  
          this.firstName = firstName;  
          this.lastName = lastName;  
          this.gender = gender;  
          this.age = age;  
          this.job = job;  
          this.salary = salary;  
}
// Getter and Setter
// . . . . .
}
```
接下来,我们将创建两个list,都用来存放Person对象:
```
List<Person> javaProgrammers = new ArrayList<Person>() {
  {
    add(new Person("Elsdon", "Jaycob", "Java programmer", "male", 43, 2000));
    add(new Person("Tamsen", "Brittany", "Java programmer", "female", 23, 1500));
    add(new Person("Floyd", "Donny", "Java programmer", "male", 33, 1800));
    add(new Person("Sindy", "Jonie", "Java programmer", "female", 32, 1600));
    add(new Person("Vere", "Hervey", "Java programmer", "male", 22, 1200));
    add(new Person("Maude", "Jaimie", "Java programmer", "female", 27, 1900));
    add(new Person("Shawn", "Randall", "Java programmer", "male", 30, 2300));
    add(new Person("Jayden", "Corrina", "Java programmer", "female", 35, 1700));
    add(new Person("Palmer", "Dene", "Java programmer", "male", 33, 2000));
    add(new Person("Addison", "Pam", "Java programmer", "female", 34, 1300));
  }
};
  
List<Person> phpProgrammers = new ArrayList<Person>() {
  {
    add(new Person("Jarrod", "Pace", "PHP programmer", "male", 34, 1550));
    add(new Person("Clarette", "Cicely", "PHP programmer", "female", 23, 1200));
    add(new Person("Victor", "Channing", "PHP programmer", "male", 32, 1600));
    add(new Person("Tori", "Sheryl", "PHP programmer", "female", 21, 1000));
    add(new Person("Osborne", "Shad", "PHP programmer", "male", 32, 1100));
    add(new Person("Rosalind", "Layla", "PHP programmer", "female", 25, 1300));
    add(new Person("Fraser", "Hewie", "PHP programmer", "male", 36, 1100));
    add(new Person("Quinn", "Tamara", "PHP programmer", "female", 21, 1000));
    add(new Person("Alvin", "Lance", "PHP programmer", "male", 38, 1600));
    add(new Person("Evonne", "Shari", "PHP programmer", "female", 40, 1800));
  }
};
```
现在我们使用forEach方法来迭代输出上述列表:
```
System.out.println("所有程序员的姓名:");
javaProgrammers.forEach((p) -> System.out.printf("%s %s; ", p.getFirstName(), p.getLastName()));
phpProgrammers.forEach((p) -> System.out.printf("%s %s; ", p.getFirstName(), p.getLastName()));
```
我们同样使用forEach方法,增加程序员的工资5%:
```
System.out.println("给程序员加薪 5% :");
Consumer<Person> giveRaise = e -> e.setSalary(e.getSalary() / 100 * 5 + e.getSalary());
  
javaProgrammers.forEach(giveRaise);
phpProgrammers.forEach(giveRaise);
```
另一个有用的方法是过滤器filter() ,让我们显示月薪超过1400美元的PHP程序员:
```
System.out.println("下面是月薪超过 $1,400 的PHP程序员:")
phpProgrammers.stream()
          .filter((p) -> (p.getSalary() > 1400))  
          .forEach((p) -> System.out.printf("%s %s; ", p.getFirstName(), p.getLastName()));
```
我们也可以定义过滤器,然后重用它们来执行其他操作:
```
// 定义 filters
Predicate<Person> ageFilter = (p) -> (p.getAge() > 25);
Predicate<Person> salaryFilter = (p) -> (p.getSalary() > 1400);
Predicate<Person> genderFilter = (p) -> ("female".equals(p.getGender()));
  
System.out.println("下面是年龄大于 24岁且月薪在$1,400以上的女PHP程序员:");
phpProgrammers.stream()
          .filter(ageFilter)  
          .filter(salaryFilter)  
          .filter(genderFilter)  
          .forEach((p) -> System.out.printf("%s %s; ", p.getFirstName(), p.getLastName()));  
  
// 重用filters
System.out.println("年龄大于 24岁的女性 Java programmers:");
javaProgrammers.stream()
          .filter(ageFilter)  
          .filter(genderFilter)  
          .forEach((p) -> System.out.printf("%s %s; ", p.getFirstName(), p.getLastName()));  
```
使用limit方法,可以限制结果集的个数:
```
System.out.println("最前面的3个 Java programmers:");
javaProgrammers.stream()
          .limit(3)  
          .forEach((p) -> System.out.printf("%s %s; ", p.getFirstName(), p.getLastName()));  
  
  
System.out.println("最前面的3个女性 Java programmers:");
javaProgrammers.stream()
          .filter(genderFilter)  
          .limit(3)  
          .forEach((p) -> System.out.printf("%s %s; ", p.getFirstName(), p.getLastName()));  
```
排序呢? 我们在stream中能处理吗? 答案是肯定的。 在下面的例子中,我们将根据名字和薪水排序Java程序员,放到一个list中,然后显示列表:
```
System.out.println("根据 name 排序,并显示前5个 Java programmers:");
List<Person> sortedJavaProgrammers = javaProgrammers
          .stream()  
          .sorted((p, p2) -> (p.getFirstName().compareTo(p2.getFirstName())))  
          .limit(5)  
          .collect(toList());  
  
sortedJavaProgrammers.forEach((p) -> System.out.printf("%s %s; %n", p.getFirstName(), p.getLastName()));
   
System.out.println("根据 salary 排序 Java programmers:");
sortedJavaProgrammers = javaProgrammers
          .stream()  
          .sorted( (p, p2) -> (p.getSalary() - p2.getSalary()) )  
          .collect( toList() );  
  
sortedJavaProgrammers.forEach((p) -> System.out.printf("%s %s; %n", p.getFirstName(), p.getLastName())); 
```
如果我们只对最低和最高的薪水感兴趣,比排序后选择第一个/最后一个 更快的是min和max方法:
```
System.out.println("工资最低的 Java programmer:");
Person pers = javaProgrammers
          .stream()  
          .min((p1, p2) -> (p1.getSalary() - p2.getSalary()))  
          .get()  
  
System.out.printf("Name: %s %s; Salary: $%,d.", pers.getFirstName(), pers.getLastName(), pers.getSalary())
  
System.out.println("工资最高的 Java programmer:");
Person person = javaProgrammers
          .stream()  
          .max((p, p2) -> (p.getSalary() - p2.getSalary()))  
          .get()  
  
System.out.printf("Name: %s %s; Salary: $%,d.", person.getFirstName(), person.getLastName(), person.getSalary())
```
上面的例子中我们已经看到 collect 方法是如何工作的。 结合 map 方法,我们可以使用 collect 方法来将我们的结果集放到一个字符串,一个 Set 或一个TreeSet中:
```
System.out.println("将 PHP programmers 的 first name 拼接成字符串:");
String phpDevelopers = phpProgrammers
          .stream()  
          .map(Person::getFirstName)  
          .collect(joining(" ; ")); // 在进一步的操作中可以作为标记(token)     
  
System.out.println("将 Java programmers 的 first name 存放到 Set:");
Set<String> javaDevFirstName = javaProgrammers
          .stream()  
          .map(Person::getFirstName)  
          .collect(toSet());  
  
System.out.println("将 Java programmers 的 first name 存放到 TreeSet:");
TreeSet<String> javaDevLastName = javaProgrammers
          .stream()  
          .map(Person::getLastName)  
          .collect(toCollection(TreeSet::new));
```
Streams 还可以是并行的(parallel)。 示例如下:
```
System.out.println("计算付给 Java programmers 的所有money:");
int totalSalary = javaProgrammers
          .parallelStream()  
          .mapToInt(p -> p.getSalary())  
          .sum();  
```
我们可以使用summaryStatistics方法获得stream 中元素的各种汇总数据。 接下来,我们可以访问这些方法,比如getMax, getMin, getSum或getAverage:
```
//计算 count, min, max, sum, and average for numbers
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
IntSummaryStatistics stats = numbers
          .stream()  
          .mapToInt((x) -> x)  
          .summaryStatistics();  
  
System.out.println("List中最大的数字 : " + stats.getMax());
System.out.println("List中最小的数字 : " + stats.getMin());
System.out.println("所有数字的总和   : " + stats.getSum());
System.out.println("所有数字的平均值 : " + stats.getAverage()); 
```
