# SPARQL 1.1查询语言
 W3C 推荐 2013年3月21日
# 摘要
RDF是一种定向的，有标记的图形数据格式，用于表示Web中的信息。此规范定义了RDF的SPARQL查询语言的语法和语义。SPARQL可用于表示跨不同数据源的查询，无论数据是本机存储为RDF还是通过中间件查看为RDF。SPARQL包含查询必需和可选图形模式及其连接和析取的功能。SPARQL还支持聚合、子查询、否定、通过表达式创建值、可扩展值测试以及通过源RDF图限制查询。SPARQL查询的结果可以是结果集或RDF图。

# 1 介绍
RDF是一种定向的，有标记的图形数据格式，用于表示Web中的信息。RDF通常用于表示个人信息，社交网络，有关数字工件的元数据，以及提供跨不同信息源的集成方式。此规范定义了RDF的SPARQL查询语言的语法和语义。

SPARQL查询语言旨在满足RDF Data Access Working Group在 RDF Data Access Use Cases and Requirements [UCNR]和SPARQL New Features and Rationale [UCNR2]中的用例和需求。

## 1.1 文本大纲
除非在标题中另有说明，否则本文中的所有章节和附录均为规范性的。

第1章介绍SPARQL查询语言规范。介绍文档的组织以及整个文档中使用的约定。
第2章通过一系列示例查询和查询结果介绍SPARQL查询语言。第3章继续介绍SPARQL查询语言，其中包含更多示例，这些示例演示SPARQL能够表达对查询结果中出现的RDF项的约束。
第4章介绍SPARQL查询语言的语法细节。定义用于表示IRIs，空白节点，文字和变量的语法结构。除此之外，还定义几个语法结构，这些结构作为更详细表达式的语法糖。

第5章介绍基本图模式和组图模式，用于构建更为复杂的SPARQL查询模式。第6,7和8章介绍将SPARQL图形模式组合成更大图形模式的结构。特别是，第6章介绍使查询的某些部分可选的功能;第7章介绍表达替代图模式分离的能力;第8章介绍测试信息缺失的模式。

第9章添加图形模式匹配的属性路径，给出查询的紧凑表示，以及匹配图中任意长度路径的能力。

第10章描述SPARQL中可能的赋值形式。

第11章介绍分组和汇总结果机制，可以将其作为子查询合并到第12章中。

第13章介绍将查询的某些部分约束到特定源图的能力。还介绍SPARQL定义查询源图的机制。

第14章引用单独的文档SPARQL 1.1 Federated Query。

第15章定义通过对一系列解决方案进行排序，切片，投影，限制和删除重复项来影响查询解决方案的结构。

第16章定义以不同形式产生结果的四种类型的SPARQL查询。

第17章定义SPARQL的可扩展值测试和表达框架。它提供可用于约束查询结果中出现的值的函数和运算符，还可以计算查询返回的新值。

第18章是SPARQL图模式和解决方案修改器评估的正式定义。

第19章包含SPARQL查询和SPARQL更新语言的语法的规范性定义，由EBNF表示法表示的语法给出。

## 1.2 文本约定
### 1.2.1 命名空间
在本文档中，示例假设以下命名空间前缀绑定，除非另有说明：

| Prefix | IRI |
| ----- | ----- |
|rdf: |http://www.w3.org/1999/02/22-rdf-syntax-ns# |
|rdfs: |http://www.w3.org/2000/01/rdf-schema# |
|xsd: |http://www.w3.org/2001/XMLSchema# |
|fn: |http://www.w3.org/2005/xpath-functions# |
|sfn: |http://www.w3.org/ns/sparql# |

### 1.2.2 数据描述
本文档使用Turtle [TURTLE]数据格式显示每个三元组。Turtle允许IRIs缩写为前缀：

```html
@prefix dc: <http://purl.org/dc/elements/1.1/> .
@prefix : <http://example.org/book/> .
:book1 dc:title "SPARQL Tutorial" .
```


### 1.2.3 结果描述
结果集以表格形式说明。

| x | y | z |
| ----- | ----- | ----- |
| "Alice" | <http://example/a> |  |

“绑定”是一对（变量，RDF术语）。在此结果集中，有三个变量：x，y和z（显示为列标题）。每个解决方案在表格的主体中显示为一行。这里有一个解决方案，其中变量x绑定到“Alice”，变量y绑定到<example：//example/a>，变量z不绑定到RDF术语。变量不需要在解决方案中绑定。

### 1.2.4 术语
SPARQL语言包括IRIs，它是省略空格的RDF URI引用的子集。注意，SPARQL查询中的所有IRIs都是绝对的; 它们可能包含也可能不包含片段标识符[RFC3987，第3.1节]。IRIs包括URIs[RFC3986]和URLs。解析SPARQL语法中的缩写形式（相对IRIs和前缀名称）以生成绝对IRIs。

以下术语在RDF Concepts and Abstract Syntax [CONCEPTS] 中定义并在SPARQL中使用：
> * IRI
> * literal
> * lexical form
> * plain literal
> * language tag
> * typed literal
> * datatype IRI
> * blank node

此外，我们定义以下术语：
> * RDF Term 包括 IRIs、blank nodes和literals
> * Simple Literal 覆盖不含language tag或datatype IRI的literals

# 2 简单查询
大多数形式的SPARQL查询都包含一组称为基本图形模式的三元组模式。三元组模式类似于RDF三元组，除了主语，谓语和对象中的每一个都可以是变量。当来自该子图的RDF项可以替换变量并且结果是RDF图等效于子图时，基本图模式匹配RDF数据的子图。

## 2.1 写一个简单的查询
下面的示例显示一个SPARQL查询，用于从给定的数据图中查找书籍的标题。该查询由两部分组成：SELECT子句标识要在查询结果中显示的变量，WHERE子句提供与数据图表匹配的基本图形模式。 此示例中的基本图形模式由单个三元组模式组成，在对象位置具有单个变量（？title）。

```html
Data:

<http://example.org/book/book1> <http://purl.org/dc/elements/1.1/title> "SPARQL Tutorial" .

```

```html
Query:
SELECT ?title
WHERE
{
  <http://example.org/book/book1> <http://purl.org/dc/elements/1.1/title> ?title .
}
```

这个查询针对上面数据有一个结果：

|title|
|-----|
|“SPARQL Tutorial”|

## 2.2 多个匹配

查询的结果是解决方案序列，对应于查询的图形模式与数据匹配的方式。查询可能有零个，一个或多个解决方案。

```html
Data:
@prefix foaf:  <http://xmlns.com/foaf/0.1/> .

_:a  foaf:name   "Johnny Lee Outlaw" .
_:a  foaf:mbox   <mailto:jlow@example.com> .
_:b  foaf:name   "Peter Goodguy" .
_:b  foaf:mbox   <mailto:peter@example.org> .
_:c  foaf:mbox   <mailto:carol@example.org> .

```

```html
Query:
PREFIX foaf:   <http://xmlns.com/foaf/0.1/>
SELECT ?name ?mbox
WHERE
  { ?x foaf:name ?name .
    ?x foaf:mbox ?mbox }
```

Query Result:

| name | mbox |
| ----- | ----- |
| "Johnny Lee Outlaw" | <mailto:jlow@example.com> |
| "Peter Goodguy" | <mailto:peter@example.org> |

每种解决方案都提供一种方法，可以将所选变量绑定到RDF术语，以便查询模式与数据匹配。结果集提供了所有可能的解决方案。在上面的示例中，以下两个数据子集提供了两个匹配项。

```html
_:a foaf:name  "Johnny Lee Outlaw" .
 _:a foaf:box   <mailto:jlow@example.com> .
```

```html
 _:b foaf:name  "Peter Goodguy" .
 _:b foaf:box   <mailto:peter@example.org> .
```

这是一个基本的图形模式匹配;查询模式中使用的所有变量必须绑定在每个解决方案中。

## 2.3 匹配RDF字面量

以下数据包括三个RDF文字：

```html
@prefix dt:   <http://example.org/datatype#> .
@prefix ns:   <http://example.org/ns#> .
@prefix :     <http://example.org/ns#> .
@prefix xsd:  <http://www.w3.org/2001/XMLSchema#> .

:x   ns:p     "cat"@en .
:y   ns:p     "42"^^xsd:integer .
:z   ns:p     "abc"^^dt:specialDatatype .
```

注意到，在Turtle中"cat"@en是一个RDF字面量，具有词汇"cat"和语言标签"en";
"42"^^xsd:integer是一个http://www.w3.org/2001/XMLSchema#integer类型的字面量;
"abc"^^dt:specialDatatype是一个类型为http://example.org/datatype#specialDatatype的字面量。
该RDF数据是2.3.1-2.3.3节中查询示例的数据图。

### 2.3.1 匹配带有语言标签的字面量

SPARQL中的语言标签使用@和标签进行表示，如 Best Common Practice 47 [BCP47]中的定义。
以下查询没有查询结果，因为"cat"和RDF字面量"cat"@en不相同。

```html
SELECT ?v WHERE {?v ?p "cat"}
```

| v |
| ----- |
| |

但是，下面的查询会发现一个方案，变量v绑定到:x，因为语言标签被给定并匹配到给定的数据：


```html
SELECT ?v WHERE {?v ?p "cat@en"}
```

| v |
| ----- |
| <http://example.org/ns#x> |

### 2.3.2 匹配数字类型字面量

SPARQL查询中的整数表示RDF中类型为xsd:integer的字面量。例如，42是 "42"^^<http://www.w3.org/2001/XMLSchema#integer>的缩写形式。

下面的查询会发现一个方案，变量v绑定到:y。
```html
SELECT ?v WHERE {?v ?p 42}
```

| v |
| ----- |
| <http://example.org/ns#y>|

4.1.2小节定义了SPARQL中xsd:float和xsd:double中的缩写形式。

### 2.3.3 匹配任意数据类型字面量

以下查询具有一个解决方案将变量v绑定到:z。查询处理器不必理解数据类型空间的值。因为词法形式和数据类型IRI都匹配，所以字面量匹配。

```html
SELECT ?v WHERE {?v ?p "abc"^^<http://example.org/datatype#specialDatatype> }
```

| v |
| ----- |
| <http://example.org/ns#z> |

## 2.4 查询结果中的空节点标签

查询结果中可以包含空节点。本文档中示例结果集中的空节点以"_:"形式写入，后跟空节点标签。

空节点标签的范围限定为结果集（请参阅"SPARQL Query Results XML Format"和"SPARQL 1.1 Query Results JSON Format"），或者对于CONSTRUCT查询表单结果图。在结果集中使用相同的标签表示相同的空节点。

```html
Data

@prefix foaf:  <http://xmlns.com/foaf/0.1/> .

_:a  foaf:name   "Alice" .
_:b  foaf:name   "Bob" .
```


```html
Query：

PREFIX foaf:   <http://xmlns.com/foaf/0.1/>
SELECT ?x ?name
WHERE  { ?x foaf:name ?name }
```

| x | name |
| ------ | ----- |
|_:c | "Alice" |
|_:d | "Bob" |

上述结果同样可以用不同的空节点标签给出，因为结果中的标签仅表明解决方案中的RDF术语是相同还是不同。

| x | name |
| ----- | ------ |
| _:r | "Alice" |
| _:s | "Bob" |

这两个结果具有相同的信息：用于匹配查询的空节点在两个解决方案中是不同的。结果集中的标签_：a与具有相同标签的数据图中的空白节点之间不需要有任何关系。

应用程序编写者不应期望查询中的空节点标签指代数据中的特定空节点。

## 2.5 使用表达式创建值

SPARQL 1.1允许从复杂表达式创建值。下面的查询显示如何使用CONCAT函数连接foaf数据中的名字和姓氏，然后使用SELECT子句中的表达式分配值，并使用BIND表单分配值。

```html
Data:

@prefix foaf:  <http://xmlns.com/foaf/0.1/> .
          
_:a  foaf:givenName   "John" .
_:a  foaf:surname  "Doe" .

```
```html
Query:

PREFIX foaf:   <http://xmlns.com/foaf/0.1/>
SELECT ( CONCAT(?G, " ", ?S) AS ?name )
WHERE  { ?P foaf:givenName ?G ; foaf:surname ?S }
```

```html
Query:

PREFIX foaf:   <http://xmlns.com/foaf/0.1/>
SELECT ?name
WHERE  { 
   ?P foaf:givenName ?G ; 
      foaf:surname ?S 
   BIND(CONCAT(?G, " ", ?S) AS ?name)
}
```

|name|
|-----|
|"John Does"|

## 2.6 建立RDF图

SPARQL有几种查询形式。SELECT查询返回变量绑定。CONSTRUCT查询返回RDF图。该图是基于模板构建的，该模板用于基于匹配查询的图形模式的结果生成RDF三元组。

```html
Data:

@prefix org:    <http://example.com/ns#> .

_:a  org:employeeName   "Alice" .
_:a  org:employeeId     12345 .

_:b  org:employeeName   "Bob" .
_:b  org:employeeId     67890 .

```

```html
Query:

PREFIX foaf:   <http://xmlns.com/foaf/0.1/>
PREFIX org:    <http://example.com/ns#>

CONSTRUCT { ?x foaf:name ?name }
WHERE  { ?x org:employeeName ?name }

```

```html
Results:

@prefix foaf: <http://xmlns.com/foaf/0.1/> .
      
_:x foaf:name "Alice" .
_:y foaf:name "Bob" .

```

可以在RDF/XML序列化为:
```xml
<rdf:RDF
    xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
    xmlns:foaf="http://xmlns.com/foaf/0.1/"
    >
  <rdf:Description>
    <foaf:name>Alice</foaf:name>
  </rdf:Description>
  <rdf:Description>
    <foaf:name>Bob</foaf:name>
  </rdf:Description>
</rdf:RDF>

```
# 3 RDF术语约束

图模式匹配产生解决方案序列，其中每个解决方案都有一组变量绑定到RDF项。SPARQL中FILTER将解决方案限制为过滤器表达式计算结果为TRUE的解决方案。

本章提供对SPARQL FILTER的非正式介绍; 它们的语义在“表达式和测试值”一节中定义，其中有一个综合的函数库。 本节中的示例共享一个输入图：

```xml
Data:
@prefix dc:   <http://purl.org/dc/elements/1.1/> .
@prefix :     <http://example.org/book/> .
@prefix ns:   <http://example.org/ns#> .

:book1  dc:title  "SPARQL Tutorial" .
:book1  ns:price  42 .
:book2  dc:title  "The Semantic Web" .
:book2  ns:price  23 .
```

## 3.1 限制字符串的值

像regex这样的SPARQL FILTER函数可以测试RDF字面量。regex只匹配字符串字面量。通过使用str函数，可以使用regex匹配其他字面量的词法形式。

```xml
Query:

PREFIX  dc:  <http://purl.org/dc/elements/1.1/>
SELECT  ?title
WHERE   { ?x dc:title ?title
          FILTER regex(?title, "^SPARQL") 
        }
```


Result：

| title |
| ----- |
|"SPARQL Tutorial"|

正则表达式匹配可以使用“i”标志不区分大小写。

```html
Query：

PREFIX  dc:  <http://purl.org/dc/elements/1.1/>
SELECT  ?title
WHERE   { ?x dc:title ?title
          FILTER regex(?title, "web", "i" ) 
        }

```
查询结果：

|title|
|-----|
|"The Semantic Web"|

正则表达式语言被定义在XQuery 1.0 and XPath 2.0 Functions and Operators并且基于XML Schema Regular Expressions.

## 3.2 限制数字的值

SPARQL FILTER可以限制算术表达式。
```html
query：

PREFIX  dc:  <http://purl.org/dc/elements/1.1/>
PREFIX  ns:  <http://example.org/ns#>
SELECT  ?title ?price
WHERE   { ?x ns:price ?price .
          FILTER (?price < 30.5)
          ?x dc:title ?title . }
```

查询结果：

|title|price|
|------|-----|
|"The Semantic Web"|23|

通过限制price变量，只有:book2匹配了查询，因为只有:book2的价格低于30.5，如过滤条件需要。

## 3.3 其他项目限制

除数字类型外，SPARQL还支持xsd：string，xsd：boolean和xsd：dateTime类型（请参阅Operand Data Types）。Operator Mapping章描述运算符，Function Definitions描述可以应用于RDF项的函数。

# 4 SPARQL语法

本章介绍SPARQL用于RDF术语和三重模式的语法。完整的语法在第19章中给出。

## 4.1 RDF项语法

### 4.1.1 IRIs语法

iri生产指定了一组IRI [RFC3987]; IRIs是URIs的一般化[RFC3986]，与URIs和URLs完全兼容。PrefixedName生产指定一个带前缀的名称。从前缀名称到IRI的映射如下所述。IRI引用（相对或绝对IRI）由IRIREF生产指定，其中'<'和'>'分隔符不构成IRI参考的一部分。相对IRIs匹配2.2 ABNF章中针对IRI参考的irelative-ref参考和[RFC3987]中的IRI，并且如下所述被解析为IRIs。

RDF概念和抽象语法中定义的RDF术语集包括RDF URI引用，而SPARQL术语包括IRIs。包含“<”，“>”，“”（双引号），空格，“{”，“}”，“|”，“\”，“^”和“`”的RDF URI引用不是IRI。未定义SPARQL查询针对由此类RDF URI引用组成的RDF语句的行为。

#### 4.1.1.1 带前缀的名称

PREFIX关键字将前缀标签与IRI相关联。带前缀的名称是前缀标签和本地部分，用冒号":"分隔。通过连接前缀和本地部分关联的IRI，将前缀名称映射到IRI。前缀标签或本地部分可以为空。请注意，SPARQL本地名称允许前导数字，而XML本地名称则不允许。SPARQL本地名称还允许IRIs中非字母数字字符通过反斜杠字符转义（例如ns:id\=123）。SPARQL本地名称比CURIEs具有更多的语法限制。

#### 4.1.1.2 相对IRIs

相对IRIs与基本IRIs组合：通用语法[RFC3986]仅使用5.2节中的基本算法。既不执行基于语法的规范化也不执行基于方案的规范化（在RFC3986的6.2.2和6.2.3节中描述）。根据Internationalized Resource Identifiers (IRIs)[RFC3987]的第6.5节，IRI引用中另外允许的字符处理方式与URI引用中处理非保留字符的方式相同。

BASE关键字定义用于根据RFC3986第5.1.1节“内容中嵌入的基本URI”解析相对IRI的基本IRI。第5.1.2节“封装实体的基本URI”定义了基本IRI如何来自封装文档，例如带有xml：base指令的SOAP信封或带有Content-Location标头的mime多部分文档。 5.1.3中标识的“检索URI”，基础“检索URI中的URI”，是从中检索特定SPARQL查询的URL。 如果以上都不指定基URI，则使用默认的基本URI（第5.1.4节“默认基本URI”）。
 
以下片段是编写相同IRI的一些不同方法：

<http://example.org/book/book1>

BASE <http://example.org/book/>
<book1>

PREFIX book: <http://example.org/book/>
book:book1

## 4.1.2 字面量语法

字面量一般语法是一个字符串（用双引号括起来，“......”或单引号，'...'），带有可选的语​​言标记（由@引入）或可选的数据类型IRI或前缀名称（由^^引入）。

为方便起见，可以直接编写整数（不带引号和显式数据类型IRI），并将其解释为数据类型xsd：integer的类型字面量; 带有"."的十进制数字，在数字没有指数的是xsd:decimal类型; 带有指数的数字类型为xsd：double。xsd：boolean类型的值也可以写为true或false。

为了便于编写本身包含引号或长且包含换行符的字面量，SPARQL提供了一个额外的引用构造，其中字面量用三个单引号或双引号括起来。

SPARQL中的文字语法示例包括：

> * "chat"
> * 'chat'@fr带有语言标签"fr"
> * "xyz"^^<http://example.org/ns/userDatatype>
> * "abc"^^appNS:appDataType
> * '''The librarian said, "Perhaps you would enjoy 'War and Peace'."'''
> * 1与"1"^^xsd:integer相同
> * 1.3与"1.3"^^xsd:decimal相同
> * 1.300与"1.300"^^xsd:decimal相同
> * 1.0e6与"1.0e6"^^xsd:double相同
> * true与"true"^^xsd:boolean相同
> * false与"false"^^xsd:boolean相同

匹配产生INTEGER，DECIMAL，DOUBLE和BooleanLiteral的标记等同于带有标记的词法值和相应数据类型的字面量（xsd：integer，xsd：decimal，xsd：double，xsd：boolean）。

### 4.1.3 查询变量的句法

使用"?"或"$"标记查询变量;"?"或"$"不是变量名称的一部分。在查询中,$abc和?abc标识相同的变量。变量的可能名称在SPARQL语法中给出。

### 4.1.4 空节点句法

图模式中的空节点充当变量，而不是对查询的数据中的特定空节点的引用。

空节点由标签形式表示，例如"_:abc"或缩写形式"[]"。可以使用[]表示在查询语法中仅在一个位置使用的空节点。唯一的空节点被使用构成三元组模式。对于标记为"abc"的空节点，空节点标签写为"_:abc"。同一空节点标签不能在同一查询中的两个不同的基本图形模式中使用。

[:p :v]构造可用于三元组模式。它创建一个空节点标签，用作所有包含的谓语-宾语对的主题。创建的空节点也可以用于主语和宾语位置中的其他三元组模式。

以下两种形式

[ :p "v" ] .
[] :p "v" .

分配一个唯一的空白节点标签（这里是“b57”），相当于写：

_:b57 :p "v" .

该分配的空节点标签可以用作其他三元组模式的主语或宾语。例如，作为主语：

[ :p "v" ] :q "w" .

相当于两个三元组：

_:b57 :p "v" .
_:b57 :q "w" .

作为宾语：

:x :q [ :p "v" ] .

相当于两个三元组:

:x  :q _:b57 .
_:b57 :p "v" .

缩写的空节点语法可以与普通主语和常见谓语的其他缩写组合。

[ foaf:name  ?name ;
  foaf:mbox  <mailto:alice@example.org> ]
  
这与为某些唯一分配的空白节点标签"b18"编写以下基本图形模式相同：

_:b18  foaf:name  ?name .
_:b18  foaf:mbox  <mailto:alice@example.org> .

## 4.2 三元组语法

三元组模式被写为主语，谓语和宾语;有一些简短的方法来编写一些常见的三元组模式结构。

以下例子表示相同的查询：

PREFIX  dc: <http://purl.org/dc/elements/1.1/>
SELECT  ?title
WHERE   { <http://example.org/book/book1> dc:title ?title }

PREFIX  dc: <http://purl.org/dc/elements/1.1/>
PREFIX  : <http://example.org/book/>
SELECT  $title
WHERE   { :book1  dc:title  $title }

BASE    <http://example.org/book/>
PREFIX  dc: <http://purl.org/dc/elements/1.1/>
SELECT  $title
WHERE   { <book1>  dc:title  ?title }

### 4.2.1 谓语-宾语列表

可以编写具有共同主语的三元组模式，使得主语仅被写入一次并且通过使用";"符号而被用于多于一个三元组模式。

?x  foaf:name  ?name ;
    foaf:mbox  ?mbox .
	
和以下三元组形式相同：

?x  foaf:name  ?name .
?x  foaf:mbox  ?mbox .

### 4.2.2 宾语列表

如果三元组模式共享主语和谓词，则宾语可以用","分隔。

?x foaf:nick  "Alice" , "Alice_" .

与下面写法相同：

?x  foaf:nick  "Alice" .
?x  foaf:nick  "Alice_" .

宾语列表可以与谓语-宾语列表相结合：
?x  foaf:name ?name ; foaf:nick  "Alice" , "Alice_" .

与以下相同：

   ?x  foaf:name  ?name .
   ?x  foaf:nick  "Alice" .
   ?x  foaf:nick  "Alice_" .
   
### 4.2.3 RDF集合

可以使用语法"(element1 element2 ...)"以三元组模式编写RDF集合。形式"()"是IRI的替代方案http://www.w3.org/1999/02/22-rdf-syntax-ns#nil。当与集合元素一起使用时，例如(1？x 3 4)，为集合分配具有空节点的三元组模式。集合头部的空白节点可以用作其他三重模式中的主题或对象。由集合语法分配的空白节点不会出现在查询的其他位置。

(1 ?x 3 4) :p "w" .

是语法糖（注意b0，b1，b2和b3不会出现在查询中的任何其他位置）：

    _:b0  rdf:first  1 ;
          rdf:rest   _:b1 .
    _:b1  rdf:first  ?x ;
          rdf:rest   _:b2 .
    _:b2  rdf:first  3 ;
          rdf:rest   _:b3 .
    _:b3  rdf:first  4 ;
          rdf:rest   rdf:nil .
    _:b0  :p         "w" .

RDF集合可以嵌套，并且可以涉及其他语法形式：

(1 [:p :q] ( 2 ) ) .

语法糖是：

    _:b0  rdf:first  1 ;
          rdf:rest   _:b1 .
    _:b1  rdf:first  _:b2 .
    _:b2  :p         :q .
    _:b1  rdf:rest   _:b3 .
    _:b3  rdf:first  _:b4 .
    _:b4  rdf:first  2 ;
          rdf:rest   rdf:nil .
    _:b3  rdf:rest   rdf:nil .

### 4.2.4 rdf:type

关键字"a"可以用作三元组模式中的谓词，并且是IRI的替代方法http://www.w3.org/1999/02/22-rdf-syntax-ns#type。此关键字区分大小写。

  ?x  a  :Class1 .
  [ a :appClass ] :p "v" .
  
语法糖是：

  ?x    rdf:type  :Class1 .
  _:b0  rdf:type  :appClass .
  _:b0  :p        "v" .
  
# 5 图模式


