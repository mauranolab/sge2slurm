# sge2slurm
Compatibility wrappers for adapting SGE workflows to SLURM

See example slurm.conf, and pay special attention to these parameters for matching typical SGE behaviour:
SchedulerType=sched/builtin
PriorityType=priority/multifactor
SelectType=select/cons_res
SelectTypeParameters=CR_CPU


#Useful aliases for SGE -> SLURM:
<pre>
alias qdel='scancel'
#Based on qcheck by Shane Neph
qcheck () { squeue -o '%.8u %.2t %.6C %R' --noheader --array $* | awk -v me=`whoami` 'BEGIN {mecntr=0;waitcntr=0;allwaitcntr=0;smartcntr=0;neversatisfiedcntr=0;allsmartcntr=0;allneversatisfiedcntr=0;allnum=0} {num=$3; allnum+=num; if ($2~/R/) {  } if ($2 ~/PD/) {allwaitcntr+=num; if ($4~/Dependency/ || $4~/JobHeld/) allsmartcntr+=num; if ($4~/DependencyNeverSatisfied/) allneversatisfiedcntr+=num; } if ($1 == me) { mecntr+=num; if ($2~/PD/) {waitcntr+=num; if ($4~/Dependency/ || $4~/JobHeldUser/) smartcntr+=num; if ($4~/DependencyNeverSatisfied/) neversatisfiedcntr+=num; } } } END { print " All Jobs: " allnum; print "   Running: " allnum-allwaitcntr; print "   Waiting: " allwaitcntr; print "      Resource: " allwaitcntr-allsmartcntr; print "      Designed: " allsmartcntr; print "      Orphaned: " allneversatisfiedcntr; print " My Jobs: " mecntr; print "   Running: " mecntr-waitcntr; print "   Waiting: " waitcntr; print "      Resource: " waitcntr-smartcntr; print "      Designed: " smartcntr; print "      Orphaned: " neversatisfiedcntr; }' &&  date; }
#%S gives start; %19V gives submitted time
qstata () { squeue -o '%9F|%.3p|%45j|%.8u|%2t|%19S;%19V|%9P|%.3C|%.5K|%R' -S 'P,-t,B,-p' $* | awk -F "|" 'BEGIN {OFS=" "} {if(NR==1) {$6="SUBMIT/START       "} else {split($6, times, ";"); if($5!="PD") {$6=times[1]} else {$6=times[2]}} print}'; }
qstat () { qstata -u `whoami` $*; }
qgrep () { qstat | tail +2 | grep $* | awk '{print $1}' ; }
qnode () { squeue -o '%9F %.3p %45j %.8u %2t %19S %9P %.3C %.5K %R' -S 'P,-t,B,-p' | awk 'NR>1 && $5=="R"' | sort -k10,10 | awk '{for(i=1; i<=$8; i++) {print $10}}' | hist | sort -k2,2 -; }
alias rqsub='qsub -S `which Rscript`'
alias bqsub='qsub -S /bin/bash'
qlogin () { srun $* --pty bash; }
</pre>

#Limit on number of total slots used per-user
The best way is to create 2 QOS:
<ol>
<li> normal is limited to 144 slots per user
<li> full enables full use of the queue. Access can optionally be restricted to certain users
</ol>

You must use sacctmgr:
<pre>
sacctmgr modify qos normal set MaxCpusPerUser=144
sacctmgr create qos full 
sacctmgr modify user mauram01 set qos+=full
#we also use a third low queue
sacctmgr create qos low
sacctmgr modify qos low set MaxCpusPerUser=48
sacctmgr modify account nyu set qos+=low
</pre>

In slurm.conf:
<pre>
enable AccountingStorageEnforce=associations,limits,qos
</pre>

Users with access can do `sbatch --qos full` and also set a default:
<pre>
export SBATCH_QOS=full
</pre>
