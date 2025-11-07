# Diagnostic Steps for Agile .NET Pro Deobfuscation

## What We Know

Your files are **Agile .NET RT Pro** protected and de4dot:
- ✅ Detects the obfuscator correctly (score: 100)
- ❌ Static method decryption fails
- ❌ Falls back to 32-bit dynamic decryption (which fails on 64-bit)

## Option 1: Try 32-bit Build (Quickest)

Pull the latest code and build for x86:

```powershell
git pull
dotnet build de4dot.netcore.sln -c Release /p:Platform=x86
```

Then run from the x86 output directory. If the x86 build outputs to a different location, find it and run:

```powershell
.\de4dot.exe -v ..\..\files_to_deobfuscate\TDUPriceAction.dll -o output.dll
```

## Option 2: Skip Method Decryption (Partial Deobfuscation)

Try deobfuscating without method decryption to at least get strings and resources:

```powershell
.\de4dot.exe -v --an-methods false ..\..\files_to_deobfuscate\TDUPriceAction.dll -o output.dll
```

This will:
- ✅ Decrypt strings
- ✅ Decrypt resources
- ✅ Rename symbols
- ❌ Leave method bodies encrypted

## Option 3: Extract Signature for Debugging

We need to see what signature this newer version uses. Run with very verbose mode:

```powershell
.\de4dot.exe -vv -d ..\..\files_to_deobfuscate\TDUPriceAction.dll > agile_debug.txt 2>&1
```

Then look in `agile_debug.txt` for any signature-related output.

## Option 4: Manual Signature Extraction

Use a hex editor (like HxD) to manually find the signature:

1. Open `TDUPriceAction.dll` in a hex editor
2. Look for the Agile.NET signature near the end of the .NET metadata
3. The signature is 16 bytes and should be near offset calculated as:
   - Metadata RVA + Metadata Size (found in PE header)
4. Look for one of these known signatures:
   ```
   Old:    1F 68 9D 2B 07 4A A6 4A 92 BB 31 7E 60 7F D7 CD
   Normal: 08 44 65 E1 8C 82 13 4C 9C 85 B4 17 DA 51 AD 25
   Pro:    68 A0 BB 60 13 65 5F 41 AE 42 AB 42 9B 6B 4E C1
   ```
5. If it's different, that's our missing signature!

## Option 5: Use dnSpy to Inspect

Download dnSpy and open the DLL to see:
- Which methods are encrypted
- If there are readable type names
- If strings are encrypted

This will tell us which parts are failing.

## What to Report Back

Please try Option 1 (32-bit build) first and let me know:

1. **Does the x86 build exist?** Check if there's a `Release\net8.0-x86` or similar directory
2. **Does 32-bit work?** If yes, we're done!
3. **If not, what's the error?** Copy the full error message
4. **What does `-vv -d` show?** Run detection with very verbose and share the output

Then I can provide the exact fix needed!
