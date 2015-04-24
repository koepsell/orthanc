

# Server-Side Scripting with Lua #

Since release 0.5.2, Orthanc supports server-side scripting through the [Lua](http://en.wikipedia.org/wiki/Lua_(programming_language)) scripting language. Thanks to this major feature, Orthanc can be tuned to specific medical workflows without being driven by an external script. This page summarizes the possibilities of Orthanc server-side scripting.

## Installing a Lua Script ##

A custom Lua script can be installed either by the [configuration file](OrthancConfiguration.md), or by uploading it through the REST API.

To install it by the configuration file method, you just have to specify the path to the file containing the Lua script in the `LuaScripts` variable.

To upload a script stored in the file "`script.lua`" through the REST API, use the following command:

```
# curl -X POST http://localhost:8042/tools/execute-script --data-binary @script.lua
```

Pay attention to the fact that, contrarily to the scripts installed from the configuration file, the scripts installed through the REST API are non-persistent: They are discarded after a restart of Orthanc, which makes them useful for script prototyping. You can also interpret a single Lua command through the REST API:

```
# curl -X POST http://localhost:8042/tools/execute-script --data-binary "print(42)"
```

_Note:_ The `--data-binary` cURL option is used instead of `--data` to prevent the interpretation of newlines by cURL, which is [mandatory for the proper evaluation](http://stackoverflow.com/q/3872427/881731) of the possible comments inside the Lua script.

## Filtering Incoming DICOM Instances ##

Each time a DICOM instance is received by Orthanc (either through the DICOM protocol or the REST API), the `ReceivedInstanceFilter()` Lua function is evaluated. If this callback returns `true`, the instance is accepted for storage. If it returns `false`, the instance is dropped.

This mechanism can be used to filter the incoming DICOM instances. Here is an example of a Lua filter that only allows incoming instances of MR modality:

```lua

function ReceivedInstanceFilter(dicom, aet)
-- Only allow incoming MR images
if dicom.Modality == 'MR' then
return true
else
return false
end
end```

The argument `dicom` corresponds to a [Lua table](http://www.lua.org/pil/2.5.html) (i.e. an associative array) that contains the DICOM tags of the incoming instance. For debugging purpose, you can print this structure as follows:

```lua

function ReceivedInstanceFilter(dicom, aet)
PrintRecursive(dicom)
-- Accept all incoming instances (default behavior)
return true
end```

The argument `aet` gives the AET of the remote modality that has issued the C-Store command. This argument is set to the empty string if the DICOM instance comes from the REST API.

## Filtering Incoming REST Requests ##

Lua scripting can be used to control the access to the various URI of the REST API. Each time an incoming HTTP request is received, the `IncomingHttpRequestFilter()` Lua function is called. The access to the resource is granted if and only if this callback script returns `true`.

This mechanism can be used to implement fine-grained [access control lists](http://en.wikipedia.org/wiki/Access_control_list). Here is an example of a Lua script that limits `POST`, `PUT` and `DELETE` requests to an user that is called "admin":

```lua

function IncomingHttpRequestFilter(method, uri, ip, username)
-- Only allow GET requests for non-admin users

if method == 'GET' then
return true
elseif username == 'admin' then
return true
else
return false
end
end```

Here is a description of the arguments of this Lua callback:
  * `method`: The HTTP method (`GET`, `POST`, `PUT` or `DELETE`).
  * `uri`: The path to the resource (e.g. `/tools/generate-uid`).
  * `ip`: The IP address of the host that has issued the HTTP request (e.g. `127.0.0.1`).
  * `username`: If HTTP Basic Authentication is enabled in the configuration file, the name of the user that has issued the HTTP request (as defined in the `RegisteredUsers` configuration variable). If the authentication is disabled, this argument is set to the empty string.


## Auto-Routing of DICOM Images ##

Since release 0.8.0, the routing of DICOM flows can be very easily automated with Orthanc. All you have to do is to declare your destination modality in the [configuration file](OrthancConfiguration.md) (section `DicomModalities`), then to create and install a Lua script. For instance, here is a sample script:

```
function OnStoredInstance(instanceId, tags, metadata)
   Delete(SendToModality(instanceId, 'sample'))
end
```

If this script is loaded into Orthanc, whenever a new DICOM instance is received by Orthanc, it will be routed to the modality `sample` (through a `Store-SCU` command), then it will be removed from Orthanc. In other words, this is a **one-liner script to implement DICOM auto-routing**.

Very importantly, thanks to this feature, you do not have to use the REST API or to create external scripts in order to automate simple imaging flows. The scripting engine is entirely contained inside the Orthanc core system.

Thanks to Lua expressiveness, you can also implement conditional auto-routing. For instance, if you wish to route only patients whose name contains "David", you would simply write:

```lua

function OnStoredInstance(instanceId, tags, metadata)
-- Extract the value of the "PatientName" DICOM tag
local patientName = string.lower(tags['PatientName'])

if string.find(patientName, 'david') ~= nil then
-- Only route patients whose name contains "David"
Delete(SendToModality(instanceId, 'sample'))

else
-- Delete the patients that are not called "David"
Delete(instanceId)
end
end```

It is also to modify the received instances before routing them. For instance, here is how you would replace the "`StationName`" DICOM tag:

```lua

function OnStoredInstance(instanceId, tags, metadata)
-- Ignore the instances that result from a modification to avoid
-- infinite loops
if (metadata['ModifiedFrom'] == nil and
metadata['AnonymizedFrom'] == nil) then

-- The tags to be replaced
local replace = {}
replace['StationName'] = 'My Medical Device'

-- The tags to be removed
local remove = { 'MilitaryRank' }

-- Modify the instance, send it, then delete the modified instance
Delete(SendToModality(ModifyInstance(instanceId, replace, remove, true), 'sample'))

-- Delete the original instance
Delete(instanceId)
end
end```

The source distribution of Orthanc contains [other Lua samples](https://code.google.com/p/orthanc/source/browse/#hg%2FResources%2FSamples%2FLua).