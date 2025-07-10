

en /HPC/pablo/cfinder/pili-viento-ptaChoros2024Junio.sh

```bash

fechaInicio="2024-06-04"
HH="12"

DD=$(dateNas +"%d" --date="$fechaInicio")
MM=$(dateNas +"%m" --date="$fechaInicio")
YYYY=$(dateNas +"%Y" --date="$fechaInicio")

DDsst=$(dateNas +"%d" --date="$fechaInicio -1 day")
MMsst=$(dateNas +"%m" --date="$fechaInicio -1 day")
YYYYsst=$(dateNas +"%Y" --date="$fechaInicio -1 day")

nCores="64"
nDiasPre="8"     # entero
nDiasPro="8.0"

# paso 2 preproceso
./correrPreprocesoConSST_hist.sh -a ${YYYY}${MM}${DD} \
-b ${HH} 
-c "/geo/pablo/GFS_g2/ds084.1/${YYYY}${MM}${DD}" \
-d "/HPC/GFS_g2/sst.${YYYYsst}${MMsst}${DDsst}" \
-e 0p25 
-f "/HPC/pablo/WRF4.3/grillaWRFv17" 
-g "/HPC/pablo/cfinder/run/pili_${YYYY}${MM}${DD}" 
-h 2 -i $nDiasPre -j $nDiasPre -k hist -l nogeo

```


