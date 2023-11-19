# 建立可執行檔

我們首先要瞭解，為什麼我們可以直接執行某些 jar 檔? 像 <code class="hljs">java -jar <a href="https://search.maven.org/artifact/com.h2database/h2" target="_blank">h2-2</a>.2.224.jar</code>

原因很簡單，jar 本身就是一個 zip 壓縮檔，本身包含一份艙單在 META-INF/MANIFEST.MF。

當`java -jar`執行時，**java**會先開啟這份艙單，讀取相關的資訊，包可執行那個類別或外部應有相依的Library。

然後就像正常的執行 class 一樣開始執行這個程式。

所以當我們打包這個 jar 檔時，就必須在相應的外掛中，設定相關的資料。

```xml
  <properties>
    <exec.mainClass>完整類別名稱(有main方法的class)</exec.mainClass>
  <properties>
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-jar-plugin</artifactId>
        <version>3.3.0</version>
        <configuration>
          <archive>
            <manifest>
              <addClasspath>true</addClasspath>
              <mainClass>${exec.mainClass}</mainClass>
            </manifest>
          </archive>
        </configuration>
      </plugin>
    </plugins>
  </build>
```

上述的作用僅僅是把預設執行的Class寫入艙單內，若是還需要外部的Jar協同執行的話，您就必須凖備好相應的Jar檔，並把這些jars檔加入Classpath中(也可以相對的徑，寫入艙單內，然後自動加入ClassPath)，不過一般除了JDBC的Jar外不太這樣做，因為我們不知道這些相關的Jars檔是否會迭失，或是User自行換了更新版本的Jar導致程式不相容的情況發生，所以一般可以把這些要用到的Jars檔，把所有的classes一起打成一包，做法改用Pluing如下:

```xml
  <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>3.5.0</version>
    <configuration>
      <archive>
        <manifest>
          <mainClass>${exec.mainClass}</mainClass>
        </manifest>
      </archive>
      <descriptorRefs>
        <descriptorRef>jar-with-dependencies</descriptorRef>
      </descriptorRefs>
    </configuration>
  </plugin>
```

剛剛我為什麼提JDBC的Jar通常不會用這種方式包進來?建議您找個JDBC的Jar解Zip出來看看，其實META-INF/內包含的不只是艙單，有時會有像services/這樣的目錄用來記錄service provider的Class有那些，更甚者，像spring framework還有自訂的目錄結構，而如何處理這樣複雜的結構就必須要使用不同的外掛與處理。

此時我們需要改用<a href="https://maven.apache.org/plugins/maven-shade-plugin/" target="_blank" style="font-weight:bold">maven-shade-plugin</a>來建立這個Jar檔。

下面是針對使用 spring 與 <a href="https://logging.apache.org/log4j/2.x/" target="_blank">Log4j2</a>的設定範例。

```xml
  <properties>
    <exec.mainClass>完整類別名稱(有main方法的class)</exec.mainClass>
  <properties>
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-shade-plugin</artifactId>
        <version>3.5.1</version>
        <dependencies>
          <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-transform-maven-shade-plugin-extensions</artifactId>
            <version>0.1.0</version>
          </dependency>
        </dependencies>
        <executions>
          <execution>
            <phase>package</phase><!--生命週期章節再解釋-->
            <goals><goal>shade</goal></goals>
            <configuration>
              <artifactSet>
                 <excludes><!--打包時排除那些您認為多餘的Library -->
                    <exclude>org.slf4j:slf4j-api:jar:</exclude><!--最後省略的是版本號-->
                    <exclude>com.microsoft.sqlserver:mssql-jdbc:jar:</exclude>
                 </excludes>
              </artifactSet>
              <transformers><!--指定那些可以處理打包的類別，各自進來處理打包-->
                <transformer implementation="org.apache.logging.log4j.maven.plugins.shade.transformer.Log4j2PluginCacheFileTransformer"/>
                <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer"/>
                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                  <mainClass>${exec.mainClass}</mainClass>
                  <manifestEntries><!--jar檔是否包含不同JDK版本的class-->
                    <Multi-Release>false</Multi-Release>
                  </manifestEntries>
                 </transformer>
              </transformers>
              <filters>
                <filter>
                  <artifact>*:*</artifact>
                  <excludes><!--打包時排除索引與簽章檔-->
                    <exclude>META-INF/*.SF</exclude>
                    <exclude>META-INF/*.DSA</exclude>
                    <exclude>META-INF/*.RSA</exclude>
                  </excludes>
                </filter>
              </filters>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
```

其中<code>&lt;artifactSet&gt;</code>是用來排除不想被包進來的的Jar檔，<code>&lt;transformers&gt;</code>則是指指定讓哪些外部程式進來處理META-INF/，此例需要外部class進來處理Log4j的打包，所以引用了要第三方dependency，要如何建立可執行檔，請參考[官方文檔](https://maven.apache.org/plugins/maven-shade-plugin/examples/executable-jar.html#)，也建議看一下[Resource Transformers](https://maven.apache.org/plugins/maven-shade-plugin/examples/resource-transformers.html)章節，裡面就有包含怎麼打包Spring程式的說明。
