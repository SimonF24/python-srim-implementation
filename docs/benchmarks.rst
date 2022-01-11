Benchmarks
==========

Docker vs Standard Linux
------------------------

I wanted to benchmark running srim calculations on normal linux vs
within a docker container. TRIM on linux showed more variability
between job runs. When I refer to TRIM on linux it is equivalent to
``wine TRIM.exe``.

I used the parallel command to test differing number of cores. Yes
this is a simple approach but will be in the right ballpark and trends
are certainly compelling for the docker container. All test we done on
my two core Lenovo t440s.

Docker startup cost is 8.5 seconds vs 5.8 seconds for standard linux
process. But the docker container is significantly faster using
=xvfb-run= best performance is linux 8.3 ions/second vs docker 13.2
ions/second. These tests indicate that the docker container may have
better performance due to rending in a virtual X frame buffer.

The python simple script to be run is a Nickel in Nickel irradiation.

.. code-block:: python

   import os
   from srim import Ion, Layer, Target, TRIM

   ion = Ion('Ni', energy=3.0e6)
   layer = Layer({
           'Ni': {
               'stoich': 1.0,
               'E_d': 30.0,
               'lattice': 0.0,
               'surface': 3.0
           }}, density=8.9, width=20000.0)
   target = Target([layer])
   trim = TRIM(target, ion, number_ions=100, calculation=1)
   srim_executable_directory = '/tmp/srim'
   results = trim.run(srim_executable_directory)
   os.makedirs('/tmp/output', exist_ok=True)
   TRIM.copy_output_files('/tmp/srim', '/tmp/output')

And the benchmarks results.

.. code-block:: bash

   time parallel -j 6 python ni.py -- 1 2

+------+-------+-----------+---------------+--------------------+
| ions | cores | linux [s] | linux [ion/s] | linux [ion/s core] |
+======+=======+===========+===============+====================+
|    1 |     1 |       5.8 |    0.17241379 |         0.17241379 |
+------+-------+-----------+---------------+--------------------+
|  100 |     1 |        29 |     3.4482759 |          3.4482759 |
+------+-------+-----------+---------------+--------------------+
|  200 |     1 |        53 |     3.7735849 |          3.7735849 |
+------+-------+-----------+---------------+--------------------+
|  200 |     2 |        35 |     5.7142857 |          2.8571429 |
+------+-------+-----------+---------------+--------------------+
|  400 |     2 |        63 |     6.3492063 |          3.1746032 |
+------+-------+-----------+---------------+--------------------+
|  300 |     3 |        43 |     6.9767442 |          2.3255814 |
+------+-------+-----------+---------------+--------------------+
|  600 |     3 |        79 |     7.5949367 |          2.5316456 |
+------+-------+-----------+---------------+--------------------+
|  400 |     4 |        55 |     7.2727273 |          1.8181818 |
+------+-------+-----------+---------------+--------------------+
|  800 |     4 |       101 |     7.9207921 |          1.9801980 |
+------+-------+-----------+---------------+--------------------+
| 1600 |     4 |       192 |     8.3333333 |          2.0833333 |
+------+-------+-----------+---------------+--------------------+
|  500 |     5 |        65 |     7.6923077 |          1.5384615 |
+------+-------+-----------+---------------+--------------------+
| 1000 |     5 |       121 |     8.2644628 |          1.6528926 |
+------+-------+-----------+---------------+--------------------+

.. code-block:: bash

   time parallel -j 6 \
            docker run \
                 -v $PWD/examples/docker/:/opt/pysrim/ \
                 -v /tmp/output:/tmp/output
                 -it costrouc/pysrim sh -c "xvfb-run -a python3.6 /opt/pysrim/ni.py" -- 1 2

+------+-------+------------+----------------+---------------------+
| ions | cores | docker [s] | docker [ion/s] | docker [ion/s core] |
+======+=======+============+================+=====================+
|    1 |     1 |        8.5 |     0.11764706 |          0.11764706 |
+------+-------+------------+----------------+---------------------+
|  100 |     1 |         27 |      3.7037037 |           3.7037037 |
+------+-------+------------+----------------+---------------------+
|  200 |     1 |         46 |      4.3478261 |           4.3478261 |
+------+-------+------------+----------------+---------------------+
|  200 |     2 |         31 |      6.4516129 |           3.2258065 |
+------+-------+------------+----------------+---------------------+
|  400 |     2 |         52 |      7.6923077 |           3.8461538 |
+------+-------+------------+----------------+---------------------+
|  300 |     3 |         39 |      7.6923077 |           2.5641026 |
+------+-------+------------+----------------+---------------------+
|  600 |     3 |         69 |      8.6956522 |           2.8985507 |
+------+-------+------------+----------------+---------------------+
|  400 |     4 |         39 |      10.256410 |           2.5641026 |
+------+-------+------------+----------------+---------------------+
|  800 |     4 |         67 |      11.940299 |           2.9850746 |
+------+-------+------------+----------------+---------------------+
| 1600 |     4 |        121 |      13.223140 |           3.3057851 |
+------+-------+------------+----------------+---------------------+
|  500 |     5 |         51 |      9.8039216 |           1.9607843 |
+------+-------+------------+----------------+---------------------+
| 1000 |     5 |         69 |      14.492754 |           2.8985507 |
+------+-------+------------+----------------+---------------------+
