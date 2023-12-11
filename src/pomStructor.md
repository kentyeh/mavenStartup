# Maven 的Project管理

Maven的管理設定主要靠Pom.xml進行，打開剛才建立的專案設定檔，內容大致說明如下(跟之前建立的pom.xml不盡完全相同)：

``` xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <!--本專案的識別-->
  <groupId>idv.kentyeh.software</groupId>
  <artifactId>firstmaven</artifactId>
  <version>1.0-SNAPSHOT</version>

  <!--表示打包專案的型態,可能為Jar、war、ear或pom，若是使用了android 則應為apk，
  若無此元素則預設為jar-->
  <packaging>jar</packaging>

  <!--以下是給工具看的,主要是本專案的顯示資訊，可以省略-->
  <name>firstmaven</name>

  <!--設定一些變數-->
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <maven.compiler.source>21</maven.compiler.source>
    <maven.compiler.target>21</maven.compiler.target>
    <!--也可以自行設定變數-->
    <surefire.version>3.2.2</surefire.version>
  </properties>

  <!--設定引用函式庫-->
  <dependencies>
    <dependency>
      <groupId>org.testng</groupId>
      <artifactId>testng</artifactId>
      <version>7.8.0</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <!--專案建置的相關設定-->
  <build>
    <finalName>${project.artifactId}</finalName>
    <resources>
      <resource>
        <directory>${project.basedir}/src/main/resources</directory>
        <filtering>true</filtering>
      </resource>
    </resources>
    <testResources>
      <testResource>
        <directory>${project.basedir}/src/test/resources</directory>
        <filtering>true</filtering>
      </testResource>
    </testResources>
    <!--Plugin 預設的識別管理-->
    <pluginManagement>
      <plugins>
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.11.0</version>
          <configuration>
            <source>${maven.compiler.source}</source>
            <target>${maven.compiler.target}</target>
            <showDeprecation>true</showDeprecation>
            <verbose>false</verbose>
            <compilerArgs>
              <arg>-parameters</arg>
            </compilerArgs>
          </configuration>
        </plugin>
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-surefire-plugin</artifactId>
          <version>${surefire.version}</version>
        </plugin>
      </plugins>
    </pluginManagement>
    <!--專案建置時使用的外掛-->
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <configuration>
          <!--外掛設定-->
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```
首先要說明的是dependencies段落，比如說，Project內會用到 testng 的Library 來進行測試，我們可以在此加入新的dependency段落，
``` xml
  <dependency>
    <groupId>org.testng</groupId>
    <artifactId>testng</artifactId>
    <version>7.8.0</version>
    <scope>test</scope>
  </dependency>
```
上述除了專案識別外，多了一個scope，其值說明如下：

| compile| 預設值，可以不填，表示此專案需要這個Library才能運作，所以打包程式時需要一併被打包 |
| --- | --- |
| provided | 表示編譯會用到，但是系統或容器在需要的時候會提供，打包專案不要含進去，例如J2EE的Library，像是servlet-api，就是由App Server(例如Tomcat)所提供 |
| runtime | 表示編譯時用不到，只有執行時會用到，所以發佈程式時須要一併打包，如JDBC各種資料庫的驅動程式 |
| test | 只有在單元測試時會用到，執行時期的程式時並不會用到，所以不會被打包 |
| <s>system</s> | <s>與provided相似，但是固定存在系統檔案,須以 systemPath 指定路徑</s>  |

*註*:強烈建議不要用system scope，有些專屬函式，像DB2的JDBC Driver db2jcc4.jar，可以用下列指令指定識別並將檔案加入到本地端的函式庫。

```shell
mvn install:install-file -Dfile=db2jcc4.jar \
    -DgroupId=com.ibm.db2 -DartifactId=db2jcc4 -Dversion=4.14.111 \
    -Dpackaging=jar
```

<b>&lt;finalName&gt;</b>是指定專案最後產出的打包名稱，若不指定則最後的打包檔案會附加版本資訊。

<b>&lt;resources&gt;</b>主要是進行資源管理。下節再進行說明。

<b>&lt;plugins&gt;</b>是指定專案中會用到的一些外掛，Maven主要目的是進行專案管理，但要完成專案建置則必須協同許多外部程式一同進行。

例如單元測試時就會用到<b>maven-surefire-plugin</b>您也許注意到了，在<b>&lt;plugins&gt;&lt;plugin&gt;</b>下的<b>maven-surefire-plugin</b>的識別管理沒有指定版本，
照理來說識別管理是少不了宣告<b>&lt;version&gt;</b>，那是因為上述多了一個<b>&lt;pluginManagement&gt;</b>段落已經預先宣告了這個外掛，所以不用特別再宣告版本，那麼為什麼會有這個<b>&lt;pluginManagement&gt;</b>段落呢?

<b>&lt;pluginManagement&gt;</b>通常用於多模組管理時，要保持外掛的一致性，允許各模組進設不同設定，但不允這各自選用不同版本。像上述範例中的<b>maven-compiler-plugin</b>已經做了相關的編譯設定，各模組編譯專案時無需再進行個別設定。

也就是說不複雜的專案，<b>&lt;pluginManagement&gt;</b>段落是不必要的。

*註1*: Maven規定：當&lt;groupId&gt;為**org.apache.maven.plugins**時，可以省略不寫。

*註2*: 同理，在<b>&lt;project&gt;</b>下也可以有<b>&lt;dependencyManagement&gt;</b>標籤為多模組專案，進行全面的的**depenbdencies**管控。
