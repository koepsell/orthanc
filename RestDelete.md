# Deleting Resources from Orthanc #

Deleting DICOM resources (i.e. patients, studies, series or instances) from Orthanc is as simple as using a HTTP DELETE on the [URI](http://en.wikipedia.org/wiki/URI) of this resource.

Concretely, you would first explore the resources that are stored in Orthanc [as explained on this page](RestContent.md):

```
# curl http://localhost:8042/patients
# curl http://localhost:8042/studies
# curl http://localhost:8042/series
# curl http://localhost:8042/instances
```

Secondly, once you have spotted the resources to be removed, you would use the following command-line syntax to delete them:

```
# curl -X DELETE http://localhost:8042/patients/dc65762c-f476e8b9-898834f4-2f8a5014-2599bc94
# curl -X DELETE http://localhost:8042/studies/c4ec7f68-9b162055-2c8c5888-5bf5752f-155ab19f
# curl -X DELETE http://localhost:8042/series/cceb768f-e0f8df71-511b0277-07e55743-9ef8890d
# curl -X DELETE http://localhost:8042/instances/8e289db9-0e1437e1-3ecf395f-d8aae463-f4bb49fe
```

# Clearing Logs #

## Changes Logs ##

As [described on this page](RestChanges.md), Orthanc keeps track of all the changes that occur in the DICOM store. This so-called "Changes Log" is accessible at the following URI:

```
# curl http://localhost:8042/changes
```

To clear the Changes Log, simply DELETE this URI:

```
# curl -X DELETE http://localhost:8042/changes
```

## Log of Exported Resources ##

For medical traceability, Orthanc stores a log of all the resources that have been exported to remote modalities:

```
# curl http://localhost:8042/exports
```

To clear this log (e.g. to prevent the SQLite from growing indefinitely  when doing auto-routing), simply DELETE this URI:

```
# curl -X DELETE http://localhost:8042/exports
```