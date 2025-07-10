en la carpeta de trabajo crear:

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
nDiasPro="8.0" # decimal

phConfig="c05"  # configuracion, param fisicas WRF

./script-cfinder_hist.sh -i "0p25_${YYYY}${MM}${DD}${HH}" \
-r ${YYYY}${MM}${DD}${HH} \
-n $nCores \
-h 1 \
-p ${phConfig} \
-w "pili_${YYYY}${MM}${DD}" \
-x 2 \
-y "/geo/pablo/WRFOUT_pili/${phConfig}_${YYYY}${MM}${DD}${HH}" \
-d $nDiasPro \
-g 0p25



```


donde script-cfinder_hist.sh es:

```bash
#!/bin/bash

set -x 
run_preproc=No
run_real=Yes
run_wrf=Yes


export CFINDERHOME=$(dirname $(readlink -f $0))


if [ $# -lt 5 ]; then
	echo "faltan argumentos"
	echo "       : -r <rundate>, format YYYYMMDDHH"
	echo "       : -p <phID>, id de paremetrizaciones fisicas"
	echo "otros:"
	echo "       : -i <wrfrunid>"
	echo "       : -w <wpsrunid>, (archivos met_em.d0*)"
	echo "       : -g <gfsres>"
	echo "       : -n <nprocs>"
	echo "       : -h <nhosts>"
	echo "       : -d <ndias>"
	echo "       : -s <ndiasDom1y2>"
	echo "       : -x <maxdom>"
	echo "       : -y <results_folder>"
	
	exit 1
fi

#valores x defecto
GFSRES="1p00"
NPROCS=152
NDIAS=5
NDIAS1Y2=10
MAXDOM=3
PHID="c05"

while getopts r:i:w:p:g:n:h:d:x:y: option
do
	case "${option}"
		in
		r) RUNDATE=${OPTARG};;
		i) WRFRUNID=${OPTARG};;
		w) WPSRUNID=${OPTARG};;
		p) PHID=${OPTARG};;
		g) GFSRES=${OPTARG};;
		n) NPROCS=${OPTARG};;
		h) NHOSTS=${OPTARG};;
		d) NDIAS=${OPTARG};;
		x) MAXDOM=${OPTARG};;
		y) RESULTSFOLDER=${OPTARG};;
	esac
done


#CFHOME=$(pwd)
#PPP WRF4

#CFHOME=/oceano/pablo/WRF4.1
#WRFFOLDER=WRF-4.1.1

CFHOME=/HPC/pablo/WRF4.3
WRFFOLDER=WRF


#MET_EM=${CFHOME}/run/${WPSRUNID}/${GFSRES}"_"${RUNDATE}
#PPP WRF4
MET_EM=/HPC/pablo/cfinder/run/${WPSRUNID}/${GFSRES}"_"${RUNDATE}

YYYY=${RUNDATE:0:4}
MM=${RUNDATE:4:2}
DD=${RUNDATE:6:2}
HH=${RUNDATE:8:2}

nhoras=$(echo $NDIAS*24|bc|awk '{printf("%i",$1)}')


echo $nhoras

#DDend=$(dateNas +%d --date="$YYYY-$MM-$DD +$nhoras hours")
#MMend=$(dateNas +%m --date="$YYYY-$MM-$DD +$nhoras hours")
#YYYYend=$(dateNas +%Y --date="$YYYY-$MM-$DD +$nhoras hours")

DDend=$(gawk -v fecha="$YYYY $MM $DD $HH 00 00" -v nhoras=$nhoras 'BEGIN{unixtime=mktime(fecha)+nhoras*60*60;print strftime("%d",unixtime);}')
MMend=$(gawk -v fecha="$YYYY $MM $DD $HH 00 00" -v nhoras=$nhoras 'BEGIN{unixtime=mktime(fecha)+nhoras*60*60;print strftime("%m",unixtime);}')
YYYYend=$(gawk -v fecha="$YYYY $MM $DD $HH 00 00" -v nhoras=$nhoras 'BEGIN{unixtime=mktime(fecha)+nhoras*60*60;print strftime("%Y",unixtime);}')
endHour=$(gawk -v fecha="$YYYY $MM $DD $HH 00 00" -v nhoras=$nhoras 'BEGIN{unixtime=mktime(fecha)+nhoras*60*60;print strftime("%H",unixtime);}')

if [[ ! ${WRFRUNID} ]]
then
	WRFRUNID=$(dateNas +"%Y%m%d%H%M%S")
fi

if [[ ! ${WPSRUNID} ]]
then
	WPSRUNID=$(dateNas +"%Y%m%d%H%M%S")
fi

echo "CFHOME  : "$CFHOME
echo "rundate : "$RUNDATE
echo "wrfrunid: "$WRFRUNID
echo "wpsrunid: "$WPSRUNID
echo "phid    : "$PHID
echo "gfsres  : "$GFSRES
echo "nprocs  : "$NPROCS
echo "nhosts  : "$NHOSTS
echo "ndias   : "$NDIAS
echo "maxdom  : "$MAXDOM

#=======================================================================
if [ "$run_wrf" == "Yes" ]; then
	#WRFRUN=${CFHOME}/WRFV3/test/${PHID}"_"${WRFRUNID}
	
	#PPP WRF4
	WRFRUN=${CFHOME}/${WRFFOLDER}/test/${PHID}"_"${WRFRUNID}

	echo "limpiando carpeta de trabajo ..."
	rm -rf ${WRFRUN} || { echo "error en rm -rf ${WRFRUN} ... " ; exit 1; }
	echo "copiando template wrf ..."

	#cp -rf ${CFHOME}/WRFV3/test/template_em_real ${WRFRUN}
	#PPP WRF4
	cp -rf ${CFHOME}/${WRFFOLDER}/test/em_real ${WRFRUN}

	cd ${WRFRUN}
	rm -f met_em.d*
	echo "enlazando archivos met_em.d0* ..."
	
	nMetEm=$(ls ${MET_EM}/met_em.d*|wc -l)
	if [ $nMetEm -eq 0 ]; then
		echo "no hay archivos met_em.d0* en ${MET_EM} "
		exit 1
	fi	
	
	ln -s ${MET_EM}/met_em.d* .
	
	
	#--------creacion de archivo namelist.input -------------------
	echo "creando archivo namelist.input ..."
	rm -f namelist.input
	cat /dev/null > namelist.input

        echo " &time_control                                    " >> namelist.input
        echo " run_days                            = 0,         " >> namelist.input
        echo " run_hours                           = 0,         " >> namelist.input
        echo " run_minutes                         = 0,         " >> namelist.input
        echo " run_seconds                         = 0,         " >> namelist.input
        echo " output_diagnostics = 1,                          " >> namelist.input
        echo " auxhist3_outname = \"wrfxtrm_d<domain>_<date>\", " >> namelist.input
        echo " auxhist3_interval = 60,  60,   60,               " >> namelist.input
        echo " frames_per_auxhist3 = 1000, 1000, 1000,          " >> namelist.input
        echo " io_form_auxhist3 = 2,                            " >> namelist.input
        echo " iofields_filename = \"my_file_d01.txt\", \"my_file_d02.txt\", \"my_file_d03.txt\"        " >> namelist.input
        echo " ignore_iofields_warning = .true.,        " >> namelist.input

        echo " start_year                          = $YYYY, $YYYY, $YYYY," >> namelist.input
        echo " start_month                         = $MM,   $MM,   $MM," >> namelist.input
        echo " start_day                           = $DD,   $DD,   $DD," >> namelist.input
        echo " start_hour                          = $HH,   $HH,   $HH," >> namelist.input
        echo " start_minute                        = 00,   00,   00," >> namelist.input
        echo " start_second                        = 00,   00,   00," >> namelist.input
        echo " end_year                            = $YYYYend, $YYYYend, $YYYYend," >> namelist.input
        echo " end_month                           = $MMend,   $MMend,   $MMend," >> namelist.input
        echo " end_day                             = $DDend,   $DDend,   $DDend," >> namelist.input
        echo " end_hour                            = $endHour,   $endHour,   $endHour," >> namelist.input
        echo " end_minute                          = 00,   00,   00,                  " >> namelist.input
        echo " end_second                          = 00,   00,   00,                  " >> namelist.input
        echo " interval_seconds                    = 21600                            " >> namelist.input
        echo " input_from_file                     = .true.,.true.,.true.,            " >> namelist.input
        echo " history_interval                    = 60,  60,   60,                   " >> namelist.input
        echo " frames_per_outfile                  = 1000, 1000, 1000,                " >> namelist.input
        echo " restart                             = .false.,                         " >> namelist.input
        echo " restart_interval                    = 4320,                            " >> namelist.input
        echo " io_form_history                     = 2                                " >> namelist.input
        echo " io_form_restart                     = 2                                " >> namelist.input
        echo " io_form_input                       = 2                                " >> namelist.input
        echo " io_form_boundary                    = 2                                " >> namelist.input
        echo " debug_level                         = 0                                " >> namelist.input
        echo "/                                                                       " >> namelist.input



        echo " &domains                 " >> namelist.input
#       echo "time_step                = 54,             " >> namelist.input
        echo "time_step                = 36,             " >> namelist.input
        echo "time_step_fract_num      = 0,               " >> namelist.input
        echo "time_step_fract_den      = 1,               " >> namelist.input
        echo "max_dom                  = $MAXDOM,         " >> namelist.input
        echo "s_we                     = 1,        1,     " >> namelist.input
        echo "e_we                     = 168,      316,   " >> namelist.input
        echo "s_sn                     = 1,        1,     " >> namelist.input
        echo "e_sn                     = 162,      310,   " >> namelist.input
        echo "s_vert                   = 1,        1,     " >> namelist.input
        echo "e_vert                   = 43,       43,    " >> namelist.input
        echo "num_metgrid_levels       = 34,              " >> namelist.input
        echo "dx                       = 9000,     3000, " >> namelist.input
        echo "dy                       = 9000,     3000, " >> namelist.input
        echo "grid_id                  = 1,        2,     " >> namelist.input
        echo "parent_id                = 1,        1,     " >> namelist.input
        echo "i_parent_start           = 1,       32,     " >> namelist.input
        echo "j_parent_start           = 1,       30,     " >> namelist.input
        echo "parent_grid_ratio        = 1,        3,     " >> namelist.input
        echo "parent_time_step_ratio   = 1,        3,     " >> namelist.input
        echo "feedback                 = 1,               " >> namelist.input
        echo "smooth_option            = 0,               " >> namelist.input
        echo "/                                           " >> namelist.input

        #param fisicas
        #cat ${CFHOME}/ph_configs/${PHID} >> namelist.input

        #PPP 
	
        cat ${CFINDERHOME}/ph_configs/${PHID} >> namelist.input

        echo " &fdda                                            " >> namelist.input
        echo " /                                                " >> namelist.input

        # config copiada de sims paleo de orlando
        echo "&dynamics                                     " >> namelist.input
        echo "w_damping           = 1,                      " >> namelist.input
        echo "diff_opt            = 1,      1,              " >> namelist.input
        echo "km_opt              = 4,      4,              " >> namelist.input
        echo "khdif               = 0,      0,              " >> namelist.input
        echo "kvdif               = 0,      0,              " >> namelist.input
        echo "non_hydrostatic     = .true., .true.,         " >> namelist.input
        echo "/                                             " >> namelist.input

        echo "                                               " >> namelist.input
        echo "&bdy_control                                   " >> namelist.input
        echo "spec_bdy_width           = 5,                  " >> namelist.input
        echo "spec_zone                = 1,                  " >> namelist.input
        echo "relax_zone               = 4,                  " >> namelist.input
        echo "specified                = .true.,  .false.,   " >> namelist.input
        echo "nested                   = .false.,   .true.,  " >> namelist.input
        echo "/                                              " >> namelist.input


        echo " &grib2                                           " >> namelist.input
        echo " /                                                " >> namelist.input

	echo "&namelist_quilt" >> namelist.input
	echo "nio_tasks_per_group = 0," >> namelist.input
	echo "nio_groups = 1," >> namelist.input
	echo "/" >> namelist.input



	#-------- fin archivo namelist.input -------------------
	
	if [ "$run_real" == "Yes" ]; then
		echo "corriendo real.exe ..."
#PPP
		./real.exe # || { echo "error en real.exe" ; exit 1; }
		
	fi

	# creando archivo para slurm
	echo "creando archivo slurm.sh ..."

	jobName=${PHID}$(dateNas +"%d" --date="${YYYY}-${MM}-${DD}")
	cat /dev/null > slurm.sh
	echo "#!/bin/bash" > slurm.sh
	echo "#SBATCH -J ${jobName} 		                                            " >> slurm.sh

	# si NHOSTS	esta definido
#	if [ ! -z $NHOSTS ]; then 
#		echo "#SBATCH -N ${NHOSTS}                                                    " >> slurm.sh
#	fi

	# determinar si entrar en part3, part2 o part1
	freePart3=$(sinfo |grep part3|grep idle|awk '{print $4}')
	freePart1=$(sinfo |grep part1|grep idle|awk '{print $4}')


	[[  $freePart3 && ${freePart3-x} ]] &&  echo "" || freePart3=0;
	[[  $freePart1 && ${freePart1-x} ]] &&  echo "" || freePart1=0;




#	if [ $freePart3 -gt 1 ]; then
#		echo "#SBATCH --partition=part3                                                   " >> slurm.sh
#		echo "#SBATCH -n 24                                                               " >> slurm.sh
#	elif [ $freePart1 -gt 0 ]; then
#                echo "#SBATCH --partition=part1                                                   " >> slurm.sh
#                echo "#SBATCH -n 16                                                               " >> slurm.sh
#	else
#                echo "#SBATCH --partition=part2                                                   " >> slurm.sh
#                echo "#SBATCH -n 24                                                               " >> slurm.sh
#	fi

echo "#SBATCH --partition=debug                                                    " >> slurm.sh
echo "#SBATCH -n ${NPROCS}                                                               " >> slurm.sh

	echo "source /opt/intel/compilers_and_libraries/linux/bin/compilervars.sh intel64 " >> slurm.sh
	echo "cd $WRFRUN                                                                  " >> slurm.sh
	echo "time mpirun ./wrf.exe                                                       " >> slurm.sh
	
	# si RESULTSFOLDER esta definido
	if [ ! -z $RESULTSFOLDER ]; then 
		echo "mkdir -p ${RESULTSFOLDER}/                 					      " >> slurm.sh

#PPP	queda el de abajo comentada
		echo "(mv -v wrfout_d* wrfxtrm_d* ${RESULTSFOLDER}/ ) && ( touch ${RESULTSFOLDER}/GFSGFSRESOLUTION_${GFSRES}  ) " >> slurm.sh
#		echo "(cp    wrfout_d* wrfxtrm_d* ${RESULTSFOLDER}/ ) && ( touch ${RESULTSFOLDER}/GFSGFSRESOLUTION_${GFSRES}  ) " >> slurm.sh




	fi

	echo "encolando trabajo en slurm ..."
#PPP
	sbatch slurm.sh

#PPP comentar!
#        if [ ! -z $RESULTSFOLDER ]; then
#                mkdir -p ${RESULTSFOLDER}/
#                touch ${RESULTSFOLDER}/wrfout_prueba  
#		touch ${RESULTSFOLDER}/GFSGFSRESOLUTION_${GFSRES}
#        fi


	sleep 2
	squeue
fi
echo "fin script"
echo ""



```
