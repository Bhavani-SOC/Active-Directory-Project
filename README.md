# Active-Directory-Project

Our objective of this project is to build home lab. In this project we setup Active directory environment which we will learn more about IT administration and how a domain works. And we are going to use a splunk instance in this project to ingest events from windows server.


### Skills Learned

At the end of this project, we will learn how to set up AD , promoting AD to domain controller, adding users in domain  and perform a brute force attack on one of the domain users in windows server from target machine and see what kind of events generating in splunk.

**Logical Diagram**  
![image](https://github.com/user-attachments/assets/a68a59f9-2286-44be-bc9c-a4c9f6aa0718)
  **Requirements:**  
1Ubuntu server(.Splunk)  
2.Windows Server(AD)  
3.Kali linux  
4.Windows 10(Target machine)  



**Setting up Virtual machines**  
Download virtual box from virtualbox.org  
Next download  all these iso’s Windows 10 , pre build VB of kali linux , windows server , Ubuntu server. And set up all these in virtual box.  

**Now, we going to set the Active directory in virtual box**  
1.Go to virtual box, click on tools. Click on NAT network and file required fields as below.  
name :AD Project  
ipv4:192.168.10.0/24  
check mark on dhcp  
save  
2. Now select “AD Project” is our network in all machines. By selecting network field as NAT network.  
3.Now setup a static ip in splunk server.  
For that we going to use this below configuration.  
network:  
     version:2  
     renderer:networkd  
ethernets:  
     enpos3:  
          addresses:  
             192.168.101.10/24  
          dhcp4:false  
         dhcp6:false  
         nameservers:  
              addresses:  
                  192.168.10.1  

routes:  
-	to: default  
via: 192.168.10.1  

4.Now, check the connectivity by pinging ‘google.com’  
$Ping google.com  
Check configured static ip of splunk  
$ip a  

Next  install guest add-ons for VB.  
$SUDO APT-GET INSTALL VISTUALBOX  
$sudo apt-get install virtualbox-guest-additions-iso  
After installation of guest add ons   

**Login to the splunk server and add ouruser to the “vbox SF group”**  
$pseudo adduser mydfir vboxsf  
If we got any error like “group vboxsf” does not exist. Then we need to install additional guest installations that VB offers.  
$sudo apt-get install virtualbox-guest-utils  
$sudo rebot  
 
2.Now create a directory called ‘share ‘ in splunk  
$mkdir share  
$ls  
$ls-la  

3.Now, use a below command to mount our shared folder onto our directory called share.  
$sudo mount –t vboxsf -0 uid=1000 , gid=1000 splunk_installer  
$exit  
$ls-la  
$cd share/  
~/share$ $ls-la  

4.Now we are going to install splunk in our Ubuntu server  
$share$ sudo dpkg –i (paste that splunk installer file here)  

5.Now change the directory , where splunk installer is located on to our server.  
$/share$cd /opt/splunk  
mydfir@splunk:opt/splunk$ls-la  

6.Now lets change into user splunk /opt/splunk$sudo –U splunk bash  
$cd bin   
Splunk:~/bin$ ./splunk start  

7.Now one more command to make sure that our splunk starts up everytime our virtual machine reboots.  
Now exit directory ‘splunk’ and change directory to ‘bin’  

mydfir@splunk:/opt/splunk$cd bin  
:/opt/splunk/bin$ sudo ./splunk enable boot-start –user splunk  


**Setting up target machine and Install Splunk universal forwarder, sysmon on target machine** 

1.Goto target machine- here we need to change the hostname of target machine as ‘target’. For that goto PC select properties and then rename the pc.  
After changing the hostname as ‘ target’ restart the target mmachine and check does our changes reflected or not. 

2.Now, set up the static ip address of target machine.  
Right click on network icon. And open network & internet settings->change adaptor options->right click on adaptor-.propertis-.double click on internet protocol version 4-.select properties-set static ip address here  
Ip address:192.168.10.100  
Subnetmask:255.255.255.0  
Default gateway:192.168.10.1  
Preferred dns:192.168.10.7  
 Save  
Now , check our changes are reflected or not  
$ipconfig  

3.Now download the splunk universal forwarder from splunk.com and Install splunk forwarder in target machine   
Now download and install the sysmon config from sysmon sysinternals.  

Open powershell with administrative privileges run as administrator  
C:\\windows\system32>cd c:\users\bib\downloads\sysmon  
C:\users\bob\downloads\sysmon > .\sysmon64.exe –I ..\sysmonconfig.xml  

4.Now download sysmon olaf config configuration file from github.We need to instruct our splunk forwarder on what we want to send over to our splunk sevrer. For that we must configure a file called inputs.conf(this file will instruct our splunk forwarder  to push events related to application,security,system,sysmon over to our splunk server)  
This file actually located under cdrive-.program file-. Splunk universal forwarder->etc->system->default-inputs.conf  

5.Now, we are going to create a new file under local directory. Because we don’t want to change the original file.In case, if anything messup we can replace that with the default config file.  

6.Now, restart the spunk universal fowwarder from services whenever we updated our inputs.conf file.  

7.We have to enable the RDP on our target machine.  
Pc->properties->advanced system settings  
It will prompt for login with admin account. After login with admin account a system properties tab will open->now click on remote->select remote conncetions to the computer->select users  
Select the object type:users or groups  
From the location:MYDFIR.LOCAL  
Enter the object names to select jerry smith, terry smith->ok ->apply   

**Setting up windows server(AD) and Install Splunk universal forwarder, sysmon on windows server**  

1.Here we need to change the hostname of windows server as ‘ADDCO1’. For that goto PC select properties and then rename the pc.  
After changing the hostname as ‘ADDCO1’ restart the server and check does our changes reflected or not.  

2.Now, set up the static ip address of windows server.  
Right click on network icon. And open network & internet settings->change adaptor options->right click on adaptor-.propertis-.double click on internet protocol version 4-.select properties-set static ip address here  
Ip address:192.168.10.7  
Subnetmask:255.255.255.0  
Default gateway:192.168.10.1  
Preferred dns:8.8.8.8  
Save  
Now , check our changes are reflected or not  
$ipconfig  

3.Now download the splunk universal forwarder from splunk.com and Install splunk forwarder in target machine   

4.Now download and install the sysmon config from sysmon sysinternals.  
Open powershell with administrative privileges run as administrator  
C:\\windows\system32>cd c:\users\bib\downloads\sysmon  
C:\users\bob\downloads\sysmon > .\sysmon64.exe –I ..\sysmonconfig.xml  

5.Now download sysmon olaf config configuration file from github.We need to instruct our splunk forwarder on what we want to send over to our splunk sevrer. For that we must configure a file called inputs.conf(this file will instruct our splunk forwarder  to push events related to application,security,system,sysmon over to our splunk server)  
This file actually located under cdrive-.program file-. Splunk universal forwarder->etc->system->default-inputs.conf  

6.Now, we are going to create a new file under local directory. Because we don’t want to change the original file.In case, if anything messup we can replace that with the default config file.  
Now, restart the spunk universal fowwarder from services whenever we updated our inputs.conf file.  

**Enable our splunk server to receive data**
1.Splunk console->settings->forward & receive data ->uder the recive data->click on configure receiving ->click on new receiving port-> give default port as 9997(as we given while installing splunk forwarder)  
Check whether we are getting datatypes are not .Got to app->search & reporting ->index=endpoint  

**Install and configure active directory on windows server and then promote it to a domain controller.**  
1.Install active directory domain services and the promote our server as domain controller for doing that  
Deployment configuration, select new forest(because we rae creating a new domain)  
Root domain name:mydfir.local  
Give password:  
Paths: database folder -> c:\windows\NTDS  
Log files folder: -> c:\windows\NTDS  
Sysvol folder->-> c:\windows\sysvol  
These paths are used to store our database file named ntds:dit  
Once setup is completed our server will automatically restart.  
Now, we can see login screen with username mydfir\adminstrator. \ backslash indicates successfully installed and our server promoted to domain controller.  

2.Create some users in server  
Server manager dashboard->tools->active directory users and computers(here we can create objects such as users, computers,groups)  
Expand our domain->mydfir.local(we can see couple of folders)  
Rigt click ondomain->new->select  organizanal unit)  
Name :IT->save(now new organizanal unit is created)  
In this unit, we will create new users  
Right click on IT->new->user  
First name:jenny  
Last name->smith  
Jsmith  
Password:  
Next->finish  

Create other organizational unit.  
Right click on domain->new ->select(organizational unit)  
->HR->Save  
Right click on HR->new user  
First name->terry  
Last name->smith  
Tsmith  
Password:  

**Join our windows target machine to our newly created domain called mydfir and authenticate using jennysmith account on our windows 10 machine:**  
1.Now join the target machine to domain called mydfir.  
Pc->properties->advanced settings->click on tab->computername->select change  
Computername:target-pc  
Domain;MYDFIR.LOCAL  
Click ok  
Cmd->ipconfig  
Computername:target-pc  
Domain:MYDFIR.LOCAL  
Windows security->computer name /domain changes  
Administrator  
Password  

2.Restart the machine.Now, it will show the logon screen with our newly created user called jenny smith.  
Make sure that our  sign in is pointing to our domain & here we can see it as ‘MYDFIR’  

3.We successfully configured our own active directory sever and creating two users and joining a machine to your newly created domain.  

**Setting up kali linux to perform brute force attack**  
1.Set up a statis ip address to kali linux.go to settings->ipv4 setting->method:manual  
Add:192.168.10.250  
Netmask:24  
Gateway:192.168.10.1  
Dns server:8.8.8.8  
Save  
Click on Ethernet connections->disconnect-> and again select wired connection(which we configured static ip)  
Open terminal  
$ip a  
$ping google.com  
$ping 192.168.10.10  

2.Now upgrade and update repositories.  
$sudo apt-get update && sudo apt-get upgrade –y   
Now create a new directory called “ad-project” on our desktop.  
$mkdir ad-project  

3.Now, we can use wordlist called ‘rockyou’ that comes with kali linux. This file will found under the below directory.  
$cd /usr/share/wordlists/  
$ls  

4.Now, we have to copy this rockyou file to our folder “ad-project’  
$cp rockyou.txt ~/Desktp/ad-project  
$cd ~/desktop/ad-project  
$ls-lh  

5.We will use only first 20lines of passwords in this file.  
$head –n 20 rockyou.txt  
We will put these 20 lines of passwords in a file called ‘password.txt’  
$head –n 20 rockyou.txt > passwords.txt  
$ls  
$cat passwords.txt  

6.Now we have to edit passwards.txt file to add our target password.  
$nano passwords.txtx  
We can add our password to the file and save it.  
$cat password.txt  

7.Install crowbar tool for brute force attack  
For brute force attack we are going to use crowbar tool. Now install that tool.  
$sudo apt-get install –y crowbar  


**Brute force attack**  
1.Go to kali linux  
[~/desktop/ad-project]  
$crowbar –h  
$crowbar –b rdp –u tsmith –c passwords.txt –s 192.168.10.100/32  
$crowbar –b rdp –u tsmith –c passwords.txt –s 192.168.10.100/32  
Now, it will be check all passwords which is available in passwords.txt. If password matches it will show RDP-success 192.168.10.100:3389 –tsmith:password  

2.Goto splunk to see the telemetry  
Splunk->search & reporting  
Index=endpoint  



