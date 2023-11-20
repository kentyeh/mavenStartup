# 測試與整合測試

我們再用下列指令建立一個新專案
```bash
mvn archetype:generate -DarchetypeRepository=local \
-DarchetypeGroupId=com.github.kentyeh \
-DarchetypeArtifactId=springJdbiArch -DarchetypeVersion=4.0.2
```
專案識別與相關問題輸入完成後(後列階假設rtifactI為springJdbi3)，這是一個整合[Spring](https://spring.io/projects/spring-framework),[Spring Security](https://spring.io/projects/spring-security) 與 [JDBI3](http://jdbi.org/) 的專案.

這裡必須介紹整體較複雜的兩個Plugins.

首先是[maven-failsafe-plugin](https://maven.apache.org/surefire/maven-failsafe-plugin/)，這個外掛與之前介紹過的[maven-surefire-plugin](https://maven.apache.org/surefire/maven-surefire-plugin/)兩者互為表裡(所以兩者的版本號會保持一致)，`maven-surefire-plugin`著重在單元測試，而`maven-failsafe-plugin`注重在整合測試，顧名思義，單元測試針對每個函式進行輸入與輸出的測試，而整合測試則必須考慮測試標的的前後關係(如修改密碼前必須先登錄帳號)。

`maven-failsafe-plugin`最重要的Goal有兩個，一個是`integration-test` ，另一個就是`verify`，兩個Goal皆對綁定到同名的生命周期。

此專案是一個Web程式，依之前介紹過的[直接執行程式](runDirect.md)，可以直接執行。

```bash
mvn jetty:run
```
由於我一貫性使用[TestNG](https://testng.org/)，所以設定檔指定了測試程式`TestIntegration.java`

```xml
<plugin>
  <artifactId>maven-failsafe-plugin</artifactId>
  <version>${surefire.version}</version>
  <configuration>
    <suiteXmlFiles>
      <suiteXmlFile>${project.build.testOutputDirectory}/testng-integration.xml</suiteXmlFile>
    </suiteXmlFiles>
  </configuration>
</plugin>
```
進行整合測試之前，我們應該先將Web Server上起來，而在測試程式完成後則應把Web Server關閉，所以我們必須將`jetty-maven-plugin`的啟動與關閉的Goal綁定到生命週期的`pre-integration-test`與`post-integration-test`。

```xml
<plugin>
  <groupId>org.eclipse.jetty</groupId>
  <artifactId>jetty-maven-plugin</artifactId>
  <version>11.0.17</version>
  <executions>
    <execution>
      <id>start-jetty</id>
      <phase>pre-integration-test</phase><!--整合測試前-->
      <goals><goal>start</goal></goals><!--啟動Jetty-->
    </execution>
    <execution>
      <id>stop-jetty</id>
      <phase>post-integration-test</phase><!--整合測試後-->
      <goals><goal>stop</goal></goals><!--關閉Jetty-->
    </execution>
  </executions>
</plugin>
```
在整合測試期間，整個Web是處於可運作狀態，測試程式得以對Web進行測試。

在此程式，使用[HtmlUnit](https://htmlunit.sourceforge.io/)進行Web測試。

`HtmlUnit`被稱作為 `Headless Broser`，也就是看不見的瀏覽器，但幾乎實作了完整的瀏覽器功能，重點是我們可以用程式操縱這個看不見的瀏覽器。可以想見，她很適合來寫爬蟲程式。

但`HtmlUnit`的缺點也是看不見，所以很難想像，畫面執行是長什麼樣子，為了彌補這個缺陷，我通常建立在您常用的瀏覽器加裝[Selenium IDE](https://www.selenium.dev/selenium-ide/)這個外掛([火狐](https://addons.mozilla.org/en-GB/firefox/addon/selenium-ide/)、[孔龍妹](https://chrome.google.com/webstore/detail/selenium-ide/mooikfkahbdckldjjndioackbalphokd)與[Edge](https://microsoftedge.microsoft.com/addons/detail/selenium-ide/ajdpfmkffanmkhejnopjppegokpogffp))，可記錄瀏覽器執行過程並重播(當然測試，免不了要自行編寫一些Assertion)。

`Selenium IDE`工具其實還支持將她的測試腳本匯出為程式碼([Java](https://search.maven.org/artifact/org.seleniumhq.selenium/selenium-java))，讓程式碼直接操縱瀏覽器直接進行整合測試，然後直接在程式裡直接Assertion，我不建議這麼作(雖然還蠻炫的)，因為這樣做的缺點也很明顯-綁特定的瀏覽器，在多人開發環境下是無法保證大家都預設用同款瀏覽器。

所以我備們直接執行下列指令進行整點測試與確認吧

```bash
mvn verify
```

或者只要進行整合測試
```bash
mvn integration-test
```
