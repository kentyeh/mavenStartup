# 目錄架構

以建立web程式來說，其目錄架構說明如下：

```
Project目錄
├src[程式碼目錄]
│ │
│ ├main[主要目錄]
│ │ │
│ │ ├java[java程式目錄]
│ │ │ │
│ │ │ └[idv.kentyeh.software....程式套件目錄]
│ │ │
│ │ ├resources[資源目錄,會copy到編譯路徑，以web來說，會依目錄層級放到WEB-INF/classes下]
│ │ │ │
│ │ │ └[各種資源(設定)檔...]
│ │ │
│ │ └webapp[web目錄]
│ │   │
│ │   └[其它資料…]
│ │
│ └test[測試相關目錄]
│   │
│   ├java[java測試程式目錄]
│   │ │
│   │ └[...]
│   │
│   └resources[測試資源目錄…]
│
└target[各種處理後生成的資料，包含最終生成的打包標的]
```
上述架構若不是Web專案．則無**webapp**目錄。

**resources**目錄負責存放靜態資訊，如設定檔、圖檔、字型檔等資源。

**main/resources**儲放主程式會用到資料，包裝專案時會一併被打包，裝進 target/classes 目錄下。

**test/resources**則只有在測試階段時才會被使用到，不會被打包，會生成入 target/test-classes 目錄下, 地位與/classes相當。

## 專案變數(&lt;properties&gt;)

我們建立的第一個 專案 的 [pom.xml](./pomStructor.html)檔案內有一個<properties>段落，可以讓我們定義一些變數， 例如Spring 通常含有多種Library，其引用版本應該一致，所以我通常會定義一個變數，然後在專案識別的版本中引用，以保持各個Spring函式版本的一致性。

``` xml
  <properties>
    <spring.version>6.0.12</spring.version>
  </properties>
```

引用範例

```xml
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>${spring.version}</version>
  </dependency>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context-support</artifactId>
    <version>${spring.version}</version>
  </dependency>
```

之前的[pom.xml](./pomStructor.html)也出現像

```xml
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  <maven.compiler.source>21</maven.compiler.source>
  <maven.compiler.target>21</maven.compiler.target>
```
又有什麼不同?

主要有3個層面，Maven本身內建的變數、外掛定義的變數與我們自訂的變數。

常用的Maven內建的變數，說明如下(另外網路上也有人整理出詳細的[列表](https://github.com/cko/predefined_maven_properties/blob/master/README.md))。

| | |
| --- | --- |
| ${project.basedir} | 表示包含pom.xml的目錄路徑 |
| ${project.version} | 程式的版本編號(maven通常建議不要直接使用${version}) |
| ${project.build.directory} | 就是target目錄 |
| ${project.build.outputDirectory} | 就是target/classes目錄 |
| ${project.build.testOutputDirectory}  | 就是target/test-classes目錄 |
| ${project.name} | 就是 pom.xml &lt;name&gt;所指定的名稱 |
| ${project.groupId} | 就是 pom.xml &lt;groupId&gt;所指定的名稱 |
| ${project.build.finalName} | Project的打包名稱 |
| ${java.home} | Java安裝目錄 |
| ${env.變數名} | 參考系統變數，例如有JAVA_HOME的系統變數，就可以直接參考${env.JAVA_HOME} |

另外一種就是各種外掛的定義變數。

我們打開**maven-compiler-plugin**的[說明頁](https://maven.apache.org/plugins/maven-compiler-plugin/compile-mojo.html#encoding)，

`${project.build.sourceEncoding}`就是javac編譯時使用的 -encoding 參數。

不意外，`${maven.compiler.source}`與`${maven.compiler.target}`則分別是javac編譯時使用的 -source 與 -target 參數。

## 資源管理

上一節提到<b>&lt;build&gt;</b>下有<b>&lt;resources&gt;</b>與<b>&lt;testResources&gt;</b>，其內部文件會分別拷貝到 /casses 或 /test-classes下。

但其實<b>&lt;resources&gt;</b>與<b>&lt;testResources&gt;</b>最重要的功能是對資料進行過慮。

以web.xml為例，設定檔內可以加入<b>&lt;display-name&gt;</b>以便war檔部署到Tomcat時，可以在Manager的管理頁面程現可讀性較強的文字。
這時過濾的功能就派上用場了。

我的web.xml有段設定如下：

```xml
<web-app version="6.0">
    <display-name>${project.name}</display-name>
</web-app>
```
註:若POM.xml內有引用spring-boot-maven-plugin，因為`application.properties`或`application.yml`也使用`${…}`，所以替代符被變更為`@…@`, 所以web.xml應該是

```xml
<web-app version="6.0">
    <display-name>@project.name@</display-name>
</web-app>
```

希望建置War檔時Maven能夠將之替換為專案名稱，但又為了建置時不要這個檔案被放到/classes裡面，所以我把這個檔放在 專案目錄/src/test/resources 內(這個目錄不會被打包)。
然後Maven進行資源處理時會被拷貝到 target/test-classes/ 下，但是一般正常的web.xml應該在WEB-INF下才對，所以我必須在POM內告訴打包war的外掛，正確的
web.xml所在。

```xml
  <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-war-plugin</artifactId><!--負責打包War的外掛-->
    <version>3.4.0</version>
    <configuration>
      <failOnMissingWebXml>false</failOnMissingWebXml>
      <webXml>${project.build.testOutputDirectory}/web.xml</webXml>
    </configuration>
  </plugin>
```
同樣的方式，若我有個system.properties檔案、或是一些Spring的設定檔，不想把一些參數寫死，也可以放在 src/main/resources 下讓Maven進行替換。

很多時後我們也會把字型檔，放在 src/man/resources 目錄下，然後一併被打包到 /classes 裡，但又怕陰錯陽差的被替換到內容，咋辦?

我們注意到<b>&lt;build&gt;</b>下的<b>&lt;resources&gt;</b>是複數，所以可設定如下：

```xml
  <resources>
    <resource>
      <directory>${project.basedir}/src/main/resources</directory>
      <filtering>true</filtering><!--要過濾-->
      <excludes>
        <exclude>**/*.ttf</exclude>
      </excludes>
    </resource>
    <resource>
      <directory>${project.basedir}/src/main/resources</directory>
      <filtering>false</filtering><!--只拷貝不過濾-->
      <includes>
        <include>**/*.ttf</include>
      </includes>
    </resource>
  </resources>
```

或許您會覺得，過濾替換好像沒有特別的需要?

想像一下這樣的開發場景：

我在本機開發是Windows，但公司的測試部署環境是Linux，而客戶的部署環境又不一樣(也許是路徑不同)。
難道在打包程式時，自已再去修這些參數嗎?

當然是在不同的<b>&lt;profiles&gt;</b>設定好各自的參數，然後打包時告訴Maven要用那組<b>&lt;profile&gt;</b>
進行打包(將客戶那組設為預設，不指定<b>&lt;profile&gt;</b>時就以客戶那組為準)。

那麼假設客戶要求打包檔時要連Source Code一起包進去，那麼請問您要怎麼做才能將源碼一起打包?
