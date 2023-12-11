# 多模組管理
當您的功能繁雜，可能需要把某些功能分割出來，例如我們把程式分割為服務與界面兩個模組，此時必然有部份資料結構是共同的情況下，我們必須把這些共用的部分抽離獨立成為一個單獨的Jar,然後被這兩個模組所共用使用；或是開發EJB程式會同時包含多個程式。

以第一個例子來看，目錄架構可能如下：
```
 MyWeb
 ├pom.xml
 ├Datas
 │├pom.xml
 │└src...
 ├Service
 │├pom.xml
 │└src...
 └UI
   ├pom.xml
   └src...
```
而最頂端(MyWeb)的pom.xml負慣管理，共同的組件，例如：引用的Dependencies，Pluings …等等，
大緻如下：
```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.company</groupId>
  <artifactId>MyWeb</artifactId>
  <version>1.0</version>
  <packaging>pom</packaging>
  <name>MyWeb模組餐理</name>
  <modules>
    <module>Datas</module>
    <module>Service</module>
    <module>UI</module>
  </modules>
  <properties>
    ...
  </properties>
  <dependencyManagement><!--這裡先訂義相依Dependencies的版本，以避免各自模組引用到不用版本
    (當然，若因部署環境需要用到不同版本的的Dependencies，那就到子轉案的pom.xml設定) -->
    <dependencies>
      <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>2.22.0</version>
      </dependency>
      <dependency>
        ...
      </dependency>
    </dependencies>
  </dependencyManagement>
  <build>
    <pluginManagement><!--同樣，plugin也可以共同在此管理版本的設定，當然，也可以留給讓模組自已去訂義-->
      <plugins>
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.11.0</version>
          <configuration>
            <compilerArgs>
              <arg>-parameters</arg>
            <compilerArgs>
          </configuration>
        </plugin>
        <plugin>
          ...
        </plugin>
      </plugins>
    </pluginManagement>
  </build>
</project>
```
Datas pom.xml
```xml
<pom>
  <modelVersion>4.0.0</modelVersion>
  <parent> <!-- 指定父模組 -->
    <groupId>com.company</groupId>
    <artifactId>MyWeb</artifactId>
    <version>1.0</version>
  </parent>
  <artifactId>Datas</artifactId>
  <packaging>jar</packaging>
  <name>共用資料結構</name>
  <dependencies>
    <dependency><!--父模組有設定，我們只需聲明引用就可以了-->
      <groupId>org.apache.logging.log4j</groupId>
      <artifactId>log4j-core</artifactId>
    </dependency>
  </dependencies>
</pom>
```
Service與UI的 pom.xml大概都長得差不多
```xml
<pom>
  <modelVersion>4.0.0</modelVersion>
  <parent> <!-- 指定父模組 -->
    <groupId>com.company</groupId>
    <artifactId>MyWeb</artifactId>
    <version>1.0</version>
  </parent>
  <artifactId>UI</artifactId>
  <packaging>war</packaging>
  <name>前端界面</name>
  <dependencies>
    <dependency><!-- 引用共用資料結構的模組 -->
      <groupId>${project.groupId}</groupId>
      <artifactId>Datas</artifactId>
      <version>${project.version}</version>
    </dependency>
    <dependency><!--父模組有設定，我們只需聲明引用就可以了-->
      <groupId>org.apache.logging.log4j</groupId>
      <artifactId>log4j-core</artifactId>
    </dependency>
  </dependencies>
</pom>
```

當我們下指令 `mvn package` 時，每個模組都會依序走完各自的生命週期。
