# Sending Resources to Remote Modalities #

Orthanc can send its DICOM instances to remote DICOM modalities (C-Store SCU).

## Configuration ##

You first have to declare the AET, the IP address and the port number of the remote modality inside the [configuration file](OrthancConfiguration.md). For instance, here is how to declare a remote modality:

```
...
"DicomModalities" : {
    "sample" : [ "STORESCP", "127.0.0.1", 2000 ]
},
...
```

Such a configuration would enable Orthanc to connect to another DICOM store (for instance, another Orthanc instance) that listens on the localhost on the port 2000. The modalities that are known to Orthanc can be queried:

```
# curl http://localhost:8042/modalities
```

## Sending One Resource ##

Once you have identified the Orthanc identifier of the DICOM resource that would like to send ([as explained on this page](RestContent.md)), you would use the following command to send it:

```
# curl -X POST http://localhost:8042/modalities/sample/store -d c4ec7f68-9b162055-2c8c5888-5bf5752f-155ab19f
```

The `/sample/` component corresponds to the identifier of the remote modality, as specified above in the configuration file.

Note that you can send isolated DICOM instances with this command, but also entire patients, studies or series.

## Bulk Store SCU ##

Each time a POST request is made to `/modalities/.../store`, a new DICOM connection is established. This may lead to a large communication overhead if sending multiple isolated instances. Starting with Orthanc 0.5.2, it is possible to send multiple instances with a single POST request (so-called "Bulk Store SCU"):

```
# curl -X POST http://localhost:8042/modalities/sample/store -d '["d4b46c8e-74b16992-b0f5ca11-f04a60fa-8eb13a88","d5604121-7d613ce6-c315a5-a77b3cf3-9c253b23","cb855110-5f4da420-ec9dc9cb-2af6a9bb-dcbd180e"]'
```

The list of the resources to be sent are given as a JSON array. In this case, a single DICOM connection is used. This approach may dramatically improve the performance of auto-routing. [Sample code is available](https://code.google.com/p/orthanc/source/browse/Resources/Samples/Python/HighPerformanceAutoRouting.py).