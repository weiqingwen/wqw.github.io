---
layout: post
title:  "不止是SQL，Java 8也可以GROUP BY"
date:   2021-08-11
last_modified_at: 2021-08-11
categories: [Java]
tags: [java8, jdk]
---
GROUP BY是SQL语言中一个常用的分组操作。在Java 8中，我们也可以进行类似的分组操作。假设有一个Persons列表，那么如何按年龄或性别对这些人进行分组，我们可以通过for循环，检查每个人，并将他们放在同一个城市或年龄的HashMap列表中，这种方法虽然是实现的一种，但是显然不够优雅。这时我们可以使用`java.util.stream.Collectors`类中的`groupingBy()`方法来实现类似SQL语言中的GROUP BY操作。

具体代码如下：

```
class Person{
    private String name;
    private String city;
    private int age;

    public Person(String name, String city, int age) {
        this.name = name;
        this.city = city;
        this.age = age;
    }

    // setters, getters......
}
```

```
import java.io.IOException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Objects;
import java.util.stream.Collectors;

public class GroupByDemoInJava8 {
  public static void main(String args[]) throws IOException {

        List<Person> people = new ArrayList<>();
        people.add(new Person("小李", "Beijing", 21));
        people.add(new Person("小黄", "Beijing", 21));
        people.add(new Person("老潘", "Beijing", 23));
        people.add(new Person("大黄", "Shanghai", 23));
        people.add(new Person("周周", "Guangzhou", 23));
        people.add(new Person("阿Mike", "Guangzhou", 31));

        // 不优雅的实现方法
        Map<String,List<Person>> personByCity = new HashMap<>(); 
        for(Person p : people){ 
          if(!personByCity.containsKey(p.getCity())){ 
            personByCity.put(p.getCity(), new ArrayList<>()); 
          } 
          personByCity.get(p.getCity()).add(p); 
        }
        System.out.println("遍历全部列表，然后按城市分组: " + personByCity);

        // 在Java 8中优雅的实现方法
        // 根据城市来分组
        personByCity =  people.stream()
                         .collect(Collectors.groupingBy(Person::getCity));
        System.out.println("根据城市来分组: " + personByCity);
        
        // 根据年龄来分组
        Map<Integer,List<Person>> personByAge = people.stream()
                          .collect(Collectors.groupingBy(Person::getAge));
        System.out.println("根据年龄来分组: " + personByAge);
}
```

输出结果
```
遍历后按城市分组: 
{
  Beijing=[小李(Beijing,21), 小黄(Beijing,21), 老潘(Beijing,23)], 
  Shanghai=[大黄(Shanghai,23)], 
  Guangzhou=[周周(Guangzhou,23), 阿Mike(Guangzhou,31)]
}

根据城市来分组: 
{
  Beijing=[小李(Beijing,21), 小黄(Beijing,21), 老潘(Beijing,23)], 
  Shanghai=[大黄(Shanghai,23)], 
  Guangzhou=[周周(Guangzhou,23), 阿Mike(Guangzhou,31)]
}

根据年龄来分组: 
{
  21=[小李(Beijing,21), 小黄(Beijing,21)], 
  23=[老潘(Beijing,23), 大黄(Shanghai,23), 周周(Guangzhou,23)], 
  31=[阿Mike(Guangzhou,31)]
}
```

从上面的例子我们可以看出，使用`groupingBy()`方法和lambda表达式，我们可以根据`List`中对象的某个属性进行分组，并且返回`Map`对象。