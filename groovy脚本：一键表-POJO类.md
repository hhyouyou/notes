# groovy脚本：一键表->POJO类



分享一个小脚本，在定义完表结构后，可以一键生成对应的实体类。





## 使用方式

1. 可以把脚本放在idea的这个位置：





2. 后面使用的话，在idea右侧的`Database` 栏中，选中对应的表，右键选择对应脚本名称即可。



## 脚本内容

整体的思路比较简单

1. 获取SQL表信息，提取列名/类型/comment等需要的信息。
2. 按java 中类格式，拼接后输出成文件。（需要import依赖的，需要加注解的都加上）



> 其中有部分注释，如果不清楚可以留言评论，我会继续补充解释。有问题的话，可以共同进步。

内容如下：

```groovy
import com.intellij.database.model.DasTable
import com.intellij.database.util.Case
import com.intellij.database.util.DasUtil

import java.text.SimpleDateFormat

/*
 * Available context bindings:
 *   SELECTION   Iterable<DasObject>
 *   PROJECT     project
 *   FILES       files helper
 */

packageName = ""
author = "hhyy"

// 类型映射，mysql中的类型，对应java中的类型
typeMapping = [
        (~/bigint/)                       : "Long",
        (~/(?i)int/)                      : "Integer",
        (~/(?i)float|double|decimal|real/): "Double",
        (~/(?i)decimal/)                  : "BigDecimal",
        (~/(?i)datetime|timestamp|time/)  : "LocalDateTime",
        (~/(?i)bool|boolean/)             : "Boolean",
        (~/(?i)date/)                     : "LocalDate",
        (~/(?i)char|text/)                : "String",
        (~/(?i)/)                         : "String"
]

FILES.chooseDirectoryAndSave("Choose directory", "Choose where to store generated files") { dir ->
    SELECTION.filter { it instanceof DasTable }.each { generate(it, dir) }
}

def generate(table, dir) {
    def className = javaName(table.getName(), true)
    def fields = calcFields(table)
    packageName = getPackageName(dir)
    new File(dir, className + "Entity.java").withPrintWriter { out -> generate(out, table, className, fields) }
}

def generate(out, table, className, fields) {

    SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss")
    def nowTime = format.format(new Date())
    def tableName = table.getName()

    out.println "package $packageName"
    out.println ""
    out.println "import lombok.Data;"
    out.println ""
    out.println "import javax.persistence.*;"
    out.println "import java.io.Serializable;"
    out.println "import java.time.LocalDateTime;"
    out.println "import java.math.BigDecimal;"
    out.println ""
    out.println ""
    out.println "/** "
    out.println " * @author $author"
    out.println " * @date $nowTime"
    out.println "**/"
    out.println "@Data"
    out.println "@Entity"
    out.println "@Table(name = \"$tableName\")"
    out.println "public class ${className}Entity implements Serializable {"
    out.println ""
    fields.each() {
        if (it.annos != "") out.println "  ${it.annos}"
        // 输出注释
        if (isNotEmpty(it.comment)) {
            out.println "\t/** "
            out.println "\t * $it.comment"
            out.println "\t**/"
        }
        if (it.name == "id") {
            // 判断自增
            if ((tableName + "_id").equalsIgnoreCase(fields[0].colum) || "id".equalsIgnoreCase(fields[0].colum)) {
                out.println "\t@Id"
                out.println "\t@GeneratedValue(strategy = GenerationType.IDENTITY)"
            }
        }
        out.println "    @Column(name = \"${it.colum}\", columnDefinition = \"${it.sqlType}\")"
        String colName = it.name

        // is_开头的列
        if (colName.startsWith("is")) {
            it.name = colName.substring(2).toLowerCase()
        }
        out.println "\tprivate ${it.type} ${it.name};"
        out.println ""
    }
    out.println "}"
}

// 处理表中的信息
def calcFields(table) {
    DasUtil.getColumns(table).reduce([]) { fields, col ->
        def spec = Case.LOWER.apply(col.getDataType().getSpecification())
        def typeStr = typeMapping.find { p, t -> p.matcher(spec).find() }.value
        // sql 字段类型处理
        String sqlTypeStr = spec;
        if (sqlTypeStr != null && !sqlTypeStr.isEmpty() && sqlTypeStr.contains("(")) {
            sqlTypeStr = sqlTypeStr.substring(0, sqlTypeStr.indexOf("("));
        }
        fields += [[
                           name   : javaName(col.getName(), false),
                           colum  : col.getName(),
                           type   : typeStr,
                           sqlType: sqlTypeStr,
                           comment: col.getComment(),
                           annos  : ""]]
    }
}

// sql列名转java列明， 主要是一个驼峰的转换
def javaName(str, capitalize) {
    def s = com.intellij.psi.codeStyle.NameUtil.splitNameIntoWords(str)
            .collect { Case.LOWER.apply(it).capitalize() }
            .join("")
            .replaceAll(/[^\p{javaJavaIdentifierPart}[_]]/, "_")
            .replaceAll(/_/, "")
    capitalize || s.length() == 1 ? s : Case.LOWER.apply(s[0]) + s[1..-1]
}

/**
 * 获取包名称
 * @param dir 实体类所在目录
 * @return
 */
static def getPackageName(dir) {
    String absolutePath = dir.toString()
    def replace = absolutePath.substring(absolutePath.indexOf("java\\") + 5).replace("\\", ".");
    return replace + ";"
}

static def isNotEmpty(content) {
    return content != null && content.toString().trim().length() > 0
}
```





## End

在理解了groovy的语法之后（和Java相似），可以根据需要的方式，修改脚本。比如一些公司实体类可能有固定的规范，需要添加一些东西。比如不用`lombok`, 自定义`get set`方法都可以，只要有了列信息，都可以做。



**将重复的工作，以脚本的形式固化下来，将大大节省时间精力。**

**小到一个定时shell脚本，再到我们编写的程序、软件都是在做这个工作，简化，再简化。**

**“懒惰”确实是人类进步的一大动力。**



