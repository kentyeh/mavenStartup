# 相依函式庫查驗

不時聽聞Java相關的Library有發現弱點，所以網路上已經有整理完整已發現的[弱點](https://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=java)，為了掃描我們專案所引用的相依函式，所以可以加入dependency-check-maven

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.owasp</groupId>
            <artifactId>dependency-check-maven</artifactId>
            <version>9.0.9</version>
        </plugin>
    </plugins>
</build>
```

然後執行下列指令進行檢查

```sehll
mvn dependency-check:check
```

第一次會下載資料庫會非常久(差不多20分鐘)，建議直接從已下載的電腦中去搬(位置在 $M2_HOME/org/owasp/dependency-check-data)。

當執行完成後，用瀏覽器開啟 target/dependency-check-report.html 查看結果。

建議詳看內容，因為有些弱點只發生在特定的JDK版本。
