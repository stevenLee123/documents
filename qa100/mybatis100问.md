## jdbc用法
使用jdbc连接mysql
```java
package com.steven.jdbc;


import lombok.extern.slf4j.Slf4j;

import java.sql.*;

/**
 * @Description:
 * @CreateDate: Created in 2024/1/4 16:38
 * @Author: lijie3
 */
@Slf4j
public class JDBCDemo {

    public static void main(String[] args) {
        //数据库连接
        Connection connection = null;
        //sql执行工具
        PreparedStatement preparedStatement = null;
        //结果集
        ResultSet resultSet = null;

        try {

            //注册驱动
            DriverManager.registerDriver(new com.mysql.cj.jdbc.Driver());
            //获取连接
            String url = "jdbc:mysql://127.0.0.1:3306/local_test?serverTimezone=UTC&useUnicode=true&zeroDateTimeBehavior=convertToNull&autoReconnect=true&characterEncoding=UTF-8&rewriteBatchedStatements=true";
            String user = "root";
            String password = "dxy123456";
            connection = DriverManager.getConnection(url,user,password);
            //获取数据库操作对象
            String sql  = "select * from user where id = ?";
            preparedStatement  = connection.prepareStatement(sql);
            preparedStatement.setInt(1,1);
            //获取结果集并解析
            resultSet = preparedStatement.executeQuery();
            if(resultSet.next()){
                log.info("id:{}",resultSet.getInt(1));
                log.info("name:{}",resultSet.getString("name"));
            }

        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            //关闭资源
            try {
                if(resultSet !=null){
                    resultSet.close();
                }
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
            try {
                if(preparedStatement != null){
                    preparedStatement.close();
                }
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
            try {
                if(connection != null){
                    connection.close();
                }
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }

    }
}
```

## mybatis的核心功能
将jdbc进行封装，用户只需要编写sql，其他的工作都交给mybatis来实现