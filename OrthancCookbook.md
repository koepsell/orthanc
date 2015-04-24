# Orthanc Cookbook #

## Binaries ##
To obtain the Orthanc binaries, you have several possibilities:
  * [Compile Orthanc by yourself](http://orthanc.googlecode.com/hg/INSTALL).
  * [Download Windows binaries of the releases](https://github.com/jodogne/Orthanc/).
  * A [Fedora package](https://apps.fedoraproject.org/packages/orthanc) is available.
  * A [Debian package](http://packages.debian.org/en/sid/orthanc) is available.
  * A Continuous Integration Server builds Orthanc each night from the mainline. The resulting binaries (notably for Windows) can be downloaded [at this location](http://www.montefiore.ulg.ac.be/~jodogne/Orthanc/NightlyBuilds.php).

## Opening Orthanc Explorer ##
The most straightforward way to use Orthanc consists in opening **Orthanc Explorer**, the embedded GUI of Orthanc, with a Web browser.

Once Orthanc is running, use Google Chrome or Mozilla Firefox to open the following URL: http://localhost:8042/app/explorer.html. Note that the port number 8042 depends on your Orthanc configuration.

You can [watch this video tutorial](https://www.youtube.com/watch?v=4dOcXGMlcFo&hd=1) that shows how to upload files to Orthanc through Orthanc Explorer.

## Instructions for Users ##
You will find below more advanced instructions about the use of Orthanc and its REST API for scripting tasks.

  * [How to configure Orthanc](OrthancConfiguration.md).
  * [How to add DICOM instances to Orthanc](StoringInstances.md).
  * How to script with the REST API:
    * [How to access the content of Orthanc](RestContent.md).
    * [How to anonymize or modify DICOM resources](Anonymization.md).
    * [How to delete content from Orthanc](RestDelete.md).
    * [How to track changes](RestChanges.md).
    * [How to send resources to remote DICOM modalities](RestStoreScu.md) (C-Store SCU).
    * [How to query the content of remote DICOM modalities](RestDicomFind.md) (C-Find SCU).
  * [Server-side scripting](ServerSideScripting.md) with Lua.

**Important Note:** All the examples are illustrated with the [cURL command-line tool](http://en.wikipedia.org/wiki/CURL), but equivalent calls can be readily transposed to any programming language that supports both [HTTP](http://en.wikipedia.org/wiki/Http) and [JSON](http://en.wikipedia.org/wiki/Json).

## Notes for Developers ##

  * The [coding style](CodingStyle.md) in Orthanc.
  * The rules for [database versioning](DatabaseVersioning.md) in Orthanc.