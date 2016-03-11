# sge2slurm
Compatibility wrappers for adapting SGE workflows to SLURM

#Useful aliases for SGE -> SLURM:
<pre>alias qdel='scancel'</pre>

Based on qcheck by Shane Neph
<pre>alias qcheck="squeue -o '%.8u %.2t %.6C %R' --noheader --array | awk -v me=`whoami` 'BEGIN {mecntr=0;waitcntr=0;allwaitcntr=0;smartcntr=0;n\
eversatisfiedcntr=0;allsmartcntr=0;allneversatisfiedcntr=0;allnum=0} {num=\$3; allnum+=num;
if ( \$2~/R/ ) {  ; } if (\$2 ~/PD/) {allwaitcntr+=num; if (\$4 ~ /\(Dependency\)/) allsmartcntr+=num; if (\$4 ~ /\(DependencyNeverSatisfie\
d\)/) allneversatisfiedcntr+=num; } if (\$1 == me) { mecntr+=num; if (\$2 ~ /PD/ ) {waitcntr+=num; if ( \$4 ~ /\(Dependency\)/) smartcntr+=\
num; if (\$4 ~ /\(DependencyNeverSatisfied\)/) neversatisfiedcntr+=num; } } } END { print \" All Jobs: \"allnum; print \"   Running: \"alln\
um-allwaitcntr; print \"   Waiting: \"allwaitcntr; print \"      Resource: \"allwaitcntr-allsmartcntr; print \"      Designed: \"allsmartcn\
tr; print \"      Orphaned: \"allneversatisfiedcntr; print \" My Jobs: \"mecntr; print \"   Running: \"mecntr-waitcntr; print \"   Waiting:\
 \"waitcntr; print \"      Resource: \"waitcntr-smartcntr; print \"      Designed: \"smartcntr; print \"      Orphaned: \"neversatisfiedcnt\
r; }' - && date"</pre>

<pre>alias qstat="squeue -u `whoami` -o '%.18F %.3p %.40j %.8u %.2t %.10M %.9P %.6C %.10K %R'"</pre>

<pre>alias qstata="squeue -o '%.18F %.3p %.40j %.8u %.2t %.10M %.9P %.6C %.10K %R'"</pre>
