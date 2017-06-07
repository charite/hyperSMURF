.. _examples_r:

============================
Examples using the R package
============================

.. _synthetic:

Simple usage examples using synthetic data
==============================================

Here we introduce some simple usage examples using the generator of synthetic imbalanced data included in the R package.
At first we load the library:

.. code-block:: r

	library(hyperSMURF)

Then we construct two imbalanced data sets (training and test set) having both 20 `positive` and 2000 `negative` examples with 10 features (dimension of input data equal to 10 - see the `Reference manual <https://CRAN.R-project.org/package=hyperSMURF>`_ on CRAN for details about the synthetic data generator):

.. code-block:: r

	train <- imbalanced.data.generator(n.pos=20, n.neg=2000, n.features=10, n.inf.features=3, sd=0.1, seed=1);
	test <- imbalanced.data.generator(n.pos=20, n.neg=2000, n.features=10, n.inf.features=3, sd=0.1, seed=2);

Then we can train and test the model with the following code:


.. code-block:: r 
	
	HSmodel <- hyperSMURF.train(train$data, train$label, n.part = 10, fp = 2, ratio = 3);
	res <- hyperSMURF.test(test$data, HSmodel);

Note that we used 10 partitions of the training data (parameter `n.part` that corresponds to the parameter `n` in the pseudo-code of the algorithm in Supplementary Note 1), a SMOTE oversampling equal to 2 (parameter `fp` corresponding to the `f` parameter in the pseudo-code), and undersampling ratio equal to 3 (parameter `ratio` corresponding to the parameter `m` of the modified second line of the \Hy~algorithm in Supplementary Note~1). In other words the negative examples were partitioned in 10 sets of equal size (200 examples). Then a different RF was trained using: a) the available 20 positive examples, plus the "augmented" 40 synthetic positive examples obtained by SMOTE convex combination of close positive examples, and b) a set of :math:`3 \times 60 = 180` negative examples randomly extracted from the partition (see Supplementary Note 1). The obtained hyperSMURF model (`HSModel`), that includes 10 different RF (one for each partition), is finally tested on the test set.

We can easily obtain the confusion matrix: 

.. code-block:: r

	y <- ifelse(test$labels==1,1,0);
	pred <- ifelse(res>0.5,1,0);
	table(pred,y);
	y
		pred    0       1
		0       1979    1
		1       21      19


The accuracy is 0.9891 and the F-score (more informative in this unbalanced context) is 0.6333.
Note that with a RF that does not adopt unbalance-aware learning strategies on the same data we obtain significantly worse results in terms of the F-score:

.. code-block:: r

	library(randomForest);
	RF <- randomForest(train$data, train$label);
	res <- predict(RF, test$data);
	y <- ifelse(test$labels==1,1,0);
	pred <- ifelse(res==1,1,0);
	table(pred,y);
	y
		pred    0    1
		0       2000 16
		1       0    4

The accuracy of the RF is high (0.9930), but the F-score is $0.3333$, only about half that of hyperSMURF [#note]_.

To perform a 5 fold CV on a given data set we need only 1 line of R code:

.. code-block:: r

	res <- hyperSMURF.cv(train$data, train$labels, kk = 5, n.part = 10, fp = 1, ratio = 1); 
	
	To compute the AUROC and the AUPRC  (respectively the area under the ROC curve and the area under the precision/recall curve) we can use the `precrec` package:

.. code-block:: r

	library(precrec);
	labels <- ifelse(train$labels==1,1,0);
	digits=4;
	sscurves <- evalmod(scores = res, labels = labels);
	m<-attr(sscurves,"auc",exact=FALSE);
	AUROC <-  round(m[1,"aucs"],digits);
	AUPRC <-  round(m[2,"aucs"],digits);
	cat ("AUROC = ", AUROC, "\n", "AUPRC = ", AUPRC, "\n");
	AUROC =  0.9972 
	AUPRC =  0.8540 

We can also apply the version of hyperSMURF that embeds a feature selection step on the training data to select the features most correlated with the labels:

.. code-block:: r
	
	res <-hyperSMURF.corr.cv.parallel(train$data, train$labels, kk = 5, n.part = 10, fp = 1, ratio = 1, mtry=3, n.feature = 6);
	sscurves <- evalmod(scores = res, labels = labels);
	m<-attr(sscurves,"auc",exact=FALSE);
	AUROC <-  round(m[1,"aucs"],digits);
	AUPRC <-  round(m[2,"aucs"],digits);
	cat ("AUROC = ", AUROC, "\n", "AUPRC = ", AUPRC, "\n");
	AUROC =  0.9982 
	AUPRC =  0.9190

.. [#note] Note that the results may vary slightly due to the randomization in the algorithm.

Usage examples with genetic data
===================================

HyperSMURF was designed to predict rare genomic variants, when the available examples of such variants are substantially less than `background` examples. This is a typical situation with genetic variants. For instance, we have only a small set of available variants known to be associated with Mendelian diseases in non-coding regions (positive examples) against the sea of background variants, i.e. a ratio of about :math:`1:36,000` between positive and negative examples [Smedley2016]_.

Here we show how to use hyperSMURF to detect these rare features using data sets obtained from the original large set of Mendelian data [Smedley2016]_. 
To provide usage examples that do not require more than 1 minute of computation time on a modern desktop computer, we considered data sets downsampled from the original Mendelian data set described in the `mendelian data` section of the main manuscript (this data set includes more than 14 millions of genetic variants).
In particular we constructed Mendelian data sets with a progressive larger imbalance between Mendelian associated mutations and background genetic variants. We start with an artificially balanced data set, and then we consider progressively imbalanced data sets with ratio `positive:negative` varying from :math:`1:10`, to  :math:`1:100` and  :math:`1:1000`.
These data sets are downloadable as compressed `.rda` R objects from `http://homes.di.unimi.it/valentini/DATA/Mendelian <http://homes.di.unimi.it/valentini/DATA/Mendelian>`_.

The `Mendelian_balanced.rda` file include 3 objects: `m.subset`, that includes the input features of the balanced examples (406 positives and 400 negatives), `labels.subset`, i.e. the corresponding labels, and `folds.subset` a vector with the number of the fold in which each example will be included according to the 10-fold cytoband-aware CV procedure (see Supplementary Note~2). 
The following lines of code load the data and perform a 10-fold cytoband-aware CV and compute the AUROC and AUPRC:

.. code-block:: r
	
	load("Mendelian_balanced.rda");
	res <- hyperSMURF.cv(m.subset, factor(labels.subset, levels=c(1,0)), kk = 10, n.part = 2, fp = 0, ratio = 1, k = 5, ntree = 10, mtry = 6,  seed = 1, fold.partition = folds.subset);
	
	sscurves <- evalmod(scores = res, labels = labels.subset);
	m<-attr(sscurves,"auc",exact=FALSE);
	AUROC <-  round(m[1,"aucs"],digits);
	AUPRC <-  round(m[2,"aucs"],digits);
	cat ("AUROC = ", AUROC, "\n", "AUPRC = ", AUPRC, "\n");
	AUROC =  0.9903 
	AUPRC =  0.9893 

Then we can perform the same computation using the progressively imbalanced data sets:

.. code-block:: r

	# Imbalance 1:10. about 400 positives and 4000 negative variants
	load("Mendelian_1:10.rda");
	
	res <- hyperSMURF.cv(m.subset, factor(labels.subset, levels=c(1,0)), kk = 10, n.part = 5, 
	fp = 1, ratio = 1, k = 5, ntree = 10, mtry = 6,  seed = 1, fold.partition = folds.subset);
	
	sscurves <- evalmod(scores = res, labels = labels.subset);
	m<-attr(sscurves,"auc",exact=FALSE);
	AUROC <-  round(m[1,"aucs"],digits);
	AUPRC <-  round(m[2,"aucs"],digits);
	cat ("AUROC = ", AUROC, "\n", "AUPRC = ", AUPRC, "\n");
	AUROC =  0.9915 
	AUPRC =  0.9583 
	
	# Imbalance 1:100. about 400 positives and 40000 negative variants
	load("Mendelian_1:100.rda");
	res <- hyperSMURF.cv(m.subset, factor(labels.subset, levels=c(1,0)), kk = 10, n.part = 10, fp = 2, ratio = 3, k = 5, ntree = 10, mtry = 6,  seed = 1, fold.partition = folds.subset);

	sscurves <- evalmod(scores = res, labels = labels.subset);
	m<-attr(sscurves,"auc",exact=FALSE);
	AUROC <-  round(m[1,"aucs"],digits);
	AUPRC <-  round(m[2,"aucs"],digits);
	cat ("AUROC = ", AUROC, "\n", "AUPRC = ", AUPRC, "\n");
	AUROC =  0.9922 
	AUPRC =  0.9 

	# Imbalance 1:1000. about 400 positives and 400000 negative variants
	load("Mendelian_1:1000.rda");
 
	res <- hyperSMURF.cv(m.subset, factor(labels.subset, levels=c(1,0)), kk = 10, n.part = 10, 
	fp = 2, ratio = 3, k = 5, ntree = 10, mtry = 6,  seed = 1, fold.partition = folds.subset);

	sscurves <- evalmod(scores = res, labels = labels.subset);
	m<-attr(sscurves,"auc",exact=FALSE);
	AUROC <-  round(m[1,"aucs"],digits);
	AUPRC <-  round(m[2,"aucs"],digits);
	cat ("AUROC = ", AUROC, "\n", "AUPRC = ", AUPRC, "\n");
	AUROC =  0.9901 
	AUPRC =  0.7737

As we can see, we have a certain decrement of the performances when the imbalance increases. Indeed when we have perfectly balanced data the AUPRC is very close to 1, while by increasing the imbalance we have a progressive decrement of the AUPRC to 0.9583, 0.9000, till to 0.7737 when we have a :math:`1:1000` imbalance ratio. Nevertheless this decline in  performance is relatively small compared to that of state-of-the-art imbalance-unaware learning methods (see Fig. 5 in the main manuscript).


We can perform the same task using parallel computation. For instance, by using 4 cores with an Intel i7-2670QM CPU, 2.20GHz, less than 1 minute is necessary to perform a full 10-fold cytoband-aware CV using 406 genetic variants known to be associated with Mendelian diseases and 400,000 background variants:

.. code-block:: r

	res <- hyperSMURF.cv.parallel(m.subset, factor(labels.subset, levels=c(1,0)), kk = 10, n.part = 10, fp = 2, ratio = 3, k = 5, ntree = 10, mtry = 6,  seed = 1, fold.partition = folds.subset, ncores=4);

Of course the training and  CV functions allow to set also the parameters of the RF ensembles, that constitute the base learners of the hyperSMURF hyper-ensemble, such as the number of decision trees to be used for each RF (parameter `ntree`) or the number of features to be randomly selected from the set of available input features at each step of the inductive learning of the decision tree (parameter `mtry`). The full description of all the parameters and the output of each function is available in the PDF and HTML documentation included in the hyperSMURF R package.


.. rubric:: References

.. [Smedley2016] Smedley, Damian, et al. "A whole-genome analysis framework for effective identification of pathogenic regulatory variants in Mendelian disease." The American Journal of Human Genetics 99.3 (2016): 595-606.
