# [Using JSON](https://docs.intersystems.com/healthconnectlatest/csp/docbook/DocBook.UI.Page.cls?KEY=GJSON)



# JSON in IRIS (2):  %JSON.Adaptor

### 对象到json的映射

- 对象到JSON: %JSONExport(); %JSONExportToStream(); %JSONExportToString()
- JSON到String/Stream/DynamicObject: %JSONImport()

> 查看online document例子

```
  Class Model.Event Extends (%Persistent, %JSON.Adaptor)
  {
    Property Name As %String;
    Property Location As Model.Location;
  }
  
  Class Model.Location Extends (%Persistent, %JSON.Adaptor)
  {
    Property City As %String;
    Property Country As %String;
  }
  
  
```

Export()出的结果：

```
  {"Name":"Global Summit","Location":{"City":"Boston","Country":"United States of America"}}
```

### Mapping with Parameters

对mapping的控制

```
Class Model.Event Extends (%Persistent, %JSON.Adaptor)
  {
    Property Name As %String(%JSONFIELDNAME = "eventName");
    Property Location As Model.Location(%JSONINCLUDE = "INPUTONLY");
  }
```

### Formatting JSON

是DynamicObject的输出更好看：Format(), FormatToStream(), FormatToString()

```
	dynObj = {"type":"string"}
  do formatter.Format(dynObj)
```


其他： Indent, IndentChars, LineTerminator













