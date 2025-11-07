# Changelog - Agile .NET Improvements

## [Latest] - 2025-11-07

### Fixed
- **InitializeDelegate removal error** - Fixed metadata errors when using `--an-methods false`
  - Error: "TypeDef InitializeDelegate (020000C8) is not defined in this module"
  - Error: "Method System.Int32 InitializeDelegate::Invoke(System.IntPtr) is not defined in this module"
  - Cause: The deobfuscator was removing the InitializeDelegate type even when methods weren't being decrypted
  - Solution: Only remove InitializeDelegate when `options.DecryptMethods` is true
  - Impact: Partial deobfuscation (`--an-methods false`) now works correctly

### Added
- **Comprehensive diagnostic logging** in MethodsDecrypter.cs
  - Logs signature type detection (Old/Normal/Pro)
  - Shows code header offset and metadata information
  - Displays which decryption versions are attempted (V10/V50/V52)
  - Captures specific exception messages for each failed attempt
  - Provides helpful tips when static decryption fails

- **Better user guidance messages**
  - Clear explanation when falling back to dynamic decryption
  - Explains 32-bit requirement for dynamic method decryption
  - Suggests `--an-methods false` as alternative approach
  - More informative error messages throughout

- **Comprehensive documentation**
  - QUICK_START.md - Immediate next steps guide
  - CURRENT_STATUS.md - Detailed status and recommendations
  - AGILE_NET_README.md - Complete reference documentation
  - diagnose_agile.md - Diagnostic procedures
  - CHANGELOG.md - This file

### Changed
- **.NET 9 compatibility** - Added warning suppressions for obsolete APIs
  - Suppressed SYSLIB0011 (BinaryFormatter obsolete)
  - Suppressed SYSLIB0021 (DESCryptoServiceProvider obsolete)
  - Suppressed SYSLIB0022 (RijndaelManaged obsolete)
  - Suppressed SYSLIB0032 (Various crypto obsolete)
  - Enabled `EnableUnsafeBinaryFormatterSerialization` for legacy support

- **Target framework** - Updated from .NET Core 3.1 to .NET 8.0
  - Better compatibility with modern .NET SDK
  - Resolved NuGet package restoration issues

- **Platform support** - Added x86 configuration for 32-bit builds
  - Enables potential 32-bit dynamic method decryption
  - Platform options: AnyCPU, x86, x64

## Technical Details

### InitializeDelegate Fix

**Problem:**
The Agile .NET deobfuscator was unconditionally removing the `InitializeDelegate` type in the `DeobfuscateBegin()` method:

```csharp
foreach (var type in module.Types) {
    if (type.FullName == "InitializeDelegate" && DotNetUtils.DerivesFromDelegate(type))
        AddTypeToBeRemoved(type, "Obfuscator type");
}
```

When using `--an-methods false`, method bodies remain encrypted and still reference this delegate, but the type was being removed, causing metadata validation errors.

**Solution:**
Wrapped the removal in a conditional check:

```csharp
if (options.DecryptMethods) {
    foreach (var type in module.Types) {
        if (type.FullName == "InitializeDelegate" && DotNetUtils.DerivesFromDelegate(type))
            AddTypeToBeRemoved(type, "Obfuscator type");
    }
}
```

Now the delegate is only removed when methods are actually being decrypted.

### Files Modified

| File | Changes | Purpose |
|------|---------|---------|
| `de4dot.code/deobfuscators/Agile_NET/Deobfuscator.cs` | Added conditional check for InitializeDelegate removal | Fix partial deobfuscation errors |
| `de4dot.code/deobfuscators/Agile_NET/MethodsDecrypter.cs` | Added comprehensive logging | Better diagnostics for decryption failures |
| `de4dot.code/de4dot.code.csproj` | Added warning suppressions and BinaryFormatter support | .NET 9 compatibility |
| `De4DotCommon.props` | Changed target framework to net8.0 | Modern .NET SDK compatibility |
| `de4dot.cui/de4dot.cui.csproj` | Added platform configurations | 32-bit build support |
| `de4dot.mdecrypt/de4dot.mdecrypt.csproj` | Added warning suppressions | .NET 9 compatibility |
| `AssemblyData/AssemblyData.csproj` | Added warning suppressions | .NET 9 compatibility |

## Commit History

```
37b8ddf Fix InitializeDelegate removal when method decryption is disabled
27a4e3f Add comprehensive Agile.NET focused README
1fa7a0c Add quick start guide for immediate next steps
a593747 Add comprehensive status document for Agile.NET deobfuscation
c76e8e2 Add comprehensive diagnostic logging to Agile.NET deobfuscator
def04b2 Add manual change instructions for Agile.NET diagnostics
12179b8 Add diagnostic guide for Agile .NET Pro deobfuscation
7aa5897 Add x86 platform support for 32-bit dynamic method decryption
4b27e21 Fix .NET 9 build errors - suppress obsolete API warnings
4f6bcd0 Improve Agile .NET deobfuscator for newer versions
```

## Testing

### Before Fix
```powershell
PS> .\de4dot.exe --an-methods false TDUPriceAction.dll -o output.dll
ERROR: TypeDef InitializeDelegate (020000C8) is not defined in this module
ERROR: Method System.Int32 InitializeDelegate::Invoke(System.IntPtr) (06000D47) is not defined
```

### After Fix
```powershell
PS> .\de4dot.exe --an-methods false TDUPriceAction.dll -o output.dll
Detected CliSecure
Cleaning TDUPriceAction.dll
Renaming all obfuscated symbols
Saving output.dll
# Success - no metadata errors
```

## Known Limitations

### Static Method Decryption
- **Issue**: Some Agile .NET RT Pro variants use unsupported encryption algorithms
- **Symptom**: "All static decryption versions failed" error
- **Workaround**: Use `--an-methods false` for partial deobfuscation or dnSpy debugger for runtime extraction
- **Root Cause**: The encryption algorithm is not implemented in de4dot
- **Status**: No fix planned - use alternative approaches

### Dynamic Method Decryption
- **Issue**: Requires 32-bit process
- **Symptom**: "Only 32-bit dynamic methods decryption is supported"
- **Workaround**: Build for x86 platform or use .NET Framework version
- **Status**: Partial - x86 configuration added but .NET Framework build requires additional dependencies

## Recommendations

### For Partial Deobfuscation
1. Pull latest changes: `git pull`
2. Rebuild: `dotnet build -c Release`
3. Run: `de4dot.exe --an-methods false input.dll -o output.dll`
4. Check output in dnSpy for decrypted strings and structure

### For Full Deobfuscation
1. Use dnSpy debugger to attach to running application
2. Let Agile .NET decrypt methods at runtime
3. Extract decrypted IL code from memory
4. This works regardless of encryption algorithm

## Contributors
- Original de4dot by 0xd4d
- Agile .NET improvements by Claude Code session (2025-11-07)
- Testing and feedback by hungmartinole

## License
GPLv3 - See LICENSE.txt
