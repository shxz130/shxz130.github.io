---
layout : life
title : "深入浅出XML之Schema/DTD"
category : 深入理解XML
duoshuo: true
date : 2014-11-08
---
------------

##DTD

------------

* 全称:**Document Type Define**

---------------

###内部声明：

 {% highlight xml %}
<?xmlversion="1.0"?>
<!DOCTYPE note[            //定义此文档是 note 类型的文档。
<!ELEMENT note(to,from,heading,body)>  //定义 note 元素有四个元素："to、from、heading,、body"
<!ELEMENT to(#PCDATA)>  //定义 to 元素为 "#PCDATA" 类型
<!ELEMENT from(#PCDATA)>  //定义 from 元素为 "#PCDATA" 类型
<!ELEMENT heading(#PCDATA)>  //定义 heading 元素为 "#PCDATA" 类型
<!ELEMENT body(#PCDATA)>  //定义 body 元素为 "#PCDATA" 类型
]>
<note>
	<to>Tove</to>
	<from>Jani</from>
	<heading>Reminder</heading>
	<body>Don'tforgetmethisweekend</body>
</note>
{% endhighlight %}

--------------

###外部声明

------------ 

假如 DTD 位于 XML 源文件的外部，那么它应通过下面的语法被封装在一个 DOCTYPE 定义中
## **<!DOCTYPE 根元素 SYSTEM "文件名">**
{% highlight xml %}
<?xmlversion="1.0"?>
<!DOCTYPEnoteSYSTEM"note.dtd"> //引用下面的not.dtd文件。
<note>
<to>Tove</to>
<from>Jani</from>
<heading>Reminder</heading>
<body>Don'tforgetmethisweekend!</body>
</note>

//这是包含DTD的"note.dtd"文件：
<!ELEMENT note(to,from,heading,body)>
<!ELEMENT to(#PCDATA)>
<!ELEMENT from(#PCDATA)>
<!ELEMENT heading(#PCDATA)>
<!ELEMENTbody(#PCDATA)>
{% endhighlight %}

###声明属性
---------
<!ATTLIST 元素名称 属性名称 属性类型 默认值>


* 类型描述
 * ID 值为唯一的 id
 * IDREF 值为另外一个元素的 id
 * IDREFS 值为其他 id 的列表
 * NMTOKEN 值为合法的 XML 名称
 * NMTOKENS 值为合法的 XML 名称的列表
 * ENTITY 值是一个实体
 * ENTITIES 值是一个实体列表
 * NOTATION 此值是符号的名称

* 属性值
 * #REQUIRED 属性值是必需的
 * #IMPLIED 属性不是必需的
 * #FIXED value 属性值是固定的 
 
-----------------

##缺点

-----------------

* DTD有自己的特殊语法，其本身不是XML文档；
* DTD只提供了有限的数据类型，用户无法自定义类型；
* DTD不支持域名机制。

-----------------

##Schema

-----------------

* 全称：可扩展标记语言架构
* 文件扩展名：xsd

-----------------

###优点：

-----------------

XML Schema 比 DTD 更强大。其优势包括以下几点
* 支持数据类型(更容易的定义数据类型，定义数据约束，更好的数据转换)
* 它使用 XML 语法
* 可保护数据通信
* 可扩展性
* 可捕获到错误

---------------

###劣点

----------------

* XML Schema语言特别是可能非常冗长,而DTD可以简洁且相对容易编辑。
* W3C XML Schema没有实现大部分提供的数据元素到文档的DTD能力。

----------------
 {% highlight xml %}
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"  //命名空间
	targetNamespace="http://tempuri.org/po.xsd"  
	xmlns="http://tempuri.org/po.xsd">
	
	<xs:element name="purchaseOrder" type="PurchaseOrderType"/> //自定义类型数据
	<xs:element name="comment" type="xs:string"/>  //定义name为comment，string类型的数据
	<xs:complexType name="PurchaseOrderType">   //complexType类型的数据
		<xs:sequence>
			<xs:element name="shipTo" type="USAddress"/>
			<xs:element name="billTo" type="USAddress"/>
			<xs:element ref="comment" minOccurs="0"/>
			<xs:element name="items" type="Items"/>
		</xs:sequence>
		<xs:attribute name="orderDate" type="xs:date"/>
	</xs:complexType>
	<xs:complexType name="USAddress">
		<xs:sequence>
			<xs:element name="name" type="xs:string"/>
			<xs:element name="street" type="xs:string"/>
			<xs:element name="city" type="xs:string"/>
			<xs:element name="state" type="xs:string"/>
			<xs:element name="zip" type="xs:decimal"/>
		</xs:sequence>
		<xs:attribute name="country" type="xs:NMTOKEN" fixed="US"/>
	</xs:complexType>
	<xs:complexType name="Items">
		<xs:sequence>
			<xs:element name="item" minOccurs="0" maxOccurs="unbounded">
				<xs:complexType>
					<xs:sequence>
						<xs:element name="productName" type="xs:string"/>
						<xs:element name="quantity">
							<xs:simpleType>
								<xs:restriction base="xs:positiveInteger">
									<xs:maxExclusive value="100"/>
								</xs:restriction>
							</xs:simpleType>
						</xs:element>
						<xs:element name="USPrice" type="xs:decimal"/>
						<xs:element ref="comment" minOccurs="0"/>
						<xs:element name="shipDate" type="xs:date" minOccurs="0"/>
					</xs:sequence>
					<xs:attribute name="partNum" type="SKU" use="required"/>
				</xs:complexType>
			</xs:element>
		</xs:sequence>
	</xs:complexType>
	<!-- Stock Keeping Unit, a code for identifying products -->
	<xs:simpleType name="SKU">
		<xs:restriction base="xs:integer">
			<xs:minInclusive value="2"/>
			<xs:maxInclusive value="10"/>
		</xs:restriction>
	</xs:simpleType>
</xs:schema>

{% endhighlight %}












