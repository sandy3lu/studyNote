
# builder
```java
// 创建名为Officer的java bean
@Builder
public class Officer {
    //之所以使用构建器模式，是因为我们要创造的对象是一个成员变量不可变的对象。Officer类的成员变量都是final的,final修饰的成员变量，需要在实例创建时就把值确定下来
    private final String id;
    private final String name;
    private final int age;
    private final String department;
}

// 调用构建器生成Officer实例
class BuilderTest {
    public static void main(String[] args) {
        Officer officer = Officer.builder().id("00001").name("simon qi")
                .age(26).department("departmentA").build();
    }
}
```

反编译lombok生成的Officer.class
```java
public class Officer {
    private final String id;
    private final String name;
    private final int age;
    private final String department;

    Officer(String id, String name, int age, String department) {
        this.id = id;
        this.name = name;
        this.age = age;
        this.department = department;
    }

    public static Officer.OfficerBuilder builder() {
        return new Officer.OfficerBuilder();
    }
//源码中有一个OfficerBuilder的静态内部类，我们在调用builder方法时，实际返回了这个静态内部类的实例。这个OfficerBuilder类，具有和Officer相同的成员变量，且拥有名为id,name,age和department的方法。这些以Officer的成员变量命名的方法，都是给OfficerBuilder的成员变量赋值，并返回this。
//这些方法返回this，其实就是返回调用这些方法的OfficerBuilder对象，也可称为“返回对象本身”。通过返回对象本身，形成了方法的链式调用。
    public static class OfficerBuilder {
        private String id;
        private String name;
        private int age;
        private String department;

        OfficerBuilder() {
        }

        public Officer.OfficerBuilder id(String id) {
            this.id = id;
            return this;
        }

        public Officer.OfficerBuilder name(String name) {
            this.name = name;
            return this;
        }

        public Officer.OfficerBuilder age(int age) {
            this.age = age;
            return this;
        }

        public Officer.OfficerBuilder department(String department) {
            this.department = department;
            return this;
        }
        //再看build方法，它是OfficerBuilder类的方法。它创建了一个新的Officer对象，并将自身的成员变量值，传给了Officer的成员变量
        public Officer build() {
            return new Officer(this.id, this.name, this.age, this.department);
        }

        public String toString() {
            return "Officer.OfficerBuilder(id=" + this.id + ", name=" + this.name + ", age=" + this.age + ", department=" + this.department + ")";
        }
    }
}
```

> 为什么这种模式叫“构建器”，因为要创建Officer类的实例,首先要创建OfficerBuilder类的实例。而这个OfficerBuilder也就是构建器，是创建Officer对象的一个过渡者。所以利用这种模式，会有中间实例的创建，会加大虚拟机内存的消耗
