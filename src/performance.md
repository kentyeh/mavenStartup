# 執行效率
這一節已經跟maven的設定無關了，純粹是為了加速執行 maven 的說明：

Maven 3開始可以使用[平行處理](https://cwiki.apache.org/confluence/display/MAVEN/Parallel+builds+in+Maven+3)的方式來加速整個執行效率，
在CPU多核應用的情況下可以在執行時，提供如下列的參數:
```shell
mvn -T 4 clean package #以4個執行緒建置
mvn -T 1C clean package #每一CPU核心以1個執行緒建置
mvn -T 1.5C clean package #每一CPU核心以1.5個執行緒建置
```
一般來說可提升20-50%的效率, 唯一要考量的是Plugin是否執行緒安全， 所幸絕大部分都是執行緒安全，我的建議是以 -T 1C 的參數是比較合適。
