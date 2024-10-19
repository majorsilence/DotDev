---
layout: post
title: 
date: 2023-02-22
last_modified: 2024-10-19
comments: true
---

See the newer 2024-10-19 post [dotnet crystal reports on linux via crystalcmd post](/posts/2024/10/19/linux-dotnet-crystalreports-via-crystalcmd.html).

------ 

old stuff

WARNING.  Do not use.   This is one big experiment.

crystalcmd is a:
> Java and c# program to load json files into crystal reports and produce PDFs.

* [https://github.com/majorsilence/CrystalCmd](https://github.com/majorsilence/CrystalCmd)


# Server side hosting

IIS

* Download a .net [crystal reports 13 runtime](https://wiki.scn.sap.com/wiki/display/BOBJ/Crystal+Reports%2C+Developer+for+Visual+Studio+Downloads).  SP 33 or newer.  Ensure it is 64 bit.
* Download a release from [https://github.com/majorsilence/CrystalCmd/releases](https://github.com/majorsilence/CrystalCmd/releases).  
    * Majorsilence.CrystalCmd.NetFrameworkServer 
* Create an 64 bit IIS Site
* Copy in the crystalcmd zip contents
* Edit the web.config and set the username and password you wish to use.


# Client side

## Curl example

```bash
curl https://c.majorsilence.com/status

curl -F "reportdata=@test.json" -F "reporttemplate=@report.rpt" https://{{YOUR SITE}}/export --output testout.pdf

# test localhost
curl -F "reportdata=@test.json" -F "reporttemplate=@report.rpt" https://{{YOUR SITE}}/export --output testout.pdf
```


## C# example

Add the [package Majorsilence.CrystalCmd.Client](https://www.nuget.org/packages/Majorsilence.CrystalCmd.Client) to your project.

```powershell
dotnet add package Majorsilence.CrystalCmd.Client 
```

cs code

```cs
DataTable dt = new DataTable();

// init reprt data
var reportData = new Majorsilence.CrystalCmd.Client.Data()
{
    DataTables = new Dictionary<string, string>(),
    MoveObjectPosition = new List<Majorsilence.CrystalCmd.Client.MoveObjects>(),
    Parameters = new Dictionary<string, object>(),
    SubReportDataTables = new List<Majorsilence.CrystalCmd.Client.SubReports>()
};

// add as many data tables as needed.  The client library will do the necessary conversions to json/csv.
reportData.AddData("report name goes here", "table name goes here", dt);

// export to pdf
var crystalReport = System.IO.File.ReadAllBytes("The rpt template file path goes here");
using (var instream = new MemoryStream(crystalReport))
using (var outstream = new MemoryStream())
{
    var rpt = new Majorsilence.CrystalCmd.Client.Report(serverUrl, username: "The server username goes here", password: "The server password goes here");
    using (var stream = await rpt.GenerateAsync(reportData, instream, _httpClient))
    {
        stream.CopyTo(outstream);
        return outstream.ToArray();
    }
}
```

