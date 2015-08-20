開始使用Maven是因為受夠了每次開始一個Project，都要重新開始撰寫
Ant build.xml，雖然有過去建立的範本，但是還是的根據不同狀況加以
改寫(比如不同的打包方式，如同時deploy 到tomcat 與jboss時用到不
同的log4J設定，或是像GWT必須加上一道Comple手續);當有新版本的
Liberary出來的時候，還得忙著下載新Jar，然後發現不相容時，還得改回來…
Maven最大的好處，就是把上面的無聊事標準化，簡單化了，用久了，還真得
少不了它。

#建立Maven環境
1. 下載Mav[](http://maven.apache.org/download.html)en後，解開到目錄，例 x:\maven\ 
2. 決定您電腦的函式庫(Maven 把所有Libery集中管理)所在，預設在 用戶目錄/.m2，若要變更不同位置，修改 x:\maven\conf\settings.xml的<localRepository>設定 
3. 設定環境變數 M2_HOME 指到 Maven目錄，並且將 Maven目錄下的 bin加到執行路徑

#用Maven建立第一個Project
在命令視窗執行 mvn archetype:generate 命令，使用互動方式建立Project， 會依序問幾個問題 


| Choose archetype: | 選擇建立Project的範本，預設是99:maven-archetype-quickstart建立一個最基本的Project |
| -- | -- |
| Choose version: | 選擇範本的版本，會列出一些範本可用的版本，其差異是就不用版本的範本可能會建立有不同的資源檔(比如可能附帶圖檔) |
| 定義groupId: | 輸入要建立Project所隸屬的組織或公司，如我自已用idv.kentyeh.software | 
| 定義artifactId: | 就是Project名稱，例如 firstMaven |
| 定義version: | Project的版本號，預設是1.0-SNAPSHOT |
| 定義package: | 初始建立的Java Package, 如 idv.kentyeh.software |

確定後建立Project的基本架構，如果您不要用互動的方式，上述動作可以以下指令完成相同的事 

```
mvn archetype:create -DgroupId=idv.kentyeh.software -DartifactId=firstmaven \
      -DpackageName=idv.kentyeh.software -DarchetypeArtifactId:maven-archetype-quickstart \
      -Dversion=1.0-SNAPSHOT
```

#Maven的識別管理

Maven的識別管理，分為三層 groupId:artifactId:version，一個組織(group) 可能存在多個Project(atrifact)，每個Project也可能存在多個版本(version)， 整個函式庫就以這種檔檔案架構進行處理，當使用的函式不存在時，Maven會到 http://repo1.maven.org/maven2/ 或是 http://repo2.maven.org/maven2/ 進行下載，下載後存到您電腦上的函式庫 

