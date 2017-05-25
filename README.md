# CKAN

#### Introduce
- CKAN은 데이터에 접근가능한 데이터 관리 시스템
    - 제공된 툴에 의해 데이터 publishing, 공유, 데이터 사용 및 검색 가능
- CKAN은 데이터가 공유되길 원하는 국제/지역 정부, 회사, 조직의 Data Publisher를 목표로 만들어짐
- CKAN은 open Data 웹사이트를 만들기 위한 tool
    + 워드프레스같은 컨텐츠 관리 시스템으로 생각하면 되나, 블로그 포스트나 웹페이지 대신에 데이터를 사용한다
- CKAN은 관리, 수집한 데이터를 업로드하는 것을 도와준다
- 국제/국내 정부, 조사 기관, 많은 데이터를 수집한 다른 조직에 의해서 사용된다
- 한번 데이터를 업로드하면, 사용자는 필요한 데이터를 브라우져에서 찾을 수 있고, 지도,그래프,테이블을 사용하여 미리 볼 수 있다

#### Installing
>**Environment**
>- OS : CentOS 7.2.1511 
>- CKAN : 2.6.2
>- JAVA : 1.8.0_65
>- PYTHON : 2.7.5

#### Install basic package
    - Python : v2.7
    - PostgreSQL : v9.2 or newer(use v9.6.3)
    - libpq
    - pip
    - virtualenv
    - Git
    - Apache Solr
    - Tomcat : v7
    - OpenJDK JDK : 1.8.0_131

#### How to Install CKAN in CENTOS7
**1. install Python**  

**2. install PostgreSQL v9.6.3**  

    # rpm -Uvh https://download.postgresql.org/pub/repos/yum/9.6/redhat/rhel-7-x86_64/pgdg-centos96-9.6-3.noarch.rpm

**3. install libpq**

    # yum install libpq

**4. install virtualenv**
    
    # pip install virtualenv    

**5. install Git**

    # yum install git

**6. install Apache Solr**

    // CKAN cannot use the latest version of solr  
    # curl http://archive.apache.org/dist/lucene/solr/1.4.1/apache-solr-1.4.1.tgz | tar xzf -

**7. install Tomcat**

**8. install CKAN**  
- make directory


    # mkdir -p ~/ckan/lib
    # sudo ln -s ~/ckan/lib /usr/lib/ckan
    # mkdir -p ~/ckan/etc
    # sudo ln -s ~/ckan/etc /etc/ckan

- create Python vitual envireonment to install CKAN into, and activate it


    # sudo mkdir -p /usr/lib/ckan/default
    # sudo chown `whoami` /usr/lib/ckan/default
    # virtualenv --no-site-packages /usr/lib/ckan/default
    # . /usr/lib/ckan/default/bin/activate

- install setuptools


    # pip install -U setuptools
    
    
- install CKAN into virtualenv
    

    // CKAN latest version
    (default)# pip install -e 'git+https://github.com/ckan/ckan.git#egg=ckan'

- install python modules for CKAN requires into virtualenv

    
    (default)# python setup.py build
    (default)# python setup.py install
    (default)# pip install -r /usr/lib/ckan/default/src/ckan/pip-requirements-docs.txt

**9. setup a PostgreSQL database**


    # sudo -u postgres psql -l

    //result
                                         List of databases
         Name     |    Owner     | Encoding |   Collate   |    Ctype    |   Access privileges   
    --------------+--------------+----------+-------------+-------------+-----------------------
     ambari       | postgres     | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =Tc/postgres         +
                  |              |          |             |             | postgres=CTc/postgres+
                  |              |          |             |             | ambari=CTc/postgres
     ckan_default | ckan_default | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
     postgres     | postgres     | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
     template0    | postgres     | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
                  |              |          |             |             | postgres=CTc/postgres
     template1    | postgres     | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
                  |              |          |             |             | postgres=CTc/postgres
    (5 rows)  


    // create a database user 
    # sudo -u postgres createuser -S -D -R -P ckan_default

    // create a new PostgreSQL database
    # sudo -u postgres createdb -O ckan_default ckan_default -E utf-8

**10. Create a CKAN config file**

    
    // create a directory to contain the site's config files
    # sudo mkdir -p /etc/ckan/default
    # sudo chown -R `whoami` /etc/ckan/
    # sudo chown -R `whoami` ~/ckan/etc

    // edit development.ini file by a text editor
    # vim /etc/ckan/default/development.ini
        sqlalchemy.url = postgresql://ckan_default:{password => xiirocks}@localhost/ckan_default
        ckan.site_id = default
        ckan.site_url = http://demo.ckan.org

    // create CKAN config file
    # paster make-config ckan /etc/ckan/default/development.ini

**11. Setup Solr**

    
    // create directories to hold multiple Solr cores
    # mkdir -p /usr/share/solr/core0 /usr/share/solr/core1 /var/lib/solr/data/core0 \
    /var/lib/solr/data/core1 /etc/solr/core0 /etc/solr/core1
    
    // Copy the Apache SOLR war to the desired location
    # cp apache-solr-1.4.1/dist/apache-solr-1.4.1.war /usr/share/solr

    // Copy the example Apache SOLR configuration to the core0 directory
    # cp -r apache-solr-1.4.1/example/solr/conf /etc/solr/core0

    // Edit the configuration file, /etc/solr/core0/conf/solrconfig.xml, as follows:
        <dataDir>${dataDir}</dataDir>

    // Copy the core0 configuration to core1
    # cp -r /etc/solr/core0/conf /etc/solr/core1

    // Create a symbolic link between the configurations in /etc and /usr
    # ln -s /etc/solr/core0/conf /usr/share/solr/core0/conf
    # ln -s /etc/solr/core1/conf /usr/share/solr/core1/conf

    // Remove the provided schema from the two configured cores and link the schema files in the CKAN source
    # rm -f /etc/solr/core0/conf/schema.xml
    # ln -s /usr/lib/ckan/default/src/ckan/ckan/config/solr/schema.xml /etc/solr/core0/conf/schema.xml
    # rm -f /etc/solr/core1/conf/schema.xml
    # ln -s /usr/lib/ckan/default/src/ckan/ckan/config/solr/schema-1.4.xml /etc/solr/core1/conf/schema.xml

    // Create a new file, called /etc/tomcat/Catalina/localhost/solr.xml
    <Context docBase="/usr/share/solr/apache-solr-1.4.1.war" debug="0" privileged="true" allowLinking="true" crossContext="true">
        <Environment name="solr/home" type="java.lang.String" value="/usr/share/solr" override="true" />
    </Context>

    // Create a new file, called /usr/share/solr/solr.xml
    <solr persistent="true" sharedLib="lib">
        <cores adminPath="/admin/cores">
            <core name="ckan-schema" instanceDir="core0">
                <property name="dataDir" value="/var/lib/solr/data/core0" />
            </core>
            <core name="ckan-schema-1.4" instanceDir="core1">
                <property name="dataDir" value="/var/lib/solr/data/core1" />
            </core>
        </cores>
     </solr>

     // Set Permissions
     // Make tomcat the owner of the Solr directories
     # chown -R tomcat:tomcat /usr/share/solr /var/lib/solr

**12. Tomcat 방화벽 설정 및 활성화**


     // Enable Tomcat
     // Configure Tomcat to start on system boot
     # systemctl enable tomcat   // 부팅시 서비스 활성화

     // start tomcat
     # systemctl start tomcat.service


**13. CKAN DB 생성**


     // enter virtualenv 
     # . /usr/lib/ckan/default/bin/activate

     // into directory
     # cd /usr/lib/ckan/default/src/ckan

     // create db
     # paster db init -c /etc/ckan/default/development.ini

- link to who.ini


    # ln -s /usr/lib/ckan/default/src/ckan/who.ini /etc/ckan/default/who.ini

**14. CKAN Server Start**


    # . /usr/lib/ckan/default/bin/activate
    # cd /usr/lib/ckan/default/src/ckan

    // check port 5000 in your browser
    # paster serve /etc/ckan/default/development.ini


#### Referrence
<http://docs.ckan.org/en/latest/maintaining/installing/install-from-source.html#setup-solr>  
<https://github.com/ckan/ckan/wiki/How-to-install-CKAN-2.5.2-on-CentOS-6.8>
