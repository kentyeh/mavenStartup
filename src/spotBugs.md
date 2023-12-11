# SpotBugs
[SpotBugs](https://spotbugs.github.io/)是我推薦一定要裝的外掛，常常您意想不到的漏洞，就是得靠她找出來。

我常用的設定如下(版本請自已找最新的)：

```xml
<build>
  <plugins>
    <plugin>
      <groupId>com.github.spotbugs</groupId>
      <artifactId>spotbugs-maven-plugin</artifactId>
      <version>4.8.2.0</version>
      <executions>
        <execution>
          <id>spotbugs</id>
          <phase>verify</phase>
          <goals><goal>check</goal></goals>
        </execution>
        <dependencies>
          <dependency>
            <groupId>com.payneteasy</groupId>
            <artifactId>ber-tlv</artifactId>
            <version>1.0-11</version>
          </dependency>
        </dependencies>
      </executions>
      <configuration>
        <plugins>
          <plugin>
            <groupId>com.h3xstream.findsecbugs</groupId>
            <artifactId>findsecbugs-plugin</artifactId>
            <version>1.12.0</version>
          </plugin>
        </plugins>
        <failOnError>false</failOnError>
        <effort>Max</effort>
        <fork>false</fork>
      </configuration>
    </plugin>
  </plugins>
</build>
```
上述定義了`mvn verify`會順帶執行源碼檢測。

若只是單純要檢查程式碼，則程式碼必須先經過編譯，所以單純執行源碼檢測可執行如下：

```shell
mvn compile spotbugs:check spotbugs:gui
```
其中 `spotbugs:check` 是用來執行源碼檢測，而 `spotbugs:gui` 則是用GUI的方式來查核源碼檢測的執行結果。

若有需要在產製專案報表(site report，後面章節會提到)時，也要一併併入SpotBugs報表，則需要新增設定如下
```xml
<pom>
  <reporting>
    <plugins>
      <groupId>com.github.spotbugs</groupId>
      <artifactId>spotbugs-maven-plugin</artifactId>
      <version>4.8.2.0</version>
      <configuration>
        <failOnError>false</failOnError>
        <xmlOutputDirectory>${project.build.directory}/site</xmlOutputDirectory>
        <effort>Max</effort><!--Detect Level:Min,Default or Max-->
        <fork>false</fork>
      </configuration>
    </plugins>
  </reporting>
</pom>
```
