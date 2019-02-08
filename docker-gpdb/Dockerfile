# Extended Dan Baskette's repo - https://github.com/dbbaskette/gpdb-docker
FROM centos:6.7

COPY pkgs/* /tmp/
COPY configs/* /tmp/
ADD get-pip.py get-pip.py
RUN echo root:pivotal | chpasswd \
        && yum install -y unzip which tar more util-linux-ng passwd openssh-clients openssh-server ed m4 tk libgomp; yum clean all \
        && yum install -y /tmp/greenplum-db-5.16.0-rhel6-x86_64.rpm \
        && cat /tmp/sysctl.conf.add >> /etc/sysctl.conf \
        && cat /tmp/limits.conf.add >> /etc/security/limits.conf \
        && rm -f /tmp/*.add \
        && echo "localhost" > /tmp/gpdb-hosts \
        && chmod 777 /tmp/gpinitsystem_singlenode \
        && hostname > ~/orig_hostname \
        && mv /tmp/run.sh /usr/local/bin/run.sh \
        && chmod +x /usr/local/bin/run.sh \
        && /usr/sbin/groupadd gpadmin \
        && /usr/sbin/useradd gpadmin -g gpadmin -G wheel \
        && echo "pivotal"|passwd --stdin gpadmin \
        && echo "gpadmin        ALL=(ALL)       NOPASSWD: ALL" >> /etc/sudoers \
        && mv /tmp/bash_profile /home/gpadmin/.bash_profile \
        && chown -R gpadmin: /home/gpadmin \
        && mkdir -p /gpdata/master /gpdata/segments \
        && chown -R gpadmin: /gpdata \
        && chown -R gpadmin: /usr/local/green* \
        && service sshd start \
        && su gpadmin -l -c "source /usr/local/greenplum-db/greenplum_path.sh;gpssh-exkeys -f /tmp/gpdb-hosts"  \
        && su gpadmin -l -c "source /usr/local/greenplum-db/greenplum_path.sh;gpinitsystem -a -c  /tmp/gpinitsystem_singlenode -h /tmp/gpdb-hosts; exit 0 "\
        && su gpadmin -l -c "export MASTER_DATA_DIRECTORY=/gpdata/master/gpseg-1;source /usr/local/greenplum-db/greenplum_path.sh;psql -d template1 -c \"alter user gpadmin password 'pivotal'\"; createdb gpadmin;  exit 0"\
        && su gpadmin -l -c "gppkg -i /tmp/DataSciencePython-1.1.1-gp5-rhel6-x86_64.gppkg" \
        && su gpadmin -l -c "echo -ne 'y' | source /usr/local/greenplum-db/greenplum_path.sh" \
        && su gpadmin -l -c "echo -ne 'y' | gpstop -ra" \
        && rm /tmp/DataSciencePython-1.1.1-gp5-rhel6-x86_64.gppkg \
        && tar -xvf /tmp/madlib-1.15.1-gp5-rhel6-x86_64.tar.gz -C /tmp/ \
        && rm /tmp/madlib-1.15.1-gp5-rhel6-x86_64.tar.gz \
        && su gpadmin -l -c "gppkg -i /tmp/madlib-1.15.1-gp5-rhel6-x86_64/madlib-1.15.1-gp5-rhel6-x86_64.gppkg" \
        && rm /tmp/madlib-1.15.1-gp5-rhel6-x86_64/madlib-1.15.1-gp5-rhel6-x86_64.gppkg \
        && su gpadmin -l -c "/usr/local/greenplum-db/madlib/bin/madpack -s madlib -p greenplum -c gpadmin@localhost:5432/gpadmin install" \
        && su gpadmin -l -c "echo -ne 'y' | source /usr/local/greenplum-db/greenplum_path.sh" \
        && su gpadmin -l -c "echo -ne 'y' | gpstop -ra" 

#&& su gpadmin -l -c "psql -f /usr/local/greenplum-db/share/postgresql/plcontainer/plcontainer_install.sql -d gpadmin" \
#&& su gpadmin -l -c "plcontainer configure --reset" \
#&& chown gpadmin -R /tmp/plcontainer-python-images-1.0.0_beta1.tar.gz \
#&& su gpadmin -l -c "plcontainer install -n plc_python_shared -i /tmp/plcontainer-python-images-1.0.0_beta1.tar.gz -c pivotaldata/plcontainer_python_shared:devel -l python;"


EXPOSE 5432 22

VOLUME /gpdata
# Set the default command to run when starting the container

CMD echo "127.0.0.1 $(cat ~/orig_hostname)" >> /etc/hosts \
        && service sshd start \
#       && sysctl -p \
        && su gpadmin -l -c "/usr/local/bin/run.sh" \
        && /bin/bash
RUN  echo /usr/local/greenplum-db/ext/python/lib/ >> /etc/ld.so.conf \
     && ldconfig
 
RUN  yum install -y /tmp/jdk-8u201-linux-x64.rpm \
    && su gpamdin -l -c "source /usr/local/greenplum-db/greenplum_path.sh;echo \"export JAVA_HOME=/data1/jdk/jdk1.8.0_191\" >> $GPHOME/greenplum_path.sh" \
    && su gpamdin -l -c "source /usr/local/greenplum-db/greenplum_path.sh;echo \"export PATH=\$JAVA_HOME/bin:\$PATH\" >> $GPHOME/greenplum_path.sh" \
    && yum install nc