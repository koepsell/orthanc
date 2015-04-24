# Storing DICOM Instances into Orthanc #

You have 3 options to upload DICOM instances into Orthanc:

  1. Use the embedded HTML interface (Orthanc Explorer);
  1. Use DICOM commands (C-Store or C-Move);
  1. Use the REST API of Orthanc.

These methods are described below. Each time a DICOM instance is successfully sent to Orthanc, it appears in [Orthanc Explorer](http://localhost:8042/app/explorer.html).

## Through Orthanc Explorer ##

The embedded HTML interface contains a user-friendly page to upload DICOM files. You have to open Google Chrome or Mozilla Firefox at http://localhost:8042/app/explorer.html#upload to reach this upload page. Then, you can drag and drop your DICOM files and click on the Upload button.

## Through DICOM Commands ##

The DICOM AET (Application Entity Title) of Orthanc can be set in its [configuration file](OrthancConfiguration.md). Once Orthanc is configured, any DICOM modality can send its instances to Orthanc.

It is of course also possible to use the command-line tool `storescu` from the [DCMTK distribution](http://dicom.offis.de/dcmtk.php.en) to send DICOM files to Orthanc, for instance:

```
# storescu -aec ORTHANC localhost 4242 *.dcm
```

## Through the REST API ##

The upload of DICOM files is also possible through raw queries to the REST API of Orthanc. Using the standard command-line tool [curl](http://curl.haxx.se/), you can use the following syntax:

```
# curl -X POST http://localhost:8042/instances --data-binary @CT.X.1.2.276.0.7230010.dcm
```

Orthanc will respond with a JSON file that contain information about the location of the stored instance, such as:

```
{
   "ID" : "e87da270-c52b-4f2a-b8c6-bae25928d0b0",
   "Path" : "/instances/e87da270-c52b-4f2a-b8c6-bae25928d0b0",
   "Status" : "Success"
}
```

Note that in the case of `curl`, setting the `Expect` HTTP Header will significantly [reduce the execution time of POST requests](http://stackoverflow.com/a/463277/881731):

```
# curl -X POST -H "Expect:" http://localhost:8042/instances --data-binary @CT.X.1.2.276.0.7230010.dcm
```

The code distribution of Orthanc contains a [sample Python script](http://orthanc.googlecode.com/hg/Resources/Samples/ImportDicomFiles/ImportDicomFiles.py) that recursively upload the content of some folder into Orthanc using the REST API:

```
# python ImportDicomFiles.py localhost 8042 ~/DICOM/
```