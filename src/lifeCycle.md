# 生命週期

之前[外掛](plugin.md)一節提過，plugin提供了各種Goal來達成各自的目的，像之前提過的

```shell
mvn clean:clean package javadoc:javadoc
```
等等,怎麼找不到 **package** 是哪個Goal?

您是否想過打包Jar、EJB或Web，它們的目錄結構會是一樣的嗎?

沒有一種打包的外掛能夠Cover各種不同的打包，也許您也會像[spring-boot-maven-plugin](https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/)自行開發打包的方式。

想想**package**這件事，至少要經過拷貝資源、編譯程式、測試等階段(Phase)，才會開始打包吧!

所以我們知道Phase是包含從開始到標的Goal的一系列集合。

而Maven所謂的生命週期(LifeCycle)就是指一連串的 Phase 所形成。

Maven有三種不同的生命週期，我們先來看一下官方所列出的表格

|Phase |以打包Jar的 plugin:goal  | 說明 |
| --- | --- | --- |
|process-resources | resources:resources | src/main/resources(也許先過濾)拷貝到 classes 目錄 |
|compile | compiler:compile | 編譯main/src源碼 |
|process-test-resources | resources:testResources | src/test/resources(也許先過濾)拷貝到 target/test-classes 目錄 |
|test-compile | compiler:testCompile | 編譯src/main/java源碼 |
|test | surefire:test | 進行單元測試 |
|package | jar:jar | 把專案打包成Jar檔 |
|install | install:install | 把Jar檔安裝到本機函式庫(之前在專案管理一節使用過**install:install-file**) |
|deploy | deploy:deploy | 把Jar檔發佈到遠端函式庫(例如Maven的中央庫) |

從上表看，我們package一個Jar檔時，不止是執行**jar:jar**這個Goal，而是在**package**這個Phase之前的Phase都要依序執行過，
這也是為什麼我們只要執行**package**時，未**compile**過的程式能被編譯的關鍵    。

當需要執行**phase**時，Maven會執行該**phase**所關聯的Goal，這也是[spring-boot-maven-plugin](https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/)能夠在**package**時進行打包。

記得在[直接執行程式](runDirect.md)一節我們透過指令直接執行程式

```shell
mvn compile exec:java
```

`exec:java`是一個獨立的Goal，所以我們要在執行之前要先確保程式經過**compile**，所以要串接兩個指令，
現在我只要指定在**test**這個**phase**執行這個Goal，就可以不用這麼麻煩了。

```xml
<plugin>
  <groupId>org.codehaus.mojo</groupId>
  <artifactId>exec-maven-plugin</artifactId>
  <version>3.1.1</version>
  <executions>
    <execution><!--設定Goal的執行方式-->
      <phase>test</phase><!--在test pahse時執行下列的Goal-->
      <goals>
        <goal>java</goal> <!--要執行的goal-->
      </goals>
    </execution>
  </executions>
  <configuration>
    <mainClass>${exec.mainClass}</mainClass>
    <cleanupDaemonThreads>false</cleanupDaemonThreads>
  </configuration>
</plugin>
```
現在我們只要直接執行 `mvn test` ，Maven就會執行**test** phase之前所有的pahse，我們就可以不用考慮源碼的編譯問題。

之前我們提到過Maven有三大生命周期，包括clean(清理目錄結構)、build(上面的**package**表格只是這個的一部份)、與 site(主管文件產製)。
強烈建議要查看[官方文件](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#lifecycle-reference)。

打開官方文件，瞭解生命周期有哪些**phase**s外，再對照各個**pahse**[關聯(Binding)](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#built-in-lifecycle-bindings)的Goal，至此，便可算Maven入了門。

我要特別提一下[Default Lifecycle](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#default-lifecycle)最後有四個phases值得注意：

|Phase | 說明 |
| --- | --- |
| pre-integration-test | 整合測試前 |
| integration-test | 整合測試 |
| post-integration-test | 整合測試後 |
| verify | 運行任何檢查以驗證打包是否有效並符合品質標準 |

一般來說單元測試只是針對函式做輸入輸出的驗證，而整合測試則是包含實際的運作(可能測試是有時間序的，比如登錄後才能測試功能)，
前一小節[直接執行程式](runDirect.md)有提過如何讓Web專案直接潤(run)起來。

再配上上面所說的&lt;**executions**&gt;，我們根本可以在整合測試前，讓Jetty Web執行後再進行整合測試，然後測試完成後，再關閉Jetty Web。

```xml
<plugin>
  <groupId>org.eclipse.jetty</groupId>
  <artifactId>jetty-maven-plugin</artifactId>
  <version>11.0.19</version>
  <executions>
    <execution>
      <id>start-jetty</id>
      <phase>pre-integration-test</phase>
      <goals>
        <goal>start</goal>
      </goals>
    </execution>
    <execution>
      <id>stop-jetty</id>
      <phase>post-integration-test</phase>
      <goals>
        <goal>stop</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```
註1：整合測試需要用到[Maven Failsafe Plugin](https://maven.apache.org/surefire/maven-failsafe-plugin/)，版本通常與[maven-surefire-plugin](https://maven.apache.org/surefire/maven-surefire-plugin/)一致。

註2:Maven 的 Goal 可會有預設綁定(default bind)的phase，如果&lt;**execution**&gt;未指定&lt;**phase**&gt;時，則依預設binding phase執行。

  例如我們沒有特別編譯就直可直接執行 `mvn jetty:run` 是因為 jetty 執行前確保會執行 compile。

  可以執行 `mvn org.eclipse.jetty:jetty-maven-plugin:help -Dgoal=run` 看一下說明。
