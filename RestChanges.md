# Tracking Changes #

Whenever Orthanc receives a new DICOM instance, this event is recorded in the so-called "Changes Log". This enables remote scripts to react to the arrival of new DICOM resources. A typical application is auto-routing, where an external script waits for a new DICOM instance to arrive into Orthanc, then forward this instance to another modality.

The Changes Log can be accessed by the following command:

```
# curl http://localhost:8042/changes
```

Here is a typical output:

```
{
   "Changes" : [
      {
         "ChangeType" : "NewInstance",
         "Date" : "20130507T143902",
         "ID" : "8e289db9-0e1437e1-3ecf395f-d8aae463-f4bb49fe",
         "Path" : "/instances/8e289db9-0e1437e1-3ecf395f-d8aae463-f4bb49fe",
         "ResourceType" : "Instance",
         "Seq" : 921
      },
      {
         "ChangeType" : "NewSeries",
         "Date" : "20130507T143902",
         "ID" : "cceb768f-e0f8df71-511b0277-07e55743-9ef8890d",
         "Path" : "/series/cceb768f-e0f8df71-511b0277-07e55743-9ef8890d",
         "ResourceType" : "Series",
         "Seq" : 922
      },
      {
         "ChangeType" : "NewStudy",
         "Date" : "20130507T143902",
         "ID" : "c4ec7f68-9b162055-2c8c5888-5bf5752f-155ab19f",
         "Path" : "/studies/c4ec7f68-9b162055-2c8c5888-5bf5752f-155ab19f",
         "ResourceType" : "Study",
         "Seq" : 923
      },
      {
         "ChangeType" : "NewPatient",
         "Date" : "20130507T143902",
         "ID" : "dc65762c-f476e8b9-898834f4-2f8a5014-2599bc94",
         "Path" : "/patients/dc65762c-f476e8b9-898834f4-2f8a5014-2599bc94",
         "ResourceType" : "Patient",
         "Seq" : 924
      }
   ],
   "Done" : true,
   "Last" : 924
}
```

This output corresponds to the receiving of one single DICOM instance by Orthanc. It records that a new instance, a new series, a new study and a new patient has been created inside Orthanc. Note that each changes is labeled by a `ChangeType`, a `Date` (in the [ISO format](http://en.wikipedia.org/wiki/ISO_8601)), the location of the resource inside Orthanc, and a sequence number (`Seq`).

Note that this call is non-blocking. It is up to the calling program to wait for the occurrence of a new event (by implementing a polling loop).

This call only returns a fixed number of events, that can be changed by using the `limit` option:

```
# curl http://localhost:8042/changes?limit=100
```

The flag `Last` records the sequence number of the lastly returned event. The flag `Done` is set to `True` if no further event has occurred after this lastly returned event. If `Done` is set to `False`, further events are available and can be retrieved. This is done by setting the `since` option that specifies from which sequence number the changes must be returned:

```
# curl 'http://localhost:8042/changes?limit=100&since=922'
```

A [sample code in the source distribution](https://code.google.com/p/orthanc/source/browse/Resources/Samples/Python/ChangesLoop.py) shows how to use this Changes API to implement a polling loop.