# SpringBoot整合screw生成数据库文档



## 简介

> screw (螺丝钉) 是一个简洁好用的数据库表结构文档生成工具，
>
> 使用插件或配置的方式以应对频繁设计和更改数据库造成的没有文档或文档未及时更新的困扰。



## 特点

- 简洁、轻量、设计良好
- 多数据库支持
- 多种格式文档
- 灵活扩展
- 支持自定义模板



## 数据库支持

- [x]  MySQL

- [x] MariaDB

- [x] TIDB

- [x] Oracle

- [x] SqlServer

- [x] PostgreSQL

- [x] Cache DB（2016）

- [ ] H2 （开发中）

- [ ] DB2 （开发中）

- [ ] HSQL （开发中）

- [ ] SQLite（开发中）

- [ ] 瀚高（开发中）

- [ ] 达梦 （开发中）

- [ ] 虚谷 （开发中）

- [ ] 人大金仓（开发中）



## 文档生成支持

- [x] html
- [x] word
- [x] markdown



## 文档截图

![HTML](..\picture\161414_74cd0b68_1407605.png)

![screw-logo](..\picture\161723_6da58c41_1407605.png)

## 引入依赖

```java
<dependency>
    <groupId>cn.smallbun.screw</groupId>
    <artifactId>screw-core</artifactId>
    <version>1.0.5</version>
</dependency>
```



## 配置代码

配置输出路径，数据库连接，文件名

运行以下方法即可生成文档 或者也可以使用插件生成

```java
public static void main(String[] args) {
    //数据源
    HikariConfig hikariConfig = new HikariConfig();
    hikariConfig.setDriverClassName("org.postgresql.Driver");
    hikariConfig.setJdbcUrl("url");
    hikariConfig.setUsername("用户名");
    hikariConfig.setPassword("密码");
    //指定数据库下面的哪个模式
    //hikariConfig.setSchema("hd");
    //设置可以获取tables remarks信息
    hikariConfig.addDataSourceProperty("useInformationSchema", "true");
    hikariConfig.setMinimumIdle(2);
    hikariConfig.setMaximumPoolSize(5);
    DataSource dataSource = new HikariDataSource(hikariConfig);
    //生成配置
    EngineConfig engineConfig = EngineConfig.builder()
            //生成文件路径
            .fileOutputDir("D:")
            //打开目录
            .openOutputDir(true)
            //文件类型
            .fileType(EngineFileType.WORD)
            //生成模板实现
            .produceType(EngineTemplateType.freemarker)
            //自定义文件名称
            .fileName("iot_hd_数据库设计文档").build();

    //忽略表
    ArrayList<String> ignoreTableName = new ArrayList<>();
    ignoreTableName.add("test_user");
    ignoreTableName.add("test_group");
    //忽略表前缀
    ArrayList<String> ignorePrefix = new ArrayList<>();
    ignorePrefix.add("test_");
    //忽略表后缀
    ArrayList<String> ignoreSuffix = new ArrayList<>();
    ignoreSuffix.add("_test");
    ProcessConfig processConfig = ProcessConfig.builder()
            //指定生成逻辑、当存在指定表、指定表前缀、指定表后缀时，将生成指定表，其余表不生成、并跳过忽略表配置
            //根据名称指定表生成
            .designatedTableName(new ArrayList<>())
            //根据表前缀生成
            .designatedTablePrefix(new ArrayList<>())
            //根据表后缀生成
            .designatedTableSuffix(new ArrayList<>())
            //忽略表名
            .ignoreTableName(ignoreTableName)
            //忽略表前缀
            .ignoreTablePrefix(ignorePrefix)
            //忽略表后缀
            .ignoreTableSuffix(ignoreSuffix).build();
    //配置
    Configuration config = Configuration.builder()
            //版本
            .version("1.0.0")
            //描述
            .description("数据库设计文档生成")
            //数据源
            .dataSource(dataSource)
            //生成配置
            .engineConfig(engineConfig)
            //生成配置
            .produceConfig(processConfig)
            .build();
    //执行生成
    new DocumentationExecute(config).execute();
}
```





## Maven插件

```java
<build>
    <plugins>
        <plugin>
            <groupId>cn.smallbun.screw</groupId>
            <artifactId>screw-maven-plugin</artifactId>
            <version>${lastVersion}</version>
            <dependencies>
                <!-- HikariCP -->
                <dependency>
                    <groupId>com.zaxxer</groupId>
                    <artifactId>HikariCP</artifactId>
                    <version>3.4.5</version>
                </dependency>
                <!--mysql driver-->
                <dependency>
                    <groupId>mysql</groupId>
                    <artifactId>mysql-connector-java</artifactId>
                    <version>8.0.20</version>
                </dependency>
            </dependencies>
            <configuration>
                <!--username-->
                <username>root</username>
                <!--password-->
                <password>password</password>
                <!--driver-->
                <driverClassName>com.mysql.cj.jdbc.Driver</driverClassName>
                <!--jdbc url-->
                <jdbcUrl>jdbc:mysql://127.0.0.1:3306/xxxx</jdbcUrl>
                <!--生成文件类型-->
                <fileType>HTML</fileType>
                <!--打开文件输出目录-->
                <openOutputDir>false</openOutputDir>
                <!--生成模板-->
                <produceType>freemarker</produceType>
                <!--文档名称 为空时:将采用[数据库名称-描述-版本号]作为文档名称-->
                <fileName>测试文档名称</fileName>
                <!--描述-->
                <description>数据库文档生成</description>
                <!--版本-->
                <version>${project.version}</version>
                <!--标题-->
                <title>数据库文档</title>
            </configuration>
            <executions>
                <execution>
                    <phase>compile</phase>
                    <goals>
                        <goal>run</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```