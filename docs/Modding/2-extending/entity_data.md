---
sidebar_position: 2
---

# Entity Data

To synchronize network data of your custom entity you first need to define data set ("header").

Then for each of the header you define method that can write this data. Then a method that writes all the data.

The reason behind being able to write individually, is to be able to change and sync data individually, instead of sending all the data every time.

WriteAllHeaders is used when a new player joins and needs the full state of the server


## Defining Data

To extend entity data you first have to define Enum with your data types.

It's important to specify enum type to byte

```cs title="Example entity header"
public enum ExampleEntityHeader : byte
{
    IsReady,
    CurrentStage
}
```


## Data Serialization


First define methods to serialize specific header

> BinaryWriter Writer property is available

```cs title="Example Write methods"
private void WriteIsReady()
{
    _insertHeader(ExampleEntityHeader.IsReady);
    Writer.Write(IsReady);
}

private void WriteCurrentStage()
{
    _insertHeader(ExampleEntityHeader.CurrentStage);
    Writer.Write(CurrentStage);
}
```

---

Then override WriteAllHeaders to serialize all headers

> Make sure to define all headers that are in the enum or else the serialization will throw error
>
> And leave the base overload so the inherited data is also serialized


```cs title="Example Write methods"
public override void WriteAllHeaders()
{
    base.WriteAllHeaders(); // Make sure to leave base overload
        
    _InsertHeaderCount(typeof(ExampleEntityHeader));
    WriteIsReady();
    WriteCurrentStage();
}
```


## Data Deserialization

To deserialize override ReadHeaders method and follow the structure of the example

> Feel free to define each header (switch case) as own method, but this is not required

```cs title="Example Read Headers"
protected override void ReadHeaders()
{
    base.ReadHeaders();

    var headerCount = _GetHeaderCount();


    for (int i = 0; i < headerCount; i++)
    {

        var header = _GetHeader<ExampleEntityHeader>();

        switch (header)
        {
            case ExampleEntityHeader.IsReady:

                IsReady = Reader.ReadBoolean();
                continue;
            
            case ExampleEntityHeader.CurrentStage:

                CurrentStage = Reader.ReadByte();
                continue;
            
            default:
                Logs.Error("Unknown header type!");
                continue;
        }
        
    }
}
```