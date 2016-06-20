---
layout: post
title: "HOW TO: Convert office documents to PDF using Open Office/LibreOffice in C#"
excerpt_separator: <!--more-->
tags:
    - c#
    - open-office
---

Lately, we had this requirement to convert office documents such as DOC, DOCX, XLS, XLSX, PPT, PPTX to PDF. After googling for sometime,
Microsoft Office doesn’t have any API or exposes any command to achieve this target. As I have some interaction with Open Office on my 
Ubuntu netbook and I knew Open Office has their API exposed with UNO. I started searching and working on a workable demo to convert office
documents to PDF.<!--more-->

Open Office has CLI implementation on Java’s UNO development environment for their API to use on .NET Framework. 
See Open Office Developer Guide for [details](http://wiki.services.openoffice.org/wiki/Documentation/DevGuide/OpenOffice.org_Developers_Guide).

### Things need to be installed

- Open Office (Libre Office now)
- Open Office SDK

NOTE: Open Office SDK is not quite required if you can copy the required assemblies from GAC to you application’s assembly folder. The required files listed below:

- cli_basetypes.dll
- cli_cppuhelper.dll
- cli_oootypes.dll
- cli_ure.dll
- cli_uretypes.dll

If you have installed Open Office SDK, you will get these files under sdkcli on you installed SDK folder.

### Implementation

Add reference to all the DLLs above on your project follow the noted methods below.

You will be needing these namespaces imported

{% highlight csharp %}
using System.Diagnostics;
using System.IO;
using System.Threading;
using uno;
using uno.util;
using unoidl.com.sun.star.beans;
using unoidl.com.sun.star.frame;
using unoidl.com.sun.star.lang;
{% endhighlight %}

This is the method you will be actually using on your assembly or expose from your class library. This method starts up Open Office executable,
initialize UNO components and saves to PDF in the end.

{% highlight csharp %}
public static void ConvertToPdf(string inputFile, string outputFile)
{
    if (ConvertExtensionToFilterType(Path.GetExtension(inputFile)) == null)
        throw new InvalidProgramException("Unknown file type for OpenOffice. File = " + inputFile);

    StartOpenOffice();

    //Get a ComponentContext
    var xLocalContext =
        Bootstrap.bootstrap();
    //Get MultiServiceFactory
    var xRemoteFactory =
        (XMultiServiceFactory)
        xLocalContext.getServiceManager();
    //Get a CompontLoader
    var aLoader =
        (XComponentLoader) xRemoteFactory.createInstance("com.sun.star.frame.Desktop");
    //Load the sourcefile

    XComponent xComponent = null;
    try
    {
        xComponent = InitDocument(aLoader,
                                PathConverter(inputFile), "_blank");
        //Wait for loading
        while (xComponent == null)
        {
            Thread.Sleep(1000);
        }

        // save/export the document
        SaveDocument(xComponent, inputFile, PathConverter(outputFile));
    }
    finally
    {
        if (xComponent != null) xComponent.dispose();
    }
}
{% endhighlight %}

Starts executable instance of `soffice.exe` where your application will be communicating with this using CLI DLLs referenced.

{% highlight csharp %}
private static void StartOpenOffice()
{
    var ps = Process.GetProcessesByName("soffice.exe");
    if (ps.Length != 0)
        throw new InvalidProgramException("OpenOffice not found.  Is OpenOffice installed?");
    if (ps.Length > 0)
        return;
    var p = new Process
                {
                    StartInfo =
                        {
                            Arguments = "-headless -nofirststartwizard",
                            FileName = "soffice.exe",
                            CreateNoWindow = true
                        }
                };
    var result = p.Start();

    if (result == false)
        throw new InvalidProgramException("OpenOffice failed to start.");
}
{% endhighlight %}

This initializes the document instance and load the source file.

{% highlight csharp %}
private static XComponent InitDocument(XComponentLoader aLoader, string file, string target)
{
    var openProps = new PropertyValue[1];
    openProps[0] = new PropertyValue {Name = "Hidden", Value = new Any(true)};

    var xComponent = aLoader.loadComponentFromURL(
        file, target, 0,
        openProps);

    return xComponent;
}
{% endhighlight %}

This method saves the processed document to a destination file.

{% highlight csharp %}
private static void SaveDocument(XComponent xComponent, string sourceFile, string destinationFile)
{
    var propertyValues = new PropertyValue[2];
    // Setting the flag for overwriting
    propertyValues[1] = new PropertyValue {Name = "Overwrite", Value = new Any(true)};
    //// Setting the filter name
    propertyValues[0] = new PropertyValue
                            {
                                Name = "FilterName",
                                Value = new Any(ConvertExtensionToFilterType(Path.GetExtension(sourceFile)))
                            };
    ((XStorable) xComponent).storeToURL(destinationFile, propertyValues);
}
{% endhighlight %}

Converts file path to OpenOffice API readable format.

{% highlight csharp %}
private static string PathConverter(string file)
{
    if (string.IsNullOrEmpty(file))
        throw new NullReferenceException("Null or empty path passed to OpenOffice");

    return String.Format("file:///{0}", file.Replace(@"\", "/"));
}
{% endhighlight %}

This methods returns the filter type required for conversion based on extension.

{% highlight csharp %}
public static string ConvertExtensionToFilterType(string extension)
{
    switch (extension)
    {
        case ".doc":
        case ".docx":
        case ".txt":
        case ".rtf":
        case ".html":
        case ".htm":
        case ".xml":
        case ".odt":
        case ".wps":
        case ".wpd":
            return "writer_pdf_Export";
        case ".xls":
        case ".xlsb":
        case ".xlsx":
        case ".ods":
            return "calc_pdf_Export";
        case ".ppt":
        case ".pptx":
        case ".odp":
            return "impress_pdf_Export";

        default:
            return null;
    }
}
{% endhighlight %}

Well, that’s it! Just invoke the method `ConvertToPdf` with input and output file name parameter.