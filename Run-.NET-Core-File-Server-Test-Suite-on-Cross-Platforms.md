 
Contents
Meetup Test Environment Architecture	2
Run Test Suite on Pre-configured Linux Driver	5
Setup Your Own Driver Computer to Run Test Suite	6
Setup a Linux Machine to run test suite	6
Run test suite with released binaries	7
Build and run test suite from the scratch	8
Run test suite in Docker image on Linux computer	10
Setup a macOS /Windows machine with Docker image to run test suite	11

 ![](‪C:\Users\vtian\Desktop\meetupenv.PNG)

# Test Environment Architecture
The Test Environment consists of 8 Driver computers (client) and 8 SUT computers (server) hosted as Azure virtual machines in a Domain environment. Users will access the Domain via the remote desktop protocol (RDP) with a predefined administrator name and password. The basic network configuration is shown as follows:

# Test Environment Configuration
8 sets of Linux Driver + Windows SUT in domain environment with proper deployments were configured already. To get one set of environment, please check the session Reserve Meetup Test Environment. Once you reserve successfully, you are ready dial in the Meetup Environment via P2S VPN, and then follow the run test case on the Linux Driver computer directly. 

In the test environment, one domain controller is setup, with the configurations deployed on the domain controller. 

* Domain name: eg. sina2020.org
* Domain admin credential: eg. sina2020\administrator,  Password01!!
 
The Windows SUTs have been configured with the below deployments. Take the SUT Meetup-SRV01 as example for the following description. 
1.Join Meetup-srv01 into the Domain SNIA2020.org 
1.Disable Firewall
1.Config the test scenario related environment including:

•	SMB2 scenario configuration
1.	SMB2 share “SMBBasic” was created for MS-SMB2 basic and MS-FSRVP test
2.	Encrypted share “SMBEncrypted” created for encryption feature
3.	A symbolic link”Symboliclink” under basic share (e.g. SMBBasic)  which links to basic share “SMBBasic” was created.
4.	A sub folder “Sub” under basic share “SMBBasic“ was created.
5.	A symbolic link “Symboliclink2” under sub folder ”SMBBasic\Sub“ which also links to the basic share (e.g. SMBBasic) was created.
6.	A share “ShareForceLevel2” was created and was set SHI1005_FLAGS_FORCE_LEVELII_OPLOCK in SHI1005_flags.
7.	A share  “SameWithSMBBasic” was created pointing to the same local path of basic share (e.g. SMBBasic)
8.	A share “DifferentFromSMBBasic” was created pointing to a different local path (e.g. C:\DifferentFromSMBBasic). 
9.	A share SMBReFSShare on ReFS volume was created.
10.	3  shadow copies on the volume were created which contains the share SMBBasic.
•	Auth scenario configuration
1.	Share permission:
A share named ”AzShare” was created with permission: 
•	NTFS Permission:         Allow Everyone
•	Share Permission: Allow Domain Admins
2.	Folder Permission: 
	A share named “AzFolder” was created with permission:
•	NTFS Permission:         Allow Domain Admins
•	Share Permission: Allow Everyone
3.	File Permission:
A share named “AzFile” was created with permission:
•	NTFS Permission:         Allow Domain Admins
•	Share Permission: Allow Everyone
4.	Claim-Based Access Control (CBAC):
A share named “AzCBAC” was created with permission:
•	NTFS Permission:      Allow Everyone
•	Share Permission: Allow Everyone
For all the shares created above, permissions were granted as following:
•	Grant Full Control Permissions to admin account
•	Grant Permissions without DELETE and GENERIC_ALL to nonadmin account

•	MS-DFSC scenario configuration
1.	An SMB2 share for DFSC test “FileShare” was created
2.	DFS namespaces, two Stand-alone namespaces: SMBDfs and Standalone were created: Root share for SMBDfs: \\Meetup-SRV01\SMBDfs ,  Root share for Standalone: \\Meetup-SRV01\Standalone
3.	Domain-based namespace DomainBased
4.	One folder “SMBDfsLink” to 1st namespace (e.g. SMBDfs) and set link target to SMB2 share \\Meetup-SRV01\SMBBasic
5.	Add two folders to 2nd namespace (e.g. Standalone)
i.	One is DFSLink, link target is \\Meetup-SRV01\FileShare
ii.	The other is Interlink, link target is \\Meetup-SRV01\SMBDfs\SMBDfsLink
6.	Add two folders to Domain-based namespace (e.g. DomainBased)
i.	One is DFSLink, link target is \\Meetup-SRV01\FileShare
7.	The other is Interlink, link target is \\Meetup-SRV01\SMBDfs\SMBDfsLink

•	MS-FSA scenario configuration
1.	An SMB2 share FileShare was created.
2.	A file in the SMB2 share “FileShare” was created with file name  “ExistingFile.txt”
3.	A folder ”ExistingFoler” was created  in the SMB2 share “FileShare”
4.	A mountpoint “mountpoint” was created in the share “FileShare” mounting to the volume
5.	A symbolic link file “link.txt” was created in the share “FileShare” linking to the file “ExistingFile.txt“

With all the configurations, the final Meetup-Srv01 share folders like below:

 

If you want to setup your own driver computer and run test suite in Linux or macOS/Windows with Docker image, please check the section Setup Your Own Driver Computer to Run Test Suite for more details.
Either, you can try to Run Test Suite on Pre-configured Linux Driver in the Meetup Environment, which is only for the IOLab event meetup purpose. After the IOlab, the environment will be not available any more. 
Run Test Suite on Pre-configured Linux Driver 
There 2 ways to run test cases on the pre-configured Linux Driver: dotnet command or PTMCli.
1.	DOTNET test command
The test suite released binaries have already been downloaded to the Linux Driver, under /home/iolab/FileServer.
In the /home/iolab/FileServer folder, run test cases with commands below for different binaries.

dotnet vstest MS-SMB2_ServerTestSuite.dll --logger:"trx;LogFileName=SMB2TestResult.trx"

dotnet vstest MS-SMB2Model_ServerTestSuite.dll --logger:"trx;LogFileName=SMB2ModelTestResult.trx"

dotnet vstest MS-FSA_ServerTestSuite.dll --logger:"trx;LogFileName=FSATestResult.trx"

dotnet vstest MS-FSAModel_ServerTestSuite.dll --logger:"trx;LogFileName=FSAModelTestResult.trx"

dotnet vstest Auth_ServerTestSuite.dll --logger:"trx;LogFileName=AuthServerTestResult.trx"

2.	PTMCli:
PTM cli now is a cross platform command line, which can run on Windows, Linux and macOS platforms.
For more information about PTM Cli command parameters, please check the GitHub Wiki. 

Setup Your Own Driver Computer to Run Test Suite
Setup a Linux Machine to run test suite
There are 3 ways to setup a Linux machine to run test suite:
1.	Run test suite with released binaries. 
2.	Build and run test suite from the scratch.
3.	Run test suite in Docker image.

Before run test suite, DNS resolution and .NET Core SDK is required to be configured properly on the Linux machine. 
1.	Edit /etc/hosts by command: 
sudo nano /etc/hosts

Append the domain controller IP address and domain name to the end of the file:
192.168.142.251        snia2020.org

Append the SUT IP address and FQDN name to the end of the file. For example, Meetup-Srv01 owns the IP of 192.168.142.12, the new line to be added will be:
192.168.142.12       meetup-srv01.snia2020.org
	
You can adjust the IP address and host name with the SUT assigned to you. Exit and save the hosts file.

2.	Install .NET Core SDK
wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
 
sudo apt-get update; sudo apt-get install -y apt-transport-https && sudo apt-get update && sudo apt-get install -y dotnet-sdk-3.1 

Run test suite with released binaries
 
1.	Download test suite source released binaries from GitHub to /home/iolab

wget https://github.com/microsoft/WindowsProtocolTestSuites/archive/3.20.1.0.zip -L -O testsuite.zip

2.	Unzip test suite binaries
tar -C /home/iolab -zxvf /home/iolab/FileServer.tar.gz

3.	Update the ptfconfig files

Located at the local config path at /home/iolab/FileServer/CommonTestSuite.deployment.ptfconfig. Update the ClientNic1IPAddress with the Linux Host IP (192.168.142.114 for example) . If you need to run Multiple Channel cases, add one more Nic to the Linux Host and update the value of one more node “ClientNic2IPAddress”  with the second Nic IP (192.168.142.115 for example).

 <Property name="ClientNic1IPAddress" value="192.168.142.114">
    <Description>
      One IP address or host name on local test drive computer to establish connections to SUT
    </Description>
  </Property>
  <Property name="ClientNic2IPAddress" value="192.168.142.115">
    <Description>
      Another IP address or host name on local test drive computer to establish connections to SUT
      If test drive computer only has one IP address, leave it blank
    </Description>
  </Property>

4.	Run Testcase under /home/iolab/FileServer with the commands below for different binaries.

dotnet vstest MS-SMB2_ServerTestSuite.dll --logger:"trx;LogFileName=SMB2TestResult.trx"

dotnet vstest MS-SMB2Model_ServerTestSuite.dll --logger:"trx;LogFileName=SMB2ModelTestResult.trx"

dotnet vstest MS-FSA_ServerTestSuite.dll --logger:"trx;LogFileName=FSATestResult.trx"

dotnet vstest MS-FSAModel_ServerTestSuite.dll --logger:"trx;LogFileName=FSAModelTestResult.trx"

dotnet vstest Auth_ServerTestSuite.dll --logger:"trx;LogFileName=AuthServerTestResult.trx"


Build and run test suite from the scratch

1.	Clone source code from GitHub website https://github.com/microsoft/WindowsProtocolTestSuites  to /home/iolab/

2.	Update the CommonTestSuite.deployment.ptfconfig 

Located at the local config path at /home/iolab/FileServer/CommonTestSuite.deployment.ptfconfig. Update the ClientNic1IPAddress with the Linux Host IP (192.168.142.114 for example) . If you need to run Multiple Channel cases, add one more Nic to the Linux Host and update the value of one more node “ClientNic2IPAddress”  with the second Nic IP (192.168.142.115 for example).

 <Property name="ClientNic1IPAddress" value="192.168.142.114">
    <Description>
      One IP address or host name on local test drive computer to establish connections to SUT
    </Description>
  </Property>
  <Property name="ClientNic2IPAddress" value="192.168.142.115">
    <Description>
      Another IP address or host name on local test drive computer to establish connections to SUT
      If test drive computer only has one IP address, leave it blank
    </Description>
  </Property>

3.	Build test suite with PowerShell or Shell script:

cd /home/iolab/WindowsProtocolTestSuites/TestSuites/FileServer/src

build.ps1
or build.sh

During the build, PTF will be downloaded from NuGet website side-by-side. 

After the build succeeds, the common folder structure should be generated in the folder `/home/iolab/WindowsProtocolTestSuites /drop\TestSuites\FileServer`.

o	`Bin`: all the built binaries including ProtoSDK, adapters and test suites.
o	`Batch`: batch files (.ps1, .sh) which can be used to launch tests.
o	Scripts`: scripts which can be used to configure test environment.
o	`Utils`: some utilities which can be used in tests.

4.	Run test suite with Batch

In the `Batch` folder under root path of the test suite, there are several scripts you can use to launch tests.

o	Run all test cases
   
   Execute `RunAllTestCases.ps1` in PowerShell, or `RunAllTestCases.sh` in shell directly.

o	Run test cases by filters

  Execute `RunTestCasesByFilter.ps1 -Filter [your filter expression]` in PowerShell, or `RunTestCasesByFilter.sh [your filter expression]` in shell directly.

 For example, you can run below command if you want to run test cases with test category `BVT` and `SMB311`:
 RunTestCasesByFilter.sh "TestCategory=BVT&TestCategory=SMB311"

For more information about how to construct the filter expression, you can refer to Filter option details.

o	Dry run

If you want to list the test cases before running them actually, you could add `-DryRun` switch to `.ps1` scripts or pass a non-empty string as the last argument to `.sh` scripts.

For example, you can run below command if you want to list test cases with test category `BVT` and `SMB311`:

RunTestCasesByFilter.sh "TestCategory=BVT&TestCategory=SMB311" "list"

Run test suite in Docker image on Linux computer
1.	Pull Docker image from Docker Hub
docker pull mcr.microsoft.com/windowsprotocoltestsuites

2.	Download ptfconfig file from GitHub
Download the ptfconfig file from the GitHub  link: https://github.com/microsoft/WindowsProtocolTestSuites/releases/download/4.20.9.0/fileserver-docker-ptfconfig.tar  
Unzip it to the local path of Linux host machine (/data/fileserver for example) before ahead. 

3.	Update the ptfconfig files

Located at CommonTestSuite.deployment.ptfconfig. Update the ClientNic1IPAddress with the Linux Host IP (192.168.142.114 for example) . If you need to run Multiple Channel cases, add one more Nic to the Linux Host and update the value of one more node “ClientNic2IPAddress”  with the second Nic IP (192.168.142.115 for example).

 <Property name="ClientNic1IPAddress" value="192.168.142.114">
    <Description>
      One IP address or host name on local test drive computer to establish connections to SUT
    </Description>
  </Property>
  <Property name="ClientNic2IPAddress" value="192.168.142.115">
    <Description>
      Another IP address or host name on local test drive computer to establish connections to SUT
      If test drive computer only has one IP address, leave it blank
    </Description>
  </Property>

4.	Mount the folder (/data/fileserver) with ptfconfig files included and pre-configured:
5.	Run the Windows protocol test suites image for FileServer with parameters:
docker run \
  --hostname <hostname> \
  --network host \
  -v /path/of/ptfconfig:/data/fileserver \
  -i windowsprotocoltestsuites:fileserver \
  [optional]$filter \
  [optional]$dryRun
o	--hostname: Required. The host name of the running container, for example: Linux-Client
o	--network: Required. The network the running container will use, using host as default. While using host, please make sure that the connection between the host which the container is runnning and the server is valid.
o	-v: Required. The /path/of/ptfconfig should include all the ptfconfig files with pre-configured, and mount this path to the fixed path /data/fileserver in the container
o	-i: Required. The image name, for example: windowsprotocoltestsuites:fileserver
o	$filter: Optional. The expression used to filter test cases. For example, "TestCategory=BVT&TestCategory=SMB311" will filter out test cases with test category BVT and SMB311.
o	$dryRun: Optional. If set as "y", just list all the test cases match the filter string instead of running them. If it's null or empty, the filtered test cases will be executed directly

For example, the command below will run the test cases with category traditional FSA categories.

docker run 
--network host
-v /Users/microsoft/Desktop/config/ptfconfig:/data/fileserver
 -i testsuiteimage.azurecr.io/windowsprotocoltestsuites:fileserver "TestCategory=Traditional&TestCategory=Fsa"

Setup a macOS /Windows machine with Docker image to run test suite

1.	Install docker desktop version on your macOS
2.	Pull image 
docker pull testsuiteimage.azurecr.io/windowsprotocoltestsuites:fileserver
3.	Create bridge to let host can communicate with container image 
docker network create --subnet=192.168.144.0/24 inter1

For Multiple Channel cases, 2 NIC are required. Since "host" and "Macvlan" network drivers are not available on Windows and macOS, some FileServer test cases involving multiple channels will fail due to the lack of multiple NICs during the container execution.

To solve this issue, the container should be connected to multiple bridges to simulate the situation the test cases need. You can follow the steps below to run FileServer Docker image if "host" network driver is not available.
Create 2 bridges by running the following commands.
docker network create --driver=bridge --subnet=192.168.144.0/16 inter1
docker network create --driver=bridge --subnet=192.168.142.0/16 inter2

By running the commands, you will get a bridge named inter1 with a subnet range of 192.168.144.0/16 and another bridge named inter2 with a subnet range of 192.168.142.0/16.

4.	Update the ptfconfig files

For multiple channel cases, the commontstsuite.deployment.ptfconfig need to be updated with both ClientNic1IPAddress and  ClientNic2IPAddress  IP addresses. Otherwise, update only the ClientNic1IPAddress value. 

Located at your local config path, and ensure that the modified values are legal IP addresses in inter1 or inter2 separately.

<Property name="ClientNic1IPAddress" value="192.168.144.33">
    <Description>
      One IP address or host name on local test drive computer to establish connections to SUT
    </Description>
  </Property>
  <Property name="ClientNic2IPAddress" value="192.168.142.43">
    <Description>
      Another IP address or host name on local test drive computer to establish connections to SUT
      If test drive computer only has one IP address, leave it blank
    </Description>
  </Property>

The ClientNic1IPAddress is now a legal IP address in inter1, and the ClientNic2IPAddress is another legal IP address in inter2. The modified IP addresses are useful in the following container creation for connections between bridges and the container. The container will establish multiple channels through these 2 IP addresses.

5.	Create a new container named fsdocker by running the following command.
docker container create
 --name fsdocker 
--hostname <your_hostname>
 --ip 172.18.0.33 
--network inter1 
-v <your_local_config_path>:/data/fileserver
 -i <image_repository>:<image_tag> 
"TestCategory=BVT&TestCategory=SMB311"

The container fsdocker will be connected to the bridge inter1 and will be assigned with an IP address of 192.168.144.33.

6.	Connect fsdocker to inter2 by running the following command.
	 docker network connect inter fsdocker --ip=192.168.142.43.

The container fsdocker will be connected to the bridge inter2 and will be assigned with an IP address of 192.168.142.43.

7.	Start the container by running the following command.
docker container start fsdocker -a

docker run 
--net mynet --ip 192.168.1.10 
--add-host=Meetup-Srv01:192.168.142.15
 -v /Users/microsoft/Desktop/config/ptfconfig:/data/fileserver 
-it testsuiteimage.azurecr.io/windowsprotocoltestsuites:fileserver 
"TestCategory=Model&TestCategory!=Fsa" 

You will see output flows out to the host console, and the test result of the test cases filtered by the filter TestCategory=BVT&TestCategory=SMB311 will be found in your local config path when the execution is complete.

Use below command then you can execute shell on docker container VM 
docker exec -it {image id} bash 

