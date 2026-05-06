# Practice 8 Coverage Report

## 1. 作业目标

Practice 8 要求在 Teedy 项目中使用 JaCoCo 生成测试覆盖率报告，并新增测试用例，使 instruction coverage 和 branch coverage 都比原始测试套件更高。

我选择在 `docs-core` 模块补充测试，因为这个模块已有 JUnit 4 测试，并且 `LocaleUtil`、`MimeTypeUtil` 是纯工具类，不依赖数据库或 Web 容器，适合做稳定的覆盖率提升演示。

## 2. 修改内容

### 2.1 配置 JaCoCo

在根目录 `pom.xml` 中加入 JaCoCo Maven Plugin：

```xml
<plugin>
  <groupId>org.jacoco</groupId>
  <artifactId>jacoco-maven-plugin</artifactId>
  <version>${org.jacoco.jacoco-maven-plugin.version}</version>
  <executions>
    <execution>
      <goals>
        <goal>prepare-agent</goal>
      </goals>
    </execution>
    <execution>
      <id>report</id>
      <phase>test</phase>
      <goals>
        <goal>report</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

说明：教程里使用的是 JaCoCo `0.8.9`。我本机 Java 是 `22.0.2`，`0.8.9` 会对 Java 22 的 class file 产生兼容性报错，所以实际配置为 `0.8.12`。

另外，原来的 `pom.xml` 中有一个误插入的第二个 `<dependencies>` 标签，导致 Maven 报 `Duplicated tag: 'dependencies'`，我先把它移除，保证项目可以解析和测试。

### 2.2 新增测试

新增测试文件：

```text
docs-core/src/test/java/com/sismics/util/TestJsonLocaleUtil.java
```

测试覆盖的代码：

- `LocaleUtil.getLocale(String localeCode)`
- `MimeTypeUtil.getFileExtension(String mimeType)`

测试设计：

- `LocaleUtil` 覆盖空值、空字符串、只有 language、有 language_country、有 language_country_variant 的路径。
- `MimeTypeUtil.getFileExtension` 覆盖所有已有 `case` 分支和 `default` 分支。
- 这些测试不依赖数据库、文件系统 MIME 探测、外部服务或 Web 容器，现场演示比较稳定。

## 3. 运行新增测试

单独执行新增测试类：

```powershell
mvn -pl docs-core -Dtest=TestJsonLocaleUtil test
```

结果：

```text
Tests run: 4, Failures: 0, Errors: 0, Skipped: 0
BUILD SUCCESS
```

这一步对应作业要求中的 `mvn -Dtest=YourNewTestClass test`。

## 4. 覆盖率对比

为了让前后数据可比，我使用同一个 JaCoCo 版本，并清理 `docs-core/target/jacoco.exec` 后分别生成报告。

### 4.1 原始覆盖率

命令：

```powershell
Remove-Item -LiteralPath docs-core\target\jacoco.exec -ErrorAction SilentlyContinue
mvn -pl docs-core test jacoco:report "-Dmaven.test.failure.ignore=true" "-Dtest=!TestJsonLocaleUtil"
```

原始覆盖率，也就是排除我新增测试后的覆盖率：

| Metric | Covered / Total | Coverage |
| --- | ---: | ---: |
| Instruction | 5026 / 20585 | 24.42% |
| Branch | 208 / 1150 | 18.09% |

原始报告位置：

```text
docs-core/target/site/jacoco/index.html
```

我也保存了一份 CSV：

```text
docs-core/target/site/jacoco/jacoco-baseline-without-TestJsonLocaleUtil.csv
```

### 4.2 新覆盖率

命令：

```powershell
Remove-Item -LiteralPath docs-core\target\jacoco.exec -ErrorAction SilentlyContinue
mvn -pl docs-core test jacoco:report "-Dmaven.test.failure.ignore=true"
```

加入新增测试后的覆盖率：

| Metric | Covered / Total | Coverage | Increase |
| --- | ---: | ---: | ---: |
| Instruction | 5071 / 20585 | 24.63% | +0.21 pp |
| Branch | 223 / 1150 | 19.39% | +1.30 pp |

新报告位置：

```text
docs-core/target/site/jacoco/index.html
```

我也保存了一份 CSV：

```text
docs-core/target/site/jacoco/jacoco-final-with-TestJsonLocaleUtil.csv
```

## 5. 现场给老师讲解流程

可以按下面顺序讲：

1. 先说明我阅读了 Practice 8，任务是用 JaCoCo 看原始覆盖率，然后新增测试，让 instruction 和 branch coverage 都提升。
2. 打开 `pom.xml`，指出我加入了 `jacoco-maven-plugin`，并解释 `prepare-agent` 负责收集测试执行数据，`report` 负责生成 HTML 覆盖率报告。
3. 说明我本机是 Java 22，所以把教程中的 JaCoCo `0.8.9` 调整为 `0.8.12`，否则会出现 class file version 兼容问题。
4. 打开 `TestJsonLocaleUtil.java`，讲测试目标：
   - `LocaleUtil.getLocale` 有 null/empty、language、country、variant 分支。
   - `MimeTypeUtil.getFileExtension` 是 switch 分支，新增测试覆盖所有已知 MIME 类型和 default 分支。
5. 执行 `mvn -pl docs-core -Dtest=TestJsonLocaleUtil test`，展示新增测试 4 个 case 全部通过。
6. 打开 JaCoCo 报告 `docs-core/target/site/jacoco/index.html`，对比前后数据：
   - Instruction coverage: 24.42% -> 24.63%
   - Branch coverage: 18.09% -> 19.39%
7. 最后说明完整 Teedy 原始测试在我的环境中有少数既有失败，所以我使用了教程建议的 `-Dmaven.test.failure.ignore=true` 让 JaCoCo 继续生成报告；新增测试本身是单独通过的。

## 6. 已知情况

完整 `docs-core` 测试在当前环境下存在 3 个既有失败：

- `TestMimeTypeUtil.test`：Windows/JDK MIME 探测把 CSV 识别为 `application/vnd.ms-excel`，而测试期望 `text/csv`。
- `TestFileUtil.extractContentScannedPdf`：`TransactionRequiredException`。
- `TestPdfFormatHandler.testIssue373`：`TransactionRequiredException`。

这些失败在排除/加入新增测试的两次 JaCoCo 运行中都存在，不影响本次新增测试带来的 coverage 对比。新增测试命令单独执行时是通过的。
