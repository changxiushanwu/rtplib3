# rtplib3

**rtplib3** offers a high level Python drop module for rtplib2 that is compatible with HIL API MAPort Python and XIL API .NET MAPort. It does this by hiding the [rtplib2 to XIL API migration guide](https://www.dspace.com/support/patches/TASC/PAPI/RTPLIB2_XIL_API_PythonNET_Migration_Guide.pdf) behind a layer of abstraction.

# Why?

Executive Summary: Combination of Windows, dSpace and Matlab compatibilities are going to make a lot of current running code obsolete. Any scripts using rtplib2 or HIL API MAPort interfaces might no longer work once machines are upgraded to Windows 10.

Microsoft's main stream support for [Windows 7 ended January 13, 2015 and extended support ends January 14, 2020.](https://support.microsoft.com/en-us/help/13853/windows-lifecycle-fact-sheet).

dSpace doesn't actually support Windows 10 *yet*:

*With dSPACE Release 2016-B TargetLink and Model Compare will support Windows 10.
Full qualified support of Windows 10 for the complete dSPACE Release 2016-B will be granted by means of a service pack available mid 2017.
Beginning with dSPACE Release 2017-A dSPACE products will generally support Windows 10.
dSPACE will support only the 64-bit Professional and Enterprise editions of Windows 10.* - [Windows 10 Support of dSPACE Products](https://www.dspace.com/en/inc/home/support/supvers/supverscompm/release_roadmap/win10.cfm)

dSpace has exposed multiple programming interfaces for automation. [rtplib2 (Python) is discontinued and will be distributed for the last time with dSPACE Release 2016-A.](https://www.dspace.com/en/inc/home/support/kb/supkbspecial/kbta/rtplib2xilapinetmaport.cfm). It was first replaced by the HIL API MAPort then by the XIL API .NET. 

There is a [rtplib2 to XIL API migration guide](https://www.dspace.com/support/patches/TASC/PAPI/RTPLIB2_XIL_API_PythonNET_Migration_Guide.pdf) however it is non-trivial and there is a large existing code base of rtplib2 Python scripts running dSpace anywhere dSpace is sold.

HIL API MAPort was a short lived program interface. There is also a [HIL API MAPort Python to XIL API MAPort .NET Migration Guide](https://www.dspace.com/en/pub/home/support/kb/supkbspecial/kbta/hilapimaportpythonxilapimig.cfm).

See Also: [Paying Down Your Technical Debt](https://blog.codinghorror.com/paying-down-your-technical-debt/)

# Examples:

## Creating and Configuring MAPorts

rtplib2:

    import rtplib2
   
    platformIdentifier = "ds1005"
    applicationPath = r"C:\Path\To\MyApplication.sdf"
    appl = rtplib2.Appl(applicationPath, platformIdentifier) 
    
**rtplib3**:

    import rtplib3 as rtplib2
   
    platformIdentifier = "ds1005"
    applicationPath = r"C:\Path\To\MyApplication.sdf"
    appl = rtplib2.Appl(applicationPath, platformIdentifier) 
    
XIL API .NET:

    # import Python for .NET
    import clr
    # make some collections and data types of the .NET framework available in Python
    from System import Array
    from System.Collections.Generic import Dictionary
    # load ASAM assemblies from the global assembly cache (GAC)
    clr.AddReference("ASAM.XIL.Implementation.TestbenchFactory, Version=2.0.1.0,
    Culture=neutral, PublicKeyToken=fc9d65855b27d387")
    clr.AddReference("ASAM.XIL.Interfaces, Version=2.0.1.0, Culture=neutral,
    PublicKeyToken=bf471dff114ae984")
    # import XIL API .NET classes from the .NET assemblies
    from ASAM.XIL.Implementation.TestbenchFactory.Testbench import TestbenchFactory
    from ASAM.XIL.Interfaces.Testbench.Common.Error import TestbenchPortException 
    
    testbenchFactory = TestbenchFactory()
    testbench = testbenchFactory.CreateVendorSpecificTestbench("dSPACE GmbH", "XIL API", "2015-B");
    
    maPortConfigFilePath = r"..\MAPortConfiguration.xml"
    maPort = testbench.MAPortFactory.CreateMAPort("DemoMAPort")
    maPortConfig = maPort.LoadConfiguration(maPortConfigFilePath)
    maPort.Configure(maPortConfig, False) 
    
## Reading and Writing Values 
    
rptlib2:

    turnSignalLever = "Model Root/TurnSignalLever[-1..1]/Value"
    turnSignalLeverVar = appl.Variable(turnSignalLever)
    readValue = turnSignalLeverVar.Read()
    print readValue
    turnSignalLeverVar.Write(1.0)
    
**rptlib3**:

    turnSignalLever = "Model Root/TurnSignalLever[-1..1]/Value"
    turnSignalLeverVar = appl.Variable(turnSignalLever)
    readValue = turnSignalLeverVar.Read()
    print readValue
    turnSignalLeverVar.Write(1.0) 
    
 **rptlib3** w/sugar:
 Since rtplib3 is an abstraction, you can use [syntactic sugar](https://en.wikipedia.org/wiki/Syntactic_sugar) to simplify new code bases.
 
    turnSignalLever = "Model Root/TurnSignalLever[-1..1]/Value"
    turnSignalLeverVar = appl.Variable(turnSignalLever)
    print turnSignalLeverVar
    turnSignalLeverVar = 1.0
    
XIL API .NET:
(*Since XIL API is a .NET module, all numbers must be cast to .NET types before use*)

    turnSignalLever = "Model Root/TurnSignalLever[-1..1]/Value"
    readValue = maPort.Read(turnSignalLever)
    print readValue.Value
    maPort.Write(turnSignalLever, testbench.ValueFactory.CreateFloatValue(1.0)) 
