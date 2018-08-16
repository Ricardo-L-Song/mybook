# DAO模式的使用

最近在上Servlet的时候，穿插讲了一些DAO设计模式，要求实现对学生表的CRUD，以下完整范例项目Students作为一个回顾。

#### 一.设计架构**原理**:

直接使用课件原图 ![image.png](https://upload-images.jianshu.io/upload_images/9003228-5d5196b03900db0e.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

完整的DAO模式其实横跨了BO,DAO,DataBase三层，层与层之间的调用以及访问通过接口访问，实现了分层设计，主要有以下几个类: **实体类javaBean**:是各层的底层媒介 通常存放在VO包下 **DataBaseConnection**: 数据层连接数据库驱动程序 数据层与数据库（DataBase）通信基础 通常存放在dbc包下 **IXXXDAO** :数据层与业务层接口 数据层与业务层通信基础 通常存放在DAO包下 **XXXDAOImpl**: 数据层实现类（继承IXXXDAO）具体实现CRUD操作 通常存放在impl包下 **DAOFactory**:数据层工厂类 业务层对数据层进行调用，必须有DAO包下的IXXXDAO接口，但不同层想取得接口对象实例，需要使用工厂设计模式 通常存放在factory包下 **IXXXService**: 业务层与其他层接口 实现业务层与数据层等通信基础 通常存放在service包下 XXXServiceImpl： 业务层实现类 通常放在impl包下 ServiceFactory ：业务层工厂类 用来获得业务层接口 供控制层调用 通常放在Factory包下

#### 二.代码**实现**

完整的项目结构如下: ![SO\[NZ\)UYGBMVS\]L~{AN\`\)@E.png](https://upload-images.jianshu.io/upload_images/9003228-837b33db71b97ea4.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

根据下图 我们从DataBase数据库开始开发 从后往前 ![CPJ5L$D3PG%}HH{52UU6}IP.png](https://upload-images.jianshu.io/upload_images/9003228-7e6e3875451b3f95.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240) 1.MySQL建student表 ![image.png](https://upload-images.jianshu.io/upload_images/9003228-ac8af68f9746ad17.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

1.Student.class 实体类映射表的字段

```text
package com.sl.vo;
//所有的javabean（简单java类保存在VO包中）
//对应数据库的表 映射 Value Object 简单java类的名称必须与表的名称对应 例如student_info对应StudentInfo Student对应Student

public class Student {
    private int idStudent;
    private String password;
    private int sex;
    private String credit;
    private String favourite;
    private String introduction;

    public int getIdStudent() {
        return idStudent;
    }

    public void setIdStudent(int idStudent) {
        this.idStudent = idStudent;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public int getSex() {
        return sex;
    }

    public void setSex(int sex) {
        this.sex = sex;
    }

    public String getCredit() {
        return credit;
    }

    public void setCredit(String credit) {
        this.credit = credit;
    }

    public String getFavourite() {
        return favourite;
    }

    public void setFavourite(String favourite) {
        this.favourite = favourite;
    }

    public String getIntroduction() {
        return introduction;
    }

    public void setIntroduction(String introduction) {
        this.introduction = introduction;
    }
}
```

2.DataBaseConnection:数据库驱动连接

```text
package com.sl.dbc;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.*;
//在本类的构造方法里要进行数据库的驱动加载和数据库的连接取得 不进行数据库操作
//数据层与数据库连接
/**
 *
 * @author sl
 */
public class DataBaseConnection {


    private static final String DBDRIVER="com.mysql.jdbc.Driver";
    private static final String DBURL="jdbc:mysql://localhost/mydb";//这里是Oracle数据库的写法
    private static final String DBUSER="root";
    private static final String PASSWORD="SL886886";
    private Connection conn=null;
    //在构造方法里为conn对象实例化，可以直接取得数据库的连接对象
    public DataBaseConnection() {
        try{
            //第一步 数据库驱动程序
        Class.forName(DBDRIVER);
            //第二部 数据库连接
        this.conn=DriverManager.getConnection(DBURL,DBUSER,PASSWORD);
            //第三步 进行数据库操作
//            Statement stmt=conn.createStatement();
//            String sql = "INSERT INTO Student(idStudent, password, sex, credit,favourite,introduction) "  + "VALUES(5,'556', 1,'体育', '音乐', '我是王五')";
//             int len=stmt.executeUpdate(sql);//执行sql 返回更新的行数
//             System.out.println("影响的行数"+len);
//            //第四步 关闭数据库
//            conn.close();
        }catch(Exception e){
        e.printStackTrace();
        }
    }
    //取得一个数据库的连接对象 返回一个Connection的实例对象
    public Connection getConnection(){
        return this.conn;
    }
    //负责数据库的关闭
    public void close(){
        if(this.conn!=null){//表示现在存在有我们的连接对象
            try{
                this.conn.close();
            }catch(SQLException e){
            e.printStackTrace();
            }
        }
    }
}
```

3.IStudentDAO:数据层接口

```text
package com.sl.dao1;
import com.sl.vo.Student;
import java.util.*;
//定义Student表数据层的操作标准（以接口形式）
/**
 *
 * @author sl
 */
//数据层与业务层通信标准 接口形式实现
public interface IStudentDAO {
    //数据的增加操作
    //VO包含了要增加数据的VO对象
    //增加成功返回true 否则返回false
    public boolean doCreate(Student vo) throws Exception;//doUpdata doEdlete等具体的增删改查
    //数据的修改操作 我们本次的修改是根据id进行全部数据的修改
    //VO包含了我们要修改数据的信息 一定要包含id内容
    //修改成功返回true 否则返回false
    public boolean doUpdate(Student vo) throws Exception;
    //数据的批量删除操作 所有要删除的数据以set集合形式保存
    //Ids包含了所要要删除的数据ID，不包含重复内容
    //删除成功返回true(删除的数据个数与传入的数据个数相同返回true)，否则返回false
    public boolean doRemoveBatch(Set<Integer> ids) throws Exception;
    //根据雇员/学生编号查询指定的雇员信息
    //id是要查询的雇员/学生编号
    //如果雇员信息存在 则将VO类以对象的形式返回 如果雇员信息不存在 则返回空
    public Student findById(Integer id) throws Exception;//如果雇员存在 则返回雇员，如果雇员不存在，则返回null
    //查询指定数据表的全部记录 并且以集合形式返回
    //如果表中有数据，那么数据库层的数据在数据层中就会封装成VO对象 利用List集合返回 如果没有数据 那么集合的长度为0（size()==0,而不是null） 
    public List<Student> findAll()throws Exception;
    //分页进行数据的模糊查询，结果以集合的形式返回
    //currentPage 当前所在的页
    //lineSize 每页显示的数据函数
    //column 要进行模糊查询的数据列
    //keyWord 模糊查询的关键字
    //如果表中有数据，那么数据库层的数据在数据层中就会封装成VO对象 利用List集合返回 如果没有数据 那么集合的长度为0（size()==0,而不是null） 
    public List<Student> findAllSplit(Integer currentPage,Integer lineSize,String column,String keyWord) 
            throws Exception;//如果表中有数据 则所有对象封装为VO对象装入List中返回

    //进行模糊查询数量的统计，如果表中没有记录 统计的结果就是0
     //column 要进行模糊查询的数据列
    //keyWord 模糊查询的关键字
    //返回表中的数据量 如果表中没有数据 则返回0
    public Integer getAllCount(String column,String keyWord)throws Exception;
}
```

1. StudentDAOImpl:数据层接口实现类

   \`\`\`

   package com.sl.dao.impl;

   //具体实现类

import com.sl.dao1.IStudentDAO; import com.sl.vo.Student; import java.sql.Connection; import java.sql.PreparedStatement; import java.sql.ResultSet; import java.util.ArrayList; import java.util.Iterator; import java.util.List; import java.util.Set;

/_\*_ 

* @author sl \*/ //数据层具体实现增删改查操作类，由于我们最好自己在业务层定义数据库开关 所以在数据层要有Connection类对象实参 具体的开关在业务层实现 传入Connection对象实参 //VO对象作为了数据层 业务层等等层的底层媒介 public class StudentDAOImpl implements IStudentDAO{//具体实现类 覆盖增删改查doCreat\(\),doUpdate等 private Connection conn; private PreparedStatement pstmt;

  public StudentDAOImpl\(Connection conn\) { //由外部传入Connection对象 this.conn = conn; } //增加 数据更新 @Override public boolean doCreate\(Student vo\) throws Exception {//vo只是作为了一个媒介 要想理解此次操作 还是看pstmt.setXXX是用来增加数据的 对应数据库的操作 String sql="INSERT INTO Student\(idStudent,password,sex,credit,favourite,introduction\) VALUES\(?,?,?,?,?,?\)"; this.pstmt=conn.prepareStatement\(sql\); pstmt.setInt\(1, vo.getIdStudent\(\)\); pstmt.setString\(2, vo.getPassword\(\)\); pstmt.setInt\(3, vo.getSex\(\)\); pstmt.setString\(4, vo.getCredit\(\)\); pstmt.setString\(5, vo.getFavourite\(\)\); pstmt.setString\(6, vo.getIntroduction\(\)\); return this.pstmt.executeUpdate\(\)&gt;0;//如果为真 就是执行了操作 返回true1 否则返回false0 } //更新 数据更新 @Override public boolean doUpdate\(Student vo\) throws Exception { String sql="UPDATE Student password=?,sex=?,credit=?,favourite=?,introduction=? WHERE idStudent=?"; this.pstmt=conn.prepareStatement\(sql\); pstmt.setString\(1, vo.getPassword\(\)\); pstmt.setInt\(2, vo.getSex\(\)\); pstmt.setString\(3, vo.getCredit\(\)\); pstmt.setString\(4, vo.getFavourite\(\)\); pstmt.setString\(5, vo.getIntroduction\(\)\); pstmt.setInt\(6, vo.getIdStudent\(\)\); return this.pstmt.executeUpdate\(\)&gt;0;//如果为真 就是执行了操作 返回true1 否则返回false0

  } //批量删除 数据更新 @Override public boolean doRemoveBatch\(Set ids\) throws Exception { if\(ids==null\|\|ids.size\(\)==0\){//如果没有要删除的对象id\(即表中idStudent\) return false; } StringBuffer sql=new StringBuffer\(\);//拼凑字符串 拼凑sql语句 sql.append\("DELETE FROM Student WHERE idStudent IN\("\);//append用来正式添加进拼凑的语句中 Iterator iter=ids.iterator\(\); while \(iter.hasNext\(\)\) { sql.append\(iter.next\(\)\).append\(","\);//添加进ids整型集合中的整型数字 每个数字用","隔开 } sql.delete\(ids.size\(\)-1, ids.size\(\)\);//删除最后一个逗号 sql.append\("\)"\);//加上IN（）的结尾小括号 this.pstmt=this.conn.prepareStatement\(sql.toString\(\)\);//toString将StringBuffer变为String return this.pstmt.executeUpdate\(\)==ids.size\(\);//删除的数量是否等于id集合的长度 } //根据id查询 数据查询 成功返回VO类对象 失败返回null @Override public Student findById\(Integer id\) throws Exception { Student vo=null; String sql="SELECT idStudent,password,sex,credit,favourite,introduction FROM Student WHRER idStudent=?"; this.pstmt=this.conn.prepareStatement\(sql\); this.pstmt.setInt\(1, id\);//根据传入的实参来删除对象 ResultSet rs=this.pstmt.executeQuery\(\);//对数据库进行查询 有结果返回 if\(rs.next\(\)\){//逐行读数据 虽然只有一行 vo=new Student\(\); vo.setIdStudent\(rs.getInt\(1\)\);//逐列读出 vo.setPassword\(rs.getString\(2\)\); vo.setSex\(rs.getInt\(3\)\); vo.setCredit\(rs.getString\(4\)\); vo.setFavourite\(rs.getString\(5\)\); vo.setIntroduction\(rs.getString\(6\)\); } return vo; } //查询全部数据 数据查询 @Override public List findAll\(\) throws Exception { List all=new ArrayList\(\); String sql="SELECT idStudent,password,sex,credit,favourite,introduction FROM Student"; this.pstmt=this.conn.prepareStatement\(sql\); ResultSet rs=this.pstmt.executeQuery\(\);//对数据库进行查询 有结果返回 while\(rs.next\(\)\){//逐行读数据 虽然只有一行 Student vo=new Student\(\);; vo.setIdStudent\(rs.getInt\(1\)\);//逐列读出 vo.setPassword\(rs.getString\(2\)\); vo.setSex\(rs.getInt\(3\)\); vo.setCredit\(rs.getString\(4\)\); vo.setFavourite\(rs.getString\(5\)\); vo.setIntroduction\(rs.getString\(6\)\); all.add\(vo\);//将从数据库返回的数据保存在底层媒介后 往泛型集合中加一个实例化对象 } return all; } //查询全部数据并分页 数据查询 @Override public List findAllSplit\(Integer currentPage, Integer lineSize, String column, String keyWord\) throws Exception { List all=new ArrayList\(\); String sql="SELECT \* FROM "

  * " SELECT idStudent,password,sex,credit,favourite,introduction FROM Student" +" WHERE "+column+" LIKE ? AND ROWNUM&lt;=?\) temp" +" WHERE temp.rn&gt;? " ; this.pstmt=this.conn.prepareStatement\(sql\); pstmt.setString\(1, "%"+keyWord+"%"\); pstmt.setInt\(2, currentPage_lineSize\); pstmt.setInt\(3, \(currentPage-1\)_lineSize\); ResultSet rs=this.pstmt.executeQuery\(\);//对数据库进行查询 有结果返回 while\(rs.next\(\)\){//逐行读数据 虽然只有一行 Student vo=new Student\(\);; vo.setIdStudent\(rs.getInt\(1\)\);//逐列读出 vo.setPassword\(rs.getString\(2\)\); vo.setSex\(rs.getInt\(3\)\); vo.setCredit\(rs.getString\(4\)\); vo.setFavourite\(rs.getString\(5\)\); vo.setIntroduction\(rs.getString\(6\)\); all.add\(vo\);//将从数据库返回的数据保存在底层媒介后 往泛型集合中加一个实例化对象 } return all; }

    @Override public Integer getAllCount\(String column, String keyWord\) throws Exception { String sql="SELECT COUNT\(idStudent\) FROM Student WHERE "+column+"LIKE ?"; this.pstmt=this.conn.prepareStatement\(sql\); this.pstmt.setString\(1, "%"+keyWord+"%"\); ResultSet rs=this.pstmt.executeQuery\(\); if\(rs.next\(\)\){ return rs.getInt\(1\); } return null; }

}

```text
5.DAOFactory：供业务层调用的工厂类
package com.sl.dao.factory;
```

/\*

* To change this license header, choose License Headers in Project Properties.
* To change this template file, choose Tools \| Templates
* and open the template in the editor.

  \*/

  import com.sl.dao.impl.StudentDAOImpl;

  import com.sl.dao1.IStudentDAO;

  import java.sql.Connection;

  /\*\*

  \*

* @author sl

  \*/

  //数据层工厂类 为了方便业务层对数据层进行操作 关闭开启数据库

  //使用工厂类的特征就是不需要知道具体的子类（第三方接口也是这样 只负责传参 不关心具体的实现类）即业务层是看不见具体的实现类StudentDAOImpl

  //除此之外的VO类，IStudentDAO，Connection等都能看见

  public class DAOFactory{//接口集合

   public static IStudentDAO getIStudentDAOInstance\(Connection conn\){//由外部传入形参 

  ```text
   return new StudentDAOImpl(conn);//双重形参
  ```

   }

}

```text
6.IStudentService：业务层通信基础 业务层接口
```

package com.sl.service;

import com.sl.vo.Student; import java.util.List; import java.util.Map; import java.util.Set;

/_\*_ 

* @author sl \*/ //业务层通信标准 业务层接口 并且继承此接口的类在业务层一定要负责数据库的打开和关闭操作 //此外继承此接口的类可以通过DAOFactory类取得IEmpDAO接口对象 //业务层能用int还是使用int型 不用包装类 便于区分 public interface IStudentService { //实现学生数据的增加操作 //本次操作调用IStudentDAO接口的如下方法 //调用IStudentDAO.findById\(\)方法看学生id是否存在 //如果不存在则调用IStudentDAO.doCreate\(\)方法增加数据 //@return 如果Id存在或者保存失败，返回false，否则返回true public boolean insert\(Student vo\)throws Exception;

  //实现学生数据的修改操作 //本次调用IStudentDAO的doUpdate\(\)方法 //@return 修改成功返回true,否则返回false public boolean update\(Student vo\)throws Exception;

  //实现学生数据的批量删除操作 //本次调用IStudentDAO中的doRemoveBatch方法 //@param ids包含了全部要删除数据的集合 没有重复数据 //return 删除成功返回true 否则返回false public boolean delete\(Set ids\)throws Exception;

  //根据学生的id编号 查找学生信息 //本次调用了IStudentDAO中的findById\(\)方法 //@param id表明要查找的学生编号 //@return 如果找到了学生信息 以Student vo类对象返回，否则返回null public Student get\(int ids\)throws Exception;

  ```text
  //查询全部学生信息 调用IStudentDAO.findAll()方法
  //@return 查询结果以list集合形式返回，如果没有数据，集合长度为0
  public List<Student> list() throws Exception;

  /**
   * 实现数据的模糊查询与数据统计
   * 要调用IStudentDAO.findAllSplit()方法，查询所有的表数据，返回的List<Student>
   * 调用IStudentDAO.getAllCount()方法，查询所有的数据量，返回Integer
   * @param currentPage 当前所在页
   * @param lineSize 每页显示的记录数目
   * @param column 模糊查询的数据列
   * @param keyWord 模糊查询的关键字
   * @return 本方法由于需要返回多种数据类型，所以使用Map集合返回，由于类型不同意，所以所有Value的类型设置为Object
   * 如果key=allStudents,value=IStudentDAO.findAllSplit()方法返回结果 List<Student>
   * 如果key=StudentCount value=IStudentDAO.getAllCount()返回结果 Integer
   * @throws Exception 
   */
  public Map<String,Object> list(int currentPage,int lineSize,String column,String keyWord) throws Exception;
  ```

  }

```text
7.StudentServiceImpl：业务层实现类
```

public class StudentServiceImpl implements IStudentService{ //数据库连接类，实例化以后就连接上了数据库 所以我们之后的方法 一定有finally 用来关闭数据库 this.dbc.close public DataBaseConnection dbc=new DataBaseConnection\(\); @Override public boolean insert\(Student vo\) throws Exception { try { //首先 由业务层传入的Connection对象负责开关 双重形参 并且返回的IStudentDAO对象调用其findById方法 看是否学生编号为空 if\(DAOFactory.getIStudentDAOInstance\(this.dbc.getConnection\(\)\).findById\(vo.getIdStudent\(\)\)==null\){ //学生id编号为空 我们就能进行增加操作了 以上以下为代码链 return DAOFactory.getIStudentDAOInstance\(this.dbc.getConnection\(\)\).doCreate\(vo\); } return false; } catch \(Exception e\) { throw e; }finally{ this.dbc.close\(\); } }

```text
@Override
public boolean update(Student vo) throws Exception {
 try {
        return DAOFactory.getIStudentDAOInstance(this.dbc.getConnection()).doUpdate(vo);
    } catch (Exception e) {
        throw e;
    }finally{
         this.dbc.close();
        }
}

@Override
public boolean delete(Set<Integer> ids) throws Exception {
 try {
        return DAOFactory.getIStudentDAOInstance(this.dbc.getConnection()).doRemoveBatch(ids);       
    } catch (Exception e) {
        throw e;
    }finally{
         this.dbc.close();
        }
}

@Override
public Student get(int ids) throws Exception {
 try {
        return DAOFactory.getIStudentDAOInstance(this.dbc.getConnection()).findById(ids);
    } catch (Exception e) {
        throw e;
    }finally{
         this.dbc.close();
        }
}

@Override
public List<Student> list() throws Exception {
 try {
       return DAOFactory.getIStudentDAOInstance(this.dbc.getConnection()).findAll();
    } catch (Exception e) {
        throw e;
    }finally{
         this.dbc.close();
        }
}

@Override
public Map<String, Object> list(int currentPage, int lineSize, String column, String keyWord) throws Exception {
 try {
        Map<String,Object> map=new HashMap<String,Object>();
        map.put("allStudents", DAOFactory.getIStudentDAOInstance(this.dbc.getConnection()).findAllSplit(currentPage, lineSize, column, keyWord));
        map.put("StudentCount", DAOFactory.getIStudentDAOInstance(this.dbc.getConnection()).getAllCount(column, keyWord));
        return map;
    } catch (Exception e) {
        throw e;
    }finally{
         this.dbc.close();
        }
}
```

}

```text
8.ServiceFactory:业务层工厂类
```

//业务层工厂类 用来获得业务层接口 public class ServiceFactory { public static IStudentService getIStudentServiceInstance\(\){ return new StudentServiceImpl\(\); } }

```text
### 三.代码**测试**:
写个增加数据的调用测试
```

package com.sl.test;

import com.sl.dao.factory.ServiceFactory; import com.sl.vo.Student;

/_\*_ 

* @author sl

  \*/

  public class TestStudentInsert {

   public static void main\(String\[\] args\) {

  ```text
   Student vo=new Student();
   vo.setIdStudent(3);
   vo.setPassword("567");
   vo.setSex(0);
   vo.setCredit("university");
   vo.setFavourite("PE");
   vo.setIntroduction("wangwu");
   try {//不同层操作 使用try catch
       System.err.println(ServiceFactory.getIStudentServiceInstance().insert(vo));
   } catch (Exception e) {
       e.printStackTrace();
   }
  ```

   }

}

\`\`\`

结果: ![KUXG\`9T939M2}XFYTI7B5DK.png](https://upload-images.jianshu.io/upload_images/9003228-6dbba8d4d3127f7e.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

搞定~

