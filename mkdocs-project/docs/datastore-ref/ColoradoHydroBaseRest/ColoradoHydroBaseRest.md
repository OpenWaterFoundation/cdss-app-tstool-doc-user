# TSTool / Datastore Reference / ColoradoHydroBaseRest #

**This documentation is under development.  An attempt is being made to keep new nomenclature consistent
with direct HydroBase use and legacy datastores,
while also leveraging new web service features.**

* [Overview](#overview)
* [Standard Time Series Properties](#standard-time-series-properties)
* [Limitations](#limitations)
* [Datastore Configuration File](#datastore-configuration-file)

--------------------

## Overview ##

The State of Colorado’s HydroBase database is the primary database for water data in Colorado.
However, using the HydroBase datastore or input type in TSTool (see the [HydroBase Datastore appendix](../CO-HydroBase/CO-HydroBase)
requires a direct connection to the database,
and a local installation of the database may not be available.
The ColoradoHydroBaseRest datastore provides internet REST web service access to historical and real-time data and is described here:

* [Colorado Division of Water Resources REST Web Services Help](https://dnrweb.state.co.us/DWR/DwrApiService/Help)
* Legacy datastores:
	+ [Colorado Water HBGuest Datastore appendix](../ColoradoWaterHBGuest/ColoradoWaterHBGuest)
	+ [Colorado Water SMS Datastore appendix](../ColoradoWaterSMS/ColoradoWaterSMS)
* Legacy web service documentation that may be useful:
	+ [Colorado HBGuest Web Service Documentation](http://www.dwr.state.co.us/HBGuest/Documents/ColoradoHBGuestWebService.pdf)
	+ [HBGuest Web Service Data Dictionary](http://www.dwr.state.co.us/HBGuest/Documents/HBGuestWebService Objects.pdf)
	+ [HBGuest Web Services landing page](http://www.dwr.state.co.us/HBGuest/default.aspx)

The ColoradoHydroBaseRest datastore is under development and uses new DWR HydroBase web services.
It is expected that a number of iterations will be needed to align TSTool requirements with web service features.

## Web Service to Time Series Mapping ##

Time series data objects in TSTool consist of various properties such as location identifier, and units,
and data arrays for data values and flags.
To populate this data typically requires joining a station's or structure's data with the time series for that location.
The TSTool main interface browsing tool requires a join of information in order to list time series for selection.
Once the data type and interval are picked, specific time series are listed.
The following table summarizes DWR web service and use this datastore.

| **Web Service**              |**Time Series Data Types**|**TSTool HydroBase Data Type**| **Comment** |
|--|--|--|--|
| Administrative Calls         | Currently none.            | Could potentially treat historical calls as time series, but would need special handling.|
| Analysis Services            | Currently none.            | Not currently integrating into TSTool. |
| Dam Safety                   | Currently none.            | Not currently integrating into TSTool. |
| Groundwater Geophysical Logs | Currently none.            | Not currently integrating into TSTool. |
| Groundwater Levels           | See<br><ul><li>***Well - WaterLevelDepth***</li><li>***Well - WaterLevelElev***</li></ul> data type below. | Comparable TSTool TSIDs are `402930104414301.DWR.WellLevelDepth.Day~ColoradoWaterHBGuest` and `402930104414301.DWR.WellLevelElev.Day~ColoradoWaterHBGuest`.  **Must use point graph to see data points since sparce.**|
| NGWMN                        | Need to review.            | Not currently integrating into TSTool - don't know what this is. |
| Parcel Use TS                | Need to review.            | Under evaluation. |
| Structures                   | See<br><ul><li>***Diversion - DivTotal***</li><li>***Diversion - DivClass***</li></ul> data type below.| Comparable TSTool TSIDs are `0300905.DWR.DivTotal.Month~ColoradoWaterHBGuest` and `0300503.DWR.DivClass-S:6 F:0300934 U:Q T:0 G:.Month~HydroBase`. **Note that additional work is needed in TSTool to support new diversion classes with account and To**. |
| Telemetry Stations           | Is this only DWR SMS stations or also historical? | Under evaluation. |
| Water Rights                 | Currently none.            | Not currently integrating into TSTool.  Could treat water rights as time series to allow accumulation. |
| Well Permits                 | Currently none.            | Not currently integrating into TSTool. |

Any of the webs services can be called using TSTool `WebGet` command, which retrieves the result of a URL and saves to a local file.
If the web service supports CSV, the resulting file can be read using commands such as
`ReadTableFromDelimitedFile` (table) and `ReadDelimitedFile` (time series).
Consequently, there is always a way to use the web services, but it is not as tightly integrated into TSTool.

## Standard Time Series Properties ##

The general form of time series identifier used by TSTool is:

```
Location.DataSource.DataType.Interval~DataStoreName
```

The standard time series identifier format for ColoradoHydroBaseRest time series is of the form:

```
Location.DataSource.DataType.Interval~ColoradoHydroBaseRest
```

The meaning of the parts is as follows (**this information is being updated as web service are implemented**):

* The `Location` is set to one of the following depending on the measurement location type:
	+ the State of Colorado’s water district identifier (WDID) for structures.
	+ the State of Colorado’s abbreviation for telemetered stations
	+ **Question:  Do telemetered stations only include SMS stations?  How can historical time series such as climate data be retrieved?**
* The `DataSource` is set to the providing agency (e.g., `DWR` for diversion data).
	+ **Issue:  Currently evaluating how to handle agency name with punctuation.  Abbreviation would be useful.**
* The `DataType` is set to the “measurement type” described in the State’s web service
documentation (e.g., `DivTotal` for total diversions through a structure).
Refer to the HydroBase Input Type appendix for a full list of time series data available in HydroBase.
In some cases, the data type used by TSTool will not exactly match HydroBase.
For example, TSTool uses `ResMeasStorage` and HydroBase uses `ResMeas` to indicate reservoir measurements,
which can contain several observations.
The additional “Storage” is needed in TSTool to uniquely identify the time series for the specific data type.
	+ **Issue:  Can these human-readable types be determined from all web services?  Don't really want to use camelCase column names from returned resources and diversion classes are too complicated.**
* `Interval` is `Day`, `Month`, or `Year`, as requested.
The interval string is converted from HydroBase conventions of `Daily`, `Monthly`, `Annual`,
and `Random` (monthly and annual diversion data are stored together in HydroBase and are identified as Annual data).
	+ **Issue:  Need to confirm real-time data interval.  Is it always 15min or should be irregular?**
* The datastore name (`ColoradoHydroBaseRest` by default) indicates that the data
are being read from the ColoradoHydroBaseRest web service.

## Limitations ##

The following limitations of the web service may impact users of the data.

* Data type – The goal of the datastore is to provide access to all time series
that are available in HydroBase.  However, data access is limited by features of the
web services.
The following data types were implemented in the legacy [ColoradoWaterHBGuest web services](../ColoradoWaterHBGuest/ColoradoWaterHBGuest), for all available intervals.
Additional data types will be supported in the future (see the
[HydroBase Datastore appendix](../CO-HydroBase/CO-HydroBase)
for a complete list of available data types).
Note that creating a complete diversion time series,
in particular for structures that are not frequently measured,
may require considering several data types, including `DivTotal` (total through headgate),
`IDivTotal` (infrequent measurement), and `DivComment` (indicates when ditches are out of service,
not currently implemented in this datastore).
The following list of data types does not completely match data that are available in HydroBase,
but additional data types will be enabled when they become available in the web services.
* Time series metadata – some metadata such as units and measurement counts
currently may not available from the web service.
This information will be displayed as blank in the time series listing.
* Roundoff – a comparison of data values read directly from HydroBase and from the web
service may show very slight differences when values are rounded.
This is due to numbers being read and formatted during processing.  The differences should not be large enough to significantly impact final results.
* Performance – time series metadata (lists of location/data type/interval combinations)
are retrievable from the web service by water district, water division, and single entry.
In order for TSTool to provide the user with “drill down”
capability starting with a full list of available data,
it is necessary to request blocks of data from the web service.
However, requesting too large a block results in performance problems
due to the bandwidth necessary to transmit data across the network.
Consequently, TSTool utilizes caching to store lists of time series metadata,
grouped by water district, data type, and interval. The cache is populated based on user requests.
Consequently, the first time that data are requested for a district,
performance will be slower while the data are retrieved.
Subsequent listing of the time series should be fast.
Time series data are not currently cached and therefore there may be noticeable slowing for large queries.
Additional optimization of data transfer will be evaluated as web service use increases.

|**ColoradoWaterHBGuest web service data type**|**Available in ColoradoHydroBaseREST web service?**|**Comments**|
|--|--|--|
|`AdminFlow – AdminFlow`|Not yet||
|`Agriculture/GIS – CropAreaAllIrrigation`|Not yet||
|`Agriculture/GIS – CropAreaDrip`|Not yet||
|`Agriculture/GIS – CropAreaFlood`|Not yet||
|`Agriculture/GIS – CropAreaFurrow`|Not yet||
|`Agriculture/GIS – CropAreaSprinkler`|Not yet||
|`Climate – EvapPan`|Not yet||
|`Climate – FrostDate`|Not yet||
|`Climate – Precip`|Not yet||
|`Climate – Snow`|Not yet||
|`Climate – SnowCourseDepth`|Not yet||
|`Climate - SnowCourseSWE`|Not yet||
|`Climate - Solar`|Not yet||
|`Climate – TempMax`|Not yet||
|`Climate – TempMean`|Not yet||
|`Climate - TempMin`|Not yet||
|`Climate – VaporPressure`|Not yet||
|`Climate - Wind`|Not yet||
|`Diversion - DivClass` **Not available in ColordoWaterHBGuest** | Not yet.| **Being evaluated**.|
|`Diversion - DivTotal`|Not yet| **Being implemented**.|
|`Diversion - IDivTotal`|Not yet||
|`Reservoir - RelTotal`|Not yet| **Being implemented**.|
|`Reservoir - IRelTotal`|Not yet||
|`Reservoir - ResEOM` – however, no data are returned from the service|Not yet||
|`Reservoir - ResEOY` – however, no data are returned from the service|Not yet||
|`Reservoir - ResMeasElev` – treated as a daily time series with many gaps|Not yet||
|`Reservoir - ResMeasEvap` – treated as a daily time series with many gaps, not sure if daily is appropriate|Not yet||
|`Reservoir - ResMeasFill` – treated as a daily time series with many gaps, not sure if daily is appropriate|Not yet||
|`Reservoir - ResMeasRelease` – treated as a daily time series with many gaps, not sure if daily is appropriate|Not yet||
|`Reservoir - ResMeasStorage` – treated as a daily time series with many gaps|Not yet||
|`Stream - Streamflow`|Not yet||
|`Well - WellLevelElev` – treated as a daily time series with many gaps|Not yet||
|`Well - WellLevelDepth` – treated as a daily time series with many gaps|Not yet||

## Datastore Configuration File ##

A datastore is configured by enabling ColoradoWaterHBGuest datastores in the main `TSTool.cfg` configuration file,
and creating a datastore configuration file for each datastore connection.
Configurations are processed at software startup.
An example of the TSTool configuration file is shown below.
Multiple datastores can be defined using the `[DataStore:DataStoreName]` syntax.
For ColoradoWaterHBGuest, this would allow, for example, accessing different versions of the web services.

```
# Configuration file for TSTool

[TSTool]
ColoradoHydroBaseRestEnabled = true
```

**<p style="text-align: center;">
TSTool Configuration File with ColoradHydroBaseRest Datastore Properties
</p>**

Properties for each datastore are specified in an accompanying configuration file described below.

The following illustrates the ColoradoHydroBaseRest datastore configuration file format,
which in this example is located in the same folder as the TSTool
configuration file and configures a datastore named `ColoradoHydroBaseRest`.

```
# Configuration information for "ColoradoHydroBaseRest" web service datastore.
# Properties are:
#
# The user will see the following when interacting with the datastore:
#
# Name - datastore identifier used in applications, for example as the
#     input type information for time series identifiers (usually a short string)
# Description - datastore description for reports and user interfaces (short phrase)
# ServiceRootURI - web service root URI, including the server name and root path

Type = "ColoradoHydroBaseRestDataStore"
Name = "ColoradoHydroBaseRest"
Description = "Colorado HydroBase REST Web Service"
ServiceRootURI = "https://dwrweb.state.co.us/DWR/DwrApiService/api/v2"
```

**<p style="text-align: center;">
ColoradoHydroBase Datastore Configuration File
</p>**