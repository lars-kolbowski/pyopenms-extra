Getting Started
===============

Import pyopenms
***************

After installation, you should be able to import pyopenms as a package

.. code-block:: python

    import pyopenms

which should now give you access to all of pyopenms. You should now be able to
interact with the OpenMS library and, for example, read and write mzML files:

.. code-block:: python

    import pyopenms
    exp = pyopenms.MSExperiment()
    pyopenms.MzMLFile().store("testfile.mzML", exp)

which will create an empty mzML file called `testfile.mzML`.

Getting help
************

There are multiple ways to get information about the available functions and
methods. We can inspect individual pyOpenMS objects through the ``help``
function: 

.. code-block:: python

    >>> import pyopenms
    >>> help(pyopenms.MSExperiment)
    Help on class MSExperiment in module pyopenms.pyopenms_2:

    class MSExperiment(__builtin__.object)
     |  Methods defined here:
     |  
     |  __copy__(...)
     |  
     |  __deepcopy__(...)
     [...]


which lists the available functions. This full list indicates that
``pyopenms.MSExperiment`` has multiple methods, among them ``__copy__``.  The
command also lists the signature for each function, allowing users to identify
the function arguments and return types. In order to get more information about
the wrapped functions, we can also consult the `pyOpenMS manual
<http://proteomics.ethz.ch/pyOpenMS_Manual.pdf>`_ which references to all
wrapped functions. For a more complete documentation of the
underlying wrapped methods, please consult the official OpenMS documentation,
in this case the `MSExperiment documentation <http://ftp.mi.fu-berlin.de/pub/OpenMS/release-documentation/html/classOpenMS_1_1MSExperiment.html>`_.


First look at data
******************

File reading
^^^^^^^^^^^^

pyOpenMS supports a variety of different files through the implementations in
OpenMS. In order to read mass spectrometric data, we can download the `mzML
example file <http://proteowizard.sourceforge.net/example_data/tiny.pwiz.1.1.mzML>`_

.. code-block:: python

    import urllib
    from pyopenms import *
    urllib.urlretrieve ("http://proteowizard.sourceforge.net/example_data/tiny.pwiz.1.1.mzML", "tiny.pwiz.1.1.mzML")
    exp = MSExperiment()
    MzMLFile().load("tiny.pwiz.1.1.mzML", exp)

which will load the content of the "tiny.pwiz.1.1.mzML" file into the ``exp``
variable of type ``MSExperiment``.
We can now inspect the properties of this object:

.. code-block:: python

    >>> help(exp)
    Help on MSExperiment object:

    class MSExperiment(__builtin__.object)
     |  Methods defined here:
     ...
     |  getNrChromatograms(...)
     |      Cython signature: size_t getNrChromatograms()
     |  
     |  getNrSpectra(...)
     |      Cython signature: size_t getNrSpectra()
     |  
     ...
  

which indicates that the variable ``exp`` has (among others) the functions
``getNrSpectra`` and ``getNrChromatograms``. We can now try these functions:

.. code-block:: python

    >>> exp.getNrSpectra()
    4
    >>> exp.getNrChromatograms()
    2

and indeed we see that we get information about the underlying MS data. We can
iterate through the spectra as follows:


Iteration
^^^^^^^^^

.. code-block:: python

    >>> for spec in exp:
    ...   print "MS Level:", spec.getMSLevel()
    ... 
    MS Level: 1
    MS Level: 2
    MS Level: 1
    MS Level: 1

This iterates through all available spectra, we can also access spectra through the ``[]`` operator:

.. code-block:: python

    >>> print "MS Level:", exp[1].getMSLevel()
    MS Level: 2

Note that ``spec[1]`` will access the *second* spectrum (arrays start at
``0``). We can access the raw peaks through ``get_peaks()``:

.. code-block:: python

    >>> spec = exp[1]
    >>> mz, i = spec.get_peaks()
    >>> sum(i)
    110

Which will access the data using a numpy array, storing the *m/z* information
in the ``mz`` vector and the intensity in the ``i`` vector. Alternatively, we
can also iterate over individual peak objects as follows (this tends to be
slower): 

.. code-block:: python

    >>> for peak in spec:
    ...   print peak.getIntensity()
    ... 
    20.0
    18.0
    16.0
    14.0
    12.0
    10.0
    8.0
    6.0
    4.0
    2.0

TIC calculation
^^^^^^^^^^^^^^^


With this information, we can now calculate a total ion current (TIC) using the
following function:

.. code-block:: python
   :linenos:

    def calcTIC(exp):
        tic = 0
        for spec in exp:
            if spec.getMSLevel() == 1:
                mz, i = spec.get_peaks()
                tic += sum(i)
        return tic

To calculate a TIC we would now call the function:

.. code-block:: python
   :linenos:

    >>> calcTIC(exp)
    240.0
    >>> sum([sum(s.get_peaks()[1]) for s in exp if s.getMSLevel() == 1])
    240.0

Note how one can compute the same property using list comprehensions in Python (see the third line above).

