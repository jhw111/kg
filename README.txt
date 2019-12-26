知识图谱

导入数据
create database kg_movie;
source C:\Users\WzNgs\Desktop\kg_movie-master\kg_movie.sql

创建本体
kg_movie_ontology.owl

下载安装D2RQ
将mysql-connector-java-5.1.44.jar文件放入d2rq的/lib文件夹中

用D2RQ的generate-mapping工具为数据库创建mapping file
在D2RQ目录执行命令：generate-mapping -u root -p 1 -o kg_movie.ttl jdbc:mysql://localhost:3306/kg_movie
去除kg_movie.ttl中的 vocab:
去除属性前缀，如：d2rq:property :actor_actor_bio;-->d2rq:property :actor_bio;

数据转化为RDF
dump-rdf.bat -o kg_movie.nt kg_movie.ttl