# Agile .NET Deobfuscation - Enhanced Edition

This branch contains significant improvements to de4dot's Agile .NET (CliSecure) deobfuscator, specifically targeting NinjaTrader 8 indicators protected with **Agile .NET RT Pro**.

## What's New

### 1. Enhanced Diagnostic Logging
Added comprehensive verbose logging throughout the Agile .NET method decryption process:
- Signature detection and analysis
- Code header offset and metadata information
- Decryption version detection (V10, V50, V52, etc.)
- Detailed failure messages for troubleshooting
- Helpful tips when decryption fails

### 2. .NET 8/9 Compatibility
- Updated from .NET Core 3.1 to .NET 8.0
- Added compatibility for .NET 9 SDK builds
- Suppressed obsolete API warnings (SYSLIB0011, SYSLIB0021, etc.)
- Enabled legacy `BinaryFormatter` serialization

### 3. Platform Support
- Added x86 platform configuration for 32-bit dynamic decryption
- Better support for mixed-mode assemblies

### 4. Comprehensive Documentation
Created detailed guides for different use cases:
- **QUICK_START.md** - Get started in 5 minutes
- **CURRENT_STATUS.md** - Comprehensive analysis and recommendations
- **diagnose_agile.md** - Diagnostic procedures
- **MANUAL_CHANGES_NEEDED.md** - Change reference (already applied)

## Quick Start

### Building

```powershell
# Clone and checkout this branch
git clone [your-repo-url]
git checkout claude/dotnet-reverse-engineering-011CUtHCGgPB1MG9sWCGQ7Ce

# Build with .NET SDK (8.0 or 9.0)
dotnet build de4dot.netcore.sln -c Release
```

### Usage Examples

**Full Deobfuscation (with verbose logging):**
```powershell
cd Release\net8.0
.\de4dot.exe -v input.dll -o output.dll
```

**Partial Deobfuscation (skip encrypted methods):**
```powershell
.\de4dot.exe --an-methods false input.dll -o output_partial.dll
```

**Diagnostic Mode (very verbose):**
```powershell
.\de4dot.exe -vv input.dll -o output.dll > debug.txt 2>&1
```

## Understanding Agile .NET RT Pro

### What It Is
Agile .NET (formerly CliSecure) is a commercial .NET obfuscator that provides:
- Method body encryption
- String encryption
- Resource encryption
- Control flow obfuscation
- Symbol renaming

### Known Limitations

**The RT Pro edition uses encryption algorithms that may not be supported by de4dot:**
- Newer versions use custom encryption schemes
- Static decryption may fail for some files
- Dynamic decryption requires 32-bit process execution

### When Full Deobfuscation Fails

If you see this error:
```
Agile.NET: All static decryption versions failed!
ERROR: Only 32-bit dynamic methods decryption is supported
```

**You have three options:**

1. **Partial Deobfuscation** (Recommended First Step)
   ```powershell
   .\de4dot.exe --an-methods false input.dll -o output_partial.dll
   ```
   This will decrypt:
   - ✅ Strings
   - ✅ Resources
   - ✅ Type/method names (renamed to readable format)
   - ❌ Method bodies remain encrypted

2. **dnSpy Debugger** (Most Reliable for Full Extraction)
   - Attach dnSpy to the running application
   - Let Agile .NET decrypt methods at runtime
   - Extract decrypted IL code from memory
   - Works regardless of encryption algorithm

3. **32-bit Build** (If dynamic decryption is needed)
   - Build for x86 platform specifically
   - Dynamic decryption loads assembly in 32-bit mode
   - May work for some protection variants

## Documentation Files

| File | Purpose | When to Read |
|------|---------|--------------|
| **QUICK_START.md** | Step-by-step immediate actions | Read this first! |
| **CURRENT_STATUS.md** | Comprehensive analysis | For understanding the full situation |
| **diagnose_agile.md** | Troubleshooting procedures | When things don't work as expected |
| **AGILE_NET_IMPROVEMENTS.md** | Technical implementation details | For developers modifying the code |
| **MANUAL_CHANGES_NEEDED.md** | Change reference | Historical reference (already applied) |

## Diagnostic Logging Output

With the enhanced logging, you'll see detailed information like:

```
[LOG] v: Agile.NET: Signature type = Normal
[LOG] v: Agile.NET: CodeHeader offset = 0x5B1234
[LOG] v: Agile.NET: Total code size = 0x3E644, Num methods = 3419
[LOG] v: Agile.NET: Will try 2 version(s): V52, V50
[LOG] v: Agile.NET: Trying version V52...
[LOG] v: Agile.NET: Version V52 failed: Exception of type 'de4dot.code.deobfuscators.InvalidMethodBody' was thrown.
[LOG] v: Agile.NET: Trying version V50...
[LOG] v: Agile.NET: Version V50 failed: Exception of type 'de4dot.code.deobfuscators.InvalidMethodBody' was thrown.
[LOG] e: Agile.NET: All static decryption versions failed!
[LOG] n: Agile.NET: Static decryption failed, trying dynamic method decryption
[LOG] n: NOTE: Dynamic decryption requires running as 32-bit process
[LOG] n: TIP: Try using --an-methods false to skip method decryption and only decrypt strings/resources
```

This helps identify:
- Which protection variant is being used
- Why static decryption is failing
- What to try next

## Technical Details

### Agile .NET Signatures

The tool recognizes three signature types:

| Signature | Hex Bytes | Protection Level |
|-----------|-----------|------------------|
| **Old** | `1F 68 9D 2B 07 4A A6 4A...` | Older versions |
| **Normal** | `08 44 65 E1 8C 82 13 4C...` | Standard protection |
| **Pro** | `68 A0 BB 60 13 65 5F 41...` | Professional edition |

### Code Header Versions

Different Agile .NET versions use different header formats:
- **V10** - Oldest format (pre-5.0)
- **V50** - Version 5.0-5.x
- **V52** - Version 5.2+ (newest supported)

### Decryption Methods

The tool attempts multiple decryption approaches:
1. **Decrypter10** - Old XOR-based encryption
2. **Decrypter5** - Double XOR with key derivation
3. **ProDecrypter** - TEA (Tiny Encryption Algorithm)

When all fail, it falls back to dynamic decryption which executes the Agile .NET runtime decryptor.

## Command Reference

### Basic Commands

```powershell
# Detect obfuscator only (no changes)
.\de4dot.exe -d input.dll

# Deobfuscate with default settings
.\de4dot.exe input.dll

# Deobfuscate with verbose output
.\de4dot.exe -v input.dll -o output.dll

# Very verbose (diagnostic mode)
.\de4dot.exe -vv input.dll -o output.dll

# Specify output file
.\de4dot.exe input.dll -o output.dll
```

### Agile .NET Specific Flags

```powershell
# Skip method decryption (partial deobfuscation)
.\de4dot.exe --an-methods false input.dll -o output.dll

# Force Agile .NET deobfuscator
.\de4dot.exe -p an input.dll -o output.dll
```

### Batch Processing

```powershell
# Process entire directory
.\de4dot.exe -r "C:\input" -o "C:\output"

# Process with file pattern
.\de4dot.exe -r "C:\input\*.dll" -o "C:\output"
```

## Troubleshooting

### Build Errors

**"CS0246: The type or namespace name 'dnlib' could not be found"**
- dnlib is included as a submodule
- Clone with `--recurse-submodules` or run `git submodule update --init --recursive`

**"SYSLIB0011: BinaryFormatter is obsolete"**
- Already fixed in this branch
- If you see this, pull the latest changes

### Runtime Errors

**"Could not load file or assembly 'dnlib'"**
- Make sure dnlib.dll is in the same directory as de4dot.exe
- Check the build output for missing dependencies

**"InvalidMethodBody exception"**
- The encryption algorithm is not supported
- Use `--an-methods false` for partial deobfuscation
- Or use dnSpy debugger for runtime extraction

### No Strings Decrypted

If strings remain encrypted after deobfuscation:
- Check if `-v` shows "Decrypted X strings"
- Some Agile .NET versions use custom string encryption
- Try very verbose mode (`-vv`) to see what's happening

## For Developers

### Modifying the Deobfuscator

Key files:
- `de4dot.code/deobfuscators/Agile_NET/MethodsDecrypter.cs` - Method decryption logic
- `de4dot.code/deobfuscators/Agile_NET/StringDecrypter.cs` - String decryption
- `de4dot.code/deobfuscators/Agile_NET/Deobfuscator.cs` - Main deobfuscator class

### Adding New Encryption Algorithms

To support new Agile .NET encryption:
1. Identify the signature in `GetSigType()`
2. Create a new `DecrypterXX` class implementing the algorithm
3. Add version detection in `GetCsHeaderVersions()`
4. Register the decrypter in `CreateCsHeader()`

### Testing Changes

```powershell
# Build
dotnet build -c Release

# Test with diagnostic output
.\de4dot.exe -vv test_file.dll -o output.dll > debug.txt 2>&1

# Check the debug output
type debug.txt
```

## Contributing

Improvements to Agile .NET support are welcome! When contributing:
1. Add diagnostic logging for new code paths
2. Update documentation to reflect changes
3. Test against multiple Agile .NET versions
4. Include sample output showing success/failure

## License

de4dot is licensed under GPLv3. See LICENSE.txt for details.

## Credits

- Original de4dot by 0xd4d
- Agile .NET diagnostic improvements by Claude Code session
- Testing and feedback by the NinjaTrader reverse engineering community

---

**Need Help?** Check **QUICK_START.md** for immediate next steps or **CURRENT_STATUS.md** for comprehensive guidance.
