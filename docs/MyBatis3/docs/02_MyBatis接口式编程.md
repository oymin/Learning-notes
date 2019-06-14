## 1. 创建接口

```
package com.koax.mybatis.dao;

import com.koax.mybatis.bean.Employee;

public interface EmployeeMapper {
	
	//根据id查询
	public Employee getEmpById(Integer id);
}
```

## 2. mapper.xml 配置

> 指定 `namespace` 的接口全限定名

```
<mapper namespace="com.koax.mybatis.dao.EmployeeMapper">
```

> 标签绑定接口方法

```
    <select id="getEmpById" resultType="com.koax.mybatis.bean.Employee">
		select id,last_name lastName,email,gender from tbl_employee where id = #{id}
	</select>
```

## 测试

```
class MyBatisTest {
	
	/**
	 * 测试 MyBatis 接口式编程
	 * @throws IOException
	 * @author oym
	 * @time 2019年4月17日 下午10:22:16
	 */
	@Test
	public void testInterface() throws IOException {
		SqlSession sqlSession = getSqlSessionFactory().openSession();
		try {
			//获取接口的实现类对象
            //会为接口自动的创建一个代理对象，代理对象取执行增删改查
			EmployeeMapper mapper = sqlSession.getMapper(EmployeeMapper.class);
			Employee employee = mapper.getEmpById(1);
			System.out.println(employee);
		} finally {
			sqlSession.close();
		}
	}

}
```

> `sqlSeesion` 和 `connection` 都是非线程安全的