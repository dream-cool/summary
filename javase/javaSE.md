---

title: java基础语法总结
date: 2019-11-17
categories: JavaSE
tags: java基础

---

# final关键字修饰变量总结

```java
@org.junit.Test
    public void fianlKeyWordTest(){
        final Dog dog = new Dog("旺财");
        Dog dog1 = new Dog("阿狗");
        dog.setName("来福");
        /**
         *  final 修饰的变量 可以改变该变量地址所指向的对象里面的内容
         *  但无法改变该变量所指向的地址，即一旦将该变量指向某一地址后就再无法更改它所指向的地址
         */
//        dog = dog1;  ×
        System.out.println(dog); // Dog{name='来福', age=null}
    }
```

