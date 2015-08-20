開始使用Maven是因為受夠了每次開始一個Project，都要重新開始撰寫
Ant build.xml，雖然有過去建立的範本，但是還是的根據不同狀況加以
改寫(比如不同的打包方式，如同時deploy 到tomcat 與jboss時用到不
同的log4J設定，或是像GWT必須加上一道Comple手續);當有新版本的
Liberary出來的時候，還得忙著下載新Jar，然後發現不相容時，還得改回來…
Maven最大的好處，就是把上面的無聊事標準化，簡單化了，用久了，還真得
少不了它。

#<a name="buildMavenEnv"></a>建立Maven環境
1. 下載Mav[](http://maven.apache.org/download.html)en後，解開到目錄，例 x:\maven\ 
2. 決定您電腦的函式庫(Maven 把所有Libery集中管理)所在，預設在 用戶目錄/.m2，若要變更不同位置，修改 x:\maven\conf\settings.xml的<localRepository>設定 
3. 設定環境變數 M2_HOME 指到 Maven目錄，並且將 Maven目錄下的 bin加到執行路徑

#<a name="firstProject"></a>用Maven建立第一個Project
在命令視窗執行 mvn archetype:generate 命令，使用互動方式建立Project， 會依序問幾個問題 


| Choose archetype: | 選擇建立Project的範本，預設是99:maven-archetype-quickstart建立一個最基本的Project |
| -- | -- |
| Choose version: | 選擇範本的版本，會列出一些範本可用的版本，其差異是就不用版本的範本可能會建立有不同的資源檔(比如可能附帶圖檔) |
| 定義groupId: | 輸入要建立Project所隸屬的組織或公司，如我自已用idv.kentyeh.software | 
| 定義artifactId: | 就是Project名稱，例如 firstMaven |
| 定義version: | Project的版本號，預設是1.0-SNAPSHOT |
| 定義package: | 初始建立的Java Package, 如 idv.kentyeh.software |

確定後建立Project的基本架構，如果您不要用互動的方式，上述動作可以以下指令完成相同的事 

```
mvn archetype:create -DgroupId=idv.kentyeh.software -DartifactId=firstmaven \
      -DpackageName=idv.kentyeh.software -DarchetypeArtifactId:maven-archetype-quickstart \
      -Dversion=1.0-SNAPSHOT
```

#<a name="identity"></a>Maven的識別管理

Maven的識別管理，分為三層 groupId:artifactId:version，一個組織(group) 可能存在多個Project(atrifact)，每個Project也可能存在多個版本(version)， 整個函式庫就以這種檔檔案架構進行處理，當使用的函式不存在時，Maven會到 http://repo1.maven.org/maven2/ 或是 http://repo2.maven.org/maven2/ 進行下載，下載後存到您電腦上的函式庫 

#<a name="projectMangement"></a>Maven 的Project管理
Maven的管理設定主要靠Pom.xml進行，打開剛才建立的Project設定檔，內容說明如下： 
```<project xmlns="http://maven.apache.org/POM/4.0.0" 
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>
    
    <!--本Project識別-->
    <groupId>idv.kentyeh.software</groupId>
    <artifactId>firstmaven</artifactId>
    <version>1.0-SNAPSHOT</version>
    
    <packaging>jar</packaging>  <!--表示打包Project的型態,可能為Jar、war、ear或pom，若是使用了android 則為apk-->
    
    <!--以下是給工具看的,主要是本Project的資訊-->
    <name>第一個MavenProject</name>
    <url>http://sites.google.com/site/gwtmemo</url>
    
    <!--設定一些變數-->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    
    <!--設定引用函式庫-->
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>3.8.1</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
</project>

首先要說明的是dependencies段落，比如說，Project內會用到 commons-loggin 的Library，我們可以在此加入新的dependency段落，

```
    <dependency>
        <groupId>commons-logging</groupId>
        <artifactId>commons-logging</artifactId>
        <version>1.1.1</version>
    </dependency>```

不知道commons-loggin的識別資料?沒關係，到這裡查。至於scope可以不填，表示打包Project(如war,ear…)時， 引用的Library會一起被打包，Scope的值說明如下： 

|compile | Scope不填時的預設值，表示Project程式須要這個Library才能運作，所以會一併被打包 |
| -- | -- |
| provided | 表示編譯會用到，但是系統在需要的時候會提供，打包Project不要含進去，例如J2ee的Library，像是servlet-api，就是由App Server提供 |
| runtime | 表示編譯時用不到，只有執行時會用到，所以發佈程式時須要一併打包，如GWT 的 gwt-servlet.jar |
| test | 只有在單元測試時會用到，發佈程式時並不會用到，所以不會被打包 |
| system | 與provided相似，但是固定存在系統檔案,須以 systemPath 指定路徑  |

至於commons-loggin是否須要使用到其它的Library，根本完全不用在意，因為Maven會自動導入相關的Library 

#<a name="archetype"></a>Project目錄架構

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
└target[各種處理後產生的資料，包含最終生的的打包標的]
```

#<a href="thirdParty"></a>Maven 引用第三方函式庫

[之前](#identity)有說過，Maven有專門存放函式的repository (檔案庫)， 但是[ASF](http://www.apache.org/)再厲害，也不可能搜羅所有Library，所以必要的時候，我們必須引用第三方的函式檔案庫， 以下為可能會用到的來源(加入到Pom.xml)：
```
<repositories>
    <repository><!--J2ee 最新的函式庫在此-->
        <id>java.net2</id>
        <name>Repository hosting the jee6 artifacts</name>
        <url>http://download.java.net/maven/2</url>
    </repository>
</repositories>
```

#Maven Plugin

一開始的時候，我們執行 "mvn archetype:generate" 建立Project，mvn 後面接的指令 叫 goal由 "前置字:識別字" 表示要執行的作業。這些執業作業(命令)是由所謂的 plugin所提供，plugin是一種專供 Maven本身使用的Library，同一個plugin通常使用同一前置字，然後本身會帶有說明提供那些作業的自我解說檔(metadata)， 通常這個自我解說檔，也會定義每種作業隸屬於那種[phase](#phase)(作業階段)；Maven本身[早已知道](http://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Built-in_Lifecycle_Bindings)許許多多的goal的定義，當Maven執行特定的 goal時，缺少的Plugin，Maven就會自動去Repository下載相關的檔案。 

Maven 的指令可以串接，例如以下指令 

```
mvn clean package javadoc:javadoc exec:exec
```
表示，先刪除 target 目錄後再行打包Project，然後產生文件，再執行專案(需要額外設定)。

上述的 clean、package(打包）都很容易理解，可是為何又出現了 javadoc:javadoc、exec:exec 這樣的表示， 這是因為Plugin是Maven裡面的一種特殊專案，她裡面存在一些task，當這些Plugin被包含進專案的pom.xml時， 例如要執行maven-javadoc-plugin這個Plugin的javadoc task時，完整的命令下法應該是 
```
mvn org.apache.maven.plugins:maven-javadoc-plugin:2.10.1:javadoc
```
但是因為在的Jar的Metadata中已經標注了它的prefix，在Maven執行時，會去讀取每個plugin java的metadata， 而maven-javadoc-plugin的metadata標記為javadoc，所以才省略為 javadoc:javadoc，

至於前面的clean package 等則是[maven lifecycle](#phase)的一部分，會綁定特定的Plugin，然後執行 該Plugin的task。

plugin的定義結構通常如下 :
```
<project ...>
    ...
    <build>
        <plugins>
            <!--可定義多個plugin-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId><!--若是此值，則可省略-->
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                ...各種設定
                </configuration>
            </plugin>
        <plugins>
        <finalName>firstmvn</finalName>
        ...
    <build>
    ...
</project>
```

#<a name="property"></a>Project變數

我們建立的第一個 Project 的 [pom.xml](#projectMangement)檔案內有一個<properties>段落，可以讓我們定義一些變數， 例如Spring 通常含有多種Library，其引用版本應該一致，所以我通常會定義一個變數 

```
<org.springframework.version>4.2.0.RELEASE</org.springframework.version>
```

然後，定義版本別的地方會寫成 

```
<version>${org.springframework.version}</version>
```

以後當版本變更的時候，只要修改&lt;properties&gt;下的&lt;org.springframework.version&gt;4.2.0.RELEASE&lt;/org.springframework.version&gt;，就可以引用新的版本。 初建立時的pom.xml內已經有一個變數 

```
<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
```
