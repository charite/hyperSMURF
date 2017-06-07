[![Build Status](https://travis-ci.org/charite/hyperSMURF.svg?branch=master)](https://travis-ci.org/charite/hyperSMURF)
[![Documentation](https://readthedocs.org/projects/jannovar/badge/?version=master)](https://hyperSMURF.readthedocs.org)
[![API Docs](https://img.shields.io/badge/api-v0.3-blue.svg?style=flat)](https://javadoc.io/doc/de.charite.compbio/hyperSMURF/)
[![license](https://img.shields.io/badge/licence-GNU%20GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0.txt)


# hyperSMURF

Weka implementation of hyperSMURF using EasyEnsemble and SMOTE.

Please read the [manual](https://hyperSMURF.readthedocs.org) for more information, a detailed installation descriotion and a tutorial.

## Requirements

hyperSMURF requires java 8 and higher. It can be used as a [Weka](http://www.cs.waikato.ac.nz/~ml/weka/) plugin using version 3.9 or higher. To build the program it is recommended to use [Maven](https://maven.apache.org/).

## Installation from Github sources

To use hyperSMURF in [Weka](http://www.cs.waikato.ac.nz/~ml/weka/) three steps are needed.

1. Clone this repository.
2. Compile the java classes and create a Weka plugin using Maven   
3. Load the plugin into your Weka Package Manager

### Clone this repository

Use your terminal and go to a folder where you want to checkout hyperSMURF. Then run:

```
git clone https://github.com/charite/hyperSMURF.git
```

### Compile the java classes and create the Weka plugin

Go to your repository and create a jar file of hyperSMURF using Maven.

```
cd hyperSMURF

mvn clean install package
```

Now you should have the  `hyperSMURF-0.3.jar` in the folder `target/`. The package phase of Maven creates also the Weka  file `hyperSMURF-0.3-weka.zip`. It is located in the `target/` folder.

### Load the plugin into your Weka Package Manager

Open Weka, go to the package manager, and load the file `hyperSMURF-0.3-weka.zip` into it.  Look at the [Weka wiki](http://weka.wikispaces.com/How+do+I+use+the+package+manager%3F) for more information about the Weka Package Manager.

## Installation using Maven

If you want to include hyperSMURF into your java project you can use Maven to download the necessary files from Maven Central. You have to add this code under the `dependencies` section in your `pom.xml`:

```
<dependency>
	<groupId>de.charite.compbio</groupId>
	<artifactId>hyperSMURF</artifactId>
	<version>0.3</version>
</dependency>
```  

## Citation

Please cite our [Scientific Reports article](https://doi.org/10.1038/s41598-017-03011-5):

```
M. Schubach, M. Re, P. N. Robinson, and G. Valentini. (2017).
Imbalance-Aware Machine Learning for Predicting Rare and Common Disease-Associated Non-Coding Variants.
Scientific Reports, 7.
```
