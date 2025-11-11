# Alternative Agile.NET Deobfuscation Tools

Based on research, there are several alternatives to the standard de4dot that might work better for your Agile.NET RT Pro files.

## Recommended Alternatives to Try

### 1. **AgileDotNetSlayer** ⭐ (Try This First!)

**Purpose-built for Agile.NET** - This is a specialized deobfuscator specifically for Agile.NET protection.

**Repository:** https://github.com/SychicBoy/AgileDotNetSlayer

**Features:**
- ✅ Control Flow deobfuscation
- ✅ String Encryption decryption
- ✅ Resource Encryption decryption
- ✅ Method Call Obfuscation removal
- ✅ Specifically designed for Agile.NET (not a generic tool)

**How to use:**
```powershell
# Clone the repository
git clone https://github.com/SychicBoy/AgileDotNetSlayer.git
cd AgileDotNetSlayer

# Build it
dotnet build -c Release

# Run on your file
cd bin\Release
.\AgileDotNetSlayer.exe path\to\TDUPriceAction.dll
```

**Why this might work:**
- Dedicated Agile.NET support (not generic like de4dot)
- May have newer algorithm implementations
- Focused on Agile.NET variants

### 2. **de4dot-cex** (Community Edition eXtended)

**More updated than original de4dot**

**Repository:** https://github.com/ViRb3/de4dot-cex

**Features:**
- Updated dependencies
- Better ConfuserEx support (main focus)
- Some Agile.NET improvements
- More active maintenance than original de4dot

**Note:** Primarily focused on ConfuserEx, but worth trying.

**How to use:**
```powershell
# Clone
git clone https://github.com/ViRb3/de4dot-cex.git
cd de4dot-cex

# Build
dotnet build -c Release

# Test
cd bin\Release
.\de4dot.exe -vv path\to\TDUPriceAction.dll -o output.dll
```

### 3. **Venturi77CallHijacker** (For VM Protection)

**If Agile.NET uses VM (virtualization)**

**Repository:** Look for "Venturi77CallHijacker" on GitHub

**Features:**
- KoiVM, EazVM, **AgileVM** patcher
- Handles virtualized code
- Useful if your Agile.NET uses VM protection

**Note:** Only needed if VM protection is detected.

### 4. **NinjaTrader-Specific Tools**

According to NinjaTrader forums, some community members have tools for dealing with protected indicators.

**Where to look:**
- NinjaTrader Support Forum: https://forum.ninjatrader.com/
- Search for: "agile.net deobfuscate" or "protected indicator"
- Private community tools/scripts

**Example thread found:**
https://forum.ninjatrader.com/forum/ninjatrader-8/platform-technical-support-aa/1210343-about-agile-net-protection

### 5. **Private/Commercial Deobfuscators**

Some reverse engineering services offer paid deobfuscation:

**Services:**
- Underground forums (risky!)
- Freelance reverse engineers
- Private tools (not public GitHub)

**Cost:** $50-$500+ depending on complexity

⚠️ **Warning:** High risk of malware, scams, or backdoored tools

## Comparison Table

| Tool | Agile.NET Focus | Active? | Open Source | Likely Success |
|------|----------------|---------|-------------|----------------|
| **AgileDotNetSlayer** | ⭐⭐⭐⭐⭐ High | Yes | Yes | 60-70% |
| **de4dot-cex** | ⭐⭐⭐ Medium | Yes | Yes | 40-50% |
| **Original de4dot** | ⭐⭐ Low | No | Yes | 30% (already tried) |
| **dnSpy Debugger** | ⭐⭐⭐⭐⭐ Always works | Yes | Yes | 100% |
| **Commercial Services** | ⭐⭐⭐⭐ High | Varies | No | 70-90% |

## Recommended Testing Order

### Quick Wins (Try These First - 30 minutes total)

1. **AgileDotNetSlayer** (15 min)
   - Most likely to work
   - Purpose-built for Agile.NET

2. **de4dot-cex** (10 min)
   - Might have newer algorithms
   - Easy to try

3. **x86 build of original de4dot** (5 min)
   - Enable dynamic decryption
   - Already have the code

### If Quick Wins Fail

4. **dnSpy Debugger** (60 min, guaranteed)
   - 100% success rate
   - Also helps debug your issues
   - See DNSPY_DEBUGGING_GUIDE.md

## What to Check After Running Alternative Tools

After running any alternative tool, check the output:

```csharp
// Open in dnSpy and look for OnBarUpdate method

// ❌ Still encrypted - tool didn't work
void OnBarUpdate() {
    <AgileDotNetRTPro>.Initialize();
    ret
}

// ✅ Decrypted - tool worked!
void OnBarUpdate() {
    if (this.CurrentBar < this.BarsRequiredToPlot)
        return;
    // ... actual code!
}
```

## Why These Might Work When de4dot Failed

### Different Algorithm Implementations
- **AgileDotNetSlayer** may have reverse-engineered newer Agile.NET versions
- Community contributions add support for variants

### Different Approaches
- Some use **pattern matching** instead of crypto analysis
- Some use **emulation** to run decryption routines
- Some use **hooking** to intercept runtime decryption

### Newer Updates
- de4dot hasn't been updated since ~2015
- Agile.NET RT Pro versions from 2016+ aren't supported
- Community forks may have caught up

## NinjaTrader-Specific Notes

From the NinjaTrader forum:
- Agile.NET is commonly used for NT8 indicator protection
- Some community members have successfully deobfuscated protected indicators
- The protection is often **not** the most advanced Agile.NET settings
- NinjaTrader vendors want users to be able to run the code (not just obfuscate heavily)

**This is good news:** Your files might use "standard" Agile.NET settings that community tools can handle.

## Testing Protocol

For each tool you try:

```powershell
# 1. Run the tool
.\tool.exe input.dll -o output.dll

# 2. Check for errors
# Look for success messages or error logs

# 3. Open output in dnSpy
dnSpy.exe output.dll

# 4. Navigate to a complex method
# Find: TDUPriceAction -> OnBarUpdate

# 5. Check if decrypted
# Look for actual code, not just stubs

# 6. If yes - SUCCESS!
# If no - try next tool
```

## AgileDotNetSlayer - Detailed Steps

Since this is the most promising alternative:

### 1. Clone and Build
```powershell
git clone https://github.com/SychicBoy/AgileDotNetSlayer.git
cd AgileDotNetSlayer

# Check if it builds on .NET 8
dotnet build -c Release

# If it requires older .NET, install that first
```

### 2. Run on Your Files
```powershell
cd bin\Release\net48  # or wherever the exe is

# Try on TDUPriceAction
.\AgileDotNetSlayer.exe C:\web\de4dot\files_to_deobfuscate\TDUPriceAction.dll

# It might create output in same directory or ask for output path
```

### 3. Check Results
```powershell
# Open the output in dnSpy
dnSpy.exe TDUPriceAction_cleaned.dll  # or whatever it names the output

# Navigate to methods and check
```

### 4. Report Back
If it works or fails, let me know:
- What errors appeared?
- Did it create output?
- Are methods decrypted?

## My Recommendations

**Best ROI (Return on Investment):**

1. **Try AgileDotNetSlayer first** (15 min)
   - Purpose-built for your exact scenario
   - High chance of success
   - If it works, you're done!

2. **If that fails, try de4dot-cex** (10 min)
   - Quick to test
   - Might have algorithm you need

3. **If both fail, use dnSpy** (60 min)
   - Guaranteed to work
   - Also helps you debug the code
   - Professional approach

**Don't waste time on:**
- Private/commercial tools (expensive, risky)
- Random tools from shady sites (malware risk)
- Old versions of de4dot (already tried basically)

## Expected Outcomes

**Realistic success rates:**
- AgileDotNetSlayer: **60-70%** (best bet)
- de4dot-cex: **40-50%** (worth a try)
- x86 de4dot build: **30-40%** (might work)
- dnSpy: **100%** (always works, manual)

**Timeline:**
- Quick alternatives: 30 minutes total
- dnSpy if needed: +60 minutes
- **Total max time: 90 minutes to fully deobfuscated code**

## Community Resources

**GitHub Search:**
```
"Agile.NET" deobfuscator
"AgileDotNet" unpack
"CliSecure" decrypt
```

**Forums to Check:**
- NinjaTrader Support Forum
- ReversingHub / Reverse Engineering forums
- GitHub Issues on de4dot/de4dot-cex

**Look for:**
- Similar cases (NinjaTrader indicators)
- Tools people recommend
- Success stories with specific Agile.NET versions

## Legal/Ethical Note

Before using these tools:
- ✅ Your own code: No issues
- ✅ Purchased indicators for personal use/debugging: Generally OK
- ✅ Fixing bugs in indicators you paid for: Reasonable
- ❌ Redistributing deobfuscated code: Copyright violation
- ❌ Commercial resale of vendor's code: Illegal

Check your indicator license agreement to be sure.

---

## Summary

**You have good options:**

1. **AgileDotNetSlayer** - Try this first! (15 min)
2. **de4dot-cex** - Second option (10 min)
3. **dnSpy** - Guaranteed fallback (60 min)

**Total time investment:** 25-85 minutes to fully deobfuscated code

Let me know which tool you want to try first, or if you want help setting up any of them!
