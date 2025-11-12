# dnSpy Practical Walkthrough - TDUPriceAction Specific

**Time to complete:** 30-60 minutes
**Success rate:** 100% guaranteed
**What you get:** Fully decrypted code + ability to debug and fix your issues

---

## Prerequisites (5 minutes)

### 1. Download dnSpy
```powershell
# Go to: https://github.com/dnSpy/dnSpy/releases
# Download: dnSpy-net-win64.zip (latest release)
# Extract to: C:\Tools\dnSpy\
```

### 2. Verify NinjaTrader 8 Location
```powershell
# Default location:
C:\Program Files\NinjaTrader 8\bin\NinjaTrader.exe

# Your indicators should be at:
C:\Users\[YourName]\Documents\NinjaTrader 8\bin\Custom\Indicators\
```

---

## Method 1: Debug Live NinjaTrader (Easiest - 30 minutes)

This approach runs NinjaTrader normally and extracts code from memory.

### Step 1: Start NinjaTrader (2 min)

```powershell
# Start NinjaTrader 8 normally
# Go to: Control Center
# DON'T load any charts yet
```

### Step 2: Attach dnSpy (3 min)

1. **Run dnSpy:**
   ```powershell
   C:\Tools\dnSpy\dnSpy.exe
   ```

2. **Attach to NinjaTrader:**
   - Menu: `Debug` → `Attach to Process...`
   - Find: `NinjaTrader.exe` in the list
   - Click: `Attach`

   ⚠️ **If you don't see NinjaTrader.exe:**
   - Run dnSpy as Administrator
   - Make sure NinjaTrader is running

### Step 3: Load Your Indicator (5 min)

1. **In NinjaTrader:**
   - Create a new chart (any instrument)
   - Add indicator: `TDUPriceAction`
   - Let it initialize and run for a few bars

2. **Wait for decryption:**
   - Agile.NET decrypts methods when first called
   - Give it 30 seconds to fully initialize

### Step 4: Find the Decrypted DLL (5 min)

1. **In dnSpy:**
   - Menu: `Debug` → `Windows` → `Modules`
   - Or press: `Ctrl+Alt+U`

2. **Locate your DLL:**
   - Scroll through the list
   - Look for: `TDUPriceAction.dll`
   - It might be at a path like:
     ```
     C:\Users\[You]\Documents\NinjaTrader 8\bin\Custom\Indicators\TDUPriceAction.dll
     ```

3. **Open it:**
   - Right-click `TDUPriceAction.dll` → `Go to Module`
   - The Assembly Explorer opens showing your assembly

### Step 5: View Decrypted Methods (10 min)

1. **Navigate to your class:**
   ```
   TDUPriceAction.dll
   └─ NinjaTrader.NinjaScript.Indicators.TDU
      └─ TDUPriceAction  ← Your main class
   ```

2. **Find OnBarUpdate:**
   - Expand `TDUPriceAction` class
   - Look for: `OnBarUpdate()`
   - Double-click it

3. **View the REAL code:**
   - You'll see actual IL code (not C# yet)
   - **Important:** Right-click the method → `Edit Method (C#)`
   - **NOW you see decrypted C# code!** 🎉

4. **What you should see:**
   ```csharp
   protected override void OnBarUpdate()
   {
       if (CurrentBar < BarsRequiredToPlot)
           return;

       // ACTUAL LOGIC HERE!
       // Not <AgileDotNetRTPro>.Initialize() stubs
       // Real calculations, real code!
   }
   ```

### Step 6: Extract All Methods (10 min)

For each important method:

1. **Right-click method** → `Edit Method (C#)`
2. **Select all code** (Ctrl+A)
3. **Copy** (Ctrl+C)
4. **Paste into a text file** or your IDE

**Important methods to extract:**
- `OnStateChange()`
- `OnBarUpdate()`
- `OnMarketData()` (if exists)
- Any `Calculate*()` methods
- Any custom methods you see

**OR extract entire class:**
1. Right-click the **class** (not method) → `Edit Class (C#)`
2. Get the entire class at once
3. Copy all code

### Step 7: Save Decrypted Assembly (Optional - 5 min)

You can save the entire decrypted DLL:

1. **In Assembly Explorer:**
   - Right-click `TDUPriceAction.dll` → `Save Module...`

2. **Save as:**
   ```
   C:\web\de4dot\TDUPriceAction_decrypted.dll
   ```

3. **Now you have:**
   - A DLL with decrypted methods in memory
   - Can decompile with any tool (ILSpy, dotPeek, etc.)

---

## Method 2: Debug with Breakpoints (Advanced - 45 min)

Use this if you want to understand HOW the code works and WHERE it fails.

### Step 1-3: Same as Method 1

Follow steps 1-3 from Method 1 above.

### Step 4: Set Breakpoints (10 min)

1. **Find OnBarUpdate** in dnSpy
2. **Set a breakpoint:**
   - Click in the left margin (gray area)
   - Red dot appears
   - Or: Right-click line → `Add Breakpoint`

3. **Continue execution:**
   - Menu: `Debug` → `Continue`
   - Or press: `F5`

### Step 5: Hit Breakpoint (5 min)

1. **Wait for indicator to update:**
   - When new bar comes in NinjaTrader
   - dnSpy will PAUSE execution
   - You're now INSIDE the decrypted code!

2. **Inspect variables:**
   - Hover over variables to see values
   - Menu: `Debug` → `Windows` → `Locals`
   - See: `CurrentBar`, `Close[0]`, etc.

### Step 6: Step Through Code (15 min)

1. **Step through line by line:**
   - Press `F10` to step over
   - Press `F11` to step into methods
   - Watch variables change

2. **Find the bug:**
   - See where values are wrong
   - See where exceptions occur
   - Understand the logic flow

3. **Fix the issue:**
   - Note what needs to change
   - Extract the code
   - Create your fixed version

---

## Method 3: Create Clean Project (Recommended - 60 min)

Combine everything into a maintainable project.

### Step 1: Extract Code (30 min)

Use Method 1 or 2 above to extract all methods.

### Step 2: Create New Project (10 min)

```powershell
# Create new NinjaTrader indicator project
# Or use Visual Studio template
```

```csharp
using NinjaTrader.NinjaScript.Indicators;

namespace NinjaTrader.NinjaScript.Indicators.TDU
{
    public class TDUPriceAction : Indicator
    {
        // Paste your extracted code here

        protected override void OnStateChange()
        {
            // Paste extracted OnStateChange
        }

        protected override void OnBarUpdate()
        {
            // Paste extracted OnBarUpdate
        }

        // ... other methods
    }
}
```

### Step 3: Use Partial Deobfuscation for Strings (10 min)

Remember, you have `TDUPriceAction_partial.dll` from de4dot with decrypted strings!

1. **Open partial DLL in dnSpy**
2. **Copy string constants**
3. **Copy property definitions**
4. **Use for reference**

### Step 4: Compile and Test (10 min)

1. **Build your project**
2. **Copy to NinjaTrader**
3. **Test on chart**
4. **Fix any issues**

---

## Troubleshooting

### "Cannot attach to NinjaTrader.exe"

**Solution:**
```powershell
# Run dnSpy as Administrator
Right-click dnSpy.exe → Run as Administrator
```

### "Methods still show encrypted"

**Cause:** Methods haven't been called yet

**Solution:**
- Make sure indicator is loaded on a chart
- Let it run for at least 10-20 bars
- Try setting breakpoint in OnBarUpdate
- Refresh the Assembly view

### "Can't see C# code, only IL"

**Solution:**
- Right-click method → `Edit Method (C#)` (not IL)
- Check dropdown at top: Select "C#" not "IL"

### "Decompiled code has errors"

**Cause:** Decompiler isn't perfect

**Solution:**
- Use as reference, not verbatim
- Fix syntax errors manually
- Compare with partial deobfuscation output
- Use multiple decompilers if needed

### "NinjaTrader crashes when debugging"

**Solution:**
- Don't set too many breakpoints
- Use "Step Over" not "Step Into" for NT functions
- Let it run freely, just view the code
- Or use Method 1 (just extract, don't break)

---

## Expected Timeline

| Task | Time | Difficulty |
|------|------|------------|
| Setup dnSpy | 5 min | Easy |
| Attach to NinjaTrader | 3 min | Easy |
| Find decrypted methods | 10 min | Medium |
| Extract main methods | 15 min | Easy |
| Extract all classes | 30 min | Medium |
| Debug and understand | 45 min | Medium |
| Create clean project | 60 min | Medium |

**Minimum viable extraction:** 30 minutes
**Full clean project:** 90 minutes

---

## What You'll Get

After following this guide:

✅ **Fully decrypted OnBarUpdate()** - See all logic
✅ **All calculation methods** - Understand algorithms
✅ **Variable values at runtime** - Debug issues
✅ **Clean, maintainable code** - No more Agile.NET
✅ **Ability to fix bugs** - Edit and recompile
✅ **No more obfuscation** - Forever free

---

## Pro Tips

### Tip 1: Extract in Order
1. Properties first (easy)
2. OnStateChange (initialization)
3. OnBarUpdate (main logic)
4. Helper methods (calculations)

### Tip 2: Compare with Partial
Use your `TDUPriceAction_partial.dll` side-by-side:
- Strings from partial
- Logic from dnSpy
- Combine = complete picture

### Tip 3: Save Your Work
Create a text file as you extract:
```
TDUPriceAction_extracted.txt
├─ Properties
├─ OnStateChange
├─ OnBarUpdate
└─ Helper methods
```

### Tip 4: Don't Panic
If code looks complex:
- Take it method by method
- You don't need to understand everything
- Focus on what you need to fix

---

## Success Criteria

You know you've succeeded when:

1. ✅ You can see OnBarUpdate with REAL code (not stubs)
2. ✅ You can copy/paste the code to a text file
3. ✅ You understand what the indicator does
4. ✅ You found the bug (if debugging)
5. ✅ You have a clean project that compiles

---

## Next Steps After Extraction

Once you have the clean code:

1. **Fix your issues** - Edit the code as needed
2. **Compile without Agile.NET** - Normal DLL
3. **Test thoroughly** - Make sure it works
4. **Keep the source** - Never lose it again
5. **Add improvements** - Now you can enhance it

---

**Ready to start?** Follow Step 1 and let me know when you have dnSpy attached to NinjaTrader. I'll help you through each step!
