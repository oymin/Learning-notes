## 1. 项目结构

```
├─conf
│      EmployeeMapper.xml
│      log4j.xml
│      mybatis-config.xml
│
├─lib
│      log4j.jar
│      mybatis-3.4.1.jar
│      mysql-connector-java-5.1.37-bin.jar
│
└─src
    └─com
        └─koax
            └─mybatis
                │  Employee.java
                │
                └─test
                        MyBatisTest.java
```

## 2. mybatis-config.xml 全局配置文件

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
 PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<environments default="development">
		<environment id="development">
			<transactionManager type="JDBC" />
			<dataSource type="POOLED">
				<property name="driver" value="com.mysql.jdbc.Driver" />
				<property name="url" value="jdbc:mysql://localhost:3306/mybatis" />
				<property name="username" value="root" />
				<property name="password" value="root" />
			</dataSource>
		</environment>
	</environments>
	<!-- 将我们写好的sql映射文件（EmployeeMapper.xml）一定要注册到全局配置文件（mybatis-config.xml）中 -->
	<mappers>
		<mapper resource="EmployeeMapper.xml" />
	</mappers>
</configuration>
```

## 3. mapper.xml sql映射文件
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
 PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.atguigu.mybatis.dao.EmployeeMapper">
<!-- 
	namespace:名称空间;指定为接口的全类名
	id：唯一标识
	resultType：返回值类型
	#{id}：从传递过来的参数中取出id值	
	public Employee getEmpById(Integer id);
 -->
	<select id="getEmpById" resultType="com.atguigu.mybatis.bean.Employee">
		select id,last_name lastName,email,gender from tbl_employee where id = #{id}
	</select>
</mapper>
```

## 3. Test 测试

```
class MyBatisTest {

	/**
	 * 1.根据xml配置文件（创建一个SqlSessionFactory）
	 *   使用SqlSessionFactory工厂获取到sqlSession对象执行增删改查操作
	 *   一个sqlSession就是代表和数据库的一次会话，用完关闭
	 * @throws IOException
	 * @author oym
	 * @time 2019年4月17日 下午8:40:51
	 */
	public SqlSessionFactory getSqlSessionFactory() throws IOException {
		String resource = "mybatis-config.xml";
		InputStream inputStream = Resources.getResourceAsStream(resource);
		return new SqlSessionFactoryBuilder().build(inputStream);
	}
	
	/**
	 * 2.获取sqlSeesion实例，能直接执行已经映射的sql语句
	 * @author oym
	 * @throws IOException 
	 * @time 2019年4月17日 下午9:23:50
	 */
	@Test
	public void test() throws IOException {
		SqlSessionFactory factory = getSqlSessionFactory();
		SqlSession sqlSession = factory.openSession();
		try {
			Employee employee = sqlSession.selectOne("com.koax.mybatis.dao.EmployeeMapper.getEmpById", 1);
			System.out.println(employee.toString());
		} finally {
			//最后关闭sqlSession会话
			sqlSession.close();
		}
	}

}
```




