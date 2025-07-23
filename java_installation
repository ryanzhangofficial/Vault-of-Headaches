# Java Installation (Windows)

This guide provides steps to install and get Java working on your computer. Especially helpful if you're using a work computer with restrictions.

## Full Steps (ZIP method)

1. **Download & extract**
   - Grab `jdk-24.0.2_windows-x64_bin.zip` or a version of your choice (Oracle/OpenJDK).
   - Extract to `C:\Java\jdk-24.0.2` (any path works; just be consistent).

2. **Set environment variables (Admin PowerShell)**

   ```powershell
   setx /M JAVA_HOME "C:\Java\jdk-24.0.2"
   setx /M PATH "%JAVA_HOME%\bin;%PATH%"
   ```

   These write to the registry; they won’t change your current shell.

3. **Verify (in a fresh shell)**

   ```powershell
   java -version
   javac -version
   ```

4. **(Optional) One-off test without reopening**

   ```powershell
   $env:JAVA_HOME = "C:\Java\jdk-24.0.2"
   $env:Path = "$env:JAVA_HOME\bin;$env:Path"
   java -version
   ```

## Common Errors & Quick Fixes

| Symptom / Error | Likely Cause | Fix |
| --- | --- | --- |
| `java : The term 'java' is not recognized` | `%JAVA_HOME%\bin` not in `PATH` or didn’t reopen shell | Add to PATH, open new terminal |
| `curl : Cannot find drive 'https'` | PowerShell alias confusion | Use `curl.exe` or proper `Invoke-WebRequest` |
| WinGet `0x8A150040` | Corrupt/outdated source or App Installer | Update App Installer, `winget source update` |
| Gradle warning: `Use --enable-native-access=ALL-UNNAMED` | JDK 24+ restricted native access | Add `org.gradle.jvmargs=--enable-native-access=ALL-UNNAMED` to `gradle.properties` |
