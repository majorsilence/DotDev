---
layout: base
title: Crystal Reports
date: 2018-02-14
last_modified: 2024-03-30
description: C# crystal report examples
tags: c# crystal reports
---

Examples to use crystal reports from c#.

Make sure you have the crystal reports runtime installed.  It
can be downloaded from [https://wiki.scn.sap.com/wiki/display/BOBJ/Crystal+Reports%2C+Developer+for+Visual+Studio+Downloads](https://wiki.scn.sap.com/wiki/display/BOBJ/Crystal+Reports%2C+Developer+for+Visual+Studio+Downloads).

All examples below require references for __CrystalDecisions.CrystalReports.Engine__ and __CrystalDecisions.Shared__ to be added to your project.

Newer Crystal runtime, such as SP 35

```xml
<ItemGroup>
		<Reference Include="CrystalDecisions.Windows.Forms, Version=13.0.4000.0, Culture=neutral, PublicKeyToken=692fbea5521e1304, processorArchitecture=MSIL">
			<HintPath>C:\Windows\Microsoft.NET\assembly\GAC_MSIL\CrystalDecisions.Windows.Forms\v4.0_13.0.4000.0__692fbea5521e1304\CrystalDecisions.Windows.Forms.dll</HintPath>
		</Reference>
		<Reference Include="CrystalDecisions.CrystalReports.Engine, Version=13.0.4000.0, Culture=neutral, PublicKeyToken=692fbea5521e1304, processorArchitecture=MSIL">
			<HintPath>C:\Windows\Microsoft.NET\assembly\GAC_MSIL\CrystalDecisions.CrystalReports.Engine\v4.0_13.0.4000.0__692fbea5521e1304\CrystalDecisions.CrystalReports.Engine.dll</HintPath>
		</Reference>
		<Reference Include="CrystalDecisions.Shared, Version=13.0.4000.0, Culture=neutral, PublicKeyToken=692fbea5521e1304, processorArchitecture=MSIL">
			<HintPath>C:\Windows\Microsoft.NET\assembly\GAC_MSIL\CrystalDecisions.Shared\v4.0_13.0.4000.0__692fbea5521e1304\CrystalDecisions.Shared.dll</HintPath>
		</Reference>
	</ItemGroup>
```

Old Crystal SP 20 runtime or older references
```xml
<Reference Include="CrystalDecisions.CrystalReports.Engine, Version=13.0.2000.0, 
Culture=neutral, PublicKeyToken=692fbea5521e1304, processorArchitecture=MSIL" />
<Reference Include="CrystalDecisions.Shared, Version=13.0.2000.0,
 Culture=neutral, PublicKeyToken=692fbea5521e1304, processorArchitecture=MSIL" />
```

# Set data using DataTables

Example initializing a report and passing in a DataTable.

```cs
using CrystalDecisions.CrystalReports.Engine;
using CrystalDecisions.Shared;

// Pass a DataTable to a crystal report table
public static SetData(string crystalTemplateFilePath, 
    string pdfFilename, DataSet val)
{
    using (var rpt = new ReportDocument())
    {
        rpt.Load(crystalTemplateFilePath);
   
        rpt.Database.Tables["tableName"].SetDataSource(val);
    }
}

// Pass any generic IEnumerable data to a crystal report DataTable
public static SetData(string crystalTemplateFilePath, 
    IEnumerable<T> val)
{
    using (var rpt = new ReportDocument())
    {
        rpt.Load(crystalTemplateFilePath);
   
        var dt = ConvertGenericListToDatatable(val);
        rpt.Database.Tables[tableName].SetDataSource(val);
    }
}


// Found somewhere on the internet.  I know longer remember where.
public static DataTable ConvertGenericListToDatatable<T>(IEnumerable<T> dataLst)
{
    DataTable dt = new DataTable();

    foreach (var info in dataLst.FirstOrDefault().GetType().GetProperties())
    {
        dt.Columns.Add(info.Name, info.PropertyType);
    }

    foreach (var tp in dataLst)
    {
        DataRow row = dt.NewRow();
        foreach (var info in typeof(T).GetProperties())
        {
            if (info.Name == "Item") continue;
            row[info.Name] = info.GetValue(tp, null) == null ? DBNull.Value : info.GetValue(tp, null);
        }
        dt.Rows.Add(row);
    }
    dt.AcceptChanges();
    return dt;
}
```

# Set report parameters

Set report parameters from code.

```cs
using CrystalDecisions.CrystalReports.Engine;
using CrystalDecisions.Shared;

public static void SetParameterValueName(string crystalTemplateFilePath, object val)
{
    using (var rpt = new ReportDocument())
    {
        rpt.Load(crystalTemplateFilePath);

        string name = "ParameterName";
        if (rpt.ParameterFields[name] != null)
        {
            this.MyReportDoc.SetParameterValue(name, val);
        }
    }
}
```

# Move report objects

Example moving an object.

```cs
using CrystalDecisions.CrystalReports.Engine;
using CrystalDecisions.Shared;

public static MoveObject(string crystalTemplateFilePath)
{
    using (var rpt = new ReportDocument())
    {
        rpt.Load(crystalTemplateFilePath);
   
        rpt.ReportDefinition.ReportObjects["objectName"].Left = 15;
        rpt.ReportDefinition.ReportObjects["objectName"].Top = 15;
    }
}
```



# Export to pdf or other file type

Load a report and export it to pdf.  You can pass in data or set other properties before the export.

```cs
using CrystalDecisions.CrystalReports.Engine;
using CrystalDecisions.Shared;

public static ExportPdf(string crystalTemplateFilePath, 
    string pdfFilename)
{
    using (var rpt = new ReportDocument())
    {
        rpt.Load(crystalTemplateFilePath);
   
        var exp = ExportFormatType.PortableDocFormat;
        rpt.ExportToDisk(exp, pdfFilename);
    }
}
```


# net6.0 amd newer

As of March 30, 2024 the dotnet crystal reports runtime only works with .netframework 4.8.  There is no support for dotnet core and the new net6.0 and newer versions of dotnet.

To work with legacy crystal reports in a modern dotnet environment see [CrytalCMD](https://github.com/majorsilence/CrystalCmd).  CrystalCMD is a tool to permit sending crystal reports rpt files and data to an external .net framework 4.8 server or console application that will return a pdf document that can be viewed in a net6.0 or newer application.  Use libraries such as [IronPDF](https://ironpdf.com/blog/using-ironpdf/pdf-viewer-csharp-windows-application-tutorial/) or Telerik pdf viewer options.

A few Telerik options include:
- [UI for WinForms - WinForms PDF Viewer](https://www.telerik.com/products/winforms/pdf-viewer.aspx)
- [UI for ASP.NET Core - ASP.NET Core PDF Viewer](https://www.telerik.com/aspnet-core-ui/pdf-viewer)
- [UI for Blazor - Blazor PDF Viewe](https://www.telerik.com/blazor-ui/pdf-viewer)
- [UI for .NET MAUI - .NET MAUI PDF Viewer](https://www.telerik.com/maui-ui/pdf-viewer)

If the application is a web application another option is to let the browser display the pdf using the browsers native pdf viewer.