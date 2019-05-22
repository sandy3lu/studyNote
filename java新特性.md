# 接口默认实现default
```java
/**
    定义接口
*/
interface DAO {
    default int getInt(){
        return 5;
    }
}

/**
 *  接口实现类
 * */
class BasicDAO implements DAO{

    public int getIntDefaultValue(){
        return getInt();
    }
}

/**
 * 测试类
 */
public class Test {

    public static void main(String[] args) {
        BasicDAO basicDAO = new BasicDAO();
        //结果为 5 
        System.out.println(basicDAO.getIntDefaultValue());
    }
}


/**
 *  接口实现类
 * */
class BasicDAO implements DAO{

    public int getIntDefaultValue(){
        return getInt();
    }

     @Override
    public int getInt(){
        return 88;
    }
}


/**
 * 测试类
 */
public class Test {

    public static void main(String[] args) {
        BasicDAO basicDAO = new BasicDAO();
        //结果为 88 
        System.out.println(basicDAO.getIntDefaultValue());
    }
}
```




