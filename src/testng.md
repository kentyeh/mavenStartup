# TestNG設定

我個人覺得[TestNG](https://testng.org/doc/)比Junit好用多了，文件也漂亮多了，更不用像Junit 3->4->5 像在擠牙膏一樣，功能慢慢加，
每次都要重新學，TestNG從一開始就功能強大，學習的CP值比較高(但您若要使用spring-boot這種框架，Junit就具有天生的優勢，不熟的人要用就比較會有困擾)。

Maven的外掛設定如下：

```xml
<project>
  <properties>
    <surefire.version>3.2.2</surefire.version>
  </properties>
  <dependencies>
    <dependency>
      <groupId>org.testng</groupId>
      <artifactId>testng</artifactId>
      <version>7.8.0</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId><!--Maven 的單元測試外掛-->
        <version>${surefire.version}</version>
        <configuration>
           <systemPropertyVariables>
             <catalina.home>${project.build.directory}</catalina.home><!--這是我Log4J2會用到的系統變數，跟TestNG無關-->
           <systemPropertyVariables>
           <parallel>classes</parallel><!--以類別的方式進行平行測試，也就是測試Class之間無關聯，可同時開啟測試-->
           <suiteXmlFiles><!--TestNG設定檔-->
             <suiteXmlFile>${project.build.testOutputDirectory}/testng-unit.xml</suiteXmlFile>
           </suiteXmlFiles>
        </configuration>
      </plugin>
    </plugins>
  <build>
</project>
```

從上面的設定瞭解到，在 src/test/resources下有一支testng-unit.xml描述測試如何進行

```xml
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd" >
<suite name="${project.name}單元測試"  thread-count="4" skipfailedinvocationcounts="false" parallel="classes" verbose="10">
  <test name="分群測試功能" enabled="true" enabled="true">
      <parameter name="contextPath" value="/${project.artifactId}"/>
      <parameter name="http.port" value="8080"/>
      <classes>
          <class name="you.wanna.testClass.className">
            <!--<methods>
               <include name="testSomeMethod1" />
               <include name="testSomeMethod2" />
            <methods>-->
          </class>
      </classes>
  </test>
</suite>
```
&lt;parameter&gt;是一些自訂的參數，可以代入測試類別內。

&lt;classes&gt;指定那些要進行測試的類別。

測試類別可能包含一堆要測試的方式，常常我們的測試類別內，包含多個測試，但若測試不通過時，我們可以將某些測試Methos隔離出來，單獨測試並查找出錯誤原因。
那時就可以在&lt;method&gt;指定那些特別methos進行測試。

下面特別簡單說明一下TestNG的測試

```java
public class TestSomeUnit {
    private String contextPath;
    private int httpPort;

    @BeforeClass //類別執行前必定先執行的段落，可以帶參數，並決少時給預設值
    public void setup(@Optional("8080") int httpPort,
     @Optional("/HelloWorld") String contextPath){
     this.contextPath = contextPath;
     this.port = port;
   }

   @AfterClass
   public void tearDown() {
      //類別測試結束後，一定要到此進行善後
   }

   @Test(description = "登錄")
   void testLogin() throws Exception {
      //進行登錄
   }

   @Test(description="查詢個資",dependsOnMethods="testLogin")
   void testMyprofile() throws Exception{
      //查詢測試
   }

   @Test(description="Other Test")
   void testOther(){
      //其它測試
   }
}
```
上述測試碼中的testMyprofile()，本身包含先後順序(有相依性)，所以需要隔離出來單獨測試時，兩個Methods都一定要被隔離出來。

另外TestNG也提供檢測一定要發生的錯誤，或者平行測試、多執行緒測試等多種測試功能。

當進行過單元測試 `mvn test` 後，我們就可以用瀏覽器開啟下列檔案，查看測試報告。
```text
 ./target/surefire-reports/index.html
```
