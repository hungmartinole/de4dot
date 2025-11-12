# Reverse Engineering Agile.NET Encryption - Implementation Guide

## Overview

Since all public tools failed, we can reverse engineer the encryption algorithm and implement it in de4dot. This is a legitimate reverse engineering project.

---

## What We Need to Find

The Agile.NET RT Pro protection uses:
- **Method body encryption** - Our target
- **String encryption** - Already works in de4dot
- **Control flow obfuscation** - Secondary concern
- **Resource encryption** - Already works

**Our goal:** Find and implement the method body decryption algorithm.

---

## Phase 1: Locate the Decryption Routine (4-8 hours)

### Step 1: Find the Runtime DLL

The decryption happens in Agile.NET's runtime component:

```powershell
# Look for these files in your protected DLL's directory or embedded resources:
- AgileDotNetRTPro.dll
- CliSecure.dll
- Native decryption DLL (might be embedded)
```

**Extract it:**
```csharp
// The protected DLL likely has embedded resources
// Open TDUPriceAction.dll in dnSpy
// Look in Resources for embedded DLLs
// Extract them
```

### Step 2: Identify the Decryption Function

**Tools needed:**
- **IDA Pro** (paid, $500-1500) or **Ghidra** (free)
- **x64dbg** (free debugger)
- **dnSpy** (already have)

**Process:**

1. **Load the runtime DLL in IDA/Ghidra:**
   ```
   Open AgileDotNetRTPro.dll
   Let it analyze
   Look for functions related to decryption
   ```

2. **Search for string references:**
   ```
   Search for strings like:
   - "decrypt"
   - "method"
   - "body"
   - Error messages you've seen
   ```

3. **Find the JIT hook:**
   Agile.NET hooks the .NET JIT compiler. Look for:
   ```
   - ICorJitCompiler interface implementation
   - compileMethod function
   - Method decryption before JIT
   ```

### Step 3: Analyze the Algorithm

**What to look for:**

```assembly
; Typical decryption routine pattern:

mov     ecx, [encrypted_data]
mov     edx, [decryption_key]
call    decrypt_function

decrypt_function:
    ; Loop over bytes
    xor     byte [data + i], [key + (i % keylen)]
    ; Or more complex:
    ; TEA cipher
    ; AES
    ; Custom algorithm
    ret
```

**Common algorithms in Agile.NET:**
- TEA (Tiny Encryption Algorithm) - Already in de4dot
- XOR with key derivation - Already in de4dot
- **Custom TEA variant** - Likely what you have
- **Hybrid approach** - TEA + XOR

---

## Phase 2: Understand the Key Derivation (2-4 hours)

### The Key Problem

Your files show:
```
Signature: Normal (not Pro)
Version: V52
```

But standard V52 decryption fails. Why?

**Possible reasons:**
1. **Custom key derivation** - Key is computed differently
2. **Different TEA rounds** - Not 32 rounds, maybe 64?
3. **Modified TEA constants** - Not standard 0x9E3779B8
4. **Offset calculation** - Different from standard V52

### How to Find It

**Dynamic analysis:**

```csharp
// Create a test program
public static void Main() {
    // Load the protected DLL
    Assembly asm = Assembly.LoadFile("TDUPriceAction.dll");

    // Get a type
    Type t = asm.GetType("NinjaTrader.NinjaScript.Indicators.TDU.TDUPriceAction");

    // Create instance (triggers decryption)
    object instance = Activator.CreateInstance(t);

    // Set breakpoint HERE in x64dbg
    // Step into the decryption
}
```

**In x64dbg:**
1. Attach to your test program
2. Set breakpoint on CreateInstance
3. Step through
4. Watch when decryption happens
5. Analyze the assembly code

### Key Extraction

You need to find:
- Where the key comes from (embedded? computed?)
- How it's derived (hash? transform?)
- What parameters are used (method offset, size, RVA?)

---

## Phase 3: Implement in de4dot (8-16 hours)

### Create New Decrypter Class

Based on what you found, create:

```csharp
// de4dot.code/deobfuscators/Agile_NET/MethodsDecrypter.cs

// Add new decrypter class
class DecrypterV52Custom : DecrypterBase {
    readonly uint[] customKey = new uint[4];
    readonly int rounds;  // Might be different from 32

    public DecrypterV52Custom(MyPEImage peImage, CodeHeader codeHeader)
        : base(peImage, codeHeader) {
        // Initialize based on reverse engineering findings

        // Example: Custom key derivation
        for (int i = 0; i < 4; i++) {
            customKey[i] = DeriveKeyPart(codeHeader.decryptionKey, i);
        }

        // Maybe different round count?
        rounds = DetectRounds(codeHeader);
    }

    uint DeriveKeyPart(byte[] key, int index) {
        // Based on what you found in reverse engineering
        // Maybe it's not just ReadUInt32_be?

        // Example custom derivation:
        uint part = ReadUInt32_be(key, index * 4);
        part ^= 0x12345678;  // Custom XOR constant you found
        part = RotateLeft(part, 7);  // Custom rotation you found
        return part;
    }

    public override MethodBodyHeader Decrypt(MethodInfo methodInfo, out byte[] code, out byte[] extraSections) {
        byte[] data = peImage.OffsetReadBytes(endOfMetadata + methodInfo.codeOffs, (int)methodInfo.codeSize);

        // Implement the algorithm you reversed
        int numBlocks = (int)(methodInfo.codeSize / 8);
        for (int i = 0; i < numBlocks; i++) {
            int offset = i * 8;
            uint v0 = ReadUInt32_be(data, offset);
            uint v1 = ReadUInt32_be(data, offset + 4);

            // Your custom TEA variant
            const uint magic = 0x????????;  // Found from RE
            uint sum = magic * rounds;

            for (int j = 0; j < rounds; j++) {
                v1 -= ((v0 << 4) + customKey[2]) ^ (sum + v0) ^ ((v0 >> 5) + customKey[3]);
                v0 -= ((v1 << 4) + customKey[0]) ^ (sum + v1) ^ ((v1 >> 5) + customKey[1]);
                sum -= magic;
            }

            WriteUInt32_be(data, offset, v0);
            WriteUInt32_be(data, offset + 4, v1);
        }

        return GetCodeBytes(data, out code, out extraSections);
    }
}
```

### Integrate into Version Detection

```csharp
// Modify GetCsHeaderVersions or CreateCsHeader

ICsHeader CreateCsHeader(CsHeaderVersion version) {
    switch (version) {
    case CsHeaderVersion.V30: return new CsHeader30(this);
    case CsHeaderVersion.V40: return new CsHeader40(this);
    case CsHeaderVersion.V45: return new CsHeader45(this);
    case CsHeaderVersion.V50: return new CsHeader5(this, 0x28);
    case CsHeaderVersion.V52: return new CsHeader5(this, 0x30);
    case CsHeaderVersion.V52Custom: return new CsHeader5Custom(this, 0x30);  // NEW
    default: throw new ApplicationException("Unknown CS header");
    }
}
```

### Add Detection Logic

```csharp
// How do we know when to use V52Custom?
// Need to detect based on signature or other markers

List<CsHeaderVersion> GetCsHeaderVersions(uint codeHeaderOffset, MDTable methodDefTable) {
    if (sigType == SigType.Old)
        return new List<CsHeaderVersion> { CsHeaderVersion.V10 };

    if (!IsOldHeader(methodDefTable)) {
        // Try to detect V52Custom variant
        if (IsV52CustomVariant(codeHeaderOffset)) {
            return new List<CsHeaderVersion> {
                CsHeaderVersion.V52Custom,  // Try custom first
                CsHeaderVersion.V52,        // Fallback to standard
            };
        }
        return new List<CsHeaderVersion> { CsHeaderVersion.V52 };
    }

    // ... rest of logic
}

bool IsV52CustomVariant(uint codeHeaderOffset) {
    // Detect based on patterns you found
    // Examples:
    // - Check specific bytes at offset
    // - Check method count range
    // - Check total code size
    // - Check for custom markers

    // Example:
    byte marker = peImage.OffsetReadByte(codeHeaderOffset + 0x2F);
    return marker == 0x42;  // Custom marker you found
}
```

---

## Phase 4: Test and Debug (4-8 hours)

### Test Framework

```csharp
// Create test project
public class AgileDotNetTests {
    [Test]
    public void TestV52CustomDecryption() {
        var peImage = new MyPEImage("TDUPriceAction.dll");
        var module = ModuleDefMD.Load("TDUPriceAction.dll");
        var csRtType = new CliSecureRtType(module);

        var decrypter = new MethodsDecrypter();
        DumpedMethods dumpedMethods = null;

        bool result = decrypter.Decrypt(peImage, module, csRtType, ref dumpedMethods);

        Assert.IsTrue(result);
        Assert.IsNotNull(dumpedMethods);
        Assert.Greater(dumpedMethods.Count, 0);

        // Verify first method decrypted correctly
        var firstMethod = dumpedMethods[0];
        Assert.IsNotNull(firstMethod.code);

        // Check for valid method header
        Assert.That(firstMethod.code[0], Is.OneOf(0x02, 0x03, 0x13, 0x1B));
    }
}
```

### Debug Process

1. **Run de4dot with verbose logging**
2. **Check decrypted bytes** - Do they look valid?
3. **Compare with known good bytes** - Decrypt manually, compare
4. **Iterate on algorithm** - Adjust until it works

---

## Tools and Resources

### Required Tools

| Tool | Cost | Purpose |
|------|------|---------|
| **IDA Pro** | $500-1500 | Best disassembler (or Ghidra free) |
| **Ghidra** | Free | NSA's reverse engineering tool |
| **x64dbg** | Free | Dynamic analysis/debugging |
| **dnSpy** | Free | .NET debugging (already have) |
| **HxD** | Free | Hex editor |

### Learning Resources

**Reverse Engineering:**
- "Practical Malware Analysis" book
- "The IDA Pro Book"
- OpenSecurityTraining.info courses

**Cryptography:**
- "Cryptography Engineering" book
- Understanding TEA cipher
- Block cipher modes

**.NET Internals:**
- ".NET IL Assembler" book
- ECMA-335 specification
- CLR internals

---

## Estimated Timeline

### For Experienced Reverse Engineer:
- **Week 1:** Locate and analyze decryption (20 hours)
- **Week 2:** Implement and test (20 hours)
- **Total: 40 hours**

### For Learning As You Go:
- **Week 1-2:** Learn tools and RE basics (40 hours)
- **Week 3-4:** Locate decryption routine (40 hours)
- **Week 5-6:** Understand algorithm (40 hours)
- **Week 7-8:** Implement and debug (40 hours)
- **Total: 160 hours (1 month full-time)**

### For Complete Beginner:
- **Months 1-2:** Learn reverse engineering
- **Month 3:** Learn .NET internals
- **Month 4:** Actual RE work
- **Total: 300+ hours**

---

## Cost-Benefit Analysis

### Benefits of RE Approach:
✅ Fully automated solution
✅ Can process many files
✅ Learn valuable RE skills
✅ Contribute to open source
✅ Solve it once, use forever

### Drawbacks:
❌ Weeks/months of work
❌ Requires specialized skills
❌ Might hit anti-analysis protections
❌ Still doesn't help debug the bug
❌ Tools cost money (or time to learn free ones)

### Benefits of dnSpy Approach:
✅ 30 minutes to solution
✅ No specialized skills needed
✅ Also helps debug the bug
✅ See runtime values
✅ Guaranteed to work
✅ Free tools

### Drawbacks:
❌ Manual per-file
❌ Don't learn RE skills
❌ Don't solve root problem

---

## My Recommendation

### If you have:
- ✅ 100+ DLLs to deobfuscate
- ✅ Time (weeks/months)
- ✅ RE experience or willingness to learn
- ✅ $500+ for tools (or patience with Ghidra)

→ **Do the RE work!** It's valuable and worth it.

### If you have:
- ✅ 2-10 DLLs to process
- ✅ Need solution NOW
- ✅ Want to fix bugs and move on
- ✅ Limited RE experience

→ **Use dnSpy!** 30 min vs months.

---

## Hybrid Approach (Best of Both Worlds?)

**Quick solution now:**
1. Use dnSpy to extract code (30 min)
2. Fix your bugs and continue work
3. Have working indicators

**Learn RE in parallel:**
1. Start learning RE in spare time
2. Work on Agile.NET as practice project
3. Eventually implement custom decrypter
4. Contribute back to de4dot
5. Help the community!

**Timeline:**
- **Day 1:** Working code via dnSpy ✅
- **Months 1-3:** Learn RE skills
- **Month 4:** Implement decrypter
- **Result:** Both immediate solution AND long-term skills

---

## How I Can Help

**If you choose RE path:**
- I can help analyze the encryption
- Guide you through IDA/Ghidra
- Review your findings
- Help implement in de4dot
- Debug the implementation

**If you choose dnSpy:**
- Walk you through step-by-step
- Help extract the code
- Guide debugging process
- Help create clean project

**If you choose hybrid:**
- Help with dnSpy NOW
- Support your RE learning
- Be available when you're ready to implement

---

**What would you like to do?**
