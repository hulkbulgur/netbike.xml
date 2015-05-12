NetBike.Xml
============

Fast, flexible and high customizable XML serializer for converting between .NET objects and XML.

* Flexible model of contracts via IXmlContractResolver
* Custom formatting and construction of objects via IXmlConverter
* Built-in support for the Nullable<>, List<>, Dictionary<> and more
* Supports for base and derived types via IXmlTypeResolver
* Partial supports for XML attributes of System.Xml.XmlSerializer
* .NET 4

Install
-------

Install via [NuGet package](https://www.nuget.org/packages/NetBike.Xml/):

	PM> Install-Package NetBike.Xml

Documentation
-------------

A long time ago in a galaxy far, far away..

Examples
--------

A simple object serialization:

```csharp
var serializer = new XmlSerializer();
serializer.Settings.Namespaces.Clear();
serializer.Settings.OmitXmlDeclaration = true;

/*
    Output XML:
    <foo><id>1</id><name>test</name></foo>
*/

serializer.Serialize(Console.Out, new Foo
{
    Id = 1,
    Name = "test"
});
```

A simple object deserialization:

```csharp
var serializer = new XmlSerializer();
var xml = "<foo><id>1</id><name>test</name></foo>";
var foo = serializer.Deserialize<Foo>(new StringReader(xml));
```

An advanced object serialization:

```csharp
public class Bar
{
    [XmlAttribute]
    public int Id { get; set; }

    public string Name { get; set; }

    public Dictionary<int, string> Numbers { get; set; }
}

public static void Example()
{
    var keyValuePairContract = new XmlObjectContractBuilder<KeyValuePair<int, string>>()
        .SetProperty(x => x.Key, "id", XmlMappingType.Attribute)
        .SetProperty(x => x.Value, "value", XmlMappingType.InnerText)
        .Build();
            
    var settings = new XmlSerializationSettings
    {
        Indent = true,
        OmitXmlDeclaration = true,
        NullValueHandling = XmlNullValueHandling.Include,
        ContractResolver = new XmlCustomContractResolver(
            new XmlContract[] { keyValuePairContract },
            fallbackResolver: new XmlContractResolver(ignoreSystemAttributes: false))
    };

    var serializer = new XmlSerializer(settings);

    var bar = new Bar
    {
        Id = 1,
        Name = null,
        Numbers = new Dictionary<int, string>
        {
            { 1, "one" },
            { 2, "two" }
        }
    };

    /*
        Output XML:
        <bar xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" id="1">
            <name xsi:nil="true" />
            <numbers>
            <item id="1">one</item>
            <item id="2">two</item>
            </numbers>
        </bar>
    */

    serializer.Serialize(Console.Out, bar);
}
```

License
-------
Copyright (c) 2015 NetBike

The [MIT License](https://github.com/netbike/netbike.xml/blob/master/LICENSE)
