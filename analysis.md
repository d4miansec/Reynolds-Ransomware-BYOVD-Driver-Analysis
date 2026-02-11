# Static Analysis
This report documents the static analysis of the “Reynlods” ransomware sample.
The executable is a 64-bit C++ PE compiled for Windows Vista (Timestamp: November 2025) and exhibits typical Ransomware behavior including data encryption techniques and abusing vulnerable drivers.

**SHA-256:** 6BD8A0291B268D32422139387864F15924E1DB05DBEF8CC75A6677F8263FA11D

**MD5:**     F0BDB2ADD62B0196A50E25E45E370CC5

<img width="807" height="261" alt="VirtualBoxVM_pOxpVrwX0E" src="https://github.com/user-attachments/assets/68a62245-f5d2-4bb3-a825-302db3b78f8e" />

## Anti Analysis: UPX Packed
This sample is packed with UPX. UPX is a very common packer used by many threat actors. It allows the actual program code to be stored encoded in the binary, and at runtime extracted into memory and executed unpacked.
This is done to prevent software from scanning the payload and detecting the malware.

To unpack the sample you just have to download the original UPX packer from github and use the "-d" command line to reverse the sample.
Now that we have unpacked it, we can observe typical ransomware strings.

<img width="512" height="522" alt="image" src="https://github.com/user-attachments/assets/f37e9d50-3fb1-4941-bcbd-f06e351d80ff" />
<img width="743" height="116" alt="image" src="https://github.com/user-attachments/assets/8e363202-bd10-4b86-ab33-c0659e68961d" />
<img width="459" height="617" alt="image" src="https://github.com/user-attachments/assets/f6dbca09-dbe2-4037-82c2-6150a1e24e43" />

In my point of view, this ransomware seems to still be in development because of lacking default features like network propagation or backup deletion.

## Exploring BYOVD

Next, as we analyze the malware using Ghidra, we can see the BYOVD (Bring Your Own Vulnerable Driver) technique in action.
The actual vulnerable driver is embedded in the ransomware sample, decrypted at runtime using RC4, and then stored in the ProgramData folder as "02.sys".

<img width="761" height="650" alt="image" src="https://github.com/user-attachments/assets/cba5c7a1-1bb5-4338-ae75-23ceda102ec2" />

The "SystemFunction032" function is often linked to cryptographic operations, specifically for encrypting and decrypting data with the RC4 algorithm.
If the vulnerable driver is dropped successfully, it attempts to load the driver using the low-level function "NtLoadDriver".
To load the driver the initial program needs specific Token privileges "SeLoadDriverPrivilege". 

<img width="659" height="449" alt="image" src="https://github.com/user-attachments/assets/473cdc8c-3a26-49e4-b9f6-16dca16e42e1" />

To set up the driver, the malware creates a new registry key named "NSecKrnl". 
It then loads the driver into the kernel using "NtLoadDriver" and proceeds to send malicious IOCTL commands to terminate EDR, AV, and monitoring processes.

<img width="541" height="122" alt="image" src="https://github.com/user-attachments/assets/3d1045cd-a115-4646-9a2c-fd64549ec9ac" />
<img width="670" height="668" alt="image" src="https://github.com/user-attachments/assets/24eb550d-7234-4bc5-ab49-07a8320f3473" />

Here is the list of all AV and EDR solutions that the Reynolds Ransomware terminates if successful:

* **SEDService.exe**
* **SophosHealth.exe**
* **SophosFS.exe**
* **SSPService.exe**
* **SophosFileScanner.exe**
* **McsAgent.exe**
* **McsClient.exe**
* **SophosLiveQueryService.exe**
* **SophosNetFilter.exe**
* **SophosNtpService.exe**
* **hmpalert.exe**
* **Sophos.Encryption.BitLockerService.exe**
* **SophosOsquery.exe**
* **ccSvcHst.exe**
* **SymCorpUI.exe**
* **SISIPSService.exe**
* **SISIDSService.exe**
* **SmcGui.exe**
* **sisipsutil.exe**
* **sepWscSvc64.exe**
* **MsMpEng.exe**
* **CSFalconService.exe**
* **cydump.exe**
* **cyreport.exe**
* **cyrestart.exe**
* **cyrprtui.exe**
* **cyserver.exe**
* **cytool.exe**
* **cytray.exe**
* **cyuserserver.exe**
* **CyveraConsole.exe**
* **tlaworker.exe**
* **ekrn.exe**
* **eguiProxy.exe**
* **egui.exe**
* **aswEngSrv.exe**
* **aswidsagent.exe**
* **AvastUI.exe**

## Conclusion

The analysis of Reynolds Ransomware highlights its sophisticated use of BYOVD techniques and typical ransomware behaviors, such as encryption and process termination. Despite its current developmental state, which reflects missing features like network propagation, the integration of a vulnerable driver poses significant risks. Organizations must remain vigilant and implement robust security measures, including regular updates and monitoring, to defend against such evolving threats. Understanding these tactics is crucial for improving cybersecurity strategies and ensuring resilience in the face of ransomware attacks.
