

crear o copiar estos 2 scripts en la carpeta de trabajo:

```bash

fechaInicio="2024-06-04"
HH="12"

DD=$(dateNas +"%d" --date="$fechaInicio")
MM=$(dateNas +"%m" --date="$fechaInicio")
YYYY=$(dateNas +"%Y" --date="$fechaInicio")

DDsst=$(dateNas +"%d" --date="$fechaInicio -1 day")
MMsst=$(dateNas +"%m" --date="$fechaInicio -1 day")
YYYYsst=$(dateNas +"%Y" --date="$fechaInicio -1 day")

nDiasPre="8"     # entero, n° de dias a correr preproceso

# preproceso
./correrPreprocesoConSST_hist.sh -a ${YYYY}${MM}${DD} \  # fecha
-b ${HH} \                                               # hora
-c "/geo/pablo/GFS_g2/ds084.1/${YYYY}${MM}${DD}" \       # rutaGFS
-d "/HPC/GFS_g2/sst.${YYYYsst}${MMsst}${DDsst}" \        # rutaSST
-e 0p25 \                                                # GFSRESOLUTION
-f "/HPC/pablo/WRF4.3/grillaWRFv17" \                    # WPSTEMPLATE ruta donde estan los archivos de dominio geo_em.d0*.nc
-g "/HPC/pablo/cfinder/run/pili_${YYYY}${MM}${DD}" \     # WPSRUN ruta donde se correra el preproceso
-h 2 \         # n° de dominios
-i $nDiasPre \ # n° de dias para dominio3, si hubiera
-j $nDiasPre \ # n° de dias para dominios 1 y 2
-k hist \      # tipo: hist={historico GFS}
-l nogeo       # no correr geogrid, eso de hace solo 1 vez cuando se define el dominio

```


donde el script correrPreprocesoConSST_hist.sh es:

```bash
#!/bin/bash


if [ $# -eq 0 ]; then
	echo "Uso: ./script [options]"
	echo "-a dateStartForecast"
	echo "-b gfsHour          "
	echo "-c rutaGFS          "
	echo "-d rutaSST          "
	echo "-e GFSRESOLUTION    "
	echo "-f WPSTEMPLATE      "
	echo "-g WPSRUN           "
	echo "-h maxDom           "
	echo "-i NDIAS            "
	echo "-j NDIASDOM1Y2      "
	echo "-k gfsVersion       "
	echo "-l run_geogrid      "
	exit 1
fi

set -x

export run_ungrib=ungrib


#if [ $# -lt 12 ]; then
#	echo "./correrPreprocesoConSST.sh <StartForecast> <gfsHour> <rutaGfs> <rutaSST> <resGFS> <wpsTemplate> <wpsRun> <maxDom> <nDias> <nDiasdom1y2> <gfsVersion{op|hist}> <runGeogrid{geo|nogeo}>"
#	echo "./correrPreprocesoConSST.sh 20180502"
#	echo "                            00                                              "
#	echo "                            /HPC/GFS_g2/20180502-00	                      "
#	echo "                            /HPC/GFS_g2/sst.20180501	                      "
#	echo "                            0p25		                                      "
#	echo "                            /HPC/pablo/cfinder/3grillasOpCeaza	          "
#	echo "                            /HPC/pablo/cfinder/run/manual_OP_20180502 	  "
#	echo "                            3			                                      "
#	echo "                            5		                                          "
#	echo "                            10				                              "
#	echo "                            op						                      "
#	echo "                            nogeo"                                            
#	exit 1
#fi

export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/wrf/LIBS/grib2/lib
export logFile=/HPC/pablo/log.preproc.$(dateNas +%Y%m%d%H%M%S)

echo "logFile: "${logFile}

datelog() {
	echo "~~~ ["$(dateNas +"%Y-%m-%d %H:%M:%S")"]" $1 >> $logFile
}




while getopts a:b:c:d:e:f:g:h:i:j:k:l: option
do
	case "${option}"
		in
		a) dateStartForecast="${OPTARG}";;
		b) gfsHour="${OPTARG}";;
		c) rutaGFS="${OPTARG}";;
		d) rutaSST="${OPTARG}";;
		e) GFSRESOLUTION="${OPTARG}";;
		f) WPSTEMPLATE="${OPTARG}";;
		g) WPSRUN="${OPTARG}";;
		h) maxDom="${OPTARG}";;
		i) NDIAS="${OPTARG}";;
		j) NDIASDOM1Y2="${OPTARG}";;
		k) gfsVersion="${OPTARG}";;
		l) run_geogrid="${OPTARG}";;
	esac
done

YYYY=${dateStartForecast:0:4}
MM=${dateStartForecast:4:2}
DD=${dateStartForecast:6:2}
HH=$gfsHour

DDend=$(dateNas +%d --date="$YYYY-$MM-$DD +$NDIAS day")
MMend=$(dateNas +%m --date="$YYYY-$MM-$DD +$NDIAS day")
YYYYend=$(dateNas +%Y --date="$YYYY-$MM-$DD +$NDIAS day")
DDendDom1y2=$(dateNas +%d --date="$YYYY-$MM-$DD +$NDIASDOM1Y2 day")
MMendDom1y2=$(dateNas +%m --date="$YYYY-$MM-$DD +$NDIASDOM1Y2 day")
YYYYendDom1y2=$(dateNas +%Y --date="$YYYY-$MM-$DD +$NDIASDOM1Y2 day")

DDsst=$(dateNas +%d --date="$YYYY-$MM-$DD -1 day")  
MMsst=$(dateNas +%m --date="$YYYY-$MM-$DD -1 day")
YYYYsst=$(dateNas +%Y --date="$YYYY-$MM-$DD -1 day")
day=$(dateNas --date="$YYYY-$MM-$DD -1 day" +"%Y%m%d")


echo "fecha: $YYYY $MM $DD"

# ---------------------------------------------------------------------
export gfsFolder=${rutaGFS}

datelog " -- inicio script --"
datelog "gfsFolder: $gfsFolder"

#		#comprobar cantidad de archivos grib gfs
#		nGribFiles=93  #para resoluciones 1p00 y 0p50
#		if [ "$GFSRESOLUTION" == "0p25" ]; then
#			nGribFiles=173
#		fi
#
#		nFilesDown=$(ls $gfsFolder/gfs.*| grep $GFSRESOLUTION | grep t${HH}z |wc -l)
#		while [ ${nFilesDown} -lt ${nGribFiles} ] ; do
#			if [ $(ps xau|grep $USER|grep  "bajarGfs.sh ${DD} ${MM} ${YYYY} ${HH}"|grep -v grep|wc -l) -lt 1 ]; then
#				datelog 'aun no corre script de descarga, iniciando ...'
#				/HPC/pablo/cfinder/bajarGfs.sh ${DD} ${MM} ${YYYY} ${HH}
#			fi
#			datelog "cantidad de archivos grib (${nFilesDown}) es menor a ${nGribFiles}, esperando ..."
#			sleep 60 #
#			nFilesDown=$(ls $gfsFolder/gfs.*| grep "${GFSRESOLUTION}" | grep t${HH}z |wc -l)
#		done
#
#		if [ "$GFSRESOLUTION" == "0p25" ]; then
#			# como son archivos mas grandes, talvez hay que esperar a que terminen de descargar
#			menorTam=$(du -sm $gfsFolder/gfs.t${HH}z.pgrb2.${GFSRESOLUTION}.f???|sort -n | head -1 |gawk '{print $1}')
#			while [ $menorTam -lt 120 ] ; do
#				datelog "tam menor: ${menorTam}, aun faltan por descargar totalmente algunos archivos, esperando ..."
#				sleep 5
#				menorTam=$(du -sm $gfsFolder/gfs.t${HH}z.pgrb2.${GFSRESOLUTION}.f???|sort -n | head -1 |gawk '{print $1}')
#			done
#		fi
#
#		datelog "cantidad de archivos grib OK."
#
#		# comprobar existencia de archivo grib sst
#		/HPC/pablo/cfinder/bajarSstHrGrb.sh /HPC/GFS_g2 ${YYYYsst}${MMsst}${DDsst} 0.083
#		while [ $(ls ${rutaSST}/rtgssthr_grb_* | grep grib2$ | wc -l) -lt 1 ] ; do
#			datelog "no hay archivo grib SST, reintentando..."
#			sleep 120
#			/HPC/pablo/cfinder/bajarSstHrGrb.sh /HPC/GFS_g2 ${YYYYsst}${MMsst}${DDsst} 0.083	
#		done
#
#
#
#		datelog "Archivo grib SST OK."


datelog "copiando template WPS ..."
rm -rf ${WPSRUN}
mkdir -p ${WPSRUN}



cp -rf ${WPSTEMPLATE}/* ${WPSRUN}/ || { datelog " error:  cp -rf $WPSTEMPLATE/* $WPSRUN/ " ; exit 1; }
chmod -R u+w $WPSRUN

cd $WPSRUN 
rm -f GRIBFILE.*
datelog "enlazando archivos GFS ..."
if [ "$gfsVersion" == "hist" ]; then
	# version ds084.1 (gfs historico)
	echo "path para linkgrib ${rutaGFS}/gfs.${GFSRESOLUTION}.${dateStartForecast}${HH}.f"
	./link_grib.csh "${rutaGFS}/gfs.${GFSRESOLUTION}.${dateStartForecast}${HH}.f" || { datelog " error linkbrib!" ; exit 1; }
else 
	#version operacional
	./link_grib.csh "${rutaGFS}/gfs.t${HH}z.pgrb2.${GFSRESOLUTION}.f" || { datelog " error linkbrib!" ; exit 1; }
fi


datelog "enlazando Vtable para GFS ..."
ln -sf ungrib/Variable_Tables/Vtable.GFS Vtable

cat /dev/null > namelist.wps
echo "&share               " >> namelist.wps
echo " wrf_core = 'ARW',   " >> namelist.wps
echo " max_dom = $maxDom,  " >> namelist.wps
echo "start_date = '"$YYYY"-"$MM"-"$DD"_"$HH":00:00','"$YYYY"-"$MM"-"$DD"_"$HH":00:00','"$YYYY"-"$MM"-"$DD"_"$HH":00:00'," >> namelist.wps
echo "end_date = '"$YYYYendDom1y2"-"$MMendDom1y2"-"$DDendDom1y2"_"$HH":00:00','"$YYYYendDom1y2"-"$MMendDom1y2"-"$DDendDom1y2"_"$HH":00:00','"$YYYYend"-"$MMend"-"$DDend"_"$HH":00:00'," >> namelist.wps
#echo "end_date = '"$YYYY"-"$MM"-"$DD"_06:00:00','"$YYYY"-"$MM"-"$DD"_06:00:00','"$YYYY"-"$MM"-"$DD"_06:00:00'," >> namelist.wps
echo " interval_seconds = 21600,                                 " >> namelist.wps
echo " io_form_geogrid = 2,                                      " >> namelist.wps
echo " debug_level = 0,                                          " >> namelist.wps
echo "/                                                          " >> namelist.wps
echo "                                                         " >> namelist.wps
echo "&geogrid                                                          " >> namelist.wps
echo " parent_id         = 1,1                                  " >> namelist.wps
echo " parent_grid_ratio = 1,3                                  " >> namelist.wps
echo " i_parent_start    = 1,32                         " >> namelist.wps
echo " j_parent_start    = 1,30                         " >> namelist.wps
echo " e_we          = 168,316,                                 " >> namelist.wps
echo " e_sn          = 162,310,                                 " >> namelist.wps
echo " geog_data_res = '5m','5m'                                " >> namelist.wps
echo " dx = 9000,                                               " >> namelist.wps
echo " dy = 9000,                                                       " >> namelist.wps
echo "map_proj =  'mercator',   "                               >> namelist.wps
echo "geog_data_path = '/AUX/pablo/geog/' "                     >> namelist.wps
echo "ref_lat   = -30.166,                                      " >> namelist.wps
echo "ref_lon   = -72.455,                                      " >> namelist.wps
echo "truelat1  = -30.166,                                      " >> namelist.wps
echo "truelat2  = 0,                                            " >> namelist.wps
echo "stand_lon = -72.455,                                      " >> namelist.wps
echo "ref_x = 84.0,                                             " >> namelist.wps
echo "ref_y = 81.0,                                             " >> namelist.wps
echo "/                                                          " >> namelist.wps
echo "                                                           " >> namelist.wps
echo "&ungrib                                                    " >> namelist.wps
echo " out_format = 'WPS',                                       " >> namelist.wps
echo " prefix = 'GFS2',                                          " >> namelist.wps
echo "/                                                          " >> namelist.wps
echo "                                                           " >> namelist.wps
echo "&metgrid                                                   " >> namelist.wps
echo " io_form_metgrid = 2,                                      " >> namelist.wps
echo " fg_name =  'GFS2'                                         " >> namelist.wps
echo "constants_name = 'SST:"$YYYYsst"-"$MMsst"-"$DDsst"_00'     " >> namelist.wps
echo "/                                                          " >> namelist.wps


echo "&mod_levs                                " >> namelist.wps
echo " press_pa = 201300 , 200100 , 100000 ,   " >> namelist.wps
echo "             95000 ,  90000 ,            " >> namelist.wps
echo "             85000 ,  80000 ,            " >> namelist.wps
echo "             75000 ,  70000 ,            " >> namelist.wps
echo "             65000 ,  60000 ,            " >> namelist.wps
echo "             55000 ,  50000 ,            " >> namelist.wps
echo "             45000 ,  40000 ,            " >> namelist.wps
echo "             35000 ,  30000 ,            " >> namelist.wps
echo "             25000 ,  20000 ,            " >> namelist.wps
echo "             15000 ,  10000 ,            " >> namelist.wps
echo "              5000 ,   1000              " >> namelist.wps
echo " /                                       " >> namelist.wps




if [ "$run_geogrid" == "geo" ]; then
	datelog 'corriendo geogrid.exe ...'
	./geogrid.exe >> $logFile 
fi

# ----------------- primer ungrib ------------------------------
if [ "$run_ungrib" == "ungrib" ]; then
	datelog 'borrando GFS2:* ...'
	rm -f "GFS2\:"*
	datelog 'corriendo ungrib.exe para gfs...'
	./ungrib.exe  >> $logFile  || { datelog "error en ungrib.exe para gfs." ; exit 1; }
fi

datelog "ungrib para datos GFS ok"



datelog 'respaldando namelist.wps actual para despues ocuparlo con metgrid.exe ...' 
cp -f namelist.wps namelist.wps_paraGFS

# actualizar namelist.wps para SST
cat /dev/null > namelist.wps # lo deja vacio
echo "&share               " >> namelist.wps
echo " wrf_core = 'ARW',   " >> namelist.wps
echo " max_dom = $maxDom,  " >> namelist.wps
echo "start_date = '"$YYYYsst"-"$MMsst"-"$DDsst"_00:00:00','"$YYYYsst"-"$MMsst"-"$DDsst"_00:00:00','"$YYYYsst"-"$MMsst"-"$DDsst"_00:00:00'," >> namelist.wps
echo "end_date   = '"$YYYYsst"-"$MMsst"-"$DDsst"_00:00:00','"$YYYYsst"-"$MMsst"-"$DDsst"_00:00:00','"$YYYYsst"-"$MMsst"-"$DDsst"_00:00:00'," >> namelist.wps
echo " interval_seconds = 21600,                                 " >> namelist.wps
echo " io_form_geogrid = 2,                                      " >> namelist.wps
echo "/                                                          " >> namelist.wps
echo "                                                           " >> namelist.wps
echo "&geogrid                                                          " >> namelist.wps
echo " parent_id         = 1,1                                  " >> namelist.wps
echo " parent_grid_ratio = 1,3                                  " >> namelist.wps
echo " i_parent_start    = 1,32                         " >> namelist.wps
echo " j_parent_start    = 1,30                         " >> namelist.wps
echo " e_we          = 168,316,                                 " >> namelist.wps
echo " e_sn          = 162,310,                                 " >> namelist.wps
echo " geog_data_res = '5m','5m'                                " >> namelist.wps
echo " dx = 9000,                                               " >> namelist.wps
echo " dy = 9000,                                                       " >> namelist.wps
echo "map_proj =  'mercator',   "                               >> namelist.wps
echo "geog_data_path = '/AUX/pablo/geog/' "                     >> namelist.wps
echo "ref_lat   = -30.166,                                      " >> namelist.wps
echo "ref_lon   = -72.455,                                      " >> namelist.wps
echo "truelat1  = -30.166,                                      " >> namelist.wps
echo "truelat2  = 0,                                            " >> namelist.wps
echo "stand_lon = -72.455,                                      " >> namelist.wps
echo "ref_x = 84.0,                                             " >> namelist.wps
echo "ref_y = 81.0,                                             " >> namelist.wps
echo "/                                                          " >> namelist.wps
echo "                                                           " >> namelist.wps
echo "&ungrib                                                    " >> namelist.wps
echo " out_format = 'WPS',                                       " >> namelist.wps
echo " prefix = 'SST',                                           " >> namelist.wps
echo "/                                                          " >> namelist.wps
echo "                                                           " >> namelist.wps
echo "&metgrid                                                   " >> namelist.wps
echo " io_form_metgrid = 2,                                      " >> namelist.wps
echo " fg_name =  'GFS2'                                         " >> namelist.wps
echo "constants_name = 'SST:"$YYYYsst"-"$MMsst"-"$DDsst"_00'     " >> namelist.wps
echo "/                                                          " >> namelist.wps

echo "&mod_levs                                " >> namelist.wps
echo " press_pa = 201300 , 200100 , 100000 ,   " >> namelist.wps
echo "             95000 ,  90000 ,            " >> namelist.wps
echo "             85000 ,  80000 ,            " >> namelist.wps
echo "             75000 ,  70000 ,            " >> namelist.wps
echo "             65000 ,  60000 ,            " >> namelist.wps
echo "             55000 ,  50000 ,            " >> namelist.wps
echo "             45000 ,  40000 ,            " >> namelist.wps
echo "             35000 ,  30000 ,            " >> namelist.wps
echo "             25000 ,  20000 ,            " >> namelist.wps
echo "             15000 ,  10000 ,            " >> namelist.wps
echo "              5000 ,   1000              " >> namelist.wps
echo " /                                       " >> namelist.wps



datelog "enlazando archivos grib para SST, considera ambas carpetas (/HPC/GFS_g2/sst.**** y /sata1_boletin/SST/rtg_sst_grb_hr_0.083.****  ..."
./link_grib.csh ${rutaSST}/rtgssthr_grb_0.083.grib2
#./link_grib.csh /sata1_boletin/SST/rtg_sst_grb_hr_0.083.${YYYYsst}${MMsst}${DDsst}

datelog "enlazando Vtable para SST ..."
ln -sf ungrib/Variable_Tables/Vtable.SST Vtable


# ----------------- segundo ungrib (este es mas rapido)------------------------
if [ "$run_ungrib" == "ungrib" ]; then
	datelog 'borrando SST:* ...'
	rm -f "SST\:"*
	datelog "corriendo ungrib.exe para datos SST ..."	
	./ungrib.exe  >> $logFile  || { datelog "error en ungrib.exe para sst." ; exit 1; }
fi

datelog "ungrib para SST ok"

# metrgid
ln -sf ungrib/Variable_Tables/Vtable.GFS Vtable # necesario?
datelog "restaurando namelist.wps para todo el rango temporal ..."
cp -f namelist.wps namelist.wps_paraSST
cp -f namelist.wps_paraGFS namelist.wps
datelog "iniciando metgrid.exe ..."
srun --partition=debug -n 1 ./metgrid.exe >> $logFile
datelog "metgrid terminado ok"

datelog "moviendo archivos met_em.d0 en ${GFSRESOLUTION}_${dateStartForecast}${HH}"
mkdir -p ${GFSRESOLUTION}"_"${dateStartForecast}${HH}
mv -v met_em.d0* ${GFSRESOLUTION}"_"${dateStartForecast}${HH}/

#datelog "eliminando archivos temporales ..."
#rm -f GFS2*
#ls|grep -v ungrib.log|grep -v metgrid.log |xargs rm 
#rm -rf arch geogrid metgrid ungrib util




datelog 'fin script'
datelog ' '

```
