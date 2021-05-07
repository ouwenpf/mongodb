FROM source_centos7:latest

ENV basedir mongodb
ENV datadir mongodb27017

	
RUN if [ ! -d /usr/local/mongodb ];then ln -s /application/$basedir   /usr/local/mongodb; fi

RUN if ! id mongodb &> /dev/null ;then useradd mongodb ; fi
RUN if [ ! -d /data/mongodb/$datadir ];then mkdir -p /data/mongodb/$datadir/{data,logs,conf}  &&  chown -R mongodb.mongodb /data/$basedir/$datadir ;fi

RUN sed   -ri 's#local/bin#local/bin:/application/mongodb/bin#g'     /home/mongodb/.bash_profile



COPY mongodb.conf /data/mongodb/$datadir/conf
#COPY sysctl.conf /etc/ 
COPY oneinstack.sh /etc/profile.d/
#ADD key.tar.gz /root/.ssh 
RUN chmod +x  /etc/rc.d/rc.local  \
&& echo "sed  -ri  '/sed/,/insecure/d'  /etc/rc.local" >> /etc/rc.d/rc.local 
WORKDIR /data/mysql/$datadir
EXPOSE 27017 127017













