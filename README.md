# Verint.Microsoft.Speech

[NuGet package](https://www.nuget.org/packages/verint.microsoft.speech.x64) for the Microsoft.Speech managed code class library for x64 runtimes. Install with:

```
Install-Package Verint.Microsoft.Speech
```

To use this package, you must have the [Microsoft Speech SDK](https://www.microsoft.com/en-us/download/details.aspx?id=27226) installed. This package includes three Microsoft.Speech library versions, all for x64 runtimes:

* For .NET Framework targets, the official, signed libraries from the SDK.
* For .NET Standard 1.0-2.0 targets, a recompiled (unsigned) .NET Framework version with one slight edit to allow for .NET Standard compatibility.
* For .NET Standard 2.1+ targets, a recompiled (unsigned) .NET Standard version with very slight edits to allow for real compatibility without any warnings.

Note that while we have tested the speech recognition portion of the library under .NET Core, it's impossible to say that it is 100% compatible. Caveat emptor!

## .NET Standard 1.0-2.0 Edit

The official Microsoft.Speech library crashes under .NET Core apps due to a failed bit of reflection in the `HKEYfromRegKey()` function.  The original code has the following line:

```cs
Type typeFromHandle = typeof(RegistryKey);
BindingFlags bindingAttr = BindingFlags.Instance | BindingFlags.NonPublic;
FieldInfo field = typeFromHandle.GetField("hkey", bindingAttr);
```

It's a sneaky way to get the internal handle for the `RegistryKey` class.  In .NET Core, the name of that field has been changed to `_hkey`, so the edited code looks like this:

```cs
Type typeFromHandle = typeof(RegistryKey);
BindingFlags bindingAttr = BindingFlags.Instance | BindingFlags.NonPublic;
FieldInfo field = typeFromHandle.GetField("hkey", bindingAttr) ??
                  typeFromHandle.GetField("_hkey", bindingAttr)
```

In other words, the resulting DLL can handle both .NET Framework and .NET Core apps.

## .NET Standard 2.1+ Edit

Porting the library to .NET Standard proper requires a dependency on the **Microsoft.Win32.Registry** package, and I had to remove the security attributes in the AssemblyInfo:

```cs
[assembly: SecurityPermission(SecurityAction.RequestMinimum, SkipVerification = true)]
[assembly: FileIOPermission(SecurityAction.RequestOptional, Unrestricted = true)]
[assembly: EnvironmentPermission(SecurityAction.RequestOptional, Unrestricted = true)]
[assembly: RegistryPermission(SecurityAction.RequestOptional, Unrestricted = true)]
[assembly: SecurityPermission(SecurityAction.RequestOptional, Unrestricted = true)]
[assembly: PermissionSet(SecurityAction.RequestOptional, Unrestricted = true)]
```

Those are all gone in the new version.
