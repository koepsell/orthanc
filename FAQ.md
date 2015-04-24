

# Troubleshooting #

  * **I cannot login to Orthanc Explorer**: For security reasons, access to Orthanc from remote hosts is disabled by default. Only the localhost is allowed to access Orthanc. You have to set the `RemoteAccessAllowed` option in the [configuration file](OrthancConfiguration.md) to `true`. It is then strongly advised to set `AuthenticationEnabled` to `true` and to add a user to the `RegisteredUsers` option, also in the configuration file.
  * **Orthanc Explorer is slow under Windows on the localhost**: You have to disable the IPv6 support. This is a Windows-specific problem that is discussed [here](http://superuser.com/questions/43823/google-chrome-is-slow-to-localhost) and [here](http://stackoverflow.com/questions/1726585/firefox-and-chrome-slow-on-localhost-known-fix-doesnt-work-on-windows-7).
  * Under Windows, Orthanc creates the "`OrthancStorage`" folder, and crashes with the error "**SQLite: Unable to open the database**": Your directory name is either too long, or it contains special characters. Please try and run Orthanc in a folder with a simple name such as "`C:\Orthanc`".
  * Orthanc does not compile with **Microsoft Visual Studio 2012**. Aaron Boxer has fixed the build: Apply [patch 1](https://groups.google.com/forum/?hl=fr#!topic/orthanc-users/M7sLKLitp7E) and [patch 2](https://groups.google.com/forum/?hl=fr#!topic/orthanc-users/kbYj_o-8gp4).

# Please explain the version numbering scheme of Orthanc #

Each release of Orthanc is identified by a version that is made of three parts: `API.MAJOR.MINOR` (e.g. 0.5.1).

  * `API` (currently, 0) changes when the REST API is refactored.
  * `MAJOR` changes when a new major feature is introduced (either in the REST API or in the DICOM support), when an incompatibility in the database schema is introduced, or when an important refactoring is achieved.
  * `MINOR` changes after each introduction of a minor feature, after a bugfix, or after a speed or GUI improvement. It also changes when an experimental feature is introduced.

# I use the Linux distribution XXX, how can I build Orthanc? #

  * **Orthanc >= 0.7.1**: See the instructions [inside the source package](https://code.google.com/p/orthanc/source/browse/LinuxCompilation.txt).
  * **Orthanc <= 0.7.0**: See [this Wiki page](LinuxCompilationUpTo070.md).

# I use Mac OS X, how can I build Orthanc? #

The mainline of Orthanc can compile under Apple OS X, with the XCode compiler, since June 24th, 2014. Build instructions [are available](https://code.google.com/p/orthanc/source/browse/DarwinCompilation.txt).

_Older answers:_
  * Cían Hughes has created a [fork of the Orthanc mainline](https://code.google.com/r/cian-orthanc-mac/) that is tested on Mavericks (10.9), using the [Homebrew](http://brew.sh/) package manager.
  * Ryan Walklin has created a [fork of Orthanc 0.5.2](https://code.google.com/r/ryanwalklin-mac/) that compiles and runs under Mac OS X.

# What are the Recycling/Protection/Compression Features? #

Because of its focus on low-end computers, Orthanc implements **disk space recycling**: The oldest series of images can be automatically deleted when the available disk space drops below a threshold, or when the number of stored series grows above a threshold. This enables the automated control of the disk space.

Recycling is controlled by the `MaximumStorageSize` and the `MaximumPatientCount` options in the [Orthanc configuration file](OrthancConfiguration.md).

It is possible to prevent important data from being automatically recycled. This mechanism is called **protection**. Each patient, each study and each series can be individually protected against recycling by using the `Unprotected/Protected` switch that is available in Orthanc Explorer.

If your disk space is limited, you should also consider using **disk space compression**. When compression is enabled, Orthanc compresses the incoming DICOM instances on-the-fly before writing them to the filesystem, using [zlib](http://en.wikipedia.org/wiki/Zlib). This is useful, because the images are often stored as raw,
uncompressed arrays in DICOM instances: The size of a typical DICOM
instance can hopefully be divided by a factor 2 through lossless
compression. This compression process is transparent to the user, as
Orthanc automatically decompresses the file before sending it back to
the external world.

Compression is controlled by the `StorageCompression` option in the [Orthanc configuration file](OrthancConfiguration.md).


# How can I upgrade the database version? #

Different versions of Orthanc might use a different database version, as explained on [this page](DatabaseVersioning.md). To **upgrade an existing database** for a newer version of Orthanc, you can use this [sample script](https://code.google.com/p/orthanc/source/browse/Resources/Samples/ImportDicomFiles/ImportDicomFiles.py). This script will import all the DICOM files of the existing database through the REST API of Orthanc. You just have to apply this script on the place where the older database is stored (presumably in the `OrthancStorage` folder, but this depends on your configuration). For instance:

```
# ImportDicomFiles.py localhost 8042 OrthancStorage
```


# How can I install Orthanc as a Debian/Ubuntu daemon? #

To install Orthanc as a Linux daemon on a Debian/Ubuntu system, you can:
  1. [Download this service script](http://anonscm.debian.org/viewvc/debian-med/trunk/packages/orthanc/trunk/debian/orthanc.init?view=markup) (this file is part of the official [Debian package of Orthanc](http://debian-med.alioth.debian.org/tasks/imaging#orthanc)),
  1. Adapt some of its variables to reflect the configuration of your system,
  1. Copy it in `/etc/init.d` as root (the filename cannot contain dot, otherwise it is not executed), make it belong to root, and tag it as executable:
```
# sudo mv orthanc.init /etc/init.d/orthanc
# sudo chown root:root /etc/init.d/orthanc
# sudo chmod 755 /etc/init.d/orthanc
```
  1. If you wish the daemon to be automatically launched at boot time and stopped at shutdown:
```
# sudo update-rc.d orthanc defaults
```
  1. If you wish to remove the automatic launching at boot time later on:
```
# sudo update-rc.d -f orthanc remove
```

_Note:_ You can use `rcconf` to easily monitor the services that are run at startup (`sudo apt-get install rcconf`).


# How can I access an Orthanc instance that is running inside a Fedora/RHEL/CentOS box? #

For remote access to Orthanc, you will have to open the 4242 and the 8042 ports on `iptables`:

```
# sudo iptables -I INPUT -p tcp --dport 8042 -j ACCEPT
# sudo iptables -I INPUT -p tcp --dport 4242 -j ACCEPT
# sudo iptables-save 
```


# How can I run Orthanc behind Apache? #

It is possible to make Orthanc run behind Apache through a [reverse proxy](https://en.wikipedia.org/wiki/Reverse_proxy). To map the REST API of an Orthanc server listening on the port 8000 on the URI `/Orthanc`, paste the following code in your `/etc/apache2/httpd.conf`:

```
LoadModule proxy_module /usr/lib/apache2/modules/mod_proxy.so
LoadModule proxy_http_module /usr/lib/apache2/modules/mod_proxy_http.so
ProxyRequests On
ProxyPass /Orthanc/ http://localhost:8000/ retry=0
```

_Note:_ These instructions are for Ubuntu 11.10. You perhaps have to adapt the absolute paths above to your distribution.

# How can I run Orthanc behind nginx? #

Similarly to Apache, [Orthanc is reported](https://groups.google.com/d/msg/orthanc-users/oTMCM6kElfw/uj0r062mptoJ) to be able to run as a [reverse proxy](https://en.wikipedia.org/wiki/Reverse_proxy) behind nginx. Here is the configuration snippet for nginx:

```
server{
       listen  80  default_server;
       ...
       location  /orthanc/  {
                proxy_pass http://localhost:8042;
		proxy_set_header HOST $host;
                proxy_set_header X-Real-IP $remote_addr;
                rewrite /orthanc(.*) $1 break;
        }
        ...
}
```

_Note:_ Thanks to Qaler for submitting this information!


# How can I use HTTPS (SSL encryption) with Orthanc? #

You need to obtain a [X.509 certificate](http://en.wikipedia.org/wiki/X.509) in the [PEM format](http://en.wikipedia.org/wiki/X.509#Certificate_filename_extensions), and prepend its content with your private key. You can then modify the `SslEnabled` and `SslCertificate` variables in the [Orthanc configuration file](OrthancConfiguration.md).

Here are simple instructions to create a self-signed SSL certificate that is suitable for test environments with the [OpenSSL](http://en.wikipedia.org/wiki/Openssl) command-line tools :

```
# openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout private.key -out certificate.crt
# cat private.key certificate.crt > certificate.pem
```

Some interesting references about this topic can be found [here](http://devsec.org/info/ssl-cert.html), [here](http://www.akadia.com/services/ssh_test_certificate.html), and [here](http://stackoverflow.com/questions/991758/how-to-get-an-openssl-pem-file-from-key-and-crt-files).

# Orthanc Fails to Store a DICOM File, Complaining About an "Inexistent Tag" #

If you receive the `"Exception while storing DICOM: Inexistent tag"` error while [storing a DICOM instance into Orthanc](StoringInstances.md), please make sure that all the 4 following tags do exist in the DICOM file:

  * `PatientID` (0010,0020),
  * `StudyInstanceUID` (0020,000d),
  * `SeriesInstanceUID` (0020,000e),
  * `SOPInstanceUID` (0008,0018).

These tags are all used to index the incoming DICOM instances. In Orthanc, each patient, study, series and instance is indeed assigned with an unique identifier that is computed as the [SHA-1 hash](http://en.wikipedia.org/wiki/Sha-1) of a combination of these four tags.

The error message is more explicit starting with Orthanc 0.7.4.

_Note:_ The actual implementation of the hashing is carried on by the [DicomInstanceHasher class](https://code.google.com/p/orthanc/source/browse/Core/DicomFormat/DicomInstanceHasher.cpp?name=Orthanc-0.6.0).

# Orthanc Does not Compile, Complaining About Missing `uuid-dev` Package #

This problem seems to occur when fist building Orthanc without the `uuid-dev` package installed, then installing `uuid-dev`, then rebuilding Orthanc. It seems that the build scripts do not update the cached variable about the presence of `uuid-dev`.

To solve this problem, as reported by [Peter Somlo](https://groups.google.com/d/msg/orthanc-users/hQYulBBvJvs/S1Pm125o59gJ), it is necessary to entirely remove the build directory (e.g. with `rm -rf Build`) and start again the build from a fresh directory.


# How Can I Run a C-Find SCU Query (experimental)? #

An _experimental_ C-Find SCU support is implemented in Orthanc. This is a three-step API:

  1. Retrieve the PatientID:
```
# curl http://localhost:8042/modalities/pacs/find-patient -X POST -d '{"PatientName":"JOD*","PatientSex":"M"}'
```
  1. Retrieve the studies of this patient (using the "PatientID" returned from Step 1):
```
# curl http://localhost:8042/modalities/pacs/find-study -X POST -d '{"PatientID":"0555643F"}'
```
  1. Retrieve the series of one study (using the "PatientID" from Step 1, and the "StudyInstanceUID" from Step 2):
```
# curl http://localhost:8042/modalities/pacs/find-series -X POST -d '{"PatientID":"0555643F","StudyInstanceUID":"1.2.840.113704.1.111.2768.1239195678.57"}' 
```

You will have to define the modality "pacs" in the [configuration file](OrthancConfiguration.md) of Orthanc (under the section `DicomModalities`).

Please however note that this unstable API will most likely change in
future versions of Orthanc.

# Using Orthanc to Ease WADO Querying #

As of Orthanc 0.6.1, it will be possible to use Orthanc to easily gather the three identifiers that are required to run a [WADO query](ftp://medical.nema.org/medical/dicom/2006/06_18pu.pdf) against a remote modality (without storing the files inside Orthanc). These identifiers are:

  * `StudyInstanceUID` (0020,000d),
  * `SeriesInstanceUID` (0020,000e),
  * `ObjectUID`, that exactly corresponds to the `SOPInstanceUID` tag (0008,0018) (cf. the [WADO specification](ftp://medical.nema.org/medical/dicom/2006/06_18pu.pdf), Section 8.1.4).

The trick consists in using the experimental C-Find SCU API, going down to the instance level:

```
# curl http://localhost:8042/modalities/pacs/find-patient -X POST -d '{"PatientName":"JOD*","PatientSex":"M"}'
# curl http://localhost:8042/modalities/pacs/find-study -X POST -d '{"PatientID":"0555643F"}'
# curl http://localhost:8042/modalities/pacs/find-series -X POST -d '{"PatientID":"0555643F","StudyInstanceUID":"1.2.840.113704.1.111.2768.1239195678.57"}' 
# curl http://localhost:8042/modalities/pacs/find-instance -X POST -d '{"PatientID":"0555643F","StudyInstanceUID":"1.2.840.113704.1.111.2768.1239195678.57","SeriesInstanceUID":"1.3.46.670589.28.2.7.2200939417.2.13493.0.1239199523"}'
```

The first three steps are described in [this FAQ entry](https://code.google.com/p/orthanc/wiki/FAQ#How_Can_I_Run_a_C-Find_SCU_Query_(experimental)?). The fourth step retrieves the list of the instances of the series. The latter query was not possible until Orthanc 0.6.1.

As a result of this sequence of four commands, the `StudyInstanceUID`, `SeriesInstanceUID` and `SOPInstanceUID` are readily available for each instance of the series.

_Warning:_ The C-Find SCU API is still experimental and might change in future versions of Orthanc.

# Configuring DICOM Query/Retrieve #

Starting with release 0.7.0, Orthanc supports DICOM Query/Retrieve (i.e. Orthanc can act as C-Find SCP and C-Move SCP). To configure this feature, follow these instructions:

  * Get the AET (Application Entity Title), the IP address and the port number of your DICOM client.
  * Add an entry in the `DicomModalities` section of your Orthanc [configuration file](OrthancConfiguration.md) to reflect these settings.
  * Restart Orthanc with the updated configuration file.
  * Symmetrically, in your DICOM client, configure an additional DICOM node corresponding to your Orthanc AET (see the `DicomAet` parameter of your Orthanc configuration, by default, `ORTHANC`), IP address et port number (cf. `DicomPort`, by default 4242).

# How to Backup Orthanc #

To backup an instance of Orthanc, you need to get a copy of 3 elements:
  1. Your configuration file.
  1. The DICOM files (by default, the `OrthancStorage` subdirectories).
  1. The SQLite index (by default, the `OrthancStorage/index*` files).

Karsten Hilbert provided us with [a sample backup script](https://code.google.com/p/orthanc/source/browse/Contributions/backup-orthanc.sh?repo=wiki) for the official Debian package of Orthanc.

In this script, the call to the SQLite command-line tool is used to force the [WAL replay](http://www.sqlite.org/wal.html). This manual replay will not be necessary for Orthanc >= 0.7.3.


# Why "Orthanc"? #

The spelling "[Orthanc](http://en.wikipedia.org/wiki/Orthanc#Orthanc)" originates from J.R.R. Tolkien’s work. Orthanc is the black tower of Isengard that houses one of the **palantíri**. A [palantír](http://en.wikipedia.org/wiki/Palant%C3%ADr) is a spherical stone whose name literally means "One that Sees from Afar". When one looks into a palantír, one can communicate with other such stones and anyone who might be looking into them. This is quite similar to the Orthanc server, which is designed for a simple, transparent, programmatic access to medical images in the entire hospital DICOM topology.

"Orthanc" also contains the trigram "RTH", standing for **Radiotherapy**. This is a reference to the fact that Orthanc originates from the research of the [Radiotherapy Service](http://www.chu.ulg.ac.be/jcms/c_13318/radiotherapie) of the [CHU of Liège](http://www.chu.ulg.ac.be/).