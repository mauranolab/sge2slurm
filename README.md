# sge2slurm
Compatibility wrappers for adapting SGE workflows to SLURM

See example slurm.conf, and pay special attention to these parameters for matching typical SGE behaviour:
SchedulerType=sched/builtin
PriorityType=priority/multifactor
SelectType=select/cons_res
SelectTypeParameters=CR_CPU


#Useful aliases for SGE -> SLURM:
<pre>alias qdel='scancel'</pre>

Based on qcheck by Shane Neph
<pre>qcheck () { squeue -o '%.8u %.2t %.6C %R' --noheader --array $* | awk -v me=`whoami` 'BEGIN {mecntr=0;waitcntr=0;allwaitcntr=0;smartcntr=0;neversatisfiedcntr=0;allsmartcntr=0;allneversatisfiedcntr=0;allnum=0} {num=$3; allnum+=num; if ($2~/R/) {  } if ($2 ~/PD/) {allwaitcntr+=num; if ($4~/Dependency/ || $4~/JobHeldUser/) allsmartcntr+=num; if ($4~/DependencyNeverSatisfied/) allneversatisfiedcntr+=num; } if ($1 == me) { mecntr+=num; if ($2~/PD/) {waitcntr+=num; if ($4~/Dependency/ || $4~/JobHeldUser/) smartcntr+=num; if ($4~/DependencyNeverSatisfied/) neversatisfiedcntr+=num; } } } END { print " All Jobs: " allnum; print "   Running: " allnum-allwaitcntr; print "   Waiting: " allwaitcntr; print "      Resource: " allwaitcntr-allsmartcntr; print "      Designed: " allsmartcntr; print "      Orphaned: " allneversatisfiedcntr; print " My Jobs: " mecntr; print "   Running: " mecntr-waitcntr; print "   Waiting: " waitcntr; print "      Resource: " waitcntr-smartcntr; print "      Designed: " smartcntr; print "      Orphaned: " neversatisfiedcntr; }' &&  date; }
</pre>

<pre>alias qstat="squeue -u `whoami` -o '%9F %.3p %45j %.8u %2t %19S %5P %.3C %.10K %R' -S 'P,-t,B,-p'"</pre>

<pre>alias qstata="squeue -o '%9F %.3p %45j %.8u %2t %19S %5P %.3C %.10K %R' -S 'P,-t,B,-p'"</pre>
