
<h1 align="center">
  <br>
  <img src="http://www.girliemac.com/assets/images/articles/2017/01/cover-facebook-apiai-bot.png" alt="ELK Stack" width="700">
  <br>
Entrepôt de données pour les services d’enseignement  <br>
</h1>

<h4 align="center">My part consist of an implementation of a data warehouse using ELK stack (Elastic Search - Logstash - Kibana), in order to track the evolution of teaching services of the university in the course of the years the application COSTER was running</h4> 

<br>

## Demo
<div align="center" >


<img src="https://github.com/GueddouChaouki/Google-Calendar-AWS/blob/master/Totu/AWS1.gif" width="980">


<img src="https://github.com/ilyes16K/ChatBot_nodeJS_TER_Projet/blob/master/Main_App%20(API.AI)/workspace/screenshots/ezgif.com-video-to-gif2.gif" width="280">

</div>

## prerequisite
You need to download and install : 
* [LogStach](https://www.elastic.co/guide/en/logstash/current/installing-logstash.html)
*  [Elastic Search](https://www.elastic.co/guide/en/beats/libbeat/current/elasticsearch-installation.html)
*  [Kibana](https://www.elastic.co/guide/en/kibana/6.2/install.html)
* [MySql](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-16-04)
---
## Development Setup
### 1. LogStach
To configure Logstash, you create a config file that specifies which plugins you want to use and settings for each plugin.
the configuration file consist mainly of tree parts witch are the input, filter, and the output and it can be used with several data sources such as CSV files and Data Bases:

#### 1.1. Command to execute the LogStash script 
```
sudo /usr/share/logstash/bin/logstash -f /home/gueddou/Desktop/postgres_R_ELK_stack/DbLogStash.config
```
#### 1.2. LogStach with CSV file
##### 1.2.1. Structure of the configuration file
```
# under file 
# we specify the path, start_position, sincedb_path(optionaly)

input{
	file{
		path => "/home/gueddou/Desktop/postgres_R_ELK_stack/servicesBIOTBallYears.csv"
		start_position => "beginning"
		sincedb_path => "/dev/null"
	}
}

# under csv we specify the separator and the columns of the csv file
# under mutate we have the possibility to convert the type of  previousily specified columns or add/drop a column
# under date we can set an attribute as a timestamp


filter {
	csv{
		separator => ","          
		columns=> ["Grade","NOM","PRENOM","DU","RED","FAIT","BILAN","CM","TD","TP","HEURES COMPL","PRIMES","ANNEE"]
	   }
	mutate {convert => ["DU", "float"]}
	mutate {convert => ["RED", "float"]}
	mutate {convert => ["FAIT", "float"]}
	mutate {convert => ["BILAN", "float"]}
	mutate {convert => ["CM", "float"]}
	mutate {convert => ["TD", "float"]}
	mutate {convert => ["TP", "float"]}
	mutate {convert => ["HEURES COMPL", "float"]}
	mutate {convert => ["PRIMES", "float"]}
	

   	date {
	    locale => "fr"
		match => ["ANNEE", "yyyy", "ISO8601"]
		target => "ANNEE"
           }	
	
	mutate {
  		add_field => {
    			"NomPrenom" => "%{NOM} %{PRENOM}"
  			}
	}

}

# under elasticsearch 
# we specify the hosts, index, document_type(optionaly)

output {
	elasticsearch {
		hosts => "localhost"
		index => "sept_ans_service_bio"
		document_type => "sept_ans_service_bio_stat"
	}
	stdout {}
}
```


#### 1.3. LogStach with DataBase file

##### 1.3.1. Structure of the configuration file
```
# under jdbc 
# we specify the path to jar file, jdbc_driver_class, jdbc_user and password, and finaly the SQL statement.

input {
  jdbc {
    jdbc_driver_library => "/home/gueddou/Desktop/postgres_R_ELK_stack/mysql-connector-java-5.1.45-bin.jar"
    jdbc_driver_class => "com.mysql.jdbc.Driver"
    jdbc_connection_string => "jdbc:mysql://localhost/svrbio_2013"

    jdbc_user => "root"
	jdbc_password => "85208520"
    statement => "SELECT sub1.nom, sub1.prenom, sub1.enseignantID, codecourt, SUM( heuresCMTOT ) AS CM, SUM( heuresCMTOT ) * 1.5 AS CMeqTD, SUM( heuresTDTOT ) AS TD, SUM( heuresTPTOT ) AS TP, SUM( heuresCMTOT ) * 1.5 + SUM( heuresTDTOT ) + SUM( heuresTPTOT) AS TOTAL, grades.heures-(SUM( heuresCMTOT ) * 1.5 + SUM( heuresTDTOT )+ SUM( heuresTPTOT )) as bilan, decharge, reduction, heures, 2013 as annee_current
FROM (

SELECT nom, prenom, sum( heuresCM ) AS heuresCMTOT, 0 AS heuresTDTOT, 0 AS heuresTPTOT, e.enseignantID, codegrade
FROM enseignants AS e, modules, menusemestre, preserviceCM
WHERE menusemestre.codemod = modules.codemod
AND preserviceCM.codemodsemestre = menusemestre.codemodsemestre
AND preserviceCM.enseignantID = e.enseignantID
GROUP BY nom, prenom, codegrade
UNION SELECT nom, prenom, 0 AS heuresCMTOT, SUM( heuresTD ) AS heuresTDTOT, 0 AS heuresTPTOT, e.enseignantID, codegrade
FROM enseignants AS e, modules, menusemestre, preserviceTD
WHERE menusemestre.codemod = modules.codemod
AND preserviceTD.codemodsemestre = menusemestre.codemodsemestre
AND preserviceTD.enseignantID = e.enseignantID
GROUP BY nom, prenom, codegrade
UNION SELECT nom, prenom, 0 AS heuresCMTOT, 0 AS heuresTDTOT, SUM( heuresTP ) AS heuresTPTOT, e.enseignantID, codegrade
FROM enseignants AS e, modules, menusemestre, preserviceTP
WHERE menusemestre.codemod = modules.codemod
AND preserviceTP.codemodsemestre = menusemestre.codemodsemestre
AND preserviceTP.enseignantID = e.enseignantID
GROUP BY nom, prenom, codegrade
) AS sub1 LEFT OUTER JOIN decharges d ON sub1.enseignantID = d.enseignantID LEFT OUTER JOIN reduction r on r.enseignantID = sub1.enseignantID, grades
WHERE sub1.codegrade = grades.codegrade
GROUP BY nom, prenom
ORDER BY codecourt, nom, prenom"
	
  }
}

# under mutate we have the possibility to convert the type of 
# previousily specified columns or add/drop a column
# under date we can set an attribute as a timestamp

filter {
	mutate {convert => ["enseignantID", "float"]}
	mutate {convert => ["heures", "float"]}
	mutate {convert => ["CM", "float"]}
	mutate {convert => ["TD", "float"]}
	mutate {convert => ["TP", "float"]}
	mutate {convert => ["Reduction", "float"]}
	mutate {convert => ["Prime", "float"]}
	mutate {convert => ["heures_faites", "float"]}
	mutate {convert => ["heures_Du", "float"]}
	mutate {convert => ["bilan", "float"]}



	date {
      match => [ "annee_current", "ISO8601", "YYYY-MM-dd HH:mm:ss" ]
      target => "annee_current"
      locale => "en"
    }

   		
	
	mutate {
  		add_field => {
    			"NomPrenom" => "%{nom} %{prenom}"
  			}
	}

}


# under elasticsearch 
# we specify the hosts, index, document_type(optionaly)
# under stdout we specify the formatting of the output 

output{
	elasticsearch {
	hosts => ["localhost:9200"] 
	index => "svrens_info_index"
	user => "gueddou"
    password => "85208520"
    
	
	
}
  stdout { codec => rubydebug { metadata => true } }
   # stdout { codec => dots }
}
```

### 2. ElasticSearch
Elasticsearch is a distributed, RESTful search and analytics engine capable of solving a growing number of use cases. As the heart of the Elastic Stack.

#### 2.1. some of elasticsearch use cases
```
GET /svrens_info_index/_count
{
  "query": {
    "term" : {"nom": "lopes"}
  }
}
{
  "count": 7,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  }
}
```

```
GET svrens_info_index
{
  "svrens_info_index": {
    "aliases": {},
    "mappings": {
      "logs": {
        "properties": {
          "@timestamp": {
            "type": "date"
          },
          "@version": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "NomPrenom": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "annee_current": {
            "type": "long"
          },
 .......
```

```
POST service2017/_delete_by_query
{
  "query": { 
    "match": {
      "Nom": "Kedad-Cointot"
    }
  }
}
{
  "took": 345,
  "timed_out": false,
  "total": 11,
  "deleted": 11,
  "batches": 1,
  "version_conflicts": 0,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1,
  "throttled_until_millis": 0,
  "failures": []
}
```

```
POST _reindex
{
 "conflicts": "proceed",
 "source": {
   "index": ["svrens2", "svrens1"]
  },
  "dest": {
  "index": "all_together"
 }
}
{
  "took": 550,
  "timed_out": false,
  "total": 309,
  "updated": 309,
  "created": 0,
  "deleted": 0,
  "batches": 1,
  "version_conflicts": 0,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1,
  "throttled_until_millis": 0,
  "failures": []
}
```

### 3. Kibana
Kibana lets you visualize your Elasticsearch data and navigate the Elastic Stack.
#### 3.1. create a visualization type
we follow the steps as it is shown in the following image

<img src="https://github.com/GueddouChaouki/Google-Calendar-AWS/blob/master/Totu/AWS1.gif" width="980">

#### 3.2. adding a visualization type to a dashboard
we follow the steps as it is shown in the following image

<img src="https://github.com/GueddouChaouki/Google-Calendar-AWS/blob/master/Totu/AWS1.gif" width="980">



## add your own contribution 
- Fork it!
- Create your feature branch: git checkout -b my-new-feature
- Commit your changes: git commit -am 'Add some feature'
- Push to the branch: git push origin my-new-feature
- Submit a pull request :D
## License
[MIT](https://tldrlegal.com/license/mit-license)
