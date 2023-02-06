---
title: "Chapter-9 Write-up"
classes: wide
header:
  teaser: /assets/images/site-images/PMA_logo.jpg
ribbon: Black
description: "Chapter 9 write-up from Practical Malware Analysis Book"
categories:
  - Tutorials Summaries
toc: true
---

# Chapter 9 challenges the walkthrough

# 1- Lab09-01.exe:
---

First, this malware uninstalls itself if you don't provide the expected **argument and** password**.

So, first to be able to answer the challenge questions we need to figure out the argument and password or **patch the challenge**.

As mentioned in the challenge description we will analyze this malware using **IDA pro and OllyDBG** but instead of Olly, I'll use x64DBG.

So, when we load the malware into IDA and go to main we will see a comparison between **argc** and 1 

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-1/argc_compare.PNG)

So, as we don't enter any args so we will jump to perform sub_40100*Inside this function, it will try to open **Reg** (SOFTWARE\\Microsoft \\XPS)

but the malware fails to open this Reg because **RegOpenKeyExA** returns a nonzero error code "2" **Look At EAX**

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-1/Fail_to_open_reg.PNG)

After this, malware will take a jump to perform **sub_402410**.

After Debugging this function it will take the malware path and its name and delete it using **del** command with **cmd.exe**

So, we need to get the argument and password so let's debug the main and try to find these boys.

After re-debugging the malware with x64dbg and running it with a test password, now malware will take this password and check if it's right or not and then check the password, but now we need to know the right password.

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-1/before_checking.PNG) <!--before_checking.PNG-->

Inside this function **(sub_02510)**, it will make some checks and compare the size of the password with 4 so we need a password like this **"xyzr"**.

after passing this condition, malware will check the password character by character.

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-1/first_char_password.PNG)

So, after debugging this function we will figure out that the right password = **'abcd'**

You can save your time and tears and patching this function, you can convert every **je** to **jmp** or you can convert **xor eax, eax** to **mov eax, 1** to make sure that **eax** always will be 1 

### After this, we can use IDA to explore the rest of the code to see the right arg.

After password verification, it will check for the arg and after debugging in IDA, malware will perform some strings comparisons using **__mbscmp** 

After argument and password checking, the malware will **argc** again with 3 to make sure there are 3 args.

After that, we will see the ServiceName variable and calling to **sub_4025B0** which is taking the path of malware and split its file name 

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-1/get_filename_1.PNG)
![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-1/get_filename_2.PNG)

And now **EAX** contain the file name 

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-1/filename.PNG)


Then, back to main and continue execution we will see that **ServiceName** now is the malware name **Lab09-01**

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-1/Before_Creating_Service.PNG)


Then, calling **sub_402600** and inside it will see **%SYSTEMROOT%\\system32\\** & **.exe**  offsets and **OpenSCManagerA** calling, and we can see that a connection to the service control manager

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-1/inside_create_1.PNG)

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-1/inside_create_2.PNG)

After that, it will check if the service with the same name is already exists or not by using **OpenServiceA**

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-1/try_2_openService.PNG)

If the service already exists, malware will change its configurations to these configurations:

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-1/change_malware_config.PNG)

As we see in parameter **dwStartType**, it will make malware run automatically (**persistence**)

But in our case service doesn't exists, so malware will create it with these configurations:

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-1/create_new_service.PNG)

As we see it will also created as a startup service, and if we view services we will find service created succesfully and copy the orginal file to this service

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-1/service_created.PNG)

After malware copy itself, there are calling to function **sub_4015B0** inside this function we will see some staff of creating file and get file time 


![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-1/inside_sub_4015B0_1.PNG)

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-1/inside_sub_4015B0_2.PNG)

I don't know what is actually happened, but after returning from this function we will see **ECX** will be **wow64cpu.dll**

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-1/wow64cpu.dll.png)

But, after this we will see URL **http://www.practicalmalwareanalysis.com** and two numbers (60, 80) and **ups** as arguments to **sub_401070**

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-1/URL.PNG)

Inside this function, there are many **rep movsd** and **rep movsb** instructions, and after all of these instrunctions, the malware will Create Registry key using **RegCreateKeyExA** in **SOFTWARE\\Microsoft \\XPS**

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-1/Lab-1/Registry_key_creating.PNG)

and then sets the data and type of a specified value under a registry key using **RegSetValueExA**

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-1/Registry_key_set.PNG)

So, now malware install itself and create a registyr and set its configration 

---

### Q1- How can you get this malware to install itself?
The malware will install itself as a service in the **SYSTEMROOT** directory.

---

### Q2- What are the command-line options for this program? What is the password requirement?

Beside -in option we have other three **(-re,-c,-cc)**

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-1/args_re_c.PNG) <br>

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-1/arg_cc.PNG) <br>

- **-in**: Install the malware
- **-re**: remove the malware (this option will delete malware service and its file in **SYSTEMROOT** directory)
- **-c**: this option will also remove malware if argc != 7 and if it was = 7 it will create a registry key and set its values so I think this option to modify C2 url and the other two parameters (in our original case 60 & 80)
- **-this option will retrieve the configuration in our registry and output it

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-1/malware_view_config.PNG)

And password is **abcd**

---

### Q3- How can you use OllyDbg to permanently patch this malware, so that it doesn’t require the special command-line password?

As i mentioned above, we can modify every **je** to be **jmp** or we can convert **xor eax, eax** with to **mov eax, 1**, so we can enter any password and it will be accepted.

---

### Q4- What are the host-based indicators of this malware?

SOFTWARE\\Microsoft \\XPS Registry 

%SYSTEMROOT%\\system32\\ Directory

---

### Q5- What are the different actions this malware can be instructed to take via the network?

If we view strings in IDA, we will see some action belongs to connecting to network

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-1/network_actions.PNG)

---

### Q6- Are there any useful network-based signatures for this malware?

URL: http://www.practicalmalwareanalysis.com

---
---
### Lab1 finish

---
---

# 2- Lab09-02.exe:

---

### Q1- What strings do you see statically in the binary?

Some APIs, URLs and exe files.

---

### Q2- What happens when you run this binary?

This binary doesn't show any visible actions when running it.

---

### Q3-How can you get this sample to run its malicious payload?

If we start to debug this binary we will find that this malware compare its name with **"ocl.exe"**

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-2/malware_name_compare.PNG)

And if not equal it will shut down without doing anything.

So, we can patch our sample our rename it to be able to run malware with normal behavior.

---

### Q4-What is happening at 0x00401133?

In this location we will see construction of string **Str**:
- Str = "1qaz2wsx3edc"

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-2/loc_0x00401133.PNG)

---

### Q5-What arguments are being passed to subroutine 0x00401089?

Arguments passed to this function are Str in EDX and memory address in ECX

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-2/sub_0x00401089_args.PNG)

---

### Q6- What domain name does this malware use?

If we start to debugging sub_0x00401089, we will that some kind of decoding routine to decode Str using **XOR** operation, After the function has finished its work if we look at memory dump we will see the result is **www.practicalmalwareanalysis.com**

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-2/url_dump.PNG)

---

### Q7- What encoding routine is being used to obfuscate the domain name?

XOR encoding.

---

### Q8- What is the significance of the CreateProcessA call at 0x0040106E?

After decoding URL we will see some networking function calling like **gethostbyname** which retrieves host information corresponding to a host name from a host database and then calls to **connect**** which establishes a connection to a specified socket and if we see a network traffic in Wireshark we will see that there are DNS request to our decoded URL 


![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-2/dns_req.PNG)

So, the malware is now connected to its C2 server and after that inside **sub_401000** we will see calling to **CreateProcessA** with parameters like **cmd** which indicate that malware will receive a command from the author to execute in the machine, so this malware acts as a reverse shell in the victim machine connects to C2 URL and then establishes a cmd process to receive commands to execute 

---
---
### Lab2 finish
---
---

# 3- Lab09-03.exe:
---

### Q1- 1. What DLLs are imported by Lab09-03.exe?

- DLL1.dll
- DLL2.dll
- KERNEL32.dll
- NETAPI32.dll
- DLL3.dll (dynamically loaded)

---

### Q2- What is the base address requested by DLL1.dll, DLL2.dll, and DLL3.dll?

If we load DLLs into **PE bear** and view the image base address for each of them

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-3/DLL1-image-base.PNG)

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-3/DLL2-image-base.PNG)

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-3/DLL3-image-base.PNG)

So all DLLs have the same image base address (**0x10000000**)

---

### Q3- When you use OllyDbg to debug Lab09-03.exe, what is the assigned based address for: DLL1.dll, DLL2.dll, and DLL3.dll?

So, we first need to run the malware until all DLLs are loaded in memory, and then we can go to the memory map to see where they are loaded

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-3/memory_addrs.PNG)

- DLL1 -> 0x10000000
- DLL2 -> 0x4F0000
- DLL3 -> 0x500000

---

### Q4- When Lab09-03.exe calls an import function from DLL1.dll, what does this import function do?

The malware calls DLL1Print from DLL1 if we disassembly DLL1 and view DLL1Print we will see "DLL 1 mystery data %d\n" which is printed at the start of the program and the integer, this integer comes from "**dword_10008030**"

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-3/DLL1Print.PNG)

If we examine the XREF of this location we will find the loading of its content in DllMain from "GetCurrentProcessId" 

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-3/dword_10008030.PNG)

So this function will print the process id.

---

### Q5- When Lab09-03.exe calls WriteFile, what is the filename it writes to?

The handle to this file returned from DLL2ReturnJ

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-3/handle_to_file.PNG)

If we look at this function in IDA, it just loads "**dword_1000B078****"

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-3/DLL2ReturnJ.PNG)

Jumping to XREF for this location, we will find that this location intailized in DllMain to be a handle to a file called "**temp.txt**"

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-3/file_name.PNG)

---

### Q6-When Lab09-03.exe creates a job using NetScheduleJobAdd, where does it get the data for the second parameter?

The second parameter for NetScheduleJobAdd is the buffer, which is a pointer to an AT_INFO structure describing the job to submit.

And if we look a bit above we will see dynamic calling to **DLL3GetStructure** from DLL3 and the buffer is the input of this function 

If we examine this function in disassembly we will see that it only moves **dword_1000B0A0** and returns it 

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-3/DLL3GetStructure.PNG)

And if we look at the XREF of this variable, we will see that it initialized in the main of DLL3, inside main we will see calling to **MultiByteToWideChar** which maps a character string to a UTF-16 (wide character) string, and it take "**ping www.malwareanalysisbook.com**" as a string to convert 

So the second input to NetScheduleJobAdd is a pointer to the structure which contains information about the scheduled task which is "ping www.malwareanalysisbook.com"

When I was about NetScheduleJobAdd in Microsoft documentation I found this function was not longer used since windows 8, So I can't view details about this structure I searched about it and found that the scheduled task for this malware will be: **ping www.malwareanalysisbook.com **every day** at 1 am**

---

### Q7-While running or debugging the program, you will see that it prints out three pieces of mystery data. What are the following: DLL 1 mystery data 1, DLL 2 mystery data 2, and DLL 3 mystery data 3?

- DLL 1 mystery data 1 -> current process id for the malware
- DLL 2 mystery data 2 -> handle to a created text file (temp.txt)
- DLL 3 mystery data 3 -> (WideCharStr) Pointer to a buffer that receives the converted string by **MultiByteToWideChar**

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-3/DLL3_mystery_data.PNG)

---

### Q8- How can you load DLL2.dll into IDA Pro so that it matches the load address used by OllyDbg?

we can take the address where DLL is loaded and when we start to load DLL in IDA we will select manual load and specify the new image base

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-3/new_image_base.PNG)

![](/assets/images/tutorials-summaries/Chapter9-PMA/Lab-3/loaded_new_dll.PNG)

---
---
### Lab3 finish
---
---