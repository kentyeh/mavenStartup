# PMD
和[SonarLint](https://rules.sonarsource.com/java/)一樣，也是作源碼檢測，但著重在可以自訂規則，(官網也提供了第3方提供的[規則集](https://docs.pmd-code.org/latest/pmd_userdocs_3rdpartyrulesets.html))，算是比較老牌的源碼檢測工具。
我的設定一般如下
```xml
<pom>
  <reporting>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-pmd-plugin</artifactId>
        <version>3.21.2</version>
        <configuration>
          <failOnViolation>false</failOnViolation>
          <linkXref>true</linkXref>
          <sourceEncoding>${project.build.sourceEncoding}</sourceEncoding>
          <minimumTokens>100</minimumTokens><!--重覆的token超出這個數量視為拷貝-貼上的程式碼-->
          <targetJdk>${maven.compiler.target}</targetJdk>
        </configuration>
        <reportSets><!--產製報表時的相關設定-->
          <reportSet>
            <reports>
              <report>pmd</report>
            </reports>
          </reportSet>
        </reportSets>
      </plugin>
    </plugins>
  </reporting>
</pom>
```
PMD除了源碼檢測外(`pmd:pmd` Goal)，還可以幫忙找出程式碼中拷貝貼上的部份(`pmd:cpd`)，這兩個功能被整合到一個Goal `pmd:check`(同時進行源碼檢測與找出重覆的部份)。

但是因為本身沒有提供報表，所以必須都做一點設定，以便整合進專案報表(site report，後面章節會提到)中。
