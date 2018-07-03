
## Span&lt;T&gt; and Refs 

### *Safe memory optimizations in .NET*

**Antão Almada**<br>
*Principal Engineer @ Farfetch*<br>

@fa[creative-commons] @fa[creative-commons-by]

---

## Safe memory optimizations in .NET
@ul

- Pass-by-reference
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

- Structs
  + Boolean, Char
  + Byte, SByte, Int16, UInt16, Int32, UInt32, Int64, UInt64 
  + Single, Double, Decimal
  + User-defined

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

## Span <T>

---

**Twitter:** @AntaoAlmada
**Medium:** https://medium.com/@antao.almada/

---

