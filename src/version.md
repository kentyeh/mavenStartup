# 版本管理
[maven-dependency-plugin](https://maven.apache.org/plugins/maven-dependency-plugin/)是一個常用的pluing，通常用來檢查專案裡倒底有哪些相依的jars.

```shell
mvn dependency:tree
```

會以樹狀圖的方式顯示整個相依jars的關係。

假設pom.xml類似如下：

```xml
<project ...>
  <dependencies>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-core</artifactId>
      <version>6.1.1</version>
    </dependency>
    <dependency>
      <groupId>org.jdbi</groupId>
      <artifactId>jdbi3-spring5</artifactId>
      <version>3.41.3</version>
    </dependency>
  <dependencies>
</project>
```
則 `mvn dependency:tree` 大概顯示如下的Tree
```shell
[INFO] +- org.jdbi:jdbi3-spring5:jar:3.41.3:compile
[INFO] |  +- org.jdbi:jdbi3-core:jar:3.41.3:compile
[INFO] |  |  +- org.slf4j:slf4j-api:jar:1.7.36:compile
[INFO] |  |  \- io.leangen.geantyref:geantyref:jar:1.3.14:compile
[INFO] |  +- org.springframework:spring-beans:jar:5.3.30:compile
[INFO] |  +- org.springframework:spring-jdbc:jar:5.3.30:compile
[INFO] |  \- org.springframework:spring-tx:jar:5.3.30:compile
[INFO] \- org.springframework:spring-core:jar:6.0.13:compile
[INFO]    \- org.springframework:spring-jcl:jar:6.0.13:compile
```
仔細看，相依關係混雜著spring 5版與6版，最新版的Maven已經比舊版聰明許多，舊版會看到兩個不同的spring-core版本。
所以我們只要把pom.xml改成如下：
```xml
<project ...>
  <dependencies>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>6.0.13</version>
    </dependency>
    <dependency>
      <groupId>org.jdbi</groupId>
      <artifactId>jdbi3-spring5</artifactId>
      <version>3.41.3</version>
    </dependency>
  </dependencies>
</project>
````
相依關係就變得如下：
```shell
[INFO] +- org.springframework:spring-jdbc:jar:6.0.13:compile
[INFO] |  +- org.springframework:spring-beans:jar:6.0.13:compile
[INFO] |  +- org.springframework:spring-core:jar:6.0.13:compile
[INFO] |  |  \- org.springframework:spring-jcl:jar:6.0.13:compile
[INFO] |  \- org.springframework:spring-tx:jar:6.0.13:compile
[INFO] \- org.jdbi:jdbi3-spring5:jar:3.41.3:compile
[INFO]    \- org.jdbi:jdbi3-core:jar:3.41.3:compile
[INFO]       +- org.slf4j:slf4j-api:jar:1.7.36:compile
[INFO]       \- io.leangen.geantyref:geantyref:jar:1.3.14:compile
```
原本Spring 5.3.30的版本都不見了，那是因為我們在[jdbi](https://jdbi.org/)之前先引入了spring-jdbc，由於[spring-jdbc.6.0.13](https://search.maven.org/artifact/org.springframework/spring-jdbc/6.0.13/jar)本身已有相依的spring-tx，所以Mavven到了處理jdbi3-spring5的相依關係時，發現已經引入了相關版本的artifact，便不會再次引入，所以讓Spring保持在6.0.13版。

當然，比較保險的方式，就是直接排除我們不要的artifact，再引進需要的jar
```xml
<project ...>
  <dependencies>
    <dependency>
      <groupId>org.jdbi</groupId>
      <artifactId>jdbi3-spring5</artifactId>
      <version>3.41.3</version>
        <exclusions>
          <exclusion>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
          </exclusion>
          <exclusion>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
          </exclusion>
          <exclusion>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
          </exclusion>
          <exclusion>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
          </exclusion>
        </exclusions>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>6.0.13</version>
    </dependency>
  </dependencies>
</project>
```

附帶一提，若需要下載所有相關的artifacts到本機端，請執行下列指令, 則會下載相依的Jars。

```shell
mvn dependency:copy-dependencies
```

另外，若要檢查專案是否採用最新版本的artifacts，可以執行下列指令，以出具版本報告，您再決定是否採用更新的版本。

```shell
mvn versions:display-dependency-updates
```

下列指令也可以出具外掛建議使用的版本報告(不一定是最新版)。
```shell
mvn versions:display-plugin-updates
```

不過因為[versions-maven-plugin](https://www.mojohaus.org/versions/versions-maven-plugin/index.html)無法知道您使用的是哪個版本，
 所以會列出一堆根據不同版本Maven的建議，若我們要限縮顯示的範圍。我們可以在pom.xml加本最低顯示的版本即可。

 ```xml
<project ... >
  <prerequisites>
    <maven>3.2.5</maven>
  </prerequisites>
</project>
 ````

版本管理的另一個課題是，我們如何保證，我們所開發用相關的**jar**(artifact)都是符合JDK版本需求?

例如JDK7用到專為JDK8所開發的artifact，就可能造成錯誤，例如[openpdf](https://search.maven.org/artifact/com.github.librepdf/openpdf)。

所以我們需要一個檢查機制，為此Maven推了一個所謂的[Extension](https://maven.apache.org/guides/mini/guide-using-extensions.html)，可以套用於全域的專案，但往往我們可能在同一個開發環境開下發不同JDK版本的專案，所以我還是傾向透過外掛來執成此目的

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-enforcer-plugin</artifactId>
  <version>3.4.1</version>
  <executions>
    <execution>
      <id>enforce-java</id>
      <goals><goal>enforce</goal></goals>
      <configuration>
        <rules>
          <requireJavaVersion>
            <version>${maven.compiler.target}</version>
          </requireJavaVersion>
        </rules>
      </configuration>
    </execution>
  </executions>
</plugin>
```

是的，只要將**maven.compiler.target**設為1.7，當使用到不該用的Jar時，此外掛就會報錯並阻止Maven繼續進行。

最後一個問題是，多版本JDK的問題，Maven提供了[maven-toolchains-plugin](https://maven.apache.org/plugins/maven-toolchains-plugin/usage.html)，但我照[官方指示](https://maven.apache.org/guides/mini/guide-using-toolchains.html)，執行 `mvn toolchains:toolchain` 很正常，就正當編譯時 enforcer 就一直報錯(我故意用比較低版本的JDK)，好在Linux上面切換JDK版本非常簡易，所以暫時放棄。
