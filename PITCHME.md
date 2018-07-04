@title[Span&lt;T&gt; and Refs]

## Span&lt;T&gt; and Refs 

### *Safe memory optimizations in .NET*

**Antão Almada**<br>
*Principal Engineer @ Farfetch*<br>

@fa[creative-commons] @fa[creative-commons-by] @fa[creative-commons-sa]

---

## Safe memory optimizations in .NET
@ul

- Pass-by-Reference
- Span&lt;T&gt;

@ulend

---

## Pass-by-reference

---

## .NET Types 
@ul

- Value Types
- Reference Types
- Pointer Types

@ulend

---

## Value Types
@ul[spaced-list-items]

- Byte, SByte, Int16, UInt16, Int32, UInt32, Int64, UInt64 
- Single, Double, Decimal
- Boolean, Char
- User-defined structs

@ulend

---

## Value Types
@ul[spaced-list-items]

- Allocated on the stack 
  + Except when boxed
- Arguments passed by value
  + Full contents are copied

@ulend

Note:

- Allocated on the stack **is an advantage**.
- Arguments passed by value **may be a disadvange**.

---

```csharp
struct Angle
{
    public int Degrees;
    public int Minutes;
    public double Seconds;
    
    public static void ByValue(Angle angle) {}
    public static void ByReference(ref Angle angle) {}
}
```

@[3-5]
@[7-8]

---

```csharp
public class Example 
{
    Angle angle = new Angle { Degrees = 90, Minutes = 0, Seconds = 0.0 };
    
    void CallByValue() => 
        Angle.ByValue(angle);
    
    void CallByReference() => 
        Angle.ByReference(ref angle);
}
```

@[5-6, 8-9]

NOTE:
[SharpLab.io](https://sharplab.io/#v2:C4LglgNgPgsAUAZ2AJwK4GNgAICCA7AcwgFN4BveLKrAAQGYsw9sARYg5Y4hAbkuvqNmWALJNUwbnzjVaDACYB7VACMSWAMrF0ivPN78qhubQCMANloAWLACEAngDUAhhFTEAFPiLEszwiQAlFhkAL7GgjQW1nb2AErEAGbEnHjonpyJuAG+/j7BYfDhcPCRAExYAKIAHs4AtgAO6uTG3up56gC8WHjEAO7ZPiFYbBxcCFjdAJwADAA0ouKSE93zmtq6+pNYMwB0M1ih0rIRNgDCrhAOLm6ewZ0AfFjGsm3Eu9eu7h4dxIHH1FOWAuECu8SSKWIaTukyeL2obw+4OSqXSHkyfhy/yK8CAA==)

---

```
.method private hidebysig 
instance void CallByValue () cil managed 
{
    IL_0000: ldarg.0
    IL_0001: ldfld valuetype Angle Example::angle
    IL_0006: call void Angle::ByValue(valuetype Angle)
    IL_000b: ret
}

.method private hidebysig 
instance void CallByReference () cil managed 
{
    IL_0000: ldarg.0
    IL_0001: ldflda valuetype Angle Example::angle
    IL_0006: call void Angle::ByReference(valuetype Angle&)
    IL_000b: ret
}

```

@[1-2,10-11]
@[4-7, 13-16](ldfld - Pushes the **value of field** onto the stack.<br/>ldflda - Pushes the **address of field** onto the stack.)

---

```
Example.CallByValue()
    L0000: push ebp
    L0001: mov ebp, esp
    L0003: add ecx, 0x4
    L0006: sub esp, 0x10
    L0009: movq xmm0, [ecx]
    L000d: movq [esp], xmm0
    L0012: movq xmm0, [ecx+0x8]
    L0017: movq [esp+0x8], xmm0
    L001d: call Angle.ByValue(Angle)
    L0022: pop ebp
    L0023: ret

Example.CallByReference()
    L0000: add ecx, 0x4
    L0003: call Angle.ByReference(Angle ByRef)
    L0008: ret
```

---

## Span <T>

---

@fa[twitter] [@AntaoAlmada](https://twitter.com/AntaoAlmada) <br/>

@fa[medium] [@antao.almada](https://medium.com/@antao.almada) <br/>

---

