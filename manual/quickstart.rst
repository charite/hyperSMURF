.. _quickstart:

Quickstart
===========

This short How-To guides you from downloading the hyperSMURF weka plugin and load it into weka.

#. Download the current stable |version| release from our `GitHub project <https://github.com/charite/hyperSMURF>`_ by clicking |link1|\ |version|\ |link2|\ |version|\ |link3|.
#. Download the latest 3.9 development version of Weka `here <http://www.cs.waikato.ac.nz/~ml/weka/downloading.html>`_ and install it.
#. Open Weka and navigate to the Package Manager (Tools -> Package manager).
#. Install the |zip-pre|\ |version|\ |zip-post| file (Unofficial File/URL -> Browse -> OK).
#. Restart Weka. If you open the package explorer again you will see the hyperSMURF classifier under the Installed tab.
#. Download example data `quickstart_example.arff.gz <https://github.com/charite/hyperSMURF/tree/master/data/quickstart_example.arff.gz>`_
#. Open the Weka Explorer. In the Preprocess tab click `Open file...` then navigate to the dowloaded `quickstart_example.arff.gz` and select it. Choose as filetype `*.arff.gz` and open the data.
#. Now you should be able to switch to the Classify tab and choose the hyperSMURF classifier under weka -> classifiers -> trees. After that you can start the classification and the classifier output should display results without errors. The end of the output should look like:

.. |link1| raw:: html

    <a href="https://github.com/charite/hyperSMURF/releases/download/v

.. |link2| raw:: html

    /hyperSMURF-

.. |link3| raw:: html

    -weka.zip">here</a>

.. |zip-pre| raw:: html

		<it>hyperSMURF-

.. |zip-post| raw:: html

		-weka.zip</it>


.. code-block:: text

	Time taken to build model: 7.6 seconds

	=== Stratified cross-validation ===
	=== Summary ===

	Correctly Classified Instances        8744               87.44   %
	Incorrectly Classified Instances      1256               12.56   %
	Kappa statistic                          0.7322
	Mean absolute error                      0.2089
	Root mean squared error                  0.2881
	Relative absolute error                 49.7147 %
	Root relative squared error             62.8514 %
	Total Number of Instances            10000

	=== Detailed Accuracy By Class ===

					 TP Rate  FP Rate  Precision  Recall   F-Measure  MCC      ROC Area  PRC Area  Class
					 0,823    0,006    0,997      0,823    0,902      0,758    0,993     0,997     c0
					 0,994    0,177    0,707      0,994    0,826      0,758    0,993     0,985     c1
					 Weighted Avg.    0,874    0,057    0,910      0,874    0,879      0,758    0,993     0,993

	=== Confusion Matrix ===

		a    b   <-- classified as
		5759 1239 |    a = c0
		17   2985 |    b = c1
