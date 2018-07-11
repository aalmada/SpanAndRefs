@title[Span&lt;T&gt; and Refs]

## Span&lt;T&gt; and Refs 

### *Safe memory optimizations in .NET*

**Antão Almada**<br>
*Principal Engineer @ Farfetch*<br>

@fa[creative-commons] @fa[creative-commons-by] 

---

## Safe memory optimizations in .NET
@ul

- Pass-by-Reference
- Span&lt;T&gt;

@ulend

---

## Pass-by-reference

+++

## .NET Types 
@ul

- Value Types
- Reference Types
- Pointer Types

@ulend

+++

## Value Types
@ul[spaced-list-items]

- Byte, SByte, Int16, UInt16, Int32, UInt32, Int64, UInt64 
- Single, Double, Decimal
- Boolean, Char
- User-defined structs

@ulend

+++

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

+++

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

+++

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

+++

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

+++

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

@[1-12]
@[14-17]

+++

```
          Method |      Mean |     Error |    StdDev | Scaled | ScaledSD |
---------------- |----------:|----------:|----------:|-------:|---------:|
     CallByValue | 0.0489 ns | 0.0115 ns | 0.0102 ns |   1.00 |     0.00 |
 CallByReference | 0.0164 ns | 0.0062 ns | 0.0049 ns |   0.36 |     0.15 |
```

+++

```
struct Angle
{
    public int Degrees;
    public int Minutes;
    public double Seconds;
    
    public static double ToDegrees(ref Angle angle) =>
        angle.Degrees + angle.Minutes / 60.0 + angle.Seconds / 3600.0;
}
```

@[7-8](pass-by-reference)


NOTE:

Using pass-by-reference improves performance but usually means the value will be changed, which it's not doing!

[SharpLab.io](https://sharplab.io/#v2:C4LgTgrgdgPgAgBgARwIwG4CwAoHBnYSAY2CQEEoBzAGwFMcBvHJFlAZiQEspSARWymFq08WbK3ZceSALLcIwEWIlwOAEwD2EAEZ0kAZVpENUNaOasLLVSlQA2JJp16AKhv6DheABRCAZuRUegCGQbQAlEgAvAB8VhJIoTS0AHQeQiJIANSJYSlyUAqZAPRIdggpyDlJdCmGxqZ4SKVs5RUIYgC+ODg2cABMSACiAB7BALYADnqM8X32KAAstgAc3pHxTNgAkBIAbsFgucnRSFC0AO6BJwxI6V6nAJwIADSy8opNUUhsrwZGJjMp3aSE6ygS1lQj28FGSKTc9xEvloARqEXC4JY3Ww2KAA==)

+++

```
struct Angle
{
    public int Degrees;
    public int Minutes;
    public double Seconds;
    
    public static double ToDegrees(in Angle angle) =>
        angle.Degrees + angle.Minutes / 60.0 + angle.Seconds / 3600.0;
}
```

@[7-8](readonly ref => in)

NOTE:

'in' is new keyword that means 'readonly ref' => better performance and value can't be changed

But, a copy of the angle is created before the call to ToDegrees(). 

[SharpLab.io](https://sharplab.io/#v2:D4AQDABCCMDcCwAoJBnALgJwK4GM0QEEA7AcwBsBTJAbyQnqgGYIBLI/AEQpIwopQSIGTVuwgBZNljT9BwkMwAmAeywAjShADKFHMqKKBdBsfoKo0AGwQV6zQBVlXHnxQAKNoVKaAht4oAlBAAvAB8psIQfuQUAHTOvPwQANRR/rGSRNJJAPQQlmCxkKnRlLE6egYoEHmMBYVgggC+SEjmIABMEACiAB4+ALYADpo0Ee1WUAAsFgAcbkERtIgAkMIAbj4YaTEhEEQUAO5eu9QQCa57AJxgADQSUjLVwRCMd9q6+oZ7DRBNcpEzNArm5iDFYo4Lvw3KVAgEAfQWogkUA=)

The compiler has no guarantees that the value won't change between reads of the fields.

+++

```
readonly struct Angle
{
    public readonly int Degrees;
    public readonly int Minutes;
    public readonly double Seconds;
    
    public Angle(int degrees, int minutes, double seconds) =>
        (Degrees, Minutes, Seconds) = (
        	degrees, 
        	minutes < 60 ? 
        		minutes : 
        		throw new ArgumentOutOfRangeException(nameof(minutes)), 
        	seconds < 60.0 ? 
        		seconds : 
        		throw new ArgumentOutOfRangeException(nameof(seconds))
    	);
        
    public static double ToDegrees(in Angle angle) =>
        angle.Degrees + angle.Minutes / 60.0 + angle.Seconds / 3600.0;
}
```

@[1](readonly struct)
@[3-5](readonly fields)
@[6-15](constructor)
@[17-18] 

NOTE:

´readonly struct' guarantees that the struct is immutable. The value can only be set by the constructor.

[SharpLab.io](https://sharplab.io/#v2:D4AQDABCCMDcCwAoJAnApgQwCYHsB2ANgJ4QDOALigK4DG5EAgngOYFpIDeSEPUAzBHTZ8xCAEs89ACJpm6NKQSJe/QZlyESE+gFkJVcgqUqQAoRtG4qAIzYQAymhr4si7r3c9TjFmwAU2hBYsvKkADTikhAAtvqG4UE4NnakTi6kAJQQALwAfJ4qEH4ycmgKEXp4BuUOaXiuWdlFBSoAkMGlNS28rbFV8RAAPBAAbJAA/DFxChAAXBDkABYoOADuEHho6wwozFTRaJIA8gZHAGYAShgsaACiAB40aAAO5GL4fngYBzhnfn3VTIZCLdHitVLOeqkIajMAAOgmZDqrjmC2Waw2W0Yu32h3IJ3xl2uzDujxebw+Xx+fwh6QyGQKrQyxkKPAK3hgI0SyTQEAAKjgSqEAngfKxecS2I18spWRBJWg4UKytCANTy3yKyqAiAAelhCIg6oVcMckJR+r4Y3hYCUAF8kEgOQAmCAPb7POycdkCTlQAAsUGgAA4/FkClxEK0VAA3DAoDXinKY7aavwdULzACcYAiAPi8z4uaR5tI8xtzNBQazfiY4rhAuVCj8CvpLJ4DsQnaAA===)

No copy is made now!

+++

---

## Span <T>

---

@fa[twitter] [@AntaoAlmada](https://twitter.com/AntaoAlmada) <br/>

@fa[medium] [@antao.almada](https://medium.com/@antao.almada) <br/>

---

