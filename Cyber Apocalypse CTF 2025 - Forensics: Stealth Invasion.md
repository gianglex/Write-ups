# Cyber Apocalypse CTF 2025: Stealth Invasion

## Team H-T8
Big credits to the H-T8 team. 

Members: 
ilpakka, ExcelAtSheets, ridicculus, lansiri, Mizard, EetHeet, d4ybreak

## Official 

### Description
Selene's normally secure laptop recently fell victim to a covert attack. Unbeknownst to her, a malicious Chrome extension was stealthily installed, masquerading as a useful productivity tool. Alarmed by unusual network activity, Selene is now racing against time to trace the intrusion, remove the malicious software, and bolster her digital defenses before more damage is done.

### Skills Required
- N/A

### Skills Learned
- Volatility Basics
- Chrome Extensions Basics

### Files provided
- memdump.elf: ELF 64-bit LSB core file, x86-64, version 1 (SYSV), 4.13 GB

## Additional information

### Tools used
- volatility3
- SQLiteBrowser

## Flags
Note: Scanning through the memdump takes a while

### 1. What is the PID of the Original (First) Google Chrome process
```python3 vol.py -f memdump.elf windows.pslist | grep chrome```

<img src="https://github.com/user-attachments/assets/1f277be6-1859-41e0-916b-d5ca9e53bff6" width="500"> <br/>

Flag #1: **4080**

### 2. What is the only Folder on the Desktop

Even before scanning, I knew I would likely have to stop the scanning before it finishes. The memdump is just too large and the possible amount of paths and files with the word desktop in it could be a bit too much.

```python3 vol.py -f memdump.elf windows.filescan | grep desktop```

<img src="https://github.com/user-attachments/assets/a6131fa8-f101-477a-8735-c2cf113983c0" width="500"> <br/>

After some results and after seeing the username in question, I restarted the scan with more accurate filepath to get more readable results. 

```python3 vol.py -f memdump.elf windows.filescan | grep 'Users\\selene\\Desktop'```

<img src="https://github.com/user-attachments/assets/afc4dc7d-f5d1-42b5-a29b-2832999dd22c" width="500"> <br/>

Flag #2: **malext**

### 3. What is the Extention's ID (ex: hlkenndednhfkekhgcdicdfddnkalmdm)

I googled to see check the default filepath for chromes extensions

<img src="https://github.com/user-attachments/assets/899d8dcd-22c0-4ad3-b8ae-68e72c90fbea" width="500"> <br/>

```Users > YourUsername > AppData > Local > Google > Chrome > User Data > Default > Extensions```

Based on that I could formulate my next search

```python3 vol.py -f memdump.elf windows.filescan | grep 'Users\\selene\\AppData\\Local\\Google\\Chrome\\User Data\\Default\\Extensions'```

<img src="https://github.com/user-attachments/assets/1fc30a1d-1bc4-4acf-8a22-09abb046d797" width="500"> <br/>

Unfortunately ```nmmhkkegccagdldgiimedpiccmgmieda``` was not the correct flag. 

Since the extension was not saved in the default location, I widened my search to hopefully find the correct path instead. 

```python3 vol.py -f memdump.elf windows.filescan | grep 'Chrome' | grep 'Extension'``` 

<img src="https://github.com/user-attachments/assets/c6df3029-b871-4f00-8e09-5a4b50cd627d" width="500"> <br/>

After examining results, it seems the correct path would've been ```\Users\selene\AppData\Local\Google\Chrome\User Data\Default\Local Extension Settings\```

Flag #3: **nnjofihdjilebhiiemfmdlpbdkbjcpae**

### 4. After examining the malicious extention's code, what is the log filename in which the data is stored

Based on previous search, there wouldn't be many options: 

```
0xa708c8830c80  \Users\selene\AppData\Local\Google\Chrome\User Data\Default\Local Extension Settings\nnjofihdjilebhiiemfmdlpbdkbjcpae\LOG
0xa708c8dd5be0  \Users\selene\AppData\Local\Google\Chrome\User Data\Default\Local Extension Settings\nnjofihdjilebhiiemfmdlpbdkbjcpae\MANIFEST-000001
0xa708c8dda230  \Users\selene\AppData\Local\Google\Chrome\User Data\Default\Local Extension Settings\nnjofihdjilebhiiemfmdlpbdkbjcpae\CURRENTdbtmp
0xa708c8f2b500  \Users\selene\AppData\Local\Google\Chrome\User Data\Default\Local Extension Settings\nnjofihdjilebhiiemfmdlpbdkbjcpae
0xa708c8f2d760  \Users\selene\AppData\Local\Google\Chrome\User Data\Default\Local Extension Settings\nnjofihdjilebhiiemfmdlpbdkbjcpae
0xa708caba14d0  \Users\selene\AppData\Local\Google\Chrome\User Data\Default\Local Extension Settings\nnjofihdjilebhiiemfmdlpbdkbjcpae\000003.log
```

000003.log seemed like the most likely option, altho there were so few options that you could've just tested all options to see which one passes the flag check or extracted the files to read which ones have stored information in them. 

Flag #4: **000003.log**

### 5. What is the URL the user navigated to
Next I googled default Chrome history location.

<img src="https://github.com/user-attachments/assets/06639c3c-ac43-4f82-bb57-68cb1e751e21" width="500"> <br/>

```C:\Users\<username>\AppData\Local\Google\Chrome\User Data\Default```

Based on that information, I did my next filescan. 

```python3 vol.py -f memdump.elf windows.filescan | grep 'Users\\selene\\AppData\\Local\\Google\\Chrome\\User Data\\Default\\History'```

<img src="https://github.com/user-attachments/assets/7e81a927-d8f7-4a36-81d5-cd19bcfc4fcc" width="500"> <br/>

```
0xa708c896f570  \Users\selene\AppData\Local\Google\Chrome\User Data\Default\History
0xa708ca363220  \Users\selene\AppData\Local\Google\Chrome\User Data\Default\History
```

Based on this I then moved to extract the files 

<img src="https://github.com/user-attachments/assets/283f8af1-42c9-4f8a-9d5f-9d20c5f486e1" width="500"> <br/>

```
python3 vol.py -f memdump.elf -o "/home/kali/volatility3/dumps" windows.dumpfiles --virtaddr 0xa708ca363220
python3 vol.py -f memdump.elf -o "/home/kali/volatility3/dumps" windows.dumpfiles --virtaddr 0xa708c896f570
```

<img src="https://github.com/user-attachments/assets/38eef72b-f617-40b1-9224-cefbab8babf3" width="500"> <br/>

This flag was quite finicky but after some trial and error, I was able to find the correct flag. 
While doing the next flag, I noticed there would have been an easier way to get this flag.

Flag #5: **drive.google.com**

### 6. What is the password of selene@rangers.eldoria.com

For the last flag, we want to examine the log file we found on Flag #4. 

```0xa708caba14d0  \Users\selene\AppData\Local\Google\Chrome\User Data\Default\Local Extension Settings\nnjofihdjilebhiiemfmdlpbdkbjcpae\000003.log```

For examining the file, we first have to extract the file. 

```python3 vol.py -f memdump.elf -o "/home/kali/volatility3/dumps" windows.dumpfiles --virtaddr 0xa708caba14d0```

<img src="https://github.com/user-attachments/assets/9aa1c90d-337f-43ff-848f-5fd157428cc2" width="500"> <br/>

```strings file.0xa708caba14d0.0xa708c9d90d00.DataSectionObject.000003.log.dat```

<img src="https://github.com/user-attachments/assets/b52c9896-15f2-443d-8ec7-2534ad739129" width="500"> <br/>

Based on the data, we can deduce the extension had been likely been keylogging selene. 

```loga"drive.google.comEnter\r\nselene|Shift|@rangers.eldoria.comEnter\r\nclip-mummify-proofsEnter\r\n"```

Analyzing the last line, we can deduce the last url selene visited was drive.google.com (Flag #5), her email was selene@rangers.eldoria.com and her password was clip-mummify-proofs

Flag #6: **clip-mummify-proofs**
