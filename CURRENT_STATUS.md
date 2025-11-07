# Agile .NET Deobfuscation - Current Status

## Summary

Your NinjaTrader 8 indicators (`TDUPriceAction.dll` and `ORSFusion_2_2_VendorVersion.dll`) are protected with **Agile .NET RT Pro**. We've made significant progress in diagnosing the issue, but full method decryption is not currently possible with de4dot.

## What's Been Done

### 1. Fixed Build Issues ✅
- Updated target framework from .NET Core 3.1 to .NET 8.0
- Added .NET 9 compatibility by suppressing obsolete API warnings
- Enabled `BinaryFormatter` serialization for legacy code
- Added x86 platform configuration for potential 32-bit builds

### 2. Added Comprehensive Diagnostic Logging ✅
Enhanced the Agile .NET deobfuscator with detailed logging:
- Signature detection and type identification
- Code header offset and method counts
- Version detection (V10, V50, V52, etc.)
- Detailed failure messages for each attempted decryption method
- Helpful tips when falling back to dynamic decryption

### 3. Identified the Core Issue ✅
**The files use an unsupported encryption algorithm:**
- Signature Type: Normal (not Pro)
- Code Header Version: V52
- Both XOR-based (Decrypter5) and TEA-based (ProDecrypter) algorithms fail
- The encrypted bytes suggest a custom or newer encryption scheme
- Static decryption produces invalid method bodies

## Current Situation

**What Works:**
- ✅ Agile .NET detection (100% confidence)
- ✅ String decryption (likely to work)
- ✅ Resource decryption (likely to work)
- ✅ Symbol renaming

**What Doesn't Work:**
- ❌ Method body decryption (core issue)
- ❌ 32-bit dynamic decryption (requires x86 build or .NET Framework)

## Recommended Next Steps

### Option 1: Partial Deobfuscation (RECOMMENDED - Try This First)

Use the `--an-methods false` flag to skip method decryption and at least get readable strings and structure:

```powershell
# From your Release build directory
cd Release\net8.0

# For TDUPriceAction.dll
.\de4dot.exe --an-methods false ..\..\files_to_deobfuscate\TDUPriceAction.dll -o TDUPriceAction_partial.dll

# For ORSFusion
.\de4dot.exe --an-methods false ..\..\files_to_deobfuscate\ORSFusion_2_2_VendorVersion.dll -o ORSFusion_partial.dll
```

**What this gives you:**
- Decrypted strings (very valuable for understanding functionality)
- Decrypted resources
- Renamed types and members (more readable structure)
- Original method bodies still encrypted but at least visible

**Next step:** Open the output DLLs in dnSpy to see if the strings alone provide enough insight for your needs.

### Option 2: Try dnSpy Debugger (Most Likely to Succeed)

Since the method bodies are protected at the file level but must be decrypted at runtime:

1. **Set up dnSpy debugger:**
   - Download dnSpy from https://github.com/dnSpy/dnSpy/releases
   - Configure it to debug NinjaTrader 8

2. **Debug and dump at runtime:**
   - Run NinjaTrader 8 under dnSpy debugger
   - Wait for Agile .NET to decrypt methods in memory
   - Use dnSpy's "Edit Class" to view decrypted IL code
   - Can potentially dump decrypted methods from memory

**This is the most reliable approach** because:
- Agile .NET must decrypt methods to execute them
- dnSpy can inspect memory after decryption
- No need to reverse engineer the encryption algorithm

### Option 3: Manual Extraction (Advanced)

If you need the full source code:

1. Use dnSpy debugger to extract method bodies at runtime
2. Create a new project and manually reconstruct the logic
3. Use the partial deobfuscation output for strings and structure reference

### Option 4: Contact NinjaTrader Community

The NinjaTrader ecosystem might have:
- Developers who've dealt with this protection before
- Tools specifically designed for NT8 indicator analysis
- Documentation on working with protected indicators

## Building and Testing

### Rebuild the Project

```powershell
# In Visual Studio 2022 with .NET 9 SDK
dotnet build de4dot.netcore.sln -c Release
```

### Test with Verbose Logging

```powershell
cd Release\net8.0

# Very verbose output to see all diagnostic messages
.\de4dot.exe -vv ..\..\files_to_deobfuscate\TDUPriceAction.dll -o output.dll > debug.txt 2>&1

# View the diagnostic output
type debug.txt
```

### What You'll See in the Logs

With the new diagnostic logging, you'll see messages like:
```
Agile.NET: Signature type = Normal
Agile.NET: CodeHeader offset = 0x[hex]
Agile.NET: Total code size = 0x[hex], Num methods = [count]
Agile.NET: Will try 2 version(s): V52, V50
Agile.NET: Trying version V52...
Agile.NET: Version V52 failed: [detailed error message]
Agile.NET: All static decryption versions failed!
Agile.NET: Static decryption failed, trying dynamic method decryption
NOTE: Dynamic decryption requires running as 32-bit process
TIP: Try using --an-methods false to skip method decryption and only decrypt strings/resources
```

## Technical Details

### Why Static Decryption Fails

The protected DLLs use an Agile .NET RT Pro variant that:
1. Uses a "Normal" signature (not the known Pro signature)
2. Encrypts with an algorithm not implemented in de4dot
3. Has valid method headers embedded in encrypted data (misleading)
4. Requires either:
   - Reverse engineering the new encryption algorithm
   - Runtime decryption via dynamic method decryption or debugging

### File Analysis

**TDUPriceAction.dll:**
- Size: 880 KB
- Methods: 3419
- Code Size: 0x3E644 bytes
- Signature: Normal (0x08-44-65-E1-8C-82-13-4C-9C-85-B4-17-DA-51-AD-25)
- Contains: `<AgileDotNetRTPro>` marker

**ORSFusion_2_2_VendorVersion.dll:**
- Size: 759 KB
- Similar protection scheme

## Files Modified

All changes have been committed and pushed to branch:
`claude/dotnet-reverse-engineering-011CUtHCGgPB1MG9sWCGQ7Ce`

**Key files:**
- `de4dot.code/deobfuscators/Agile_NET/MethodsDecrypter.cs` - Added diagnostic logging
- `de4dot.code/de4dot.code.csproj` - .NET 9 compatibility
- `De4DotCommon.props` - Target framework updates
- `de4dot.cui/de4dot.cui.csproj` - Platform configurations

## My Recommendation

**Start with Option 1 (partial deobfuscation)** - it's quick and will immediately show you if strings alone are valuable enough. If not, **proceed to Option 2 (dnSpy debugger)** which is the most reliable way to extract fully deobfuscated code from Agile .NET protected assemblies.

The dnSpy debugger approach is what professional reverse engineers typically use for runtime-decrypted code, and it will work regardless of which encryption algorithm Agile .NET uses.

---

**Questions or Issues?**
If you encounter any problems or need clarification on any of these steps, let me know!
