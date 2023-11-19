# 識別管理

Maven的識別管理，通常分為三層 groupId:artifactId:version[:classifiers]。

一個群組(**group**) 可能存在多個函式專案(**atrifact**)，

而每個函式專案(**atrifact**)也可能存在多個版本(**version**)。

整個函式庫就以這種檔檔案架構進行處理，當專案參而使用的函式不存在本地端時，Maven會到 [repo1.maven.org/maven2/](https://repo1.maven.org/maven2/) 或 [https://repo.maven.apache.org/maven2/](https://repo.maven.apache.org/maven2/) 進行下載，然後存到您電腦上的函式庫。

*註*: 第四層的**classifiers**很少用到，通常會用到的清況是分不同系統的版本，例如 windows、macOs…，而且Maven對**classifiers**並無定義，所以可以是任何文字。

## 函式查找
Maven對於相依的函式庫稱為Dependency，當我們知道使用了某個函式，但又不太清楚完整的函式的專案識別，
就可以開啟官方的搜尋網站 [search.maven.org](https://search.maven.org/?eh=)，然後使用關鍵字查找。

例如我們知道Excel的函式專案是 poi ，直接在搜尋列上輸入**poi**，預設會下拉出符合的函式專案侯選。

若是我們很明確知道提供該函式的群組名稱是**org.apache.poi**，那麼我們可以直接在搜尋列上鍵入**org.apache.poi:poi**
或是**g:org.apache.poi AND a:poi**。

又例如，我們看到程式引用某個class，例如**org.apache.poi.ss.usermodel.Workbook**，卻不知道是源自哪裡。
這時我們可以在搜尋列上輸入**fc:org.apache.poi.ss.usermodel.Workbook**進行查找。

若是上述情況下，我們想確定版本**5.2.4**是否包含該類別，可以輸入**fc:org.apache.poi.ss.usermodel.Workbook AND v:5.2.4**進行查找。
