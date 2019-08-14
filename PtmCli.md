Protocol Test Manager has a command line interface for you to use in automation.

`PtmCli.exe` is located in `bin` folder under installation path. By default, it is `C:\Program Files\Protocol Test Manager\bin\PtmCli.exe`.

## Syntax

```
PtmCli.exe <-p|-profile profileName>
           [-s|-selected] [-r|-report reportFile]
           [-categories categories] [-outcome pass,fail,inconclusive,notrun]
           [-sortby name|outcome] [-separator comma|space]
```

## Parameters

### -p|-profile profileName

Specifies the path of the test profile to run.

To get a valid profile. You need to export one from PTM GUI. See [Export Test Profile](#export-test-profile) section.

### -s|-selected

When specified, only the selected test cases will run. Otherwise, all the test cases in the profile will run.

### -r|-report reportFile

Specifies the result file which will be written to. If not specified, test results will be written to stdout.

### -categories categories

Specifies the categories of test cases to run. This parameter overrides the test cases in profile.

Value should be separated by comma without space.

### -outcome outcome

Specifies the outcome of the test cases to be included in the report file.

Value should be separated by comma without space.

Valid values are: `pass`, `fail`, `inconclusive`, `notrun`.

Default value is `pass,fail`.

### -sortby

Specifies the way to sort the test cases in the report.

Valid values are: `name`, `outcome`.

Default value is `name`.

### -separator

Specifies the separator used in the report file.

Valid values are: `space`, `comma`.

Default value is `space`.

## Export Test Profile

As the below image shown, when using PTM to run test cases, in the last screen, click **Export / Import** at the right bottom of the test case list, then click **Save Profile ...**. Then you can get a valid PTM profile to use with PtmCli.

![Save a profile in PTM](./images/save-profile.png)

## Modify Test Profile

If you want to run test suite using a different configuration, you can modify the test profile.

The test profile (.ptm file) itself is actually a zip file. Unzip the `.ptm` file, you will get two folders: `config` and `ptfconfig`.

In `config` folder, you can edit `playlist.xml` to update playlist and edit `profile.xml` to update the selected rules.

And all `ptfconfig` file used by test suites are located in `ptfconfig` folder, you can change them accordingly.

When you are ready, zip the folder and modify the extension to `.ptm`. You can use the new test profile to run test suites using the new configuration.