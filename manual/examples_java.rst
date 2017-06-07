.. role:: java(code)
   :language: java

.. _examples_java:

================================
Examples using the Java package
================================

In this section we will create a new Maven project, called hyperSMURF-tutorial, and using hyperSMURF together with Weka to train some tasks. Therefore we first have to set up an Maven project which will handle all libraries for us. Then we will start with a synthetic example. Afterwards we are using real genetic data for training. All files of this tutorial are available `here <https://www.github.com/charite/hyperSMURF-tutorial>`_

.. _requirements:

Requirements
=============

First we have to build a maven project and include the hyperSMURF library into the `pom.xml` file. Therefore we generate a new folder (`hyperSMURF-tutorial`) and then we generate a new `pom.xml` file.

.. code-block:: bash

	mkdir hyperSMURF-tutorial
	cd hyperSMURF-tutorial
	touch pom.xml

Then we open the `pom.xml` file in an editor and put in the following lines:

.. code-block:: xml

	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
		<modelVersion>4.0.0</modelVersion>
		<groupId>de.charite.compbio.hypersmurf</groupId>
		<artifactId>hyperSMURF-tutorial</artifactId>
		<packaging>jar</packaging>
		<version>0.2</version>
		<name>hyperSMURF-tutorial</name>
		<properties>
			<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		</properties>
		<dependencies>
			<dependency>
				<groupId>nz.ac.waikato.cms.weka</groupId>
				<artifactId>weka-dev</artifactId>
				<version>3.9.0</version>
			</dependency>
			<dependency>
				<groupId>de.charite.compbio</groupId>
				<artifactId>hyperSMURF</artifactId>
				<version>0.2</version>
			</dependency>
		</dependencies>
		<build>
			<plugins>
				<plugin>
					<artifactId>maven-assembly-plugin</artifactId>
					<executions>
						<execution>
							<phase>package</phase>
							<goals>
								<goal>single</goal>
							</goals>
						</execution>
					</executions>
					<configuration>
						<descriptorRefs>
							<descriptorRef>jar-with-dependencies</descriptorRef>
						</descriptorRefs>
					</configuration>
				</plugin>
			</plugins>
		</build>
	</project>


Now it we can generate Java files under the the folder `src/main/java` and to generate a final runnable jar-file we simply use the command `mvn clean package` to generate a jar fine into the `target` folder. Then we can run the `hyperSMURF-tutorial-0.2-jar-with-dependencies.jar` jar in `target` folder by specifying our main class (here SyntheticExample from the next section):

.. code-block:: bash

	java -cp target/hyperSMURF-tutorial-0.2-jar-with-dependencies.jar de.charite.compbio.hypersmurf.SyntheticExample

If you use Eclipse for developing a useful maven command might be `mvn eclipse:eclipse` to generate a project that can be imported into eclipse.

.. _synthetic:

Simple usage examples using synthetic data
==============================================

In this section we will add a new class `SyntheticExample.java` under the folder `src/main/java/de/charite/compbio/hypersmurf` to our maven project. This class has a main function to run it and two other main functions: (1) `generateSyntheticData` to generate synthetic imbalanced data and (2) `classify` to classify instances with a classifier using k-fold cross-validation.

The outline of the Java class `SyntheticExample.java` looks like this:

.. code-block:: java

	package de.charite.compbio.hypersmurf;

	public class SyntheticExample {

		/**
		 * We need a seed to make consistent predictions.
		 */
		private static int SEED = 42;

		public static void main(String[] args) throws Exception {

		}
	}

So the class only defines a seed to make predictions consistent. Then we use the RDG1 data generator from Weka to generate synthetic data. For example we will generate 10000 instances, each with 20 numeric attributes and set the index to the last attribute which contains class `c0` and `c1` by default. Then we randomize the data using our predefined seed:

.. code-block:: java

	RDG1 dataGenerator = new RDG1();
	dataGenerator.setRelationName("SyntheticData");
	dataGenerator.setNumExamples(10000);
	dataGenerator.setNumAttributes(20);
	dataGenerator.setNumNumeric(20);
	dataGenerator.setSeed(SEED);
	dataGenerator.defineDataFormat();
	Instances instances = dataGenerator.generateExamples();

	// set the index to last attribute
	instances.setClassIndex(instances.numAttributes() - 1);

	// randomize the data
	Random random = new Random(SEED);
	instances.randomize(random);


The problem is, that this data is not imbalanced. We can check this writing a short helper function.

.. code-block:: java

	private static int[] countClasses(Instances instances) {
		int[] counts = new int[instances.numClasses()];
		for (Instance instance : instances) {
			if (instance.classIsMissing() == false) {
				counts[(int) instance.classValue()]++;
			}
		}
		return counts;
	}

Now if we add :java:`int[] counts = countClasses(instances);` to our instance generation and print it using :java:`System.out.println("Before imbalancing: " + Arrays.toString(counts));` we will see that `c0` has 2599 and `c1` has 7401 instances.

To imbalance the data we will write some own code. For example we want to use only 50 instances of `c0`. So we have to generate a new `Instances` object add all `c1` class instances and only 50 `c0` class instances.

.. code-block:: java

	// imbalance data
	int numberOfClassOne = 50;
	Instances imbalancedInstances = new Instances(instances, counts[1] + numberOfClassOne);
	for (int i = 0; i < instances.numInstances(); i++) {
		if (instances.get(i).classValue() == 0.0) {
			if (numberOfClassOne != 0) {
				imbalancedInstances.add(instances.get(i));
				numberOfClassOne--;
			}
		} else {
			imbalancedInstances.add(instances.get(i));
		}
	}
	imbalancedInstances.randomize(random);
	counts = countClasses(imbalancedInstances);
	System.out.println("After imbalancing: " + Arrays.toString(counts));

The last line prints out the new imbalance. Now `c0` has only 50 instances.

Now we have to set up our classifier. We will use hyperSMURF with 10 partitions, oversampling factor of 2 (200%), no undersampling and each forest should have a size on 10.

.. code-block:: java

	// setup the hyperSMURF classifier
	HyperSMURF clsHyperSMURF = new HyperSMURF();
	clsHyperSMURF.setNumIterations(10);
	clsHyperSMURF.setNumTrees(10);
	clsHyperSMURF.setDistributionSpread(0);
	clsHyperSMURF.setPercentage(200.0);
	clsHyperSMURF.setSeed(SEED);


The next step will be the performance testing of hyperSMURF on the new generated imbalanced dataset. Therefore we will use a 5-fold cross-validation. To rerun this performance test using other classifiers we write everything into a new function :java:`classify(AbstractClassifier cls, Instances instances, int folds)`. The `classify` function will collect the predictions over all 5 folds in the `Evaluation` object which then can be used to print out the performance results. Here is the complete `classify` function:


.. code-block:: java

	private static void classify(AbstractClassifier cls, Instances instances, int folds) throws Exception {
		// perform cross-validation and add predictions
		Instances predictedData = null;
		Evaluation eval = new Evaluation(instances);
		for (int n = 0; n < folds; n++) {
			System.out.println("Training fold " + n + " from " + folds + "...");
			Instances train = instances.trainCV(folds, n);
			Instances test = instances.testCV(folds, n);

			// build and evaluate classifier
			Classifier clsCopy = AbstractClassifier.makeCopy(cls);
			clsCopy.buildClassifier(train);
			eval.evaluateModel(clsCopy, test);

			// add predictions
			AddClassification filter = new AddClassification();
			filter.setClassifier(cls);
			filter.setOutputClassification(true);
			filter.setOutputDistribution(true);
			filter.setOutputErrorFlag(true);
			filter.setInputFormat(train);
			Filter.useFilter(train, filter); // trains the classifier
			// perform predictions on test set
			Instances pred = Filter.useFilter(test, filter);
			if (predictedData == null)
				predictedData = new Instances(pred, 0);
			for (int j = 0; j < pred.numInstances(); j++)
				predictedData.add(pred.instance(j));
		}

		// output evaluation
		System.out.println();
		System.out.println("=== Setup ===");
		System.out.println("Classifier: " + cls.getClass().getName() + " " + Utils.joinOptions(cls.getOptions()));
		System.out.println("Dataset: " + instances.relationName());
		System.out.println("Folds: " + folds);
		System.out.println("Seed: " + SEED);
		System.out.println();
		System.out.println(eval.toSummaryString("=== " + folds + "-fold Cross-validation ===", false));
		System.out.println();
		System.out.println(eval.toClassDetailsString("=== Details ==="));

	}

Finally we can use test hyperSMURF by running :java:`classify(clsHyperSMURF, imbalancedInstances, 5);`. The output of the performance should be like this:

.. code-block:: text

	=== 5-fold Cross-validation ===
	Correctly Classified Instances        7406               99.3961 %
	Incorrectly Classified Instances        45                0.6039 %
	Kappa statistic                          0.3809
	Mean absolute error                      0.0858
	Root mean squared error                  0.1278
	Relative absolute error                637.5943 %
	Root relative squared error            156.5741 %
	Total Number of Instances             7451


	=== Details ===
	                 TP Rate  FP Rate  Precision  Recall   F-Measure  MCC      ROC Area  PRC Area  Class
	                 0.280    0.001    0.609      0.280    0.384      0.410    0.895     0.337     c0
	                 0.999    0.720    0.995      0.999    0.997      0.410    0.895     0.999     c1
	Weighted Avg.    0.994    0.715    0.993      0.994    0.993      0.410    0.895     0.995


So we will get an AUROC of 0.895 and an AUPRC of 0.337 for our minority class `c0`. We can also use a Random Forest classifier using the same number of random trees to see the differences:

.. code-block:: java

	// setup a RF classifier
	RandomForest clsRF = new RandomForest();
	clsRF.setNumIterations(10);
	clsRF.setSeed(SEED);

	// classify RF
	classify(clsRF, imbalancedInstances, 5);

Now we see that the RandomForest is only able to get an AUROC of 0.706 and an AUPRC of 0.109.

Usage examples with genetic data
===================================

HyperSMURF was designed to predict rare genomic variants, when the available examples of such variants are substantially less than `background` examples. This is a typical situation with genetic variants. For instance, we have only a small set of available variants known to be associated with Mendelian diseases in non-coding regions (positive examples) against the sea of background variants, i.e. a ratio of about :math:`1:36,000` between positive and negative examples [Smedley2016]_.

Here we show how to use hyperSMURF to detect these rare features using data sets obtained from the original large set of Mendelian data [Smedley2016]_.
To provide usage examples that do not require more than 1 minute of computation time on a modern desktop computer, we considered data sets downsampled from the original Mendelian data.
In particular we constructed Mendelian data sets with a progressive larger imbalance between Mendelian associated mutations and background genetic variants. We start with an artificially balanced data set, and then we consider progressively imbalanced data sets with ratio `positive:negative` varying from :math:`1:10`, to  :math:`1:100` and  :math:`1:1000`.
These data sets are downloadable as compressed `.arff` files, easily usable by Weka, from `https://www.github.com/charite/hyperSMURF-tutorial/data <https://www.github.com/charite/hyperSMURF-tutorial/data>`_.

The `Mendelian.balanced.arff.gz` file include 26 features, a column `class` howing the belonging class (1=positive, 0=negative) and a column `fold`. This is a numeric attribute with the number of the fold in which each example will be included according to the 10-fold cytogenetic band-aware CV procedure (0 to 9).
In total the file contains 406 positives and 400 negatives.

Now we have to write the following code in our new Java file `MendelianExample.java` in folder `src/main/java/de/charite/compbio/hypersmurf`:

* Loader of the Instances.
* Cross-validation strategy that takes the the column `fold` into account when partitioning and removing the column `fold` for training.
* Setting up our hyperSMURF classifier.

So this will be the blank `MendelianExample.java` class:

.. code-block:: java

	package de.charite.compbio.hypersmurf;

	public class MendelianExample {
		/**
		 * We need a seed to make consistent predictions.
		 */
		 private static int SEED = 42;
		 /**
		 * The number of folds are predefined in the dataset
		 */
		 private static int FOLDS = 10;

	 	public static void main(String[] args) throws Exception {

	 	}
	}


To read the data we simply can use the `ArffLoader` from Weka. We will use the first argument of the command-line arguments as our input file.

.. code-block:: java

	// read the file from the first argument of the command line input
	ArffLoader reader = new ArffLoader();
	reader.setFile(new File(args[0]));
	Instances instances = reader.getDataSet();

Then we have to set the class attribute. This is the last attribute of our instances. So we write :java:`instances.setClassIndex(instances.numAttributes() - 1);`. Because we have a balanced dataset of the Mendelian data we do not need to do over- or undersampling. So we simply run hyperSMURF with two partitions and a forest size of ten. Over- and undersampling settings have to be set to 0.

.. code-block:: java

	// setup the hyperSMURF classifier
	HyperSMURF clsHyperSMURF = new HyperSMURF();
	clsHyperSMURF.setNumIterations(2);
	clsHyperSMURF.setNumTrees(10);
	clsHyperSMURF.setDistributionSpread(0);
	clsHyperSMURF.setPercentage(0.0);
	clsHyperSMURF.setSeed(SEED);


Now we arrived at the special cytogenetic band-aware cross-validation. The folds are predefined as attribute `fold` in the instances object. So we have to select the instances on that fold but have to remove the fold attribute before training or testing a classifier. So we will write a small helper method that gives us a given fold for testing or the inverse for training. The blank method can be written like this:

 .. code-block: java

	 private static Instances getFold(Instances instances, int fold, boolean invert) throws Exception {

 	}


We will use the filter `SubsetbyExpression` to get the instances with the fold and we can simply use the `Instances` method `deteleAttributeAt(int index)` to remove the fold attribute. For `SubsetbyExpression` filter we write a regular expression like `Attribute = n` or `!(Attribute = n)` to get the `n`th fold (or all other folds). Attribute will be written by like `ATT`  with the index (one based) of the attribute. This we can get using :java:`int indexFold = instances.attribute("fold").index();` (zero based) and we have to increment it by one for our filter method. So the content of our `getFold` method can look like:

.. code-block:: java

	// filter on fold variable
	int indexFold = instances.attribute("fold").index();
	SubsetByExpression filterFold = new SubsetByExpression();
	if (invert)
		filterFold.setExpression("!(ATT" + (indexFold + 1) + " = " + fold + ")");
	else
		filterFold.setExpression("ATT" + (indexFold + 1) + " = " + fold);
	filterFold.setInputFormat(instances);
	Instances filtered = Filter.useFilter(instances, filterFold);

	// remove fold attribute
	filtered.deleteAttributeAt(indexFold);

	return filtered;

Now it is time for the cross-validation this is similar to the Synthetic Example but we will use the `getFold` method to make the train/test partitioning.

.. code-block:: java

	// perform cross-validation and add predictions
	Instances predictedData = null;
	Evaluation eval = new Evaluation(instances);
	for (int n = 0; n < FOLDS; n++) {
		System.out.println("Training fold " + (n+1) + " from " + FOLDS + "...");
		Instances train = getFold(instances, n, true);
		Instances test = getFold(instances, n, false);

		// build and evaluate classifier
		Classifier clsCopy = AbstractClassifier.makeCopy(cls);
		clsCopy.buildClassifier(train);
		eval.evaluateModel(clsCopy, test);

		// add predictions
		AddClassification filter = new AddClassification();
		filter.setClassifier(cls);
		filter.setOutputClassification(true);
		filter.setOutputDistribution(true);
		filter.setOutputErrorFlag(true);
		filter.setInputFormat(train);
		Filter.useFilter(train, filter); // trains the classifier
		// perform predictions on test set
		Instances pred = Filter.useFilter(test, filter);
		if (predictedData == null)
			predictedData = new Instances(pred, 0);
		for (int j = 0; j < pred.numInstances(); j++)
			predictedData.add(pred.instance(j));
	}

	// output evaluation
	System.out.println();
	System.out.println("=== Setup ===");
	System.out.println("Classifier: " + cls.getClass().getName() + " " + Utils.joinOptions(cls.getOptions()));
	System.out.println("Dataset: " + instances.relationName());
	System.out.println("Folds: " + FOLDS);
	System.out.println("Seed: " + SEED);
	System.out.println();
	System.out.println(eval.toSummaryString("=== " + FOLDS + "-fold Cross-validation ===", false));
	System.out.println();
	System.out.println(eval.toClassDetailsString("=== Details ==="));

If we run hyperSMURF with the settings above the command-line output will show an AUPRC 0.989 of and an AUROC of 0.989 of our class `1` which are the Mendelian regulatory mutations. This is the complete output:

.. code-block:: text

	=== 10-fold Cross-validation ===
	Correctly Classified Instances         770               95.5335 %
	Incorrectly Classified Instances        36                4.4665 %
	Kappa statistic                          0.9107
	Mean absolute error                      0.0898
	Root mean squared error                  0.1925
	Relative absolute error                 17.9538 %
	Root relative squared error             38.4915 %
	Total Number of Instances              806


	=== Details ===
	                 TP Rate  FP Rate  Precision  Recall   F-Measure  MCC      ROC Area  PRC Area  Class
	                 0.985    0.074    0.929      0.985    0.956      0.912    0.989     0.983     0
	                 0.926    0.015    0.984      0.926    0.954      0.912    0.989     0.989     1
	Weighted Avg.    0.955    0.044    0.957      0.955    0.955      0.912    0.989     0.986


Then we can perform the same computation using the progressively imbalanced data sets: `Mendelian.1_10.arff.gz`, `Mendelian.1_100.arff.gz`, and `Mendelian.1_1000.arff.gz`. Of course every time we have to adapt the settings of hyperSMURF.

Using `Mendelian.1_10.arff.gz` hyperSUMRF and the output can look like:

.. code-block:: java

	// setup the hyperSMURF classifier
	clsHyperSMURF = new HyperSMURF();
	clsHyperSMURF.setNumIterations(5);
	clsHyperSMURF.setNumTrees(10);
	clsHyperSMURF.setDistributionSpread(0);
	clsHyperSMURF.setPercentage(100.0);
	clsHyperSMURF.setSeed(SEED);

.. code-block:: text

	=== 10-fold Cross-validation ===
	Correctly Classified Instances        4310               97.8212 %
	Incorrectly Classified Instances        96                2.1788 %
	Kappa statistic                          0.8779
	Mean absolute error                      0.0577
	Root mean squared error                  0.1427
	Relative absolute error                 34.4437 %
	Root relative squared error             49.3333 %
	Total Number of Instances             4406


	=== Details ===
	                 TP Rate  FP Rate  Precision  Recall   F-Measure  MCC      ROC Area  PRC Area  Class
	                 0.981    0.044    0.995      0.981    0.988      0.880    0.990     0.999     0
	                 0.956    0.020    0.833      0.956    0.890      0.880    0.990     0.950     1
	Weighted Avg.    0.978    0.042    0.980      0.978    0.979      0.880    0.990     0.994


Increasing the imbalance with `Mendelian.1_100.arff.gz`:

.. code-block:: java

	// setup the hyperSMURF classifier
	clsHyperSMURF = new HyperSMURF();
	clsHyperSMURF.setNumIterations(5);
	clsHyperSMURF.setNumTrees(10);
	clsHyperSMURF.setDistributionSpread(0);
	clsHyperSMURF.setPercentage(100.0);
	clsHyperSMURF.setSeed(SEED);

.. code-block:: text

	=== 10-fold Cross-validation ===
	Correctly Classified Instances       39987               99.1348 %
	Incorrectly Classified Instances       349                0.8652 %
	Kappa statistic                          0.6795
	Mean absolute error                      0.0249
	Root mean squared error                  0.0851
	Relative absolute error                124.7001 %
	Root relative squared error             85.3023 %
	Total Number of Instances            40336


	=== Details ===
	                 TP Rate  FP Rate  Precision  Recall   F-Measure  MCC      ROC Area  PRC Area  Class
	                 0.992    0.071    0.999      0.992    0.996      0.705    0.991     1.000     0
	                 0.929    0.008    0.541      0.929    0.684      0.705    0.991     0.900     1
	Weighted Avg.    0.991    0.071    0.995      0.991    0.992      0.705    0.991     0.999


Again increasing the imbalance with `Mendelian.1_1000.arff.gz`:

.. code-block:: java

	// setup the hyperSMURF classifier
	clsHyperSMURF = new HyperSMURF();
	clsHyperSMURF.setNumIterations(10);
	clsHyperSMURF.setNumTrees(10);
	clsHyperSMURF.setDistributionSpread(3);
	clsHyperSMURF.setPercentage(200.0);
	clsHyperSMURF.setSeed(SEED);

.. code-block:: text

	=== 10-fold Cross-validation ===
	Correctly Classified Instances      392436               99.2597 %
	Incorrectly Classified Instances      2927                0.7403 %
	Kappa statistic                          0.2021
	Mean absolute error                      0.0233
	Root mean squared error                  0.0805
	Relative absolute error               1135.2254 %
	Root relative squared error            251.4735 %
	Total Number of Instances           395363


	=== Details ===
	                 TP Rate  FP Rate  Precision  Recall   F-Measure  MCC      ROC Area  PRC Area  Class
	                 0.993    0.079    1.000      0.993    0.996      0.323    0.989     1.000     0
	                 0.921    0.007    0.114      0.921    0.204      0.323    0.989     0.773     1
	Weighted Avg.    0.993    0.079    0.999      0.993    0.995      0.323    0.989     1.000

As we can see, we have a certain decrement of the performances when the imbalance increases. Indeed when we have perfectly balanced data the AUPRC is very close to 1, while by increasing the imbalance we have a progressive decrement of the AUPRC to 0.950, 0.900, till to 0.773 when we have a :math:`1:1000` imbalance ratio. Nevertheless this decline in  performance is relatively small compared to other machine-learning methods.


We can perform the same task using parallel computation. For instance, by using 4 cores with an Intel i7-2670QM CPU, 2.20GHz, we perform a full 10-fold cytogenic band-aware cross-validation using 406 genetic variants known to be associated with Mendelian diseases and 400000 background variants in less than 5 minutes. The best performance boost from the implementation is if we do the training of the partitioning in parallel. So we can set the number of execution slots to 4 using :java:`clsHyperSMURF.setNumExecutionSlots(4);`.

Of course the training and cross-validation functions allow to set also the parameters of the Random Forest ensembles, that constitute the base learners of the hyperSMURF hyper-ensemble, such as the number of decision trees to be used for each Random Forest (`setNumTrees(int num)`) or the number of features to be randomly selected from the set of available input features at each step of the inductive learning of the decision tree (`setNumFeatures(int num)`). The full description of the hyperSMURF class can be found in the `HyperSMURF` Java API `https://javadoc.io/doc/de.charite.compbio/hyperSMURF/ <https://javadoc.io/doc/de.charite.compbio/hyperSMURF>`_.



.. rubric:: References

.. [Smedley2016] Smedley, Damian, et al. "A whole-genome analysis framework for effective identification of pathogenic regulatory variants in Mendelian disease." The American Journal of Human Genetics 99.3 (2016): 595-606.
