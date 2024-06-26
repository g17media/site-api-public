---
title: API Reference

language_tabs: # must be one of https://github.com/rouge-ruby/rouge/wiki/List-of-supported-languages-and-lexers
  - shell
  - csharp

toc_footers:
  - <a href="https://www.lcpdfr.com/terms/">Terms of Service</a>
  - <a href="https://www.lcpdfr.com/privacy/">Privacy Policy</a>
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true

code_clipboard: true

meta:
  - name: description
    content: Documentation for the Kittn API
---

# Introduction

Welcome to the LCPDFR.com public website API.

<aside class="warning">
If calling any of these functions within, for example, a RAGEPluginHook or LSPDFR plugin, you should not make network calls within a game executing thread or fiber as this will <i>freeze the game</i>. For an example of how to call in a seperate thread, see <a href="https://gist.github.com/LSPDFR-PeterU/952de66a4f17b838d3870bdaabb6c43b">AsyncUpdateChecker by PeterU</a>.
</aside>

# Download Center

## Authorization
For public download center API calls, no authentication is needed. You should however try to ensure your client gives a *unique User-Agent*. Calls without a User-Agent may be throttled.
## Get user-facing version for a file

```shell
curl "https://www.lcpdfr.com/applications/downloadsng/interface/api.php" \
  -X GET -G \
  -H "User-Agent: CyanCallouts/1.0 (+https://www.lcpdfr.com/profile/9-cyan/)" \
  -d "do=checkForUpdates" \
  -d "fileId=7792" \
  -d "textOnly=1"
```

```csharp

  public static string GetLatestVersion(int fileId)
  {
      string updateEndpoint = "http://www.lcpdfr.com/applications/downloadsng/interface/api.php?do=checkForUpdates&fileId={0}&textOnly=true";
      string latestVersion = null;
      try {
          using(WebClient client = new WebClient()) {
              client.Headers.Add("user-agent", "CyanCallouts/1.0 (+https://www.lcpdfr.com/profile/9-cyan/)");
              latestVersion = client.DownloadString(string.Format(updateEndpoint, fileId));
          }
      }
      catch(Exception e) {
              // Failed. Maybe log the exception.
      }
      return latestVersion;
  }
```

> The above command returns a text only response of the author provided file version.

```
0.4.9 (Build 8678)
```

This endpoint retrieves the latest author provided version of a file, given a file ID.

### HTTP Request

`GET https://www.lcpdfr.com/applications/downloadsng/interface/api.php`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
do | null | Needs set to 'checkForUpdates'
fileId | null | The file ID to query
textOnly | false | Set to true to provide a text only (non JSON) response.

<aside class="notice">
Remember — the version provided is directly from the author, presented on their file page. It may or may not be accurate for automation or development purposes.
</aside>

## Get .NET assembly versions for a file

```shell
curl "https://www.lcpdfr.com/applications/downloadsng/interface/api.php" \
  -X GET -G \
  -H "User-Agent: CyanCallouts/1.0 (+https://www.lcpdfr.com/profile/9-cyan/)" \
  -d "do=getAssemblies" \
  -d "fileId=7792" 
```

```csharp
/*
 * This is an example of an async function that tries to retrieve the latest LSPDFR assembly version
 */
using System.Text.Json;
using System.Net;
using System;

public class AssemblyInfoRetriever
{
    private static readonly HttpClient client = new HttpClient();

    public static async Task<string> GetLSPDFRAssemblyVersionAsync()
    {
        string url = "https://www.lcpdfr.com/applications/downloadsng/interface/api.php?do=getAssemblies&fileId=7792";
        string assemblyVersion = null;

        try
        {
            var response = await client.GetStringAsync(url);
            using (JsonDocument doc = JsonDocument.Parse(response))
            {
                JsonElement root = doc.RootElement;
                string error = root.GetProperty("error").GetString();

                if (string.IsNullOrEmpty(error))
                {
					foreach(JsonProperty file in root.GetProperty("result").GetProperty("files").EnumerateObject()) {
						if (file.Value.TryGetProperty("plugins/LSPD First Response.dll", out JsonElement versionElement)) {
							assemblyVersion = versionElement.GetString();
						}
					}
                }
            }
			if(assemblyVersion == null) {
				throw new Exception("Couldn't find LSPDFR dll within result!");
			}
        }
        catch (Exception ex)
        {
            Console.WriteLine("Error occurred: " + ex.Message);
        }

        return assemblyVersion;
    }
}
```

> The above command returns JSON structured like this:

```json

{
  "error": "",
  "result": {
    "files": {
      "lspdfr_049_8678_setup.exe": {},
      "LSPDFR_049_8678_Manual_Install.zip": {
        "DdsConvert.dll": "1.0.0.0",
        "DiscordRpcNet.dll": "1.0.0.0",
        "EasyHook.dll": "2.7.6578.0",
        "EasyLoad64.dll": "2.7.6578.0",
        "Gwen.dll": "1.0.0.0",
        "Gwen.UnitTest.dll": "1.0.0.0",
        "LMS.Common.dll": "1.0.0.0",
        "LMS.PortableExecutable.dll": "1.0.0.0",
        "lspdfr\/LSPDFR Configurator.exe": "1.0.0.0",
        "lspdfr\/Microsoft.Expression.Drawing.dll": "4.5.0.0",
        "Microsoft.Expression.Drawing.dll": "4.5.0.0",
        "Microsoft.VisualStudio.QualityTools.UnitTestFramework.dll": "10.0.0.0",
        "Mono.Cecil.dll": "0.9.6.0",
        "Mono.Cecil.Mdb.dll": "0.9.6.0",
        "Mono.Cecil.Pdb.dll": "0.9.6.0",
        "Mono.Cecil.Rocks.dll": "0.9.6.0",
        "plugins\/LSPD First Response.dll": "0.4.8678.25591",
        "RAGEPluginHook.exe": "1.0.0.0",
        "SlimDX.dll": "4.0.13.43",
        "System.ValueTuple.dll": "4.0.2.0"
      }
    }
  }
}
```

This endpoint retrieves all .NET assemblies inside all subfiles on a file submission.

### HTTP Request

`GET https://www.lcpdfr.com/applications/downloadsng/interface/api.php`

### URL Parameters

Parameter | Default | Description
--------- | ------- | -----------
do | null | Needs set to 'getAssemblies'
fileId | null | The file ID to query

<aside class="notice">
.NET assembly versions, as well as other .NET metadata, have only started being collected recently. As a result, this metadata may not be available for all files. We intend to rebuild this data for all files uploaded before, but this hasn't been done yet. We'll update the API topic once this has been completed.
</aside>

<aside class="notice">
Filenames are chosen by the author, and as a result can and usually do change on version updates. You are recommended to iterate through the <i>files</i> object instead of hardcoding a specific file.
</aside>

## Get user-facing version of all files

```shell
curl "https://www.lcpdfr.com/applications/downloadsng/interface/api.php" \
  -X GET -G \
  -H "User-Agent: CyanCallouts/1.0 (+https://www.lcpdfr.com/profile/9-cyan/)" \
  -d "do=getAllVersions" \
  -d "categoryId=45" \
  -d "page=1"
```

> The above command returns JSON structured like this:

```json
{
  "page": "1",
  "perPage": "500",
  "totalResults": "562",
  "totalPages": "2",
  "results": [
    {
      "file_id": "47969",
      "file_name": "Simple Speed Radar",
      "file_submitter": "254392",
      "file_version": "1.5"
    },
    {
      "file_id": "47988",
      "file_name": "505 Callouts",
      "file_submitter": "615323",
      "file_version": "1.0.1"
    }
  ]
}
```

This endpoint retrieves all user-facing versions given a category ID, in bulk. It will always paginate by each 500 entries.

### HTTP Request

`GET https://www.lcpdfr.com/applications/downloadsng/interface/api.php`

### URL Parameters

Parameter | Default | Description
--------- | ------- | -----------
do | null | Needs set to 'getAllVersions'
categoryId | null | The category ID to show files for.
page | 1 | Results are paginated. This defines which page.

<aside class="notice">
.NET assembly versions are not included in this data, but may be in a future update.
</aside>