# Configuration of Orthanc #

Configuring Orthanc simply consists in copying and adapting the [default configuration file](http://orthanc.googlecode.com/hg-history/Orthanc-0.8.6/Resources/Configuration.json). This file is in the JSON file format. As of Orthanc 0.3.0, you can generate a sample configuration file with the following call:

```
# Orthanc --config=Configuration.json
```

Then, start Orthanc by giving it the path to the modified `Configuration.json` path as a command-line argument:

```
# Orthanc ./Configuration.json
```

The default configuration file would:

  * Create a DICOM server with the DICOM AET (Application Entity Title) `ORTHANC` that listens on the port 4242.
  * Create a HTTP server for the REST API that listens on the port 8042.
  * Store the Orthanc database in a folder called `OrthancStorage`.

To obtain more diagnostic, you can use the `--verbose` or the `--trace` options:

```
# Orthanc ./Configuration.json --verbose
# Orthanc ./Configuration.json --trace
```