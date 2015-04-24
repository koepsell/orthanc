

# Introduction #

Orthanc 0.5.0 introduces the anonymization of DICOM resources (i.e. patients, studies, series or instances). This page summarizes how to use this feature.

## Anonymization of a Single Instance ##

Orthanc allows to anonymize a single DICOM instance and to download the resulting anonymized DICOM file. Anonymization consists in erasing all the tags that are specified in Table E.1-1 from PS 3.15 of the DICOM standard 2008. Example:

```
# curl http://localhost:8042/instances/6e67da51-d119d6ae-c5667437-87b9a8a5-0f07c49f/anonymize -X POST -d '{}' > Anonymized.dcm
```

It is possible to control how anonymization is achieved by specifying a JSON body:

```
# curl http://localhost:8042/instances/6e67da51-d119d6ae-c5667437-87b9a8a5-0f07c49f/anonymize -X POST -d '{"Replace":{"PatientName":"hello","0010-0020":"world"},"Keep":["StudyDescription", "SeriesDescription"],"KeepPrivateTags": null}' > Anonymized.dcm
```

Explanations:
  * New UUIDs are automatically generated for the study, the series and the instance.
  * The DICOM tags can be specified either by their name (`PatientName`) or by their hexadecimal identifier (`0010-0020` corresponds to `PatientID`).
  * `Replace` is an associative array that associates a DICOM tag with its new string value. The value is dynamically cast to the proper DICOM data type (an HTTP error will occur if the cast fails). Replacements are applied after all the tags to anonymize have been removed.
  * `Keep` specifies a list of tags that should be preserved from full anonymization.
  * If `KeepPrivateTags` is absent from the JSON request, private tags (i.e. manufacturer-specific tags) are also removed. This is the default behavior, as such tags can contain patient-specific information.



## Modification of a Single Instance ##

Orthanc allows to modify a set of specified tags in a single DICOM instance and to download the resulting anonymized DICOM file. Example:

```
# curl http://localhost:8042/instances/6e67da51-d119d6ae-c5667437-87b9a8a5-0f07c49f/modify -X POST -d '{"Replace":{"PatientName":"hello","PatientID":"world"},"Remove":["InstitutionName"],"RemovePrivateTags": null}' > Modified.dcm
```

Remarks:
  * The `Remove` array specifies the list of the tags to remove.
  * The `Replace` associative array specifies the substitions to be applied (cf. anonymization).
  * If `RemovePrivateTags` is present, the private tags (i.e. manufacturer-specific tags) are removed.


## Modification of Studies or Series ##

It is possible to modify all the instances from a study or from a series in a single request. In this case, the modified instances are stored back into the Orthanc store. Here is how to modify a series:

```
# curl http://localhost:8042/series/95a6e2bf-9296e2cc-bf614e2f-22b391ee-16e010e0/modify -X POST -d '{"Replace":{"InstitutionName":"My own clinic"}}'
```

The parameters are identical to those used to modify a single instance. Orthanc will answer a JSON message that tells where the modified series has been stored:

```
{
   "ID" : "3bd3d343-82879d86-da77321c-1d23fd6b-faa07bce",
   "Path" : "/series/3bd3d343-82879d86-da77321c-1d23fd6b-faa07bce"
}
```

Similarly, here is an interaction to modify a study:

```
# curl http://localhost:8042/studies/ef2ce55f-9342856a-aee23907-2667e859-9f3b734d/modify -X POST -d '{"Replace":{"InstitutionName":"My own clinic"}}'

{
   "ID" : "1c3f7bf4-85b4aa20-236e6315-5d450dcc-3c1bcf28",
   "Path" : "/studies/1c3f7bf4-85b4aa20-236e6315-5d450dcc-3c1bcf28"
}
```


## Modification of Patients ##

Starting with Orthanc 0.7.5, Orthanc can also modify all the instances of a patient with a single REST call. Here is a sample:

```
#  curl http://localhost:8042/patients/6fb47ef5-072f4557-3215aa29-f99515c1-6fa22bf0/modify -X POST -d '{"Replace":{"PatientID":"Hello","PatientName":"Sample patient name"}}'

{
   "ID" : "f7ff9e8b-7bb2e09b-70935a5d-785e0cc5-d9d0abf0",
   "Path" : "/patients/f7ff9e8b-7bb2e09b-70935a5d-785e0cc5-d9d0abf0",
   "PatientID" : "f7ff9e8b-7bb2e09b-70935a5d-785e0cc5-d9d0abf0",
   "Type" : "Patient"
}
```

Please note that, in this case, you have to set the value of the `(0010,0020) "PatientID"` tag for Orthanc to accept this modification: This is a security to prevent the merging of patient data before and after anonymization, if the user does not explicitly tell Orthanc to do so.

## Anonymization of Patients, Studies or Series ##

Study and series can be anonymized the same way as they are modified:

```
# curl http://localhost:8042/patients/6fb47ef5-072f4557-3215aa29-f99515c1-6fa22bf0/anonymize -X POST -d '{}'
# curl http://localhost:8042/studies/ef2ce55f-9342856a-aee23907-2667e859-9f3b734d/anonymize -X POST -d '{}'
# curl http://localhost:8042/series/95a6e2bf-9296e2cc-bf614e2f-22b391ee-16e010e0/anonymize -X POST -d '{}'
```

As written above, the anonymization process can be fine-tuned by using a JSON body.