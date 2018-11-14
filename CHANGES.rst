=======
CHANGES
=======

1.2.0 (2018-11-14)
------------------

- Python 3.6 and 3.7 support. Drop Python 2.6.

- Tests now run with `python setup.py test`


1.1.1 (2015-11-09)
------------------

- Standardize namespace __init__


1.1.0 (2013-09-27)
------------------

- Added new precision value "sub-cents" (``interfaces.SUBCENTS``), which
  allows precision beyond pennies, which is needed for financial and other
  business applications.


1.0.0 (2013-08-16)
------------------

- Updated Trove classifiers.

- Moved code to GitHub.

- Changed validation to raise custom validation errors, since the upstream
  code looks at the doc string of the exception instead of the first argument.

- Improved converter to

  * Ensure proper precision of decimal after initial parsing.

  * Format the value to the proper precision.

- Changed "precision" field in `ICurrency` interface to be a choice, so that
  UIs generate nicely.

- Cleaned up code a little bit.


0.1.0 (2007-09-12)
------------------

- Initial Release

  * Implementation of ``Currency`` field supporting precision and a unit.

  * Implementation of data converter.
