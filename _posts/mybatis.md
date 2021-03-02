------

# 插入 更新 不推荐使用base sql 

# 查询可以使用

@Param

resultType

基础类型

resultMap

模型

# $与#

\#带引号，$不带

# map

```
<select id="queryNginxDomain" resultType="java.util.HashMap">
    select
    <include refid="Value_Column_List"/>
    from moni_dimension_relation
    where source = #{source}
    <if test="null != dimensionRelations and dimensionRelations.size() > 0">
        and
        <foreach collection="dimensionRelations.entrySet()" item="dimensionRelation" index="key" separator="and" open="(" close=")">
            ${key} in
            <foreach collection="dimensionRelations[key]" item="item" index="index" separator="," open="(" close=")">
                #{item}
            </foreach>
        </foreach>
    </if>
</select>
```



# collection

一对多查询：

只查询一次

```
<!-- 嵌套结果集的方式，使用collection标签定义关联的集合类型的属性封装规则 -->
<resultMap type="com.atguigu.mybatis.bean.Department" id="MyDept">
    <id column="did" property="id"/>
    <result column="dept_name" property="departmentName"/>
    <!--
    collection定义关联结合类型的属性的封装规则
    ofType：指定集合里面元素的类型
    -->
    <collection property="emps" ofType="com.atguigu.mybatis.bean.Employee">
        <!-- 定义这个集合中元素的封装规则 -->
        <id column="eid" property="id"/>
        <result column="last_name" property="lastName"/>
        <result column="email" property="email"/>
        <result column="gender" property="gender"/>
    </collection>
 
</resultMap>
 
 
<!-- public Department getDeptByIdPlus(Integer id); -->
<select id="getDeptByIdPlus" resultMap="MyDept" >
    SELECT
        d.id did,
        d.dept_name dept_name,
        e.id,
        e.last_name last_name,
        e.email email,
        e.gender gender
    FROM
        tbl_dept d
    LEFT JOIN tbl_employee e ON d.id = e.d_id
    WHERE
        d.id = #{id}
</select>
```

![img](https://img-blog.csdn.net/20180306212140955)

多次查询（不推荐）：

```
<resultMap type="com.atguigu.mybatis.bean.Department" id="MyDeptStep">
        <id column="id" property="id"/>
        <id column="dept_name" property="departmentName"/>
        <collection property="emps"
            select="com.atguigu.mybatis.dao.EmployeeMapperPlus.getEmpsByDeptId"
            column="{deptId=id}" fetchType="lazy"></collection>
</resultMap>
    <!-- public Department getDeptByIdStep(Integer id); -->
    <select id="getDeptByIdStep" resultMap="MyDeptStep">
        select id,dept_name departmentName from tbl_dept where id=#{id}
    </select>
```

![img](https://img-blog.csdn.net/201803062135397)

# ON DUPLICATE KEY UPDATE 

设置不重复键：

```
INSERT INTO tasks(task_id,subject,start_date,end_date,description)
VALUES (4,'Test ON DUPLICATE KEY UPDATE','2017-01-01','2017-01-02','Next Priority')
ON DUPLICATE KEY UPDATE
   task_id = task_id + 1,
   subject = 'Test ON DUPLICATE KEY UPDATE';//原文出自【易百教程】，商业转载请联系作者获得授权，非商业转载请保留原文链接：https://www.yiibai.com/mysql/insert-statement.html

```