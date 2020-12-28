
related link: https://www.nytimes.com/interactive/2020/09/18/opinion/wildfire-hurricane-climate.html


---

Data found via:
https://www.climate.gov/maps-data/dataset/daily-temperature-and-precipitation-reports-data-tables

Downloaded on 2020-07-15:

ftp://ftp.ncdc.noaa.gov/pub/data/ghcn/daily/by_year/2019.csv.gz
ftp://ftp.ncdc.noaa.gov/pub/data/ghcn/daily/by_year/ghcn-daily-by_year-format.rtf

```bash
gunzip 2019.csv.gz
md5 2019.csv
## MD5 (2019.csv) = d5b1fbeda6962341efff7fc48bcaa958
wc 2019.csv
## 34188762 34188762 1196914814 2019.csv
```

Cool, 34M rows.

Text from ghcn-daily-by_year-format.rtf:

```
The following information serves as a definition of each field in one
line of data covering one station-day. Each field described below is
separated by a comma ( , ) and follows the order presented in this
document.

ID = 11 character station identification code
YEAR/MONTH/DAY = 8 character date in YYYYMMDD format (e.g. 19860529 = May 29, 1986)
ELEMENT = 4 character indicator of element type
DATA VALUE = 5 character data value for ELEMENT
M-FLAG = 1 character Measurement Flag
Q-FLAG = 1 character Quality Flag
S-FLAG = 1 character Source Flag
OBS-TIME = 4-character time of observation in hour-minute format (i.e.
           0700 =7:00 am)

See section III of the GHCN-Daily readme.txt file for an explanation
of ELEMENT codes and their units as well as the M-FLAG, Q-FLAGS and
S-FLAGS.

The OBS-TIME field is populated with the observation times contained
in NOAA/NCDC’s Multinetwork Metadata System (MMS).
```

Downloading:

ftp://ftp.ncdc.noaa.gov/pub/data/ghcn/daily/by_year/readme.txt

Got "Version 3.26." And it has info on how to cite this data.

```bash
mv ~/Downloads/readme.txt ./ghcn-readme.txt
```

Okay... Simple stats!

```bash
# count the unique weather stations
time cut -d, -f1 2019.csv | sort | uniq | wc
##    39602   39602  475224
## real    2m46.411s

# count the unique dates
time cut -d, -f2 2019.csv | sort | uniq | wc
##      365     365    3285
## real    1m35.366s
# Good!
# Suggests 34188762 / 39602 / 365 = 2.4 rows per station day?
# Surely not?

# count the unique "elements"
time cut -d, -f3 2019.csv | sort | uniq | wc
##       70      70     350
## real    2m38.814s

# count the unique data values
time cut -d, -f4 2019.csv | sort | uniq | wc
##     4455    4455   21905
## real    3m48.674

# count the unique measurement flags (M-FLAGs)
time cut -d, -f5 2019.csv | sort | uniq | wc
##        5       4       9
## real    1m10.871s

# count the unique quality flags (Q-FLAGs)
time cut -d, -f6 2019.csv | sort | uniq | wc
##       15      14      29
## real    1m5.142s

# count the unique source flags (S-FLAGs)
time cut -d, -f7 2019.csv | sort | uniq | wc
##       12      12      24
## real    1m18.996s

# count the unique OBS-TIMEs
time cut -d, -f8 2019.csv | sort | uniq | wc
##       40      39     196
## real    1m14.728s

# anything in a "field 9"?
time cut -d, -f9 2019.csv | sort | uniq | wc
##        1       0       1
## real    0m44.193s
time cut -d, -f9 2019.csv | sort | uniq -c
## 34188762
## real    0m43.326s
time cut -d, -f10 2019.csv | sort | uniq -c
## 34188762
## real    0m48.951s
```

Tables!

```bash
# table of 70 unique "elements"
time cut -d, -f3 2019.csv | sort | uniq -c | sort -rh
## 10360416 PRCP
## 4414862 TMIN
## 4374247 TMAX
## 4201205 SNOW
## 3122707 SNWD
## 2307503 TAVG
## 1728384 TOBS
## 459047 WESD
## 361753 AWND
## 334625 WSF2
## 334506 WDF2
## 331469 WSF5
## 331425 WDF5
## 272900 WESF
## 154984 WT01
## 152689 WSFG
## 145615 WDFG
## 109630 DAPR
## 108680 MDPR
## 91107 PGTM
## 64920 WT03
## 49543 SX32
## 49207 SN32
## 46274 EVAP
## 37386 WT08
## 34952 WDMV
## 22068 MXPN
## 21608 MNPN
## 19480 SX52
## 19100 SN52
## 18921 WT02
## 13595 WSFI
## 13109 AWDR
## 7560 DWPR
## 7398 WT06
## 5869 SX31
## 5869 SN31
## 4597 SN33
## 4593 SX33
## 4444 WT04
## 4315 WT11
## 2741 WT05
## 2693 MDTN
## 2693 DATN
## 2647 MDTX
## 2647 DATX
## 2547 SX53
## 2547 SN53
## 2006 SN51
## 2004 SX51
## 1806 SX35
## 1440 SN35
## 1290 PSUN
## 1286 TSUN
## 1180 SX55
## 1148 SN55
## 1089 SX36
## 1078 WT09
##  971 THIC
##  757 SX56
##  726 SN56
##  723 SN36
##   91 WT07
##   31 SX57
##   27 WT10
##   21 WT16
##    4 MDSF
##    4 DASF
##    2 WT18
##    1 WT15
##
## real    2m27.033s
```

Okay well TMIN and TMAX have got to be what I'm looking for, right?

From the readme:

```bash
The five core elements are:
    PRCP = Precipitation (tenths of mm)
    SNOW = Snowfall (mm)
    SNWD = Snow depth (mm)
    TMAX = Maximum temperature (tenths of degrees C)
    TMIN = Minimum temperature (tenths of degrees C)
```

Taking a look...

```bash
grep TMIN 2019.csv | head -3
## AE000041196,20190101,TMIN,140,,,S,
## AEM00041194,20190101,TMIN,185,,,S,
## AEM00041217,20190101,TMIN,163,,,S,
```

Measurement flags (field 5) from readme:

```
    Blank = no measurement information applicable
    B     = precipitation total formed from two 12-hour totals
    D     = precipitation total formed from four six-hour totals
    H     = represents highest or lowest hourly temperature (TMAX or TMIN) 
            or the average of hourly values (TAVG)
    K     = converted from knots
    L     = temperature appears to be lagged with respect to reported
            hour of observation
    O     = converted from oktas
    P     = identified as "missing presumed zero" in DSI 3200 and 3206
    T     = trace of precipitation, snowfall, or snow depth
    W     = converted from 16-point WBAN code (for wind direction)
```

Quality flags (field 6) from readme:

```
    Blank = did not fail any quality assurance check
    D     = failed duplicate check
    G     = failed gap check
    I     = failed internal consistency check
    K     = failed streak/frequent-value check
    L     = failed check on length of multiday period
    M     = failed megaconsistency check
    N     = failed naught check
    O     = failed climatological outlier check
    R     = failed lagged range check
    S     = failed spatial consistency check
    T     = failed temporal consistency check
    W     = temperature too warm for snow
    X     = failed bounds check
    Z     = flagged as a result of an official Datzilla
            investigation
```

What does that "S" mean?

```bash
time cut -d, -f7 2019.csv | sort | uniq -c | sort -rh
## 10198314 7  # US cooperative summary of the day
## 7791663 N   # Community Collaborative Rain, Hail,and Snow (CoCoRaHS)
## 3308803 S   # Global Summary of the Day (NCDC DSI-9618)
## 3229521 W   # WBAN/ASOS Summary of the Day from NCDC's ISD
## 2045245 E   # European Climate Assessment and Dataset
## 1996312 a   # Australian data from the Australian Bureau of Meteorology
## 1966640 T   # SNOwpack TELemtry (SNOTEL) data
## 1902334 C   # Environment Canada
## 1334856 U   # Remote Automatic Weather Station (RAWS) data
## 244630 H    # High Plains Regional Climate Center real-time data
## 167829 R    # All-Russian Research Institute
## 2615 Z      # Datzilla official additions or replacements
##
## real    1m1.723s
```

Source flags (field 7) from readme:

```
    Blank = No source (i.e., data value missing)
    0     = U.S. Cooperative Summary of the Day (NCDC DSI-3200)
    6     = CDMP Cooperative Summary of the Day (NCDC DSI-3206)
    7     = U.S. Cooperative Summary of the Day -- Transmitted
            via WxCoder3 (NCDC DSI-3207)
    A     = U.S. Automated Surface Observing System (ASOS)
            real-time data (since January 1, 2006)
    a     = Australian data from the Australian Bureau of Meteorology
    B     = U.S. ASOS data for October 2000-December 2005 (NCDC
            DSI-3211)
    b     = Belarus update
    C     = Environment Canada
    D     = Short time delay US National Weather Service CF6 daily
            summaries provided by the High Plains Regional Climate
            Center
    E     = European Climate Assessment and Dataset (Klein Tank
            et al., 2002)
    F     = U.S. Fort data
    G     = Official Global Climate Observing System (GCOS) or
            other government-supplied data
    H     = High Plains Regional Climate Center real-time data
    I     = International collection (non U.S. data received through
            personal contacts)
    K     = U.S. Cooperative Summary of the Day data digitized from
            paper observer forms (from 2011 to present)
    M     = Monthly METAR Extract (additional ASOS data)
    m     = Data from the Mexican National Water Commission (Comision
            National del Agua -- CONAGUA)
    N     = Community Collaborative Rain, Hail,and Snow (CoCoRaHS)
    Q     = Data from several African countries that had been
            "quarantined", that is, withheld from public release
            until permission was granted from the respective
            meteorological services
    R     = NCEI Reference Network Database (Climate Reference Network
            and Regional Climate Reference Network)
    r     = All-Russian Research Institute of Hydrometeorological
            Information-World Data Center
    S     = Global Summary of the Day (NCDC DSI-9618)
            NOTE: "S" values are derived from hourly synoptic reports
            exchanged on the Global Telecommunications System (GTS).
            Daily values derived in this fashion may differ significantly
            from "true" daily data, particularly for precipitation
            (i.e., use with caution).
    s     = China Meteorological Administration/National Meteorological
            Information Center/Climatic Data Center (http://cdc.cma.gov.cn)
    T     = SNOwpack TELemtry (SNOTEL) data obtained from the U.S.
            Department of Agriculture's Natural Resources Conservation Service
    U     = Remote Automatic Weather Station (RAWS) data obtained
            from the Western Regional Climate Center
    u     = Ukraine update
    W     = WBAN/ASOS Summary of the Day from NCDC's Integrated
            Surface Data (ISD).
    X     = U.S. First-Order Summary of the Day (NCDC DSI-3210)
    Z     = Datzilla official additions or replacements
    z     = Uzbekistan update

    When data are available for the same time from more than one source,
    the highest priority source is chosen according to the following
    priority order (from highest to lowest):
    Z,R,D,0,6,C,X,W,K,7,F,B,M,m,r,E,z,u,b,s,a,G,Q,I,A,N,T,U,H,S
```

Okay really let's just split out the TMIN and TMAX stuff.

```bash
grep TMIN 2019.csv > 2019_TMIN.csv &
grep TMAX 2019.csv > 2019_TMAX.csv &
wc 2019_*
##  4374247 4374247 159624414 2019_TMAX.csv
##  4414862 4414862 160644176 2019_TMIN.csv
# Matches against earlier counts from field 7:
##  4374247 TMAX
##  4414862 TMIN

time cat 2019_TMIN.csv | cut -d, -f1 | sort | uniq -c | cut -c1-4 | sort -rh | uniq -c
## 5143  365
##  991  364
##  537  363
##  357  362
##  260  361
##  215  360
##  160  359
##  147  358
##  134  357
##  125  356
##   95  355
## ...

# Of this total number of statins
time cat 2019_TMIN.csv | cut -d, -f1 | sort | uniq -c | wc
##    13927   27854  236759

time cat 2019_TMAX.csv | cut -d, -f1 | sort | uniq -c | cut -c1-4 | sort -rh | uniq -c
## 5205  365
## 1006  364
##  531  363
##  369  362
##  277  361
##  217  360
##  187  359
##  158  358
##  141  357
##  128  356
##  108  355
##   85  354
## ...

time cat 2019_TMAX.csv | cut -d, -f1 | sort | uniq -c | wc
##    13861   27722  235637
```

Hmm hmm hmm... Could probably use just the complete data ones. But I
could also have _even more stations_ by including those with some
missing data!

What are my temperature bounds going to be?

I was thinking 40 and 80 degrees F, but now it's in C... I could
convert, I guess?

I'll just convert my targets, so 40 F is about 4 C (39.2 F)
and 80 F is about 27 C (80.6 F).

I can adjust these later too. Keep them as variables.

```python
cold, hot = 4, 27
```

Oh by the way: downloaded this station info, which has lat/lon.

ftp://ftp.ncdc.noaa.gov/pub/data/ghcn/daily/ghcnd-stations.txt

```bash
wc ghcnd-stations.txt
##   115082  816901 9897052 ghcnd-stations.txt
```

Lots of stations!

Is the TMIN/TMAX data all unique, as in just one reading per
station-day?

```bash
wc 2019_T*
##  4374247 4374247 159624414 2019_TMAX.csv
##  4414862 4414862 160644176 2019_TMIN.csv
cut -d, -f1-2 2019_TMAX.csv | sort | uniq | wc
##  4374247 4374247 91859187
cut -d, -f1-2 2019_TMIN.csv | sort | uniq | wc
##  4414862 4414862 92712102
```

Looks good! Don't have to worry about duplicate entries etc.!

Working in `analyze*.ipynb`...

I want more data!

Finding data via ftp://ftp.ncdc.noaa.gov/pub/data/ghcn/daily/

FTP site is being flaky...

Downloading on 2020-08-18:

 * ftp://ftp.ncdc.noaa.gov/pub/data/ghcn/daily/by_year/2018.csv.gz
 * ftp://ftp.ncdc.noaa.gov/pub/data/ghcn/daily/by_year/2017.csv.gz
 * ftp://ftp.ncdc.noaa.gov/pub/data/ghcn/daily/by_year/2016.csv.gz
 * and so on, through 2010

```bash
gunzip ~/Downloads/*gz
mv ~/Downloads/201?.csv ./
```

Ten years of weather data should fill in the map a little...


---

Returning to this on Monday 2020-12-28...

```bash
l ????.csv
## 2010.csv
## 2011.csv
## 2012.csv
## 2013.csv
## 2014.csv
## 2015.csv
## 2016.csv
## 2017.csv
## 2018.csv
## 2019.csv

# -h to avoid filenames
grep -h TMIN ????.csv > 201X_TMIN.csv &
grep -h TMAX ????.csv > 201X_TMAX.csv &
grep -h PRCP ????.csv > 201X_PRCP.csv &
```
