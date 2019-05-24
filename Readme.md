### Windows Installer Bypass using Rollback Script (RBS and RBF) - Race Condition


Thanks to @SandboxEscaper for releasing this exploit code on Gtihub, it's very interesting to analyze :)

**Source:** https://github.com/SandboxEscaper/polarbearrepo/tree/master/InstallerBypass

**Windows Installer**

Windows Installer accomplishes rollback by creating a rollback script. A rollback script is a file that contains a linear sequence of operations to perform, such as file and registry updates, configuration information updates, user interface notifications, and state information for other operations.

Each operation recorded in the rollback script is a direct response to an operation in the installation script. Rollback scripts are stored in binary format.

This improves efficiency, avoids the need for parsing the file, and discourages manual editing of the file.

Rollback script files (.RBS and .RBF) are backups of existing files. Files with an .RBS file extension are rollback script files, and files with an .RBF file extension are backups of existing files. Both are stored in hidden folders called Config.msi.

The Config.msi folders are created when the Msiexec.exe file starts copying from the installation point. The rollback script file (.RBS) is always stored in the Config.msi folder on the disk where the operating system is installed.

The .RBF files are stored in the Config.msi folder on the disk where the program that is being backed up currently resides.This is done so that there is no crossing of disks when the program files are being backed up. All rollback files and the Config.msi folders are deleted when the installation completes successfully.

**Tested on:** Windows 10 x64 <br/>
**Environment:** VM

## How exploit work?

Race condition in Windows Installer, which is exploited by creating OpLock for the directory "c:\\config.msi"  with callbackfunction and then create a  mountpoint (called as junctions or reparse points) before it writes DACL. It will provide us a small timing windows to trigger the rollback (.rbs and .rbf) action by clicking cancel button and  it will perform the rollback operation by writing the oops.dll to "c:\\windows\system32\" .  

Interesting thing is, this could  be used with malware and  programmatically trigger the rollback action to exploit it.  Maybe you can even pass the silent flag "/qn" to hide installer UI and find another way to trigger rollback (i.e through installer api, injecting into medium Integrity Level(IL) msiexec etc)

## How to reproduce?

1. Run polarbear.exe (make sure to copy `test.rbf` and `test.rbs` in the same directory.
2. Open a cmd and run an installer (has to be an autoelevating installer in C:\Windows\nsatller) this way `msiexec /fa c:\\windows\\installer\\3f223a1.msi` When we pass the repair flag, it usually gives us a little more time to press the cancel button and trigger rollback. polarbear.exe will print out when you have to press cancel. So you don't press it too early!
3. If all is successful it will write `oops.dll` to system32. If failed.. make sure to delete the following folders: config.msi, new, new2, new3 in C:\.


## Detailed Explantion about .rbf and .rbs:

Coming soon...

## Demo
 
[![Alt text](https://img.youtube.com/vi/FGUnmw7VSP0/0.jpg)](https://www.youtube.com/watch?v=FGUnmw7VSP0)
  

## :octocat:Credits:
* Umar Farook: [OSCE | Technology Security Analyst | DevSecops | Researcher](https://www.linkedin.com/in/Umar-Farook)
* FOS Team : [Fools of Security](https://www.youtube.com/channel/UCEBHO0kD1WFvIhf9wBCU-VQ)

## Support !
  
Email address: umarfarookmech712@gmail.com  or pingus@foolsofsecurity.com <br/>
Youtube: [Fools Of Security](https://www.youtube.com/channel/UCEBHO0kD1WFvIhf9wBCU-VQ)<br/>
Website: [Fools Of Security Community](https://foolsofsecurity.com) <br/>


**Reference:**

- [FileOpLock::CreateLock](https://wikileaks.org/ciav7p1/cms/page_38371356.html)
- [ReparsePoint::CreateMountPoint](https://github.com/googleprojectzero/symboliclink-testing-tools/tree/master/CreateMountPoint)

