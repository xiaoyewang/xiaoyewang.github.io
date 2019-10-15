### Athena JDBC 连接 Java sample 代码

Athena JDBC 使用官方文档 ： http://docs.aws.amazon.com/zh_cn/athena/latest/ug/connect-with-jdbc.html
>示例代码使用了第三方 maven  仓库，仅供测试参考  https://maven.atlassian.com/3rdparty/
>AthenaJDBC : https://mvnrepository.com/artifact/com.amazonaws.athena.jdbc/atl-athena-jdbc-driver    
目前发现credential provider bug,  .aws/credentials 优先于代码层配置的user/password，workaround:  在开发环境安装AWS CLI  配置好
> aws-java-sdk : https://mvnrepository.com/artifact/com.amazonaws/aws-java-sdk
>To Do : sample code with the athena sdk  https://mvnrepository.com/artifact/com.amazonaws/aws-java-sdk-athena


maven  代码  pom.xml

```
<repositories>
    <repository>
         <id>atlassian</id>
        <url> https://maven.atlassian.com/3rdparty/</url>
    </repository>
</repositories>


<dependencies>
    <!-- https://mvnrepository.com/artifact/com.amazonaws.athena.jdbc/atl-athena-jdbc-driver -->
    <dependency>
        <groupId>com.amazonaws.athena.jdbc</groupId>
        <artifactId>atl-athena-jdbc-driver</artifactId>
        <version>1.0.2-atlassian-1</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/com.amazonaws/aws-java-sdk -->
    <dependency>
        <groupId>com.amazonaws</groupId>
        <artifactId>aws-java-sdk</artifactId>
        <version>1.11.161</version>
    </dependency>

</dependencies>
```

Java 代码 ：


```
import java.sql.*;
import java.util.Properties;

import com.amazonaws.athena.jdbc.AthenaDriver;
import com.amazonaws.auth.DefaultAWSCredentialsProviderChain;


public class AthenaJDBCTest {

    static final String athenaUrl = "jdbc:awsathena://athena.us-west-2.amazonaws.com:443";

    public static void main(String[] args) {

        Connection conn = null;
        Statement statement = null;

        try {
            Class.forName("com.amazonaws.athena.jdbc.AthenaDriver");
            Properties info = new Properties();

            info.put("user", "AWSAccessKey");
            info.put("password", "AWSSecretAccessKey");
            //  staging query result goes here, must use a s3 bucket in the same region
            info.put("s3_staging_dir", "s3://aws-athena-query-results-076538427463-us-west-2/");
            info.put("aws_credentials_provider_class","com.amazonaws.auth.DefaultAWSCredentialsProviderChain");
 /* another credentials provider

            info.put("s3_staging_dir", "s3://aws-athena-query-results-076538427463-us-west-2");
            info.put("log_path", "/Users/myUser/.athena/athenajdbc.log");
            info.put("aws_credentials_provider_class", "com.amazonaws.auth.PropertiesFileCredentialsProvider");
            info.put("aws_credentials_provider_arguments", "/Users/myUser/.athenaCredentials");
*/
            String databaseName = "sampledb";

            //  Demo database     sampledb
            //  Demo table  elb_logs
            //  Demo table location  s3://athena-examples-us-west-2/elb/plaintext



            System.out.println("Connecting to Athena...");
            conn = DriverManager.getConnection(athenaUrl, info);

            System.out.println("Listing tables...");
            String sql = "show tables in " + databaseName;
            statement = conn.createStatement();
            ResultSet rs = statement.executeQuery(sql);

            while (rs.next()) {
                //Retrieve table column.
                String name = rs.getString("tab_name");

                //Display values.
                System.out.println("Name: " + name);
            }

            //select query sample
            sql = "select * from sampledb.elb_logs limit 5";
            rs = statement.executeQuery(sql);
            ResultSetMetaData rsmd = rs.getMetaData();
            int columnsNumber = rsmd.getColumnCount();
            while (rs.next()) {
                for (int i = 1; i <= columnsNumber; i++) {
                    if (i > 1) System.out.print(",  ");
                    String columnValue = rs.getString(i);
                    System.out.print(columnValue + " " + rsmd.getColumnName(i));
                }
                //Display values.
                System.out.println("");
            }


            rs.close();
            conn.close();
        } catch (Exception ex) {
            ex.printStackTrace();
        } finally {
            try {
                if (statement != null)
                    statement.close();
            } catch (Exception ex) {

            }
            try {
                if (conn != null)
                    conn.close();
            } catch (Exception ex) {

                ex.printStackTrace();
            }
        }
        System.out.printf("Finished connectivity test.");
    }


}
```




