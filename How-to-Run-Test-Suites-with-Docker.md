# Prerequisites

You need to install docker on your platform, go to the [Get Docker][] for help.

[Get Docker]: https://docs.docker.com/get-docker/


# Featured Tags

- `fileserver`: The fileserver test suite image.
  - Ubuntu 20.04 for Linux
  - `docker pull mcr.microsoft.com/windowsprotocoltestsuites:fileserver`

# About This Image

Windows Protocol Test Suites provide interoperability testing against the implementation of Windows open specifications including File Services.
If you are new to Windows Protocol Test Suites and want to learn more, go to the [windows protocol test suites][] to get started.

[windows protocol test suites]: https://github.com/microsoft/WindowsProtocolTestSuites

# How to Run this Image

To run this image, you need to mount a folder with [ptfconfig][] files included and pre-configured:

[ptfconfig]: https://github.com/microsoft/WindowsProtocolTestSuites/releases/download/4.20.9.0/fileserver-docker-ptfconfig.tar

- `--hostname`: Required. The host name of the running container, for example: FS-CLI
- `--network`: Required. The network the running container will use, using host as default. While using host, please make sure that the connection between the host which the container is runnning and the server is valid.
- `-v`: Required. The /path/of/ptfconfig should include all the ptfconfig files with pre-configured, and mount this path to the fixed path /data/fileserver in the container
- `-i`: Required. The image name, for example: windowsprotocoltestsuites:fileserver
- `$filter`: Optional. The expression used to filter test cases. For example, "TestCategory=BVT&TestCategory=SMB311" will filter out test cases with test category BVT and SMB311.
- `$dryRun`: Optional. If set as "y", just list all the test cases match the filter string instead of running them. If it's null or empty, the filtered test cases will be executed directly

To run the default windows protocol test suites image for FileServer:

```
docker run \
  --hostname <hostname> \
  --network host \
  -v /path/of/ptfconfig:/data/fileserver \
  -i windowsprotocoltestsuites:fileserver \
  [optional]$filter \
  [optional]$dryRun
```

When the test suites run finished, the result file with trx format will be generated under the location /path/of/ptfconfig

# Feedback

If you have any issues or concerns, reach out to us through a [GitHub issue](https://github.com/microsoft/WindowsProtocolTestSuites/issues/new).

# License

- Legal Notice: [Container License Information](https://aka.ms/mcr/osslegalnotice)
- [.NET Core license](https://github.com/dotnet/dotnet-docker/blob/master/LICENSE)
- Please check the [github repository](https://github.com/microsoft/WindowsProtocolTestSuites) for project license details
