Ocupar estos 2 scripts:
#### lote.sh

```bash

down_ds084.1.sh 2019 04 15 && sleep 1200
down_ds084.1.sh 2019 04 16 && sleep 1200
down_ds084.1.sh 2019 04 17 && sleep 1200
down_ds084.1.sh 2019 04 18 && sleep 1200
down_ds084.1.sh 2019 04 19 && sleep 1200
down_ds084.1.sh 2019 04 20 && sleep 1200
down_ds084.1.sh 2019 04 21 && sleep 1200
down_ds084.1.sh 2019 04 22 && sleep 1200
echo "fin script descargas"

```
#### down_ds084.1.sh

Este script (en Cshell, no en bash) descarga específicamente los datos de las 12UTC

```bash
#!/bin/csh
#################################################################
# Csh Script to retrieve 93 online Data files of 'ds084.1',
# total 31.93G. This script uses 'wget' to download data.
#
# Highlight this script by Select All, Copy and Paste it into a file;
# make the file executable and run it on command line.
#
# You need pass in your password as a parameter to execute
# this script; or you can set an environment variable RDAPSWD
# if your Operating System supports it.
#
# Contact rpconroy@ucar.edu (Riley Conroy) for further assistance.
#################################################################

if ($#argv < 3 ) then 
	echo "args: yyyy mm dd"
	exit 1
endif




set yyyy=$1
set mm=$2
set dd=$3

mkdir -p ds084.1/${yyyy}${mm}${dd}
cd ds084.1/${yyyy}${mm}${dd}

set pswd = $RDAPSWD

set v = `wget -V |grep 'GNU Wget ' | cut -d ' ' -f 3`
set a = `echo $v | cut -d '.' -f 1`
set b = `echo $v | cut -d '.' -f 2`
if(100 * $a + $b > 109) then
 set opt = 'wget -c --limit-rate=20000k  --no-check-certificate '
else
 set opt = 'wget -c --limit-rate=20000k '
endif
set opt1 = '-O Authentication.log --save-cookies auth.rda_ucar_edu --post-data'
set opt2 = "email=pablo.salinas@ceaza.cl&passwd=$pswd&action=login"
$opt $opt1="$opt2" https://rda.ucar.edu/cgi-bin/login
set opt1 = "-N --load-cookies auth.rda_ucar_edu"
#set opt2 = "$opt $opt1 https://rda.ucar.edu/data/ds084.1/"
set opt2 = "$opt $opt1 https://data.rda.ucar.edu/ds084.1/"
set filelist = ( \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f000.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f003.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f006.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f009.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f012.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f015.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f018.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f021.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f024.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f027.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f030.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f033.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f036.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f039.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f042.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f045.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f048.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f051.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f054.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f057.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f060.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f063.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f066.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f069.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f072.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f075.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f078.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f081.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f084.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f087.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f090.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f093.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f096.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f099.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f102.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f105.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f108.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f111.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f114.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f117.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f120.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f123.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f126.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f129.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f132.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f135.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f138.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f141.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f144.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f147.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f150.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f153.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f156.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f159.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f162.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f165.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f168.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f171.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f174.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f177.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f180.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f183.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f186.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f189.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f192.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f195.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f198.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f201.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f204.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f207.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f210.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f213.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f216.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f219.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f222.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f225.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f228.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f231.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f234.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f237.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f240.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f252.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f264.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f276.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f288.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f300.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f312.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f324.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f336.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f348.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f360.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f372.grib2 \
  ${yyyy}/${yyyy}${mm}${dd}/gfs.0p25.${yyyy}${mm}${dd}12.f384.grib2 \
)
while($#filelist > 0)
 set syscmd = "$opt2$filelist[1]"
 echo "$syscmd ..."
 $syscmd
 shift filelist
end

```


en cada llamada al 2do script, se crean las carpetas con la fecha, en el subdirectorio ds084.1/ que se crea en el directorio actual.





