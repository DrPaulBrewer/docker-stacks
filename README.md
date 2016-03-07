# This is my personal, experimental fork of jupyter/docker-stacks

The official site is at https://github.com/jupyter/docker-stacks

What's here:

##iopy-spark-notebook

based on all-spark-notebook

but built with
  *  openjdk-8
  *  spark-1.6.2 snapshot
  *  Apache toree pip release
  *  ubuntu docker containers instead of debian

The target environment is a large cloud VM, such as google n1-highcpu-32 or amazon EC2 c3-8xlarge or similar.

Default spark driver memory is set to 24 Gig.  You'll probably want to override this if you have less, or a lot more.

Spark master is set up as 'local[*]'

###usage

<pre>
docker run -d \
  -p 443:8888 \
  -p 80:4040 \
  -e PASSWORD=somethingclever \
  -e USE_HTTPS="yes" \
  -v /path/to/work/files:/home/jovyan/work \
   drpaulbrewer/iopy-spark-notebook
</pre>

add `-e SPARK_OPTS="--driver-memory 4G" ` or some other amount to override default of 24G set herein.
