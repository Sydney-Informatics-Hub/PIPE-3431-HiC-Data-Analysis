# Transferring data between USYD RDS to Nimbus 

To transfer your data from RDS to Nimbus you will need to run the following PBS script on Artemis. This script contains a few options, depending on the direction of transfer of files (artemis -> nimbus, nimbus <- artemis) and whether you are transferring a whole directory or just one file. 

Before running you will need to 

1. Save a copy of your pem file currently stored on your local computer to your Artemis $HOME directory

* From your local computer terminal edit and run: `scp path/to/Your.pem <YourUnikey>@hpc.sydney.edu.au: /home/<YourUnikey>`
* If that scp command doesn't work you can copy it over to Artemis using FileZilla or the file explorer panel in MobaXterm 

2. Edit the following variables in the script:

* `#PBS -P <project>`: what is your dashR project code? (i.e. SIH)
* `rcos_path=/path/to/rds/directory/` i.e /rds/PRJ-DashRName/Files
* If you are transferring a whole directory: `rcos_dir=` 
* If you are transferring only a single file: `rcos_file=` 
* `nimbusIP=`: i.e 146.118.XX.XX
* `nimbus_path`: directory to transfer to on Nimbus 
* Replace the provided pem key example (`/home/gsam0138/georgie_nimbus_final.pem`) in each command with the path to your pem key 

3. Unhash the appropriate command to be run:

Save the script and then submit with 
```
qsub datatransfer.sh
```

You can reuse the script to transfer multiple files and back up your data onto RDS once you're done with your analysis. 


## Script 

```
#!/bin/bash

#PBS -P <project>
#PBS -N transfer
#PBS -l walltime=10:00:00
#PBS -l ncpus=1
#PBS -l mem=4GB
#PBS -W umask=022
#PBS -q dtq

# requires .pem file to be on both nimbus and artemis $home for passwordless transfer
# unhash/hash the correct rsync command below depending on direction and type of transfer

# rcos credientials
rcos_path=/path/to/rds/directory/ #i.e /rds/PRJ-DashRName/Files
rcos_dir= # use this if you are transferring a whole directory
rcos_file= # use this if you are transferring a file 

# nimbus vm credentials
nimbususer=ubuntu
nimbusIP=<nimbus IP>
nimbus_path=/data/<outDIR>

# transfer a file rcos to nimbus
#rsync -tPvz --append-verify -e "ssh -i /home/gsam0138/georgie_nimbus_final.pem" \
#       ${rcos_path}/${rcos_file} ${nimbususer}@${nimbusIP}:${nimbus_path}

# transfer a directory rcos to nimbus
#rsync -rtlPvz --append-verify -e "ssh -i /home/gsam0138/georgie_nimbus_final.pem" \
#        ${rcos_path}/${rcos_dir} ${nimbususer}@${nimbusIP}:${nimbus_path}

# transfer a file from nimbus
#rsync -tPvz --append-verify -e "ssh -i /home/gsam0138/georgie_nimbus_final.pem" \
#       ${nimbususer}@${nimbusIP}:${nimbus_path} ${rcos_path}

# transfer a directory from nimbus
#rsync -rtPvz --append-verify -e "ssh -i /home/gsam0138/georgie_nimbus_final.pem" \
#        ${nimbususer}@${nimbusIP}:${nimbus_path} ${rcos_path}
```
