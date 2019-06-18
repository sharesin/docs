### Solr8.1.1源码编译环境搭建文档


#### 1. 安装 apache-ant-1.10.3 环境

官网下载地址: [http://ant.apache.org/](http://ant.apache.org/)

#### 2. 下载Solr8.1.1源码

官网下载地址: [http://lucene.apache.org/solr/](http://lucene.apache.org/solr/)

#### 3. 解压源码文件，目录如下：

```
--
	build.xml		
	dev-tools
	LICENSE.txt
	lucene
	NOTICE.txt
	README.md
	solr
```
可以查看README.md与dev-tools下面文档进行相关环境搭建。本文档现仅介绍Eclipse环境搭建。

#### 4. 进入解压目录，输入命令：ant eclipse

#### 5. 导入项目到Eclips

#### 6. 集成IK分词器,添加至solr-8.1.1/solr/webapp/web/WEB-INF/lib目录，新增配置文件至solr-8.1.1/solr/webapp/ik。

```
jar包
IKAnalyzerFS6.3.jar

配置文件
casdd_author_name_distinct.dic
IKAnalyzer.cfg.xml
stopword-agri-20170930.dic
stopword-en.dic
stopword-zh.dic
words_agri_cat_cnki_zh_20170508_28w.dic
words_agri_cat_nstl_cnki_agrovoc_en_20170509_26w.dic
words_organization_en_zh_15490_20171020.dic

```

#### 7. 集成lucene-skos

```
jar包

javax.ws.rs-api-2.1.1.jar
jena-arq-3.11.0.jar
jena-base-3.11.0.jar
jena-core-3.11.0.jar
jena-iri-3.11.0.jar
jena-shaded-guava-3.11.0.jar
libthrift-0.9.2.jar
lucene-skos-0.8.1.0-sources.jar
lucene-skos-0.8.1.0.jar
```

