para fecha pasadas se debe ocupar GFS histórico (https://rda.ucar.edu/datasets/d084001/) como forzante.

El siguiente script de ejemplo, permite descargar el pronóstico GFS para la fecha 2024-06-04:


```bash

#!/usr/bin/env csh
#
# c-shell script to download selected files from rda.ucar.edu using Wget
# NOTE: if you want to run under a different shell, make sure you change
#       the 'set' commands according to your shell's syntax
# after you save the file, don't forget to make it executable
#   i.e. - "chmod 755 <name_of_script>"
#
# Experienced Wget Users: add additional command-line flags to 'opts' here
#   Use the -r (--recursive) option with care
#   Do NOT use the -b (--background) option - simultaneous file downloads
#       can cause your data access to be blocked
set opts = "-N"
#
# Check wget version.  Set the --no-check-certificate option 
# if wget version is 1.10 or higher
set v = `wget -V |grep 'GNU Wget ' | cut -d ' ' -f 3`
set a = `echo $v | cut -d '.' -f 1`
set b = `echo $v | cut -d '.' -f 2`
if(100 * $a + $b > 109) then
  set cert_opt = "--no-check-certificate"
else
  set cert_opt = ""
endif

set filelist= ( \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f000.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f003.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f006.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f009.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f012.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f015.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f018.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f021.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f024.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f027.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f030.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f033.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f036.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f039.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f042.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f045.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f048.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f051.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f054.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f057.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f060.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f063.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f066.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f069.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f072.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f075.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f078.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f081.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f084.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f087.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f090.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f093.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f096.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f099.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f102.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f105.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f108.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f111.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f114.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f117.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f120.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f123.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f126.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f129.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f132.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f135.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f138.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f141.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f144.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f147.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f150.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f153.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f156.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f159.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f162.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f165.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f168.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f171.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f174.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f177.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f180.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f183.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f186.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f189.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f192.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f195.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f198.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f201.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f204.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f207.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f210.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f213.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f216.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f219.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f222.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f225.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f228.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f231.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f234.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f237.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f240.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f246.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f252.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f258.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f264.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f270.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f276.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f282.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f288.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f294.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f300.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f306.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f312.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f318.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f324.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f330.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f336.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f342.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f348.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f354.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f360.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f366.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f372.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f378.grib2  \
  https://data.rda.ucar.edu/d084001/2024/20240604/gfs.0p25.2024060412.f384.grib2  \
)
while($#filelist > 0)
  set syscmd = "wget -c $cert_opt $opts $filelist[1]"
  echo "$syscmd ..."
  $syscmd
  shift filelist
end

exit 0
```

descargar script: [downGfsHist_2024Junio.sh](downGfsHist_2024Junio.sh)

* En cada enlace, las partes en negrita corresponden al año y a la fecha:
		.../d084001/**2024**/**20240604**/gfs.0p25.**20240604**12.f384.grib2

* y el numero 12 corresponde a la hora (12utc):
		.../d084001/2024/20240604/gfs.0p25.20240604**12**.f384.grib2

* Los numeros de frame, es decir, los numeros junto a la "f" y antes de la extensión .grib2 son fijos y son siempre los mismos.



Para descargas por lote: [[01b descarga forzantes GFS hist, por lote]]
