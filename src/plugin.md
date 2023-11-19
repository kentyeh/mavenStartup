# 外掛

一開始的時候，我們執行 "mvn archetype:generate" 建立專案。

mvn 後面接的指令 叫 goal由 "前置字:作業標的" 表示要執行的作業。

這些執業作業(命令)是由所謂的plugin(外掛)所提供，plugin是一種專供 Maven 用於建置專案所使用的Library，同一個 plugin 使用同一前置字；

Maven本身[早已內建](http://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Built-in_Lifecycle_Bindings)許許多多的goal的定義，當Maven執行特定的 goal時，缺少的Plugin，Maven就會自動去Repository下載相關的檔案。

Maven 的指令可以串接，例如以下指令

``` shell
mvn clean package javadoc:javadoc
```

很直白，清除已有之建置，然後打包再產出Java API文件。

上述的 **clean**、**package**(打包）都很容易理解，可是為何又出現了 **javadoc:javadoc** ?

前面已經說了，Maven本身[早已內建](http://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Built-in_Lifecycle_Bindings)許許多多特殊的goal，換個說法就是：每個釋出Maven版本，本身已經關聯了許多的外掛，當您執行 `mvn compile` 時，即使pom.xml沒有宣告**maven-compiler-plugin**, 只要設定好相關的**property**，Maven也會自動正確地執行相關版本的**maven-compiler-plugin**外掛(不一定是最新的版本)。

同樣的，當執行`mvn clean`時，也必定有一版相關的[**maven-clean-plugin**](https://maven.apache.org/plugins/maven-clean-plugin/)被叫用。只不過通常用來清除**target**目錄外，很少有額外的目錄需要清除，所以也很少看到有人特別針對這個外掛做設定。

舅多媽爹!`mvn clean`或`mvn compile`這兩個goal都不是"前置字:作業標的"的形式啊!

那是因為上述的goal是屬於生命周期(Life Cycle，另外在生命週期章節中解說)之Phase(階段)的一部分，所以**前置字**被省略了，當然，您要執行 `mvn clean:clean`也不會有人反對。

也許有人會問，那我自行開發的外掛要怎麼執行?

Maven也提供了如何解析[前置字](https://maven.apache.org/guides/introduction/introduction-to-plugin-prefix-mapping.html)的規則，
要不然就只能用 `mvn 識別管理:版本:作業標的`的方式來執行外掛。

例如上述的**javadoc:javadoc**可以用下列方式執行(我想應該很少有人有特殊的需求需要自行開發外掛)：

```shell
mvn org.apache.maven.plugins:maven-javadoc-plugin:3.6.0:javadoc
```

再來介紹一些常用的外掛。
