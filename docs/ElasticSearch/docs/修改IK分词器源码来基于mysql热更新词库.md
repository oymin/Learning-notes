# 修改 IK 分词器源码来基于 mysql 热更新词库

手动下载安装：

```bash
git clone https://github.com/medcl/elasticsearch-analysis-ik
cd elasticsearch-analysis-ik
git checkout tags/{version} 
mvn clean
mvn compile
mvn package
```

## 修改源码

修改类 `org.wltea.analyzer.dic.Dictionary` ：

```java
private void loadMainDict() {
    ....
    // 添加以下内容
    // 加载 mysql 词典
    this.loadMySQLExtDict();
}

// 添加加载mysql驱动
static {
    try {
        Class.forName("com.mysql.jdbc.Driver");
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    }
}

/**
* 从mysql加载热更新词典
*/
private void loadMySQLExtDict() {
    Connection conn = null;
    Statement stmt = null;
    ResultSet rs = null;

    try {
        Path file = PathUtils.get(getDictRoot(), "jdbc-reload.properties");
        prop.load(new FileInputStream(file.toFile()));

        logger.info("[==========]jdbc-reload.properties");
        for (Object key : prop.keySet()) {
            logger.info("[==========]" + key + "=" + prop.getProperty(String.valueOf(key)));
        }

        logger.info("[==========]query hot dict from mysql, " + prop.getProperty("jdbc.reload.sql") + "......");

        conn = DriverManager.getConnection(
                prop.getProperty("jdbc.url"),
                prop.getProperty("jdbc.user"),
                prop.getProperty("jdbc.password"));
        stmt = conn.createStatement();
        rs = stmt.executeQuery(prop.getProperty("jdbc.reload.sql"));

        while (rs.next()) {
            String theWord = rs.getString("word");
            logger.info("[==========]hot word from mysql: " + theWord);
            _MainDict.fillSegment(theWord.trim().toCharArray());
        }

        Thread.sleep(Integer.valueOf(String.valueOf(prop.get("jdbc.reload.interval"))));
    } catch (Exception e) {
        logger.error("erorr", e);
    } finally {
        if (rs != null) {
            try {
                rs.close();
            } catch (SQLException e) {
                logger.error("error", e);
            }
        }
        if (stmt != null) {
            try {
                stmt.close();
            } catch (SQLException e) {
                logger.error("error", e);
            }
        }
        if (conn != null) {
            try {
                conn.close();
            } catch (SQLException e) {
                logger.error("error", e);
            }
        }
    }
}
```

```java
private void loadStopWordDict() {
    //从 mysql 加载词典
    this.loadMySQLStopwordDict();
}

/**
 * 从mysql加载停用词
 */
private void loadMySQLStopwordDict() {
    Connection conn = null;
    Statement stmt = null;
    ResultSet rs = null;

    try {
        Path file = PathUtils.get(getDictRoot(), "jdbc-reload.properties");
        prop.load(new FileInputStream(file.toFile()));

        logger.info("[==========]jdbc-reload.properties");
        for(Object key : prop.keySet()) {
            logger.info("[==========]" + key + "=" + prop.getProperty(String.valueOf(key)));
        }

        logger.info("[==========]query hot stopword dict from mysql, " + prop.getProperty("jdbc.reload.stopword.sql") + "......");

        conn = DriverManager.getConnection(
                prop.getProperty("jdbc.url"),
                prop.getProperty("jdbc.user"),
                prop.getProperty("jdbc.password"));
        stmt = conn.createStatement();
        rs = stmt.executeQuery(prop.getProperty("jdbc.reload.stopword.sql"));

        while(rs.next()) {
            String theWord = rs.getString("word");
            logger.info("[==========]hot stopword from mysql: " + theWord);
            _StopWords.fillSegment(theWord.trim().toCharArray());
        }

        Thread.sleep(Integer.valueOf(String.valueOf(prop.get("jdbc.reload.interval"))));
    } catch (Exception e) {
        logger.error("erorr", e);
    } finally {
        if(rs != null) {
            try {
                rs.close();
            } catch (SQLException e) {
                logger.error("error", e);
            }
        }
        if(stmt != null) {
            try {
                stmt.close();
            } catch (SQLException e) {
                logger.error("error", e);
            }
        }
        if(conn != null) {
            try {
                conn.close();
            } catch (SQLException e) {
                logger.error("error", e);
            }
        }
    }
}
```

**创建类调用加载方法：**

```java
package org.wltea.analyzer.dic;

import org.apache.logging.log4j.Logger;
import org.wltea.analyzer.help.ESPluginLoggerFactory;

public class HotDictReloadThread implements Runnable {

    private static final Logger logger = ESPluginLoggerFactory.getLogger(HotDictReloadThread.class.getName());

    @Override
    public void run() {
        while (true) {
            logger.info("[==========]reload hot dict from mysql......");
            Dictionary.getSingleton().reLoadMainDict();
        }
    }

}
```

**`config/jdbc-reload.properties` 配置文件**

```yml
jdbc.url=jdbc:mysql://localhost:3306/mytest?serverTimezone=GMT
jdbc.user=root
jdbc.password=root
jdbc.reload.sql=select word from hot_words
jdbc.reload.stopword.sql=select stopword as word from hot_stopwords
jdbc.reload.interval=10000
```

**maven 打包:**

```bash
mvn clean
mvn compile
mvn package
```

将 `target\releases\elasticsearch-analysis-ik-x.x.x.zip` 解压缩到 ES 的 `plugin/ik` 下。

将 mysql 驱动jar，放入 ik 的目录下, 重启es。




