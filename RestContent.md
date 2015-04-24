

# Accessing the Content of Orthanc #

Orthanc structures the stored DICOM resources using the "Patient, Study, Series, Instance" model of the DICOM standard. Each DICOM resource is associated with an unique identifier.

## Listing All the DICOM Resources ##

Here is how you would list all the DICOM resources that are stored in your local Orthanc instance:

```
# curl http://localhost:8042/patients
# curl http://localhost:8042/studies
# curl http://localhost:8042/series
# curl http://localhost:8042/instances
```

Note that the result of this command is a [JSON file](http://en.wikipedia.org/wiki/Json) that contains an array of resource identifiers. The JSON file format is lightweight and can be parsed from almost any computer language.

## Accessing a Patient ##

To access a single resource, add its identifier to the [URI](http://en.wikipedia.org/wiki/Uniform_resource_identifier). You would for instance retrieve the main information about one patient as follows:

```
# curl http://localhost:8042/patients/dc65762c-f476e8b9-898834f4-2f8a5014-2599bc94
```

Here is a possible answer from Orthanc:

```
{
   "ID" : "07a6ec1c-1be5920b-18ef5358-d24441f3-10e926ea",
   "MainDicomTags" : {
      "OtherPatientIDs" : "(null)",
      "PatientBirthDate" : "0",
      "PatientID" : "000000185",
      "PatientName" : "Anonymous^Unknown",
      "PatientSex" : "O"
   },
   "Studies" : [ "9ad2b0da-a406c43c-6e0df76d-1204b86f-78d12c15" ],
   "Type" : "Patient"
}
```

This is once again a JSON file. Note how Orthanc gives you a summary of the main DICOM tags that correspond to the patient level.

## Browsing from the Patient to the Instance ##

The field `Studies` list all the DICOM studies that are associated with the patient. So, considering the patient above, we would go down in her DICOM hierarchy as follows:

```
# curl http://localhost:8042/studies/9ad2b0da-a406c43c-6e0df76d-1204b86f-78d12c15
```

And Orthanc could answer:

```
{
   "ID" : "9ad2b0da-a406c43c-6e0df76d-1204b86f-78d12c15",
   "MainDicomTags" : {
      "AccessionNumber" : "(null)",
      "StudyDate" : "20120716",
      "StudyDescription" : "TestSUVce-TF",
      "StudyID" : "23848",
      "StudyInstanceUID" : "1.2.840.113704.1.111.7016.1342451220.40",
      "StudyTime" : "170728"
   },
   "ParentPatient" : "07a6ec1c-1be5920b-18ef5358-d24441f3-10e926ea",
   "Series" : [
      "6821d761-31fb55a9-031ebecb-ba7f9aae-ffe4ddc0",
      "2cc6336f-2d4ae733-537b3ca3-e98184b1-ba494b35",
      "7384c47e-6398f2a8-901846ef-da1e2e0b-6c50d598"
   ],
   "Type" : "Study"
}
```

The main DICOM tags are now those that are related to the study level. It is possible to retrieve the identifier of the patient in the `ParentPatient` field, which can be used to go upward the DICOM hierarchy. But let us rather go down to the series level by using the `Series` array. The next command would return information about one of the three series that have just been reported:

```
# curl http://localhost:8042/series/2cc6336f-2d4ae733-537b3ca3-e98184b1-ba494b35
```

Here is a possible answer:

```
{
   "ExpectedNumberOfInstances" : 45,
   "ID" : "2cc6336f-2d4ae733-537b3ca3-e98184b1-ba494b35",
   "Instances" : [
      "41bc3f74-360f9d10-6ae9ffa4-01ea2045-cbd457dd",
      "1d3de868-6c4f0494-709fd140-7ccc4c94-a6daa3a8",
      <...>
      "1010f80b-161b71c0-897ec01b-c85cd206-e669a3ea",
      "e668dcbf-8829a100-c0bd203b-41e404d9-c533f3d4"
   ],
   "MainDicomTags" : {
      "Manufacturer" : "Philips Medical Systems",
      "Modality" : "PT",
      "NumberOfSlices" : "45",
      "ProtocolName" : "CHU/Body_PET/CT___50",
      "SeriesDate" : "20120716",
      "SeriesDescription" : "[WB_CTAC] Body",
      "SeriesInstanceUID" : "1.3.46.670589.28.2.12.30.26407.37145.2.2516.0.1342458737",
      "SeriesNumber" : "587370",
      "SeriesTime" : "171121",
      "StationName" : "r054-svr"
   },
   "ParentStudy" : "9ad2b0da-a406c43c-6e0df76d-1204b86f-78d12c15",
   "Status" : "Complete",
   "Type" : "Series"
}
```

It can be seen that this series comes from a PET modality. Orthanc has computed that this series should contain 45 instances.

So far, we have navigated from the patient level, to the study level, and finally to the series level. There only remains the instance level. Let us dump the content of one of the instances:

```
# curl http://localhost:8042/instances/e668dcbf-8829a100-c0bd203b-41e404d9-c533f3d4
```

The instance contains the following information:

```
{
   "FileSize" : 70356,
   "FileUuid" : "3fd265f0-c2b6-41a2-ace8-ae332db63e06",
   "ID" : "e668dcbf-8829a100-c0bd203b-41e404d9-c533f3d4",
   "IndexInSeries" : 6,
   "MainDicomTags" : {
      "ImageIndex" : "6",
      "InstanceCreationDate" : "20120716",
      "InstanceCreationTime" : "171344",
      "InstanceNumber" : "6",
      "SOPInstanceUID" : "1.3.46.670589.28.2.15.30.26407.37145.3.2116.39.1342458737"
   },
   "ParentSeries" : "2cc6336f-2d4ae733-537b3ca3-e98184b1-ba494b35",
   "Type" : "Instance"
}
```

The instance has the index 6 in the parent series. The instance is stored as a raw DICOM file of 70356 bytes. You would download this DICOM file using the following command:

```
# curl http://localhost:8042/instances/e668dcbf-8829a100-c0bd203b-41e404d9-c533f3d4/file > Instance.dcm
```


## Accessing the DICOM Fields of an Instance as a JSON File ##

When one gets to the instance level, you can retrieve the hierarchy of all the DICOM tags of this instance as a JSON file:

```
# curl http://localhost:8042/instances/e668dcbf-8829a100-c0bd203b-41e404d9-c533f3d4/simplified-tags
```

Here is a excerpt of the Orthanc answer:

```
{
   "ACR_NEMA_2C_VariablePixelDataGroupLength" : "57130",
   "AccessionNumber" : null,
   "AcquisitionDate" : "20120716",
   "AcquisitionDateTime" : "20120716171219",
   "AcquisitionTime" : "171219",
   "ActualFrameDuration" : "3597793",
   "AttenuationCorrectionMethod" : "CTAC-SG",
   <...>
   "PatientID" : "000000185",
   "PatientName" : "Anonymous^Unknown",
   "PatientOrientationCodeSequence" : [
      {
         "CodeMeaning" : "recumbent",
         "CodeValue" : "F-10450",
         "CodingSchemeDesignator" : "99SDM",
         "PatientOrientationModifierCodeSequence" : [
            {
               "CodeMeaning" : "supine",
               "CodeValue" : "F-10340",
               "CodingSchemeDesignator" : "99SDM"
            }
         ]
      }
   ],
   <...>
   "StudyDescription" : "TestSUVce-TF",
   "StudyID" : "23848",
   "StudyInstanceUID" : "1.2.840.113704.1.111.7016.1342451220.40",
   "StudyTime" : "171117",
   "TypeOfDetectorMotion" : "NONE",
   "Units" : "BQML",
   "Unknown" : null,
   "WindowCenter" : "1.496995e+04",
   "WindowWidth" : "2.993990e+04"
}
```

If you need more detailed information about the type of the variables or if you wish to use the hexadecimal indexes of DICOM tags, you are free to use the following URL:

```
# curl http://localhost:8042/instances/e668dcbf-8829a100-c0bd203b-41e404d9-c533f3d4/tags
```


## Accessing the Raw DICOM Fields of an Instance ##

You also have the opportunity to access the raw value of the DICOM tags of an instance, without going through a JSON file. Here is how you would find the Patient Name of the instance:

```
# curl http://localhost:8042/instances/e668dcbf-8829a100-c0bd203b-41e404d9-c533f3d4/content/0010-0010
Anonymous^Unknown 
```

The list of all the available tags for this instance can also be retrieved easily:

```
# curl http://localhost:8042/instances/e668dcbf-8829a100-c0bd203b-41e404d9-c533f3d4/content
```

It is also possible to recursively explore the sequences of tags:

```
# curl http://localhost:8042/instances/e668dcbf-8829a100-c0bd203b-41e404d9-c533f3d4/content/0008-1250/0/0040-a170/0/0008-0104
For Attenuation Correction
```

The command above has opened the "0008-1250" tag that is a DICOM sequence, taken its first child, opened its "0040-a170" tag that is also a sequence, taken the first child of this child, and returned the "0008-0104" DICOM tag.

## Downloading Images ##

It is also possible to download a preview PNG image that corresponds to some DICOM instance:

```
# curl http://localhost:8042/instances/e668dcbf-8829a100-c0bd203b-41e404d9-c533f3d4/preview > Preview.png
```

The resulting image will be a standard graylevel PNG image that can be opened by any painting software.

## An Overview of the REST API ##

The above examples have shown you the basic principles for accessing the content of an Orthanc instance through its REST API. All the possibilities of the API have not been described. Please refer to the [full feature grid](https://docs.google.com/spreadsheet/pub?key=0Ao5aRMxCX2hldEJadzVUaWFmNW5QTWhrYTI3UHMzdXc&single=true&gid=8&output=html) to see all the operations that Orthanc supports.