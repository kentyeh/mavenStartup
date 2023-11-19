# Checkstyle
[checkstyle](checkstyle)用來檢查程式的寫法是否有遵循一定的風格(一行長度太長或命名不合規範等…)。

**maven-checkstyle-plugin**預設使用[Sun的程式風格](https://checkstyle.org/styleguides/sun-code-conventions-19990420/CodeConvTOC.doc.html)，您也可以使用其它的風格寫作，例如[Google Style](https://checkstyle.org/styleguides/google-java-style-20180523/javaguide.html)，做法就是下載[設定檔](https://github.com/checkstyle/checkstyle/blob/master/src/main/resources/google_checks.xml)到本機，然後在&lt;configuration&gt;的&lt;configLocation&gt;[指向](https://maven.apache.org/plugins/maven-checkstyle-plugin/check-mojo.html#configLocation)設定檔。

一般我的設定檔如下：
```xml
<pom>
  <reporting>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-checkstyle-plugin</artifactId>
        <version>3.3.1</version>
        <configuration>
          <failsOnError>false</failsOnError>
          <linkXRef>false</linkXRef>
        </configuration>
        <reportSets>
          <reportSet>
            <reports>
              <report>checkstyle</report>
            </reports>
          </reportSet>
        </reportSets>
      <plugin>
    </plugins>
  </reporting>
</pom>
```
和[PMD](pmd.md)一樣，Checkstyle本身沒有提供報表，所以必須都做一點設定，以便整合進專案報表中。
