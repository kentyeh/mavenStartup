# 建立第一個Project

在命令視窗執行 `mvn archetype:generate` 命令，使用互動方式建立Project， 會依序問幾個問題

| | |
| --- | --- |
| Choose artifactId: | 選擇建立Project的範本，預設是maven-archetype-quickstart建立一個最基本的Project |
| 定義groupId: | 輸入要建立Project所隸屬的組織或公司，如我自已用idv.kentyeh.software |
| 定義artifactId: | 就是Project名稱，例如 firstMaven |
| 定義version: | Project的版本號，預設是1.0-SNAPSHOT |
| 定義package: | 初始建立的Java Package, 如 idv.kentyeh.software |

確定後建立Project的基本架構，如果您不要用互動的方式，上述動作可以以下指令完成相同的事

Linux 下執行

```shell
mvn archetype:generate -DgroupId=idv.kentyeh.software -DartifactId=firstmaven \
      -DpackageName=idv.kentyeh.software -Dversion=1.0-SNAPSHOT \
      -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=No \
      -DarchetypeCatalog=https://repo.maven.apache.org/maven2/archetype-catalog.xml
```
Windows 下執行(命令模式下的換行碼與Linux不同)

```shell
mvn archetype:generate -DgroupId=idv.kentyeh.software -DartifactId=firstmaven ^
      -DpackageName=idv.kentyeh.software -Dversion=1.0-SNAPSHOT ^
      -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=No ^
      -DarchetypeCatalog=https://repo.maven.apache.org/maven2/archetype-catalog.xml
```

最後若要編譯並建置整個專案，只要執行

```shell
mvn package
```

打開專案下的 target目錄，就會發現Maven已經幫我們建置好整個專案。
