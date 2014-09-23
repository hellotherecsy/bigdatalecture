-----------accumulo 설치 ------------

먼저 ACCUMULO setting!!
SQOOP HDP 2.1 accumulo
http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.1.2.1/README-HDP-2.1.2.1.txt


1. accumulo 설치
yum install accumulo

---------------------mysql 설치 ----------------------------------
1. My sql 설치 
yum install mysql

2. start 
service mysqld start

3. root 암호 만들기
mysqladmin -u root password root


3. 접속 
mysql -u root -p  

4. database 만들기 
create database sqoop;

5. 확인 
show databases;

6. 접속
use sqoop; 

7. 유저생성
create user 'sqoop'@'localhost' identified by 'sqoop';
create user 'sqoop'@'bigdata01-01' identified by 'sqoop';


8.권한부여

grant all privileges on *.* to 'sqoop'@'localhost';
GRANT ALL PRIVILEGES ON *.* TO 'sqoop'@'localhost' IDENTIFIED BY 'sqoop' WITH GRANT OPTION;


GRANT ALL PRIVILEGES ON *.* TO 'sqoop'@'${hostname1}' IDENTIFIED BY 'sqoop' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON *.* TO 'sqoop'@'${hostname2}' IDENTIFIED BY 'sqoop' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON *.* TO 'sqoop'@'${hostname3}' IDENTIFIED BY 'sqoop' WITH GRANT OPTION;

GRANT ALL PRIVILEGES ON *.* TO 'sqoop'@'bigdata01-01' IDENTIFIED BY 'sqoop' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON *.* TO 'sqoop'@'bigdata01-02' IDENTIFIED BY 'sqoop' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON *.* TO 'sqoop'@'bigdata01-03' IDENTIFIED BY 'sqoop' WITH GRANT OPTION;



flush privileges;

------------------------ data upload ---------------

데이터 업로드 
hadoop fs -mkdir -p  /upload/acc/
hadoop fs -put Accidents0512.csv /upload/acc/.
hadoop fs -ls /upload/acc/
------------------------------------------


-----------------------------sqoop 설치--------------------

1-1. yum 설치
wget -nv http://public-repo-1.hortonworks.com/HDP/centos6/1.x/GA/1.3.0.0/hdp.repo -O /etc/yum.repos.d/hdp.repo
wget -nv http://public-repo-1.hortonworks.com/ambari/centos6/1.x/updates/1.2.4.9/ambari.repo -O /etc/yum.repos.d/ambari.repo
wget http://mirror.apache-kr.org/sqoop/1.4.4/sqoop-1.4.4.bin__hadoop-0.20.tar.gz 
yum install sqoop


export HADOOP_COMMON_HOME=/usr/local/hadoop/hadoop-1.2.1
export HADOOP_MAPRED_HOME=/usr/local/hadoop/hadoop-1.2.1

1-2. tar 설치 
mkdir /home/hadoop/sqoop
cd /home/hadoop/sqoop
wget http://mirror.apache-kr.org/sqoop/1.4.4/sqoop-1.4.4.bin__hadoop-0.20.tar.gz 
tar zxvf mysql-connector-java-5.1.31.tar.gz sqoop-1.4.4.bin__hadoop-0.20.tar.gz 

3. Mysql Connector 설치
wget  http://download.softagency.net/MySQL/Downloads/Connector-J/mysql-connector-java-5.1.31.tar.gz    
tar zxvf mysql-connector-java-5.1.31.tar.gz
cp ./mysql-connector-java-5.1.31/mysql-connector-java-5.1.31-bin.jar ./lib/.
ls -al cp ./mysql-connector-java-5.1.31/mysql-connector-java-5.1.31-bin.jar ./lib/.

-------------------------------sqoop 실행 -----------------------------------
sqoop 실행 ( user 주의 !!  : hdfs 접속 권한  ) 
1. sqoop export
sqoop-export    --connect jdbc:mysql://192.168.122.201/sqoop  --username sqoop --password sqoop  --table accidents --export-dir  /upload/acc/ --input-fields-terminated-by  "," 

2. sqoop import - key split  

(1) key split 
 : sqoop-import    --connect jdbc:mysql://192.168.122.201/sqoop  --username sqoop --password sqoop  --table accidents --target-dir /upload/acc_import/ --table accidents --split-by f1     --null-string “”
 : sqoop-import    --connect jdbc:mysql://192.168.122.201/sqoop  --username sqoop --password sqoop  --table accidents --target-dir /upload/acc_import2/ --table accidents --split-by f2     --null-string “”

(2) free query 
 : sqoop-import    --connect jdbc:mysql://192.168.122.201/sqoop  --username sqoop --password sqoop    --target-dir /upload/acc_import4/  -e "select f1,f2,f3 from accidents where \$CONDITIONS" --split-by f1    --null-string “”

(3) sequnce type / avro type import 
 : sqoop-import --connect jdbc:mysql://192.168.122.201/sqoop  --username sqoop --password sqoop  --table accidents --target-dir /upload/acc_import_seq/ --table accidents --split-by f1     --null-string “”  --as-sequencefile
 : sqoop-import --connect jdbc:mysql://192.168.122.201/sqoop  --username sqoop --password sqoop  --table accidents --target-dir /upload/acc_import_avro/ --table accidents --split-by f1     --null-string “”  --as-avrodatafile
(4) sequnce type / avro type export
 sqoop-export    --connect jdbc:mysql://192.168.122.201/sqoop  --username sqoop --password sqoop  --table accidents --export-dir  /upload/acc_import_avro/ --input-fields-terminated-by  "," 
 sqoop-export    --connect jdbc:mysql://192.168.122.201/sqoop  --username sqoop --password sqoop  --table accidents --export-dir  /upload/acc_import_seq/ --input-fields-terminated-by  "," 

(5) hive import 
  sqoop-import --connect jdbc:mysql://192.168.122.201/sqoop  --username sqoop --password sqoop  --table accidents --target-dir /upload/acc_import_hive/ --table accidents --hive-import --create-hive-table --hive-table accidents --fields-terminated-by ,  -m 1





sqoop lib 위치 
/usr/lib/sqoop/lib/mysql-connector-java.jar 








