# Agile .NET Deobfuscator Improvements

## Summary

This document outlines the improvements made to de4dot's Agile .NET deobfuscator to support newer versions of Agile .NET protector, particularly Agile .NET Pro (RT Pro) versions.

## Issues Identified

### 1. Hardcoded Type and Field Names in StringDecrypter

**Problem**: The `StringDecrypter.Find()` method only looks for exact type names `<D234>` or `<ClassD234>` which were used in older Agile .NET versions. Newer versions may use different obfuscated names.

**Location**: `de4dot.code/deobfuscators/Agile_NET/StringDecrypter.cs`, line 66-81

**Impact**: String decryption fails on newer Agile .NET protected assemblies.

**Proposed Fix**:
```csharp
public void Find() {
    stringDecrypterKey = new byte[1] { 0xFF };

    // First, try exact match for known type names (backwards compatibility)
    foreach (var type in module.Types) {
        if (type.FullName == "<D234>" || type.FullName == "<ClassD234>") {
            stringDecrypterType = type;
            foreach (var field in type.Fields) {
                if (field.FullName == "<D234> <D234>::345" ||
                    field.FullName == "<ClassD234>/D234 <ClassD234>::345") {
                    keyInitField = field;
                    stringDecrypterKey = field.InitialValue;
                    Logger.v("Found string decrypter (exact match): type={0}", type.FullName);
                    return;
                }
            }
        }
    }

    // Pattern-based detection for newer versions
    foreach (var type in module.Types) {
        if (type.FullName.StartsWith("<") && type.FullName.Contains(">") &&
            type.Fields.Count > 0 && type.Fields.Count <= 5) {
            foreach (var field in type.Fields) {
                if (field.InitialValue != null && field.InitialValue.Length > 0 &&
                    field.InitialValue.Length <= 256) {
                    // Check for non-zero bytes (valid decryption key)
                    bool hasNonZero = false;
                    foreach (var b in field.InitialValue) {
                        if (b != 0) {
                            hasNonZero = true;
                            break;
                        }
                    }
                    if (hasNonZero) {
                        stringDecrypterType = type;
                        keyInitField = field;
                        stringDecrypterKey = field.InitialValue;
                        Logger.v("Found string decrypter (pattern): type={0}, key_len={1}",
                                type.FullName, stringDecrypterKey.Length);
                        return;
                    }
                }
            }
        }
    }
}
```

### 2. Limited Method Signature Detection

**Problem**: The `MethodsDecrypter` class has three hardcoded signatures (old, normal, pro) that may not cover all Agile .NET Pro variants.

**Location**: `de4dot.code/deobfuscators/Agile_NET/MethodsDecrypter.cs`, lines 53-55

**Current Signatures**:
- Old: `1F 68 9D 2B 07 4A A6 4A 92 BB 31 7E 60 7F D7 CD`
- Normal: `08 44 65 E1 8C 82 13 4C 9C 85 B4 17 DA 51 AD 25`
- Pro: `68 A0 BB 60 13 65 5F 41 AE 42 AB 42 9B 6B 4E C1`

**Impact**: Newer Agile .NET Pro versions with different signatures won't be detected.

**Proposed Fix**: Add additional signature detection and logging:
```csharp
static readonly byte[] proSignature2 = new byte[16] {
    0x58, 0xA0, 0xBB, 0x60, 0x13, 0x65, 0x5F, 0x41,
    0xAE, 0x42, 0xAB, 0x42, 0x9B, 0x6B, 0x4E, 0xC2
}; // Example - actual signature needs to be extracted from protected files

enum SigType {
    Unknown,
    Old,
    Normal,
    Pro,
    Pro2,  // Add support for newer Pro versions
}
```

### 3. TEA Decryption in ProDecrypter

**Problem**: The TEA (Tiny Encryption Algorithm) implementation in `ProDecrypter` might need adjustments for newer Agile .NET versions.

**Location**: `de4dot.code/deobfuscators/Agile_NET/MethodsDecrypter.cs`, lines 174-219

**Current Implementation**: Uses standard TEA with magic constant `0x9E3779B8` and 32 rounds.

**Potential Issues**:
- Magic constant might be different
- Number of rounds might vary
- Key initialization might be different

**Debugging Suggestion**: Add logging to verify decryption:
```csharp
public override MethodBodyHeader Decrypt(MethodInfo methodInfo, out byte[] code, out byte[] extraSections) {
    Logger.v("ProDecrypter: Decrypting method at offset 0x{0:X}, size=0x{1:X}",
            methodInfo.codeOffs, methodInfo.codeSize);
    byte[] data = peImage.OffsetReadBytes(endOfMetadata + methodInfo.codeOffs, (int)methodInfo.codeSize);

    // Log first few bytes before decryption
    Logger.v("ProDecrypter: First 16 bytes before: {0}", BitConverter.ToString(data, 0, Math.Min(16, data.Length)));

    // ... existing decryption code ...

    // Log first few bytes after decryption
    Logger.v("ProDecrypter: First 16 bytes after: {0}", BitConverter.ToString(data, 0, Math.Min(16, data.Length)));

    return GetCodeBytes(data, out code, out extraSections);
}
```

## Analysis of Test Files

The files in `files_to_deobfuscate/` are protected with **Agile .NET RT Pro**:
- `TDUPriceAction.dll` (880 KB) - Contains strings: `<AgileDotNetRTPro>`, `AgileDotNetRTPro.dll`
- `ORSFusion_2_2_VendorVersion.dll` (759 KB) - Similar protection

## Recommended Testing Approach

1. **Extract Signature**: Run de4dot with `-d` (detect only) and add verbose logging to extract the actual signature being used
2. **Compare Decryption**: If method decryption works but strings don't, focus on StringDecrypter
3. **Incremental Testing**: Test each component separately:
   - Signature detection
   - Method decryption
   - String decryption
   - Resource decryption

## Build Instructions (Workaround for SSL Issues)

Due to NuGet SSL certificate issues in the current environment, here's how to build:

### Option 1: Use Pre-downloaded Packages
```bash
# Download packages manually
curl -L -o /tmp/dnlib.3.3.2.nupkg https://www.nuget.org/api/v2/package/dnlib/3.3.2
curl -L -o /tmp/system.drawing.common.4.7.0.nupkg https://www.nuget.org/api/v2/package/System.Drawing.Common/4.7.0

# Extract to NuGet cache
mkdir -p ~/.nuget/packages/dnlib/3.3.2
mkdir -p ~/.nuget/packages/system.drawing.common/4.7.0

cd /tmp && unzip -q dnlib.3.3.2.nupkg -d ~/.nuget/packages/dnlib/3.3.2/
cd /tmp && unzip -q system.drawing.common.4.7.0.nupkg -d ~/.nuget/packages/system.drawing.common/4.7.0/
```

### Option 2: Use Docker (Recommended)
```bash
docker run --rm -v $(pwd):/src -w /src mcr.microsoft.com/dotnet/sdk:8.0 dotnet build de4dot.netcore.sln -c Release
```

### Option 3: Windows Build
On Windows with Visual Studio or .NET SDK:
```powershell
dotnet build de4dot.netcore.sln -c Release
```

## Usage

Once built, deobfuscate the test files:

```bash
# Detect obfuscator
./de4dot -d files_to_deobfuscate/TDUPriceAction.dll

# Deobfuscate (verbose mode for debugging)
./de4dot -v files_to_deobfuscate/TDUPriceAction.dll -o files_to_deobfuscate/TDUPriceAction_deobfuscated.dll

# Deobfuscate all files in directory
./de4dot -r files_to_deobfuscate -ru -ro files_to_deobfuscate/output
```

## Next Steps

1. Build de4dot with the improvements
2. Test on the provided NinjaTrader indicators
3. If signature is unknown, extract it and add to the signature list
4. If decryption fails, analyze the specific decryption algorithm being used
5. Compare with successful deobfuscation of older Agile .NET versions to identify differences

## References

- Agile .NET Obfuscator: https://www.secureteam.net/
- de4dot Documentation: https://github.com/0xd4d/de4dot
- TEA Algorithm: https://en.wikipedia.org/wiki/Tiny_Encryption_Algorithm
