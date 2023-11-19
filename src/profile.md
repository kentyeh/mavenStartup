# Profile

有時候我們必須依環境不同，而有不同的作法，所以可以用Profile將各個Plugin作區別設定， 然後以 "mvn -P設定" 來決定要如何執行特定的Plugin。 例如在Andord開發的時候，用模擬器測試的時候，為加快速度，根本不必考慮[簽署](https://developer.android.com/studio/publish/app-signing?hl=zh-tw)apk的問題，但是發佈App時則一定要簽署才行。 所以我們設定如下:

``` xml
<project ...>
    <profiles>
        <profile>
            <id>sign</id><!--我們自行命名的Profile-->
            <build>
                <plugins>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-jarsigner-plugin</artifactId>
                        <version>3.0.0</version>
                        <executions>
                            <execution>
                                <id>signing</id><!--為執行的作業命名-->
                                <goals>
                                    <goal>sign</goal><!--我們要執行的Goal-->
                                </goals>
                                <phase>package</phase><!--在Package的Phase執行這個Goal-->
                                <inherited>true</inherited>
                                <configuration>
                                    <archiveDirectory></archiveDirectory>
                                    <includes>
                                        <include>${project.build.directory}/*.apk</include><!--要簽署的apk-->
                                    </includes>
                                    <keystore>${somewhare}/keystorefile</keystore><!--key store檔案的所在路徑-->
                                    <storepass>key-store_password</storepass><!--Key sotre 的密碼--->
                                    <!--keypass>密碼</keypass --> <!-如果使用的key有密碼的話--->
                                    <alias>key's alias name</alias><!--加簽所使用的 alias-->
                                </configuration>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
        </profile>
    </profiles>
</project>
```

所以最後要建立發佈的App時，只要執行 "mvn -Psign install "就可以為建立的apk檔案加簽了。

另外的例子如Web程式在開發時用的是Window的環境，而發佈主機則是Linux環境，因為我們須要在環境內放一個system.properties 來為不同的環境設定不同的參數， 例如我們在專案下開了兩個目錄TomcatEnv?與JbossEnv?，裡面各放了一個system.properties，然後打打包專案時，才指定要用那一個檔案。

``` xml
<project ...>
    <profiles>
        <profile>
        <id>product</id><!--我們自行命名的Profile-->
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <build>
          <plugins>
            <plugin>
              <groupId>org.apache.maven.plugins</groupId>
              <artifactId>maven-war-plugin</artifactId>
              <version>3.4.0</version>
              <configuration>
                <webResources>
                  <resource>
                    <directory>productEnv</directory><!--引用的資源目錄-->
                    <targetPath>WEB-INF/classes</targetPath><!--打包時拷貝的目的目錄-->
                  </resource>
                </webResources>
              </configuration>
            </plugin>
          </plugins>
        </build>
        </profile>
        <profile>
        <id>develop</id><!--我們自行命名的Profile-->
        <build>
          <plugins>
            <plugin>
              <groupId>org.apache.maven.plugins</groupId>
              <artifactId>maven-war-plugin</artifactId>
              <version>3.4.0</version>
              <configuration>
                <webResources>
                  <resource>
                    <directory>developEnv</directory><!--引用的資源目錄-->
                    <targetPath>WEB-INF/classes</targetPath><!--打包時拷貝的目的目錄-->
                  </resource>
                </webResources>
              </configuration>
            </plugin>
          </plugins>
        </build>
        </profile>
    </profiles>
</project>
```

上述只是示範，通常為了達成同樣的目的，我們只會用Maven本身的過濾功能(參見之前的[目錄架構](archetype.md)):

```xml
<project>
  <profiles>
    <profile>
      <id>product</id>
      <activation>
        <activeByDefault>true</activeByDefault>
      </activation>
      <properties>
        <spring.profiles.active>product</spring.profiles.active><!--配合Spring的@Profile-->
        <other4filter>d:\attach\<other4filter>
      </properties>
    </profile>
    <profile>
      <id>develop</id>
      <properties>
        <spring.profiles.active>develop</spring.profiles.active>
        <other4filter>${project.build.directory}/attach/<other4filter>
      </properties>
    </profile>
  </profiles>
  <build>
      <resources>
        <resource>
          <directory>${project.basedir}/src/main/resources</directory>
          <filtering>true</filtering>
        </resource>
      <</resources>
  </build>
</project>
```

所以當我們在開發環境時，執行Maven 總要加上 `mvn -Pdevelop xxx`，而要打包正式佈署的程式時，只要執行 `mvn package` 就可以了。
