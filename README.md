# XML Data Source for Apache Spark 3.x

# Fix: Empty `attributePrefix` Breaking XML Parsing  
  
## Problem  
When `attributePrefix` is set to `""` and the XML contains literal `\n` characters, parsing returned `NULL` instead of actual values.  
  
## Root Cause  
In `StaxXmlParser.scala`:  
  
```scala  
val attributesOnly = st.fields.forall { f =>  
  f.name == options.valueTag || f.name.startsWith(options.attributePrefix)}  
```  
  
**Bug**: `"anyString".startsWith("")` is always `true` — so when `attributePrefix = ""`, every field was incorrectly treated as an attribute, causing the parser to skip nested structures.  
  
## Fix  
Added `options.attributePrefix.nonEmpty` guard:  
  
```scala  
val attributesOnly = st.fields.forall { f =>  
  f.name == options.valueTag ||    (options.attributePrefix.nonEmpty && f.name.startsWith(options.attributePrefix))  
}  
```  
  
Same fix applied in `StaxXmlGenerator.scala` for `MapType` and `StructType` partitioning.  
  
## Result  
Parser now correctly identifies attribute fields only when `attributePrefix` is non-empty, allowing proper parsing of nested XML structures.

---
### Exception case

If the name of attribute would be the same as one of children

```xml
<foo TOP="wow">
	<top>Hello<top>
</foo>
```

```json
"foo": {
	"TOP": "wow"
	"top": "Hello"
}
```

