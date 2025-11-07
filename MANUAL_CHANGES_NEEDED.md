# Manual Changes Needed for Agile .NET Pro Support

Since automatic patching is difficult, here are the **exact manual changes** to make:

## File: `de4dot.code/deobfuscators/Agile_NET/MethodsDecrypter.cs`

### Change 1: Line ~431 - Add logging when falling back to dynamic decryption

**Find this:**
```csharp
case DecryptResult.Error:
    Logger.n("Using dynamic method decryption");
    byte[] moduleCctorBytes = GetModuleCctorBytes(csRtType);
```

**Replace with:**
```csharp
case DecryptResult.Error:
    Logger.n("Agile.NET: Static decryption failed, trying dynamic method decryption");
    Logger.n("NOTE: Dynamic decryption requires running as 32-bit process");
    Logger.n("TIP: Try using --an-methods false to skip method decryption and only decrypt strings/resources");
    byte[] moduleCctorBytes = GetModuleCctorBytes(csRtType);
```

### Change 2: Line ~481 - Add detailed logging in Decrypt2

**Find this:**
```csharp
DecryptResult Decrypt2(ref DumpedMethods dumpedMethods) {
    uint codeHeaderOffset = InitializeCodeHeader();
    if (sigType == SigType.Unknown)
        return DecryptResult.NotEncrypted;

    var methodDefTable = peImage.Metadata.TablesStream.MethodTable;

    foreach (var version in GetCsHeaderVersions(codeHeaderOffset, methodDefTable)) {
        try {
            if (version == CsHeaderVersion.V10)
                DecryptMethodsOld(methodDefTable, ref dumpedMethods);
            else
                DecryptMethods(codeHeaderOffset, methodDefTable, CreateCsHeader(version), ref dumpedMethods);
            return DecryptResult.Decrypted;
        }
        catch {
        }
    }

    return DecryptResult.Error;
}
```

**Replace with:**
```csharp
DecryptResult Decrypt2(ref DumpedMethods dumpedMethods) {
    uint codeHeaderOffset = InitializeCodeHeader();
    if (sigType == SigType.Unknown) {
        Logger.v("Agile.NET: Unknown signature, methods not encrypted");
        return DecryptResult.NotEncrypted;
    }

    Logger.v("Agile.NET: Signature type = {0}", sigType);
    Logger.v("Agile.NET: CodeHeader offset = 0x{0:X}", codeHeaderOffset);
    Logger.v("Agile.NET: Total code size = 0x{0:X}, Num methods = {1}", codeHeader.totalCodeSize, codeHeader.numMethods);

    var methodDefTable = peImage.Metadata.TablesStream.MethodTable;
    var versions = GetCsHeaderVersions(codeHeaderOffset, methodDefTable);
    Logger.v("Agile.NET: Will try {0} version(s): {1}", versions.Count, string.Join(", ", versions));

    foreach (var version in versions) {
        try {
            Logger.v("Agile.NET: Trying version {0}...", version);
            if (version == CsHeaderVersion.V10)
                DecryptMethodsOld(methodDefTable, ref dumpedMethods);
            else
                DecryptMethods(codeHeaderOffset, methodDefTable, CreateCsHeader(version), ref dumpedMethods);
            Logger.v("Agile.NET: SUCCESS with version {0}", version);
            return DecryptResult.Decrypted;
        }
        catch (Exception ex) {
            Logger.v("Agile.NET: Version {0} failed: {1}", version, ex.Message);
        }
    }

    Logger.e("Agile.NET: All static decryption versions failed!");
    return DecryptResult.Error;
}
```

### Change 3: Line ~503 - Add logging in InitializeCodeHeader

**Find this:**
```csharp
uint InitializeCodeHeader() {
    uint codeHeaderOffset = GetCodeHeaderOffset(peImage);
    ReadCodeHeader(codeHeaderOffset);
    sigType = GetSigType(codeHeader.signature);

    if (sigType == SigType.Unknown) {
        codeHeaderOffset = GetOldCodeHeaderOffset(peImage);
        if (codeHeaderOffset != 0) {
            ReadCodeHeader(codeHeaderOffset);
            sigType = GetSigType(codeHeader.signature);
        }
    }

    return codeHeaderOffset;
}
```

**Replace with:**
```csharp
uint InitializeCodeHeader() {
    uint codeHeaderOffset = GetCodeHeaderOffset(peImage);
    ReadCodeHeader(codeHeaderOffset);
    sigType = GetSigType(codeHeader.signature);

    Logger.v("Agile.NET: Signature at offset 0x{0:X}: {1}", codeHeaderOffset, BitConverter.ToString(codeHeader.signature));
    Logger.v("Agile.NET: Initial signature type: {0}", sigType);

    if (sigType == SigType.Unknown) {
        Logger.v("Agile.NET: Trying old code header location...");
        codeHeaderOffset = GetOldCodeHeaderOffset(peImage);
        if (codeHeaderOffset != 0) {
            ReadCodeHeader(codeHeaderOffset);
            sigType = GetSigType(codeHeader.signature);
            Logger.v("Agile.NET: Old signature at offset 0x{0:X}: {1}", codeHeaderOffset, BitConverter.ToString(codeHeader.signature));
            Logger.v("Agile.NET: Old signature type: {0}", sigType);
        }
    }

    return codeHeaderOffset;
}
```

## After Making Changes

1. Save the file
2. Rebuild: `dotnet build de4dot.netcore.sln -c Release`
3. Run with verbose: `.\de4dot.exe -vv ..\..\files_to_deobfuscate\TDUPriceAction.dll -o output.dll > debug2.txt 2>&1`
4. Look at `debug2.txt` for the detailed Agile.NET messages
5. Share the output so I can see exactly why static decryption is failing

This will give us the diagnostic info we need to fix the actual decryption issue!
