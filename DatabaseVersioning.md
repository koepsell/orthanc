# Database Versioning in Orthanc #

Orthanc stores the index of the DICOM instances as an embedded [SQLite database](http://www.sqlite.org/). The schema of this database has evolved across the versions of Orthanc, making the database incompatible between versions. This page clarifies which versions of Orthanc are compatible with other versions.

## Recent Versions (post-0.3.1) ##

Recent versions of Orthanc (starting 0.3.1, inclusive) include information about the version of the DB schema in the SQLite table `GlobalProperties` under the property with index `1`. Versions with the same version of the DB schema are compatible with each other.

When some version of Orthanc starts up, it checks whether it is compatible with the database version. Orthanc will **fail to start if it is not compatible with the database version**. Here is the compatibility matrix:

| | **DB v2** | **DB v3** | **DB v4** | **DB v5** |
|:|:----------|:----------|:----------|:----------|
| **Mainline**      |   | u | u | x |
| **Orthanc 0.8.5** - **Orthanc 0.8.6** |   | u | u | x |
| **Orthanc 0.7.3** - **Orthanc 0.8.4** |   | u | x |   |
| **Orthanc 0.4.0** - **Orthanc 0.7.2** |   | x |   |   |
| **Orthanc 0.3.1** | x |   |   |   |

_Note:_ "u" means that an upgrade of the DB schema will automatically take place.

## Early Versions (pre-0.3.0) ##

Early versions of Orthanc (up to version 0.3.0, inclusive) **do not check the version** of the database schema. Because of this, these early versions are incompatible with all the other versions. Pay attention to the fact that no compatibility check is done in these versions, which may result in a corrupted database.