# Site Report
顧名思義，就是用產生整個專案的報表，之前我設定了不少&lt;reporting&gt;的tag，用來設定產生報表的相關設定。

在[生命周期](lifeCycle.md)也有說過`site`生命周期主管文件產製，配合前述提過的Plugins，在此提供一份我的設定給各位參考。
```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-site-plugin</artifactId>
      <version>3.12.1</version>
      <dependencies>
        <dependency>
          <groupId>org.apache.maven.doxia</groupId>
          <artifactId>doxia-module-markdown</artifactId>
          <version>1.12.0</version>
        </dependency>
      </dependencies>
      <configuration>
        <stagingDirectory>/tmp/site</stagingDirectory><!--如果是多模組下，
          應指定各個模組報表之最終合併目錄 -->
        <locales>zh_TW</locales>
      </configuration>
    </plugin>
  </plugins>
</build>
<reporting>
  <plugins><!--開始在此放入相應報表的 pluing 設定，
    包含在軟體品質保證一節中提到的各個Pluings Reporting設定-->
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-project-info-reports-plugin</artifactId>
      <version>3.4.5</version>
      <configuration>
        <dependencyLocationsEnabled>false</dependencyLocationsEnabled>
      </configuration>
      <reportSets>
        <reportSet>
          <reports>
            <report>index</report>
            <report>plugin-management</report><!--專案外掛程式管理
               (Project Plugin Management)，設定要出哪些報告-->
            <!--report>distribution-management</report>  發佈管理-->
            <report>scm</report><!--原始碼貯藏庫 (Source Repository)-->
            <!--report>mailing-lists</report>  專案郵件列表 (Project Mailing Lists)-->
            <!--report>issue-management</report> 問題追蹤 (Issue Tracking)-->
            <!--report>ci-management</report>  持續整合 (Continuous Integration)-->
            <report>plugins</report><!--專案建構外掛程式 (Project Build Plugins)-->
            <report>summary</report><!--專案摘要 (Project Summary)-->
            <report>dependencies</report> <!--專案依賴 (Project Dependencies)-->
            <report>dependency-management</report>
            <report>dependency-convergence</report>
            <report>dependency-info</report>
            <!--<report>modules</report>-->
          </reports>
        </reportSet>
      </reportSets>
    </plugin>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-jxr-plugin</artifactId>
      <version>3.3.1</version>
      <configuration>
        <aggregate>true</aggregate>
      </configuration>
    </plugin>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-surefire-report-plugin</artifactId>
      <version>3.2.2</version>
      <configuration>
        <junitReport>true</junitReport>
        <exportAll>true</exportAll>
      </configuration>
      <reportSets>
        <reportSet>
          <id>integration-tests</id>
          <reports>
            <report>report-only</report>
            <report>failsafe-report-only</report>
          </reports>
        </reportSet>
      </reportSets>
    </plugin>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-javadoc-plugin</artifactId>
      <version>3.6.2</version>
      <configuration>
        <verbose>false</verbose>
        <locale>zh_TW</locale>
        <encoding>UTF-8</encoding>
        <docencoding>UTF-8</docencoding>
        <linksource>true</linksource>
        <doclint>accessibility,html,reference,syntax</doclint>
        <failOnError>false</failOnError>
        <source>${maven.compiler.source}</source>
        <additionalparam>-Xdoclint:none</additionalparam>
        <links>
          <link>https://docs.spring.io/spring-framework/docs/current/javadoc-api/</link>
          ...
        </links>
        <javaApiLinks>
          <property>
            <name>api_17</name>
            <value>https://docs.oracle.com/en/java/javase/17/docs/api/</value>
          </property>
        </javaApiLinks>
        <additionalJOptions>-J-Dhttp.agent=maven-javadoc-plugin-${pom‌​.name}</additionalJOptions>
        <additionalJOptions>--allow-script-in-comments</additionalJOptions>
        <doctitle>${project.name}</doctitle>
        <windowtitle>${project.name} API</windowtitle>
      </configuration>
      <reportSets>
        <reportSet>
          <id>default</id>
          <reports>
            <report>javadoc</report>
            <report>aggregate</report><!--多模組時加這設定-->
          </reports>
        </reportSet>
      </reportSets>
    </plugin>
    <plugin>
       …其它Pluings的設定
    </plugin>
  </plugins>
</reporting>
```
像要有spotbus的報表，就必須要走過compile 與 check goal(預設綁定verify生命週期)，其它的Plugins的報表也是要滿足各自的條件。
最通用的作法就是 `mvn version site`，也是是在生命週期`verify`走完成再來走 site的生命週期是最保險。

如果是多模組專案，則必須設定`<maven-site-plugin>` `<configuration>`的`<stagingDirectory>`設定後再執行 `site:stage` goal.
```shell
  mvn verify site site:stage
```
