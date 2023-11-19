# 軟體品質保證
常常聽到「這個程式，原本還好好的，怎麼又出錯了？」，咎其原因，無非是改A錯B，或是有人更動了Jar檔。

又或者程式品質不佳，在特殊的狀況下，漏洞剛好被觸發。

您能想像回頭去改多年前的程式，是多個可怕的一件事?可能只是改個小地方，升級個Jar檔就會碰觸到上面的雷。

所以上述的問題，就出在QA沒做好。

QA除了大量的測試程式外，Maven也有多種配合的外掛來幫忙管控程式品質。

<span style="color:red">在開始之前，除非您是用純文字模式開發Java，否則強列建議上網搜尋SonarLint Pluing或者到[官網](https://rules.sonarsource.com/java/)按下`Analyze you code`，;安裝適合您的編輯工具的外掛，可以用來幫忙找出Java程式碼中的錯誤、漏洞、安全熱點或者寫的不太好的地方。</span>

另外，[sonarsource](https://www.sonarsource.com/)也提供了源碼檢測工具[sonarqube](https://www.sonarsource.com/products/sonarqube/)，可以到此[下載](https://www.sonarsource.com/products/sonarqube/downloads/)社群版為您的程式進行源碼檢測。
