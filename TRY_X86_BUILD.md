# Try x86 (32-bit) Build for Dynamic Decryption

The error "Only 32-bit dynamic methods decryption is supported" means we need to run de4dot as a 32-bit process to enable dynamic method decryption.

## Option: Build as x86

### Step 1: Clean Previous Builds

```powershell
dotnet clean de4dot.netcore.sln
```

### Step 2: Build for x86 Platform

```powershell
# Try building specifically for x86
dotnet build de4dot.netcore.sln -c Release /p:Platform=x86

# Alternative: specify architecture
dotnet build de4dot.netcore.sln -c Release -a x86
```

### Step 3: Check Output Location

The x86 build might output to:
```
Release\net8.0-windows\x86\de4dot.exe
```
or
```
Release\net8.0\win-x86\de4dot.exe
```

Find the x86 executable:
```powershell
# Search for it
Get-ChildItem -Path Release -Recurse -Filter "de4dot.exe" | Select-Object FullName
```

### Step 4: Test with x86 Build

```powershell
cd [path-to-x86-build]

# Try full deobfuscation with x86 version
.\de4dot.exe -vv ..\..\files_to_deobfuscate\TDUPriceAction.dll -o TDUPriceAction_decrypted.dll
```

## Expected Results

**If it works:**
- No "Only 32-bit dynamic methods decryption is supported" error
- Methods are decrypted dynamically
- You get a fully deobfuscated DLL

**If it still fails:**
- The x86 build might not support .NET 8 properly
- Dynamic decryption might require .NET Framework specifically
- Proceed with dnSpy debugger approach (see DNSPY_DEBUGGING_GUIDE.md)

## Why x86 is Needed

The dynamic decryption feature in de4dot:
1. Loads the protected assembly into a separate process
2. Executes the Agile .NET runtime decryption routines
3. Extracts decrypted methods from memory
4. Patches the original DLL

This requires:
- 32-bit process (Agile .NET runtime code is 32-bit)
- Ability to execute untrusted code
- Inter-process communication

## If x86 Build Fails

### Error: "Could not find platform x86"

The .NET 8 SDK might not support x86 properly. Try:

```powershell
# Check available platforms
dotnet build de4dot.netcore.sln -c Release /p:Platform=AnyCPU

# List platforms
dotnet msbuild de4dot.cui/de4dot.cui.csproj /t:GetPlatforms
```

### Error: "Framework does not support x86"

.NET 8 on Windows should support x86, but if it doesn't:

**Workaround:** Use Visual Studio 2022
1. Open `de4dot.netcore.sln` in Visual Studio
2. Configuration Manager → Platform → x86
3. Build → Build Solution
4. Check `bin\x86\Release` for output

### Error: Dynamic decryption still fails

Some Agile .NET versions use protections that prevent dynamic decryption:
- Anti-debugging checks
- Virtualization
- Tamper detection

In this case, **dnSpy debugger is your only option**.

## Comparison: x86 Build vs dnSpy

### x86 Build Approach
**Pros:**
- Automated (if it works)
- Batch processing possible
- No manual intervention

**Cons:**
- Requires 32-bit build
- Might be blocked by anti-debug
- .NET 8 x86 support uncertain

### dnSpy Debugger Approach
**Pros:**
- Always works (code must decrypt to run)
- Can bypass anti-debug features
- Fine-grained control
- Industry standard

**Cons:**
- Manual process
- Requires running the app
- Per-method extraction

## My Recommendation

1. **Try x86 build first** (5 minutes)
   - If it works: Great! Automated solution.
   - If it fails: No time wasted.

2. **Use dnSpy if x86 fails** (30-60 minutes)
   - More reliable
   - Guaranteed to work
   - Better for understanding the code

Since you said "we need to edit the code since it is not working right now", dnSpy will also help you:
- Debug the actual issue
- See runtime values
- Understand why it's failing
- Test fixes immediately

---

**Bottom line:** Try x86 build, but have dnSpy ready as the backup (and more powerful) solution.
