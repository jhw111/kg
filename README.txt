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

启动D2RQ服务
d2r-server.bat kg_movie.ttl
访问地址：http://localhost:2020/

下载解压Jena，配置环境变量
JENA_HOME=XX
path+=%JENA_HOME%\bat;%JENA_HOME%\bin
命令行执行sparql -h成功

将RDF数据转换以TDB的方式存储
tdbloader.bat --loc="E:\install\apache-jena-3.13.1\tdb" "E:\install\d2rq-0.8.1\kg_movie.nt"

运行fuseki服务器
fuseki-server --loc=E:\install\apache-jena-3.13.1\tdb /kg_movie
访问
http://localhost:3030

导出mysql到csv(导出路径须和mysql配置secure-file-priv一致)
SELECT * FROM genre into outfile 'E:/install/codecheck/mysql-5.7.26-winx64/temp/kg_genre.csv' character set utf8 fields terminated by ',' optionally enclosed by '"' lines terminated by '\r\n';
SELECT * FROM actor into outfile 'E:/install/codecheck/mysql-5.7.26-winx64/temp/kg_actor.csv' character set utf8 fields terminated by ',' optionally enclosed by '"' lines terminated by '\r\n';
SELECT * FROM movie into outfile 'E:/install/codecheck/mysql-5.7.26-winx64/temp/kg_movie.csv' character set utf8 fields terminated by ',' optionally enclosed by '"' lines terminated by '\r\n';
SELECT * FROM actor_to_movie into outfile 'E:/install/codecheck/mysql-5.7.26-winx64/temp/kg_actor_to_movie.csv' character set utf8 fields terminated by ',' optionally enclosed by '"' lines terminated by '\r\n';
SELECT * FROM movie_to_genre into outfile 'E:/install/codecheck/mysql-5.7.26-winx64/temp/kg_movie_to_genre.csv' character set utf8 fields terminated by ',' optionally enclosed by '"' lines terminated by '\r\n';

导入csv到neo4j(将csv文件放于neo4j的import目录)
USING PERIODIC COMMIT 100
LOAD CSV FROM 'file:///kg_genre.csv' AS line CREATE (g:Genre { genre_id:  line[0], genre_name: line[1]  });
USING PERIODIC COMMIT 100
LOAD CSV FROM 'file:///kg_actor.csv' AS line CREATE (a:Actor { actor_id: line[0], actor_bio: line[1], actor_chName: line[2], actor_foreName: line[3],actor_nationality: line[4], actor_constellation: line[5], actor_birthPlace:  line[6], actor_birthDay: line[7], actor_repWorks: line[8], actor_achiem: line[9], actor_brokerage: line[10] })
USING PERIODIC COMMIT 100
LOAD CSV FROM 'file:///kg_movie.csv' AS line CREATE (m:Movie { movie_id: line[0], movie_bio: line[1], movie_chName: line[2], movie_foreName: line[3],movie_prodTime: line[4], movie_prodCompany: line[5], movie_director: line[6], movie_screenwriter: line[7], movie_genre: line[8], movie_star: line[9], movie_length: line[10], movie_rekeaseTime: line[11], movie_language: line[12],  movie_achiem: line[13]  });
USING PERIODIC COMMIT 300
LOAD CSV FROM 'file:///kg_actor_to_movie.csv' AS line MATCH (a:Actor), (m:Movie) WHERE a.actor_id = line[1] AND m.movie_id = line[2] CREATE (a) - [r:ACTED_IN] -> (m) RETURN r;
USING PERIODIC COMMIT 300
LOAD CSV FROM 'file:///kg_actor_to_movie.csv' AS line MATCH (a:Actor), (m:Movie) WHERE a.actor_id = line[1] AND m.movie_id = line[2] CREATE (a) - [r:ACTED_IN] -> (m) RETURN r;
USING PERIODIC COMMIT 300
LOAD CSV FROM 'file:///kg_movie_to_genre.csv' AS line MATCH (m:Movie), (g:Genre) WHERE m.movie_id = line[1] AND g.genre_id = line[2] CREATE (m) - [r:Belong_to] -> (g) RETURN r;
