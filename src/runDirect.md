# 直接執行程式

在上一節[建立可執行Jar檔](executableJar.md)是不是奇怪經常看到

```xml
  <properties>
    <exec.mainClass>完整類別名稱(有main方法的class)</exec.mainClass>
  <properties>
```
這是因為變數 <a href="https://www.mojohaus.org/exec-maven-plugin/java-mojo.html#mainClass" target="_blank"><code class="hljs">exec.mainClass</code></a>恰是
<a href="https://www.mojohaus.org/exec-maven-plugin/" target="_blank" sytle="font-weight:bold">exec-maven-plugin</a>的Goal:<a href="https://www.mojohaus.org/exec-maven-plugin/java-mojo.html" target="_blank">exec:java</a>的參數

當我們執行 `mvn compile exec:java` 時，專案就會立即編譯後立即執行指定的 class(當然，如前執行前已編譯過，可以直接執行 `mvn exec:java`)

(註：之前[外掛](plugin.md)一節提過[前置字](https://maven.apache.org/guides/introduction/introduction-to-plugin-prefix-mapping.html)的解析，讓Maven知道**exec**要使用何種外掛。)

當然，如需額外設定(例如加系統變數)，就必須在pom.xml內自行設定

```xml
<plugin>
  <groupId>org.codehaus.mojo</groupId>
  <artifactId>exec-maven-plugin</artifactId>
  <version>3.1.0</version>
  <configuration>
    <cleanupDaemonThreads>false</cleanupDaemonThreads>
    <systemProperties>
      <systemProperty>
        <key>db2.jcc.charsetDecoderEncoder</key>
        <value>3</value>
      </systemProperty>
    </systemProperties>
  </configuration>
</plugin>
```
(註：每種外掛設定系統變數的方式不儘相同，一定要看文件)

除了上述的方式，也有另一種設定(要自行設定)可以讓程式直接執行

```xml
<plugin>
  <groupId>org.codehaus.mojo</groupId>
  <artifactId>exec-maven-plugin</artifactId>
  <version>3.1.0</version>
  <configuration>
    <executable>java</executable>
    <arguments>
      <argument>-classpath</argument>
      <classpath />
      <argument>${exec.mainClass}</argument>
    </arguments>
  </configuration>
</plugin>
```
完成設定後，改為執行 `mvn compile exec:exec`，跟先前不同的是，先前整個流程都在同一個Jvm裡面完成，而此次則是叫用java來執行程式，所以同時會有2個JVM存在。

這樣的好處是可以額外加參數，例如使用JDK9以後的 module設定，就必須自行加入像<b>--module-path</b>或<b>--module</b>這樣的參數。

前述講的都是Jar，但如果我開發的是Web程式呢?

其實只要引用 **jetty-maven-plugin** 就可以了。

```xml
  <plugin>
    <groupId>org.eclipse.jetty</groupId>
    <artifactId>jetty-maven-plugin</artifactId>
    <version>11.0.17</version><!--如果是JDK8請改用9.x版-->
    <configuration>
      <webApp>
        <!--下行若不加，則會以 Root 執行-->
        <contextPath>/${project.artifactId}</contextPath>
        <!--在目錄架構一節我有提過，為了過濾web.xml，我把web.xml換了位置, 如果沒換位置下行不用加-->
        <descriptor>${project.build.testOutputDirectory}/web.xml</descriptor>
      </webApp>
      <httpConnector><!--預設就是8080，可以不用加-->
          <port>8080</port>
      </httpConnector>
      <stopKey>HALT</stopKey><!--這兩行用於執行 jetty:stop 停止，不一定要用-->
      <stopPort>9966</stopPort>
    </configuration>
  </plugin>
```

我們先建立一個Web專案

```shell
mvn archetype:generate -DarchetypeGroupId=org.apache.maven.archetypes \
  -DarchetypeArtifactId=maven-archetype-webapp -DarchetypeVersion=1.4 \
  -DarchetypeCatalog=https://repo.maven.apache.org/maven2/archetype-catalog.xml
```
然後搬動web.xml，再把**jetty-maven-plugin**加入pom.xml, 然後執行

```shell
mvn jetty:run
```

然後開啟 [http://localhost:8080/](http://localhost:8080/)就可以看到運行的web了。
