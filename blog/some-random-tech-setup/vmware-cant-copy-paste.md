---
description: >-
  Issue with vmware usually happens where the virtual machine and host machine
  doens't share the clipboard. meaning that copying text form host machine will
  be unable to paste in the virtual machine.
---

# VMWARE can't copy paste

This issue occurs with me all the time and I couldn't figure out why it was happening. I found that the VMWARE keep deleting my configuration under `kali-linux-2023.2-vmware-amd64.vmx` file.

This file is found under the directory where the Virtual Machine is located. To fix this issue find the .vmx file and open with your preferred document editor.

<figure><img src="../../.gitbook/assets/image (15) (1).png" alt=""><figcaption><p>location of .vmx file</p></figcaption></figure>

> Note that the location will vary among different users since the installed virtual machine could be setup differently.

Add the following lines to the .vmx file

```
isolation.tools.copy.disable = "FALSE"
isolation.tools.paste.disable = "FALSE"
isolation.tools.SetGUIOptions.enable = "TRUE"
```

<figure><img src="../../.gitbook/assets/image (20) (1).png" alt=""><figcaption><p>added the islation.tools configuration to the .vmx file</p></figcaption></figure>

Now restart the VMWARE workstation and copy paste should be working.

