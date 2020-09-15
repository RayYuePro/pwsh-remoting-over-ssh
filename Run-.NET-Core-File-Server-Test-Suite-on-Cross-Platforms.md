
# Contents

[Test Environment Architecture](#Test-Environment-Architecture)

[Test Environment Configuration](#Test-Environment-Configuration)

[Config Domain Controller](#Config-Domain-Controller)

[Config System Under Test](#Config-System-Under-Test)

[Setup Driver Computer to Run Test Suite](#Setup-Driver-Computer-to-Run-Test-Suite)

1. [Setup a Windows Machine to run test suite with PTM](#Setup-a-Windows-Machine-to-run-test-suite-with-ptm)

1. [Setup a Linux Machine to run test suite](#Setup-a-Linux-Machine-to-run-test-suite)

    1. [Run test suite with released binaries](#1--run-test-suite-with-released-binaries) 
    1. [Build and run test suite from scratch](#2--Build-and-run-test-suite-from-scratch)
    1. [Run test suite in Docker image](#3--Run-test-suite-in-Docker-image-on-Linux-computer)

1. [Setup a macOS or Windows machine with Docker image to run test suite](#Setup-a-macOS-or-Windows-machine-with-Docker-image-to-run-test-suite)

# Test Environment Architecture
The Test Environment consists of a Domain Controller, Driver computer (client) and SUT computers (server) in a Domain environment. 

# Test Environment Configuration
## Config Domain Controller
In the test environment, one domain controller is setup, with the configurations deployed on the domain controller. 

* Domain name: eg. sina2020.org
* Domain admin credential: eg. sina2020\administrator,  Password01!!
* Disable Firewall on domain controller
## Config System Under Test
Take a Windows SUT (computer name eg. Meetup-SRV01) as example for the configuration description.

1. Join Meetup-srv01 into the Domain SNIA2020.org 
1. Disable Firewall on the Meetup-SRV01
1. Deploy the test scenario related environment configuration including:

* **SMB2 scenario configuration**

    *  Create SMB2 share “SMBBasic” for MS-SMB2 basic and MS-FSRVP test
    *  Create an encrypted share “SMBEncrypted” for encryption feature
    *  Create a symbolic link ”Symboliclink” under basic share (e.g. SMBBasic)  which links to basic share “SMBBasic”
    *  Create a sub folder “Sub” under basic share “SMBBasic“
    *  Create a symbolic link “Symboliclink2” under sub folder ”SMBBasic\Sub“ which also links to the basic share (e.g. SMBBasic)
    *  Create a share “ShareForceLevel2” and set SHI1005_FLAGS_FORCE_LEVELII_OPLOCK in SHI1005_flags
    *  Create a share “SameWithSMBBasic” pointing to the same local path of basic share (e.g. SMBBasic)
    *  Create a share “DifferentFromSMBBasic” pointing to a different local path (e.g. C:\DifferentFromSMBBasic). 
    *  Create a share SMBReFSShare on ReFS volume 
    *  Create 3  shadow copies on the volume which contains the share SMBBasic.

* **Auth scenario configuration**
    *  Share permission:
          *  A share named ”AzShare” was created with permission: 
              *   NTFS Permission:         Allow Everyone
              *   Share Permission: Allow Domain Admins
    *   Folder Permission:
          *  A share named “AzFolder” was created with permission:
              *   NTFS Permission:         Allow Domain Admins
              *   Share Permission: Allow Everyone
    *   File Permission:
          *  A share named “AzFile” was created with permission:
              *   NTFS Permission:         Allow Domain Admins
              *   Share Permission: Allow Everyone
    *   Claim-Based Access Control (CBAC):
          *  A share named “AzCBAC” was created with permission:
              *   NTFS Permission:      Allow Everyone
              *   Share Permission: Allow Everyone

  For all the shares created above, permissions were granted as following:
     *  Grant Full Control Permissions to admin account
     *  Grant Permissions without DELETE and GENERIC_ALL to nonadmin account

* **MS-DFSC scenario configuration**
    *   An SMB2 share for DFSC test “FileShare” was created
    *   DFS namespaces, two Stand-alone namespaces: SMBDfs and Standalone were created: Root share for SMBDfs: \\\\Meetup-SRV01\SMBDfs ,  Root share for Standalone: \\\\Meetup-SRV01\Standalone
    *   Domain-based namespace DomainBased
    *   One folder “SMBDfsLink” to 1st namespace (e.g. SMBDfs) and set link target to SMB2 share \\\\Meetup-SRV01\SMBBasic
    *   Add two folders to 2nd namespace (e.g. Standalone)
        *   One is DFSLink, link target is \\\\Meetup-SRV01\FileShare
        *   The other is Interlink, link target is \\\\Meetup-SRV01\SMBDfs\SMBDfsLink
    *   Add two folders to Domain-based namespace (e.g. DomainBased)
        *   One is DFSLink, link target is \\\\Meetup-SRV01\FileShare
        *   The other is Interlink, link target is \\\\Meetup-SRV01\SMBDfs\SMBDfsLink

* **MS-FSA scenario configuration**
    *   An SMB2 share FileShare was created.
    *   A file in the SMB2 share “FileShare” was created with file name  “ExistingFile.txt”
    *   A folder ”ExistingFoler” was created  in the SMB2 share “FileShare”
    *   A mountpoint “mountpoint” was created in the share “FileShare” mounting to the volume
    *   A symbolic link file “link.txt” was created in the share “FileShare” linking to the file “ExistingFile.txt“

## Setup Driver Computer to Run Test Suite
Before config the driver computer and run test suite, please config and check the driver computer as below:

* First, check the minimum requirement of a Driver computer:
https://github.com/microsoft/WindowsProtocolTestSuites/blob/main/TestSuites/FileServer/docs/FileServerUserGuide.md#3.3.1

* Then, verify Network Computer Connectivity:
https://github.com/microsoft/WindowsProtocolTestSuites/blob/main/TestSuites/FileServer/docs/FileServerUserGuide.md#4.3

* Next, join the driver computer into a domain environment if you want to use a domain credential for testing:
https://github.com/microsoft/WindowsProtocolTestSuites/blob/main/TestSuites/FileServer/docs/FileServerUserGuide.md#-522-set-up-the-driver-computer-for-the-domain-environment

### Setup a Windows Machine to run test suite with PTM

On a Wyou can rely on the Protocol Test Manager to run test suite. More details about how to run test suite with PTM, please check the File Server SMB2 Test Suite Lab Tutorial: https://github.com/microsoft/WindowsProtocolTestSuites/blob/main/Doc/File%20Server%20SMB2%20Test%20Suite%20Lab%20Tutorial_v2.pdf


### Setup a Linux Machine to run test suite

There are 3 ways to setup a Linux machine to run test suite:

1. [Run test suite with released binaries](#1--run-test-suite-with-released-binaries) 
1. [Build and run test suite from scratch](#2--Build-and-run-test-suite-from-scratch)
1. [Run test suite in Docker image](#3--Run-test-suite-in-Docker-image-on-Linux-computer)

Before run test suite, DNS resolution and .NET Core SDK are required to be configured properly on the Linux machine. 

**Edit hosts file for DNS resolution**

    Edit /etc/hosts by command: 

    sudo nano /etc/hosts

    Append the domain controller IP address and domain name to the end of the file:

    192.168.142.251        snia2020.org

    Append the SUT IP address and FQDN name to the end of the file. For example, Meetup-Srv01 owns the IP of 192.168.142.12, the new line to be added will be:

    192.168.142.12       meetup-srv01.snia2020.org
	
    You can adjust the IP address and host name with the SUT assigned to you. Exit and save the hosts file.

**Install .NET Core SDK**

    wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb

    sudo dpkg -i packages-microsoft-prod.deb
 
    sudo apt-get update; sudo apt-get install -y apt-transport-https && sudo apt-get update && sudo apt-get install -y dotnet-sdk-3.1 

#### 1- Run test suite with released binaries
 
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

      If test drive computer only has one IP address, leave it blank`

    </Description>

  </Property>


4.	Run Testcase under /home/iolab/FileServer with the commands below for different binaries.

   `dotnet vstest MS-SMB2_ServerTestSuite.dll --logger:"trx;LogFileName=SMB2TestResult.trx"`

   `dotnet vstest MS-SMB2Model_ServerTestSuite.dll --logger:"trx;LogFileName=SMB2ModelTestResult.trx"`

   `dotnet vstest MS-FSA_ServerTestSuite.dll --logger:"trx;LogFileName=FSATestResult.trx"`

   `dotnet vstest MS-FSAModel_ServerTestSuite.dll --logger:"trx;LogFileName=FSAModelTestResult.trx"`

   `dotnet vstest Auth_ServerTestSuite.dll --logger:"trx;LogFileName=AuthServerTestResult.trx"`


#### 2- Build and run test suite from scratch
You can also clone the source code and build by yourself. Here are the steps to follow:
1.	Clone source code from GitHub website https://github.com/microsoft/WindowsProtocolTestSuites to local driver, eg. /home/iolab/

2.	Update the ptfconfig files

Located at the local config path at /home/iolab/FileServer/CommonTestSuite.deployment.ptfconfig. Update the ClientNic1IPAddress with the Linux Host IP (192.168.142.114 for example) . 
Note: 
If you need to run Multiple Channel cases, add one more Nic to the Linux Host and update the value of one more node “ClientNic2IPAddress”  with the second Nic IP (192.168.142.115 for example).

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

 `cd /home/iolab/WindowsProtocolTestSuites/TestSuites/FileServer/src`

 `build.ps1`

 or 

 `cd /home/iolab/WindowsProtocolTestSuites/TestSuites/FileServer/src`

 `build.sh`

During the build, PTF will be downloaded from NuGet website side-by-side. 

After the build succeeds, the common folder structure should be generated in the folder `/home/iolab/WindowsProtocolTestSuites /drop/TestSuites/FileServer`.

*     Bin: all the built binaries including ProtoSDK, adapters and test suites.
*     Batch: batch files (.ps1, .sh) which can be used to launch tests.
*     Scripts: scripts which can be used to configure test environment.
*     Utils: some utilities which can be used in tests.

4.	Run test suite with Batch

In the `Batch` folder under root path of the test suite, there are several scripts you can use to launch tests.

* 	Run all test cases
   
   Execute `RunAllTestCases.ps1` in PowerShell, or `RunAllTestCases.sh` in shell directly.

* 	Run test cases by filters

  Execute `RunTestCasesByFilter.ps1 -Filter [your filter expression]` in PowerShell, or `RunTestCasesByFilter.sh [your filter expression]` in shell directly.

 For example, you can run below command if you want to run test cases with test category `BVT` and `SMB311`:

` RunTestCasesByFilter.sh "TestCategory=BVT&TestCategory=SMB311"`

For more information about how to construct the filter expression, you can refer to Filter option details.

*	Dry run

If you want to list the test cases before running them actually, you could add `-DryRun` switch to `.ps1` scripts or pass a non-empty string as the last argument to `.sh` scripts.

For example, you can run below command if you want to list test cases with test category `BVT` and `SMB311`:

`RunTestCasesByFilter.sh "TestCategory=BVT&TestCategory=SMB311" "list"`

#### 3- Run test suite in Docker image on Linux computer
Refer https://github.com/microsoft/WindowsProtocolTestSuites/wiki/How-to-Run-Test-Suites-with-Docker for more details
1. Install Docker

    `sudo snap install docker`

1. Pull Docker image from Docker Hub

    `docker pull mcr.microsoft.com/windowsprotocoltestsuites:fileserver`

1. Download ptfconfig file from GitHub
    Download the ptfconfig file from the GitHub link: 
    https://github.com/microsoft/WindowsProtocolTestSuites/releases/download/4.20.9.0/fileserver-docker-ptfconfig.tar  
    Unzip it to the local path of Linux host machine (/data/fileserver for example) before ahead. 

1. Update the ptfconfig files

    Located at CommonTestSuite.deployment.ptfconfig. Update the ClientNic1IPAddress with the Linux Host IP (192.168.142.114 for example) . If 
    you need to run Multiple Channel cases, add one more Nic to the Linux Host and update the value of one more node “ClientNic2IPAddress”  
    with the second Nic IP (192.168.142.115 for example).

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

1. Mount the folder (/data/fileserver) with ptfconfig files included and pre-configured:
1. Run the Windows protocol test suites image for FileServer with parameters:

    `docker run \`

      `--hostname <hostname> \`

      `--network host \`

      `-v /path/of/ptfconfig:/data/fileserver \`

      `-i windowsprotocoltestsuites:fileserver \`

      `[optional]$filter \`

      `[optional]$dryRun`

    * 	--hostname: Required. The host name of the running container, for example: Linux-Client
    * 	--network: Required. The network the running container will use, using host as default. While using host, please make sure that the 
    connection between the host which the container is runnning and the server is valid.
    * 	-v: Required. The /path/of/ptfconfig should include all the ptfconfig files with pre-configured, and mount this path to the fixed path 
    /data/fileserver in the container
    * 	-i: Required. The image name, for example: windowsprotocoltestsuites:fileserver
    * 	$filter: Optional. The expression used to filter test cases. For example, "TestCategory=BVT&TestCategory=SMB311" will filter out test 
    cases with test category BVT and SMB311.
    * 	$dryRun: Optional. If set as "y", just list all the test cases match the filter string instead of running them. If it's null or empty, 
    the filtered test cases will be executed directly

    For example, the command below will run the test cases with category traditional FSA categories.

    `docker run `

    `--network host`

    `-v /Users/microsoft/Desktop/config/ptfconfig:/data/fileserver`

     `-i testsuiteimage.azurecr.io/windowsprotocoltestsuites:fileserver "TestCategory=Traditional&TestCategory=Fsa"`


### Setup a macOS or Windows machine with Docker image to run test suite

1.	Install docker desktop version on your macOS or Windows machine
2.	Pull image 

`docker pull testsuiteimage.azurecr.io/windowsprotocoltestsuites:fileserver`

3.	Create bridge to let host can communicate with container image 

`docker network create --subnet=192.168.144.0/24 inter1`

For Multiple Channel cases, 2 NIC are required. Since "host" and "Macvlan" network drivers are not available on Windows and macOS, some FileServer test cases involving multiple channels will fail due to the lack of multiple NICs during the container execution.

To solve this issue, the container should be connected to multiple bridges to simulate the situation the test cases need. You can follow the steps below to run FileServer Docker image if "host" network driver is not available.
Create 2 bridges by running the following commands.

`docker network create --driver=bridge --subnet=192.168.144.0/16 inter1`

`docker network create --driver=bridge --subnet=192.168.142.0/16 inter2`

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

`docker container create`

 `--name fsdocker `

`--hostname <your_hostname>`

 `--ip 172.18.0.33 `

`--network inter1 `

`-v <your_local_config_path>:/data/fileserver`

 `-i <image_repository>:<image_tag> `

`"TestCategory=BVT&TestCategory=SMB311"`

The container fsdocker will be connected to the bridge inter1 and will be assigned with an IP address of 192.168.144.33.

6.	Connect fsdocker to inter2 by running the following command.
	 
`docker network connect inter fsdocker --ip=192.168.142.43.`

The container fsdocker will be connected to the bridge inter2 and will be assigned with an IP address of 192.168.142.43.

7.	Start the container by running the following command.

`docker container start fsdocker -a`

`docker run `

`--net mynet --ip 192.168.1.10 `

`--add-host=Meetup-Srv01:192.168.142.15`

 `-v /Users/microsoft/Desktop/config/ptfconfig:/data/fileserver `

`-it testsuiteimage.azurecr.io/windowsprotocoltestsuites:fileserver `

`"TestCategory=Model&TestCategory!=Fsa" `

You will see output flows out to the host console, and the test result of the test cases filtered by the filter TestCategory=BVT&TestCategory=SMB311 will be found in your local config path when the execution is complete.

Use below command then you can execute shell on docker container VM 

`docker exec -it {image id} bash `


