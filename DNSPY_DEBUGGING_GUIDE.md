# dnSpy Debugger Guide - Extract Decrypted Methods

Since the Agile .NET encryption algorithm is not supported by de4dot's static decryption, we'll use **dnSpy debugger** to extract methods at runtime after Agile .NET decrypts them.

## Why This Works

Agile .NET encrypts method bodies on disk, but **must decrypt them in memory to execute**. dnSpy can:
1. Attach to the running NinjaTrader process
2. Wait for Agile .NET to decrypt methods
3. View/extract the decrypted IL code from memory
4. Save the clean code

This is the standard professional approach for runtime-decrypted assemblies.

---

## Step 1: Download dnSpy

1. Go to: https://github.com/dnSpy/dnSpy/releases
2. Download the latest release (e.g., `dnSpy-netframework.zip`)
3. Extract to a folder (e.g., `C:\Tools\dnSpy`)
4. Run `dnSpy.exe`

---

## Step 2: Prepare Your Environment

### Option A: Debug NinjaTrader Directly

If you can run the indicator in NinjaTrader:

1. **Launch NinjaTrader 8** (don't load the indicator yet)
2. Note the process name: `NinjaTrader.exe`

### Option B: Create a Test Harness

If you want to test the DLL independently, create a simple console app:

```csharp
// TestHarness.cs
using System;
using NinjaTrader.NinjaScript.Indicators;

class Program {
    static void Main() {
        // This will trigger Agile .NET to decrypt methods
        var indicator = new TDU.TDUPriceAction();

        Console.WriteLine("Indicator loaded. Press any key to exit...");
        Console.ReadKey();
    }
}
```

Compile this with references to the protected DLL and NinjaTrader assemblies.

---

## Step 3: Debug with dnSpy

### Start Debugging

**Method 1: Attach to Running Process**
1. Start NinjaTrader 8
2. In dnSpy: `Debug` → `Attach to Process...`
3. Find `NinjaTrader.exe` in the list
4. Click `Attach`

**Method 2: Start with Debugger**
1. In dnSpy: `Debug` → `Start Debugging`
2. Browse to NinjaTrader executable
3. Set breakpoints if needed
4. Click `Start`

### Load Your DLL

Once attached:
1. Go to `Debug` → `Windows` → `Modules`
2. Look for `TDUPriceAction.dll` in the list
3. Right-click → `Open All Modules`
4. Find your assembly in the Assembly Explorer

---

## Step 4: Extract Decrypted Methods

### View a Method

1. **Navigate to the method** you want (e.g., `OnBarUpdate`)
2. **Right-click** the method → `Edit Method (C#)`
3. You'll see the **decrypted code**!

### Example - What You'll See

**Before (in static DLL):**
```csharp
public void OnBarUpdate() {
    <AgileDotNetRTPro>.Initialize();
    ret
}
```

**After (in dnSpy debugger):**
```csharp
public void OnBarUpdate() {
    if (this.CurrentBar < this.BarsRequiredToPlot)
        return;

    this.CalculatePivots();
    this.DrawSignals();
    // ... actual logic!
}
```

### Copy the Code

**Option 1: Edit Method**
1. Right-click method → `Edit Method (C#)`
2. Copy the decompiled C# code
3. Paste into your IDE

**Option 2: Edit Class**
1. Right-click class → `Edit Class (C#)`
2. Get the entire class at once
3. Save to a .cs file

**Option 3: Export Assembly**
1. Right-click assembly → `Save Module...`
2. Save as new DLL with decrypted methods
3. Decompile later with any tool

---

## Step 5: Handle Large Indicators

If the indicator has many methods:

### Use Breakpoints

1. Set a breakpoint in `OnBarUpdate` or `OnStateChange`
2. Let NinjaTrader run until it hits
3. At this point, all methods should be decrypted
4. Browse and extract any method you need

### Batch Extract

```csharp
// Use dnSpy scripting (Debug → Windows → Scripting)
// This pseudocode shows the concept:

foreach (var type in assembly.MainModule.Types) {
    if (type.Namespace.StartsWith("NinjaTrader.NinjaScript.Indicators.TDU")) {
        foreach (var method in type.Methods) {
            // Extract method IL
            var code = method.Body.ToString();
            File.WriteAllText($"{type.Name}_{method.Name}.txt", code);
        }
    }
}
```

---

## Step 6: Reconstruct the Project

Once you have the decrypted code:

### Create New Project

```powershell
# Create new NinjaTrader indicator project
# Using your preferred IDE (Visual Studio, Rider, etc.)
```

### Copy Code

1. Use the **partial deobfuscated DLL** for structure/strings
2. Use **dnSpy extracted code** for method implementations
3. Combine them in your new project

### Fix References

The code might reference:
- NinjaTrader APIs
- Other indicators
- Custom types

Make sure to add proper NuGet packages / assembly references.

---

## Common Issues and Solutions

### Issue: "Cannot attach to process"

**Solution:** Run dnSpy as Administrator
```powershell
# Right-click dnSpy.exe → Run as Administrator
```

### Issue: "Methods still show as encrypted"

**Cause:** The methods haven't been called yet

**Solution:**
1. Let NinjaTrader fully initialize
2. Load a chart with your indicator
3. Let it run for a few bars
4. Then check dnSpy again

### Issue: "Source code not available"

**Cause:** dnSpy is showing IL instead of C#

**Solution:**
1. Right-click method → `Edit Method (C#)` (not IL)
2. Make sure C# view is selected (top-right dropdown)

### Issue: "Decompiled code doesn't compile"

**Cause:** Decompilers aren't perfect

**Solution:**
1. Use as reference, not verbatim copy
2. Fix syntax errors manually
3. Consider using multiple decompilers (ILSpy, dotPeek) and compare

---

## Alternative: Use dnSpy to Patch the DLL

Instead of extracting code, you can **edit and save** directly:

1. Debug as above
2. Find the method you want to fix
3. Right-click → `Edit Method (C#)`
4. Make your changes
5. Compile
6. `File` → `Save Module...`
7. Save as new DLL

This creates a **modified version** without Agile .NET protection!

---

## Pro Tips

### Tip 1: Use Hex Breakpoints

If you know where a bug is:
1. Set a breakpoint at the problematic line
2. Inspect variables in real-time
3. Understand what's failing

### Tip 2: Memory Dumps

Create memory dumps for later analysis:
```powershell
# While debugging in dnSpy
# Debug → Save Dump...
# Save as .dmp file
# Load later: File → Open → Dump File
```

### Tip 3: Compare with Original

Keep the partial deobfuscated output from de4dot:
- Use it for **strings** (already decrypted)
- Use it for **structure** (class/method names)
- Use dnSpy for **method implementations**

---

## What You'll Be Able to Do

After extracting with dnSpy:

✅ **View all method implementations**
- See the actual logic
- Understand how it works
- Find bugs

✅ **Edit the code**
- Fix the issues you mentioned
- Improve performance
- Add features

✅ **Create a clean version**
- Recompile without Agile .NET
- Easier to maintain
- No more encryption issues

---

## Summary of Full Workflow

```
1. Use de4dot with --an-methods false
   → Get strings and structure

2. Use dnSpy debugger on running NinjaTrader
   → Get decrypted method implementations

3. Combine both
   → Full clean source code

4. Create new project
   → Fix issues
   → Recompile
   → Done!
```

---

## Need Help?

If you get stuck at any step:

1. **Check dnSpy documentation**: https://github.com/dnSpy/dnSpy/wiki
2. **dnSpy discussions**: Look for Agile .NET specific threads
3. **Post questions** with:
   - Which step you're on
   - Exact error message
   - Screenshots if relevant

---

## Legal Note

Make sure you have the right to deobfuscate and modify these indicators:
- If you purchased them, check the license agreement
- If they're yours, no problem
- If they're from vendors, verify terms of use

Most NinjaTrader vendors allow personal modifications for debugging/fixing issues.

---

**This is the most reliable method for Agile .NET protected assemblies. Static deobfuscation has limitations, but runtime debugging always works because the code must be decrypted to execute.**
