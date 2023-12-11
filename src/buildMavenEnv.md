# 建立Maven環境

0. 先把 JDK 裝好。

1. 下載[Maven](http://maven.apache.org/download.html)後，解開到目錄，例如 x:\maven\

2. 決定您電腦的函式庫(Maven 把所有Libery集中管理)所在，預設在 用戶目錄下的 ~/.m2，若要變更不同位置，修改 x:\maven\conf\settings.xml的<localRepository>設定到您存放第三方函式庫的目錄，例如: y:\mvnrepo

    ```xml
    <settings xmlns="http://maven.apache.org/SETTINGS/1.2.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.2.0 http://maven.apache.org/xsd/settings-1.2.0.xsd">
        <localRepository>y:\mvnrepo</localRepository>
    </settings>
    ```
3. 設定環境變數 M2_HOME 指到 Maven目錄，並且將 Maven目錄下的 bin加到執行路徑的PATH變數

4. 到命令模式下，執行下列指令並確認Maven版本輸出，則安裝完成：

    ```shell
    mvn -version
    Apache Maven 3.9.6
    ```

