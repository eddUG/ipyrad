
.. include:: global.rst

.. _pedicularis_cli:


Sub-sampling data sets
=======================
In this tutorial we show how to subsample both the number of taxa in an Assembly,
and the amount of sequence data. Again, we use the 13 taxa *Pedicularis* data set
from **Eaton and Ree (2013)** for our example. 

..  use an empirical data set for the example. 
.. The data set is composed of single-end reads for a RAD-seq library prepared with 
.. the PstI enzyme for 13 individuals from the *Cyathophora* clade of the angiosperm genus
.. *Pedicularis*, originally published by 
.. (:ref:`link to open access article <eaton_and_ree>`). 


Download the empirical example data set (*Pedicularis*)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
These data are archived on the NCBI sequence read archive (SRA) under 
accession id SRP021469. For convenience, the data are also hosted at a 
public Dropbox link which is a bit easier to access. Run the code below to 
download and decompress the fastq data files, which will save them into a 
directory called ``example_empirical_data/``. The compressed file size is 
approximately 1.1GB.

.. code:: bash

    ## download fastq data from the SRA database
    >>> ipyrad --download SRP021469 example_empirical_data/


Setup a base params file
~~~~~~~~~~~~~~~~~~~~~~~~
We start by using the ``-n`` argument to create a new named Assembly. 
I'll use the name ``base`` to indicate that this is the base assembly from 
which we will later create several branches.

.. code:: bash

    >>> ipyrad -n "base"

.. parsed-literal::
    New file 'params-base.txt' created in /home/deren/Downloads


The data come to us already demultiplexed so we are going to simply set the 
**sorted\_fastq\_path** to tell ipyrad the location of the data files, 
and also set a **project\_dir**, which will group all of our analyses into 
a single directory. For the latter I use the name of our study organism, "pedicularis". 

.. parsed-literal::
    ## Use your text editor to enter the following values:
    ## The wildcard (*) tells ipyrad to select all files ending in .gz
    pedicularis                       ## [1] [project_dir] ...
    example_empirical_rad/*.gz        ## [4] [sorted_fastq_path] ...

For now we'll leave the remaining parameters at their default values.


Load the fastq Sample data
~~~~~~~~~~~~~~~~~~~~~~~~~~
When the data location is entered as a **sorted_fastq_path** step 1 
simply counts the number of reads for each Sample and parses the file names to 
extract names for each Sample. For example, the file ``29154_superba.fastq.gz`` 
will be assigned to Sample ``29154_superba``. Now, run step 1 (-s 1) and 
tell ipyrad to print the results when it is finished (-r). 

.. code:: bash

    >>> ipyrad -p params-base.txt -s 1 -r


.. parsed-literal:: 
  --------------------------------------------------
   ipyrad [v.0.2.5]
   Interactive assembly and analysis of RADseq data
  --------------------------------------------------
   New Assembly: base
   ipyparallel setup: Local connection to 4 Engines

   Step1: Linking sorted fastq data to Samples
     Linking to demultiplexed fastq files in:
       /home/deren/Downloads/example_empirical_rad/*.gz
     13 new Samples created in 'base'.
     13 fastq files linked to 13 new Samples.
   Saving Assembly.

  Summary stats of Assembly base
  ------------------------------------------------
                          state  reads_raw
  29154_superba               1     696994
  30556_thamno                1    1452316
  30686_cyathophylla          1    1253109
  32082_przewalskii           1     964244
  33413_thamno                1     636625
  33588_przewalskii           1    1002923
  35236_rex                   1    1803858
  35855_rex                   1    1409843
  38362_rex                   1    1391175
  39618_rex                   1     822263
  40578_rex                   1    1707942
  41478_cyathophylloides      1    2199740
  41954_cyathophylloides      1    2199613


Sub-sampling methods
~~~~~~~~~~~~~~~~~~~~~
Assembling this full data set takes around 3 hours on a 4-core laptop, which
is actually pretty fast. However, for very large data sets you may be interested in 
running an even faster analysis by using just a subset of your data. This would
allow you to more easily explore the affect of many different parameter settings
on your results before running the full data set. Two forms of subsampling are 
possible: first, subsampling the number of Samples in your analysis, and second,
subselecting the number of reads in your analysis. 


.. note::
    Importantly, no matter what you do in ipyrad, it will never delete or 
    modify your original fastq data files. Assembly objects simply store
    information about Samples, and Samples simply contain statistics about 
    data files. Samples can be discarded from an Assembly, in which case the
    Assembly loses some information, however, this does not delete any data files. 
    Nevertheless, to retain Sample information ipyrad only allows Samples to be 
    discarded during branching, so that Sample information is always retained 
    in the parent branch. See the example below.


**Subselecting samples**:
You can subselect Samples by creating a new branch. Here we call the new 
branch "sub4", and pass it a list of Sample names in addition to the new branch 
name. **This does NOT delete any files** (see above), but simply copies
a subset of information from "base" to the new assembly "sub4".
If you accidentally discarded the wrong Samples you could re-create "sub4" by 
simply branching "base" again with a different list of Samples. 

.. code:: bash

    ## Create new branch of base Assembly named sub4 and pass it
    ## the names of four Samples (if no names it keeps all Samples)
    ## names should be separated by a comma and spaces are optional. 
    ## The '\' character simply continues our list across a line-break
    ## for easier viewing.

    >>> ipyrad -p params-base.txt -b sub4 29154_superba 30556_thamno \
                                          30686_cyathophylla 32082_przewalskii


.. parsed-literal::
    loading Assembly: base
    from saved path: ~/Documents/ipyrad/tests/cli/cli.json
    Sample name not found: 4
    Creating a new branch called 'sub4' with 4 Samples
    Writing new params file to params-sub4.txt


.. code:: bash

    ## print stats for sub4 to confirm that Samples were discarded
    >>> ipyrad -p params-sub4.txt -r


.. parsed-literal::
  Summary stats of Assembly sub4
  ------------------------------------------------
                          state  reads_raw
  39618_rex                   1     822263
  40578_rex                   1    1707942
  41478_cyathophylloides      1    2199740
  41954_cyathophylloides      1    2199613 


**Running step 2**:


.. code:: bash

    ## run step2 
    >>> ipyrad -p params-sub4.txt -s 2 -r


.. parsed-literal::
  --------------------------------------------------
   ipyrad [v.0.2.5]
   Interactive assembly and analysis of RADseq data
  --------------------------------------------------
   loading Assembly: base
   from saved path: ~/Downloads/pedicularis/base.json
   ipyparallel setup: Local connection to 4 Engines
 
   Step2: Filtering reads 
   [####################] 100%  processing reads      | 0:02:48 
   Saving Assembly.


    Summary stats of Assembly base
    ------------------------------------------------
                            state  reads_raw  reads_filtered
    29154_superba               2     696994           92448
    30556_thamno                2    1452316           93666
    30686_cyathophylla          2    1253109           89122
    32082_przewalskii           2     964244           92016



Run step 3 (clustering and aligning)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
This is generally one of the longest running steps, depending on how
many unique clusters (loci) there are in each sample. Using more
processors will allow it run much faster. On my laptop with 4 cores this
step finishes in approximately 30 minutes. From the results you can see
that there are many clusters found in each sample (clusters\_total), but
very few are recovered at high depth (clusters\_hidepth).
The coverage would of course be much better if we did not subsample the
data set in step2. Also, this data set has fairly low coverage to begin with. 
We can either lower the mindepth setting to allow us to use more of this low
depth data, or we can decide to go ahead with our mindepth setting (currently
at the default of 6) and simply discard most of our data. I know, how about 
we create a branch so that we can do both!

.. code:: bash

    ## create a lowdepth branch
    >>> ipyrad -p params-sub4.txt -b sub4-lowdepth.

.. parsed-literal::
  loading Assembly: base
  from saved path: ~/Downloads/pedicularis/base.json
  Creating a branch of assembly base called sub4
  Writing new params file to params-sub4.txt


.. code:: bash

    ## create a lowdepth branch
    >>> ipyrad -p params-base.txt -s 3

.. parsed-literal::
   --------------------------------------------------
    ipyrad [v.0.2.5]
    Interactive assembly and analysis of RADseq data
   --------------------------------------------------
    loading Assembly: base
    from saved path: ~/Downloads/pedicularis/base.json
    ipyparallel setup: Local connection to 4 Engines
  
    Step3: Clustering/Mapping reads
    [####################] 100%  dereplicating         | 0:00:01 
    [####################] 100%  clustering            | 0:01:01 
    [####################] 100%  chunking              | 0:00:00 
    [####################] 100%  aligning              | 0:25:44 
    [####################] 100%  concatenating         | 0:00:05 
    Saving Assembly.


Use a text editor to enter the following new **mindepth_majrule** value 
in the file ``params-sub4-lowdepth.txt``:

.. parsed-literal::
    ## 2                  ## [mindepth_majrule] ...


Steps 4-5 (joint estimation of error rate & heterozygosity)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
As you can see in the results the error rate is about 10X the heterozygosity
estimate. The latter does not vary significantly across samples. With
data of greater depth the estimates will be more accurate.

.. code:: bash

    >>> ipyrad -p params-sub4.txt          -s 45 -r 
    >>> ipyrad -p params-sub4-lowdepth.txt -s 45 -r


.. parsed-literal::
    ... add output here
    

Run step 5 (consensus base calls)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
This is another step that can be computationally intensive. Here it
takes about 15 minutes on 4 cores. Although many clusters are filtered
out at this step (especially due to low depth) their information is
retained for the VCF output later so that the coverage/depth of excluded
reads can be examined.


.. code:: bash

    ipyrad -p params-base.txt -s 5
    ipyrad -p params-base.txt -r


.. parsed-literal::

    --------------------------------------------------
     ipyrad [v.0.1.70]
     Interactive assembly and analysis of RADseq data
    --------------------------------------------------
     loading Assembly: base [~/Downloads/pedicularis/base.json]
     ipyparallel setup: Local connection to 4 Engines
   
     Step5: Consensus base calling 
       Diploid base calls and paralog filter (max haplos = 2)
       error rate (mean, std):  0.00703, 0.00331
       heterozyg. (mean, std):  0.04071, 0.00396
       Saving Assembly.


    Summary stats of Assembly base
    ------------------------------------------------
                            state  reads_raw  reads_filtered  clusters_total
    29154_superba               5     696994           92448           45531 
    30556_thamno                5    1452316           93666           45745 
    30686_cyathophylla          5    1253109           89122           50306 
    32082_przewalskii           5     964244           92016           44242 
    33413_thamno                5     636625           89428           52053 
    33588_przewalskii           5    1002923           92418           46674 
    35236_rex                   5    1803858           92807           57801 
    35855_rex                   5    1409843           92883           45139 
    38362_rex                   5    1391175           93363           41580 
    39618_rex                   5     822263           92096           47295 
    40578_rex                   5    1707942           93386           45295 
    41478_cyathophylloides      5    2199740           93846           41965 
    41954_cyathophylloides      5    2199613           91756           47735 

                            clusters_hidepth  hetero_est  error_est  reads_consens  
    29154_superba                        978    0.038530   0.006630            821  
    30556_thamno                         987    0.038266   0.006009            810  
    30686_cyathophylla                   757    0.044680   0.004627            606  
    32082_przewalskii                    686    0.046796   0.007077            523  
    33413_thamno                         728    0.041466   0.004528            597  
    33588_przewalskii                    904    0.041445   0.011253            709  
    35236_rex                            767    0.042423   0.005119            629  
    35855_rex                           1106    0.035123   0.012086            844  
    38362_rex                           1140    0.041206   0.004702            943  
    39618_rex                           1258    0.040696   0.009077           1011  
    40578_rex                            832    0.045177   0.002789            689  
    41478_cyathophylloides               992    0.041085   0.004468            872  
    41954_cyathophylloides              1307    0.032387   0.013090            983  


    Full stats files
    ------------------------------------------------
    step 1: None
    step 2: ./pedicularis/base_edits/s2_rawedit_stats.txt
    step 3: ./pedicularis/base_clust_0.85/s3_cluster_stats.txt
    step 4: ./pedicularis/base_clust_0.85/s4_joint_estimate.txt
    step 5: ./pedicularis/base_consens/s5_consens_stats.txt
    step 6: None
    step 7: None
    
    


Step 6 (clustering and aligning across samples)
-----------------------------------------------

This step clusters consensus loci across Samples using the same
threshold for sequence similarity as used in step3.

.. code:: bash

    ipyrad -p params-base.txt -s 6


.. parsed-literal::

    --------------------------------------------------
     ipyrad [v.0.1.70]
     Interactive assembly and analysis of RADseq data
    --------------------------------------------------
     loading Assembly: base [~/Downloads/pedicularis/base.json]
     ipyparallel setup: Local connection to 4 Engines
   
     Step6: Clustering across 13 samples at 0.85 similarity
       Saving Assembly.



Branch the assembly
-------------------

Here we will branch the assembly to create different assemblies that we
will use as our final outputs. The main parameter we will focus on is
the ``min_samples_locus``, which is the minimum number of samples that
must have data at a locus for the locus to be retained in the data set.
We create a ``min4``, ``min8``, and ``min12`` data sets.

.. code:: bash

    ipyrad -p params-base.txt -b min4
    ipyrad -p params-base.txt -b min8
    ipyrad -p params-base.txt -b min12



.. parsed-literal::
    
      loading Assembly: base [~/Downloads/pedicularis/base.json]
      Creating a branch of assembly base called min4
      Writing new params file to params-min4.txt
    
      loading Assembly: base [~/Downloads/pedicularis/base.json]
      Creating a branch of assembly base called min8
      Writing new params file to params-min8.txt
    
      loading Assembly: base [~/Downloads/pedicularis/base.json]
      Creating a branch of assembly base called min12
      Writing new params file to params-min12.txt


Change the parameter settings in params.txt for each assembly
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. parsed-literal::

    ## Enter the changes below into the params files using a text editor
    
    ## in the file params-min4.txt
    4       ## [21] [min_samples_locus] ...
    
    ## in the file params-min8.txt
    8       ## [21] [min_samples_locus] ...
    
    ## in the file params-min12.txt
    12      ## [21] [min_samples_locus] ...



Step 7 (final filtering and create output files)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Filter and create output files for the three assemblies with different
values for the parameter ``min_samples_locus``. 

.. code:: bash

    ipyrad -p params-min4.txt -s 7 
    ipyrad -p params-min8.txt -s 7 
    ipyrad -p params-min12.txt -s 7 


.. parsed-literal::

    
     --------------------------------------------------
      ipyrad [v.0.1.70]
      Interactive assembly and analysis of RADseq data
     --------------------------------------------------
      loading Assembly: min4 [~/Downloads/pedicularis/min4.json]
      ipyparallel setup: Local connection to 4 Engines
    
      Step7: Filter and write output files for 13 Samples.
        Outfiles written to: ~/Downloads/pedicularis/min4_outfiles
        Saving Assembly.
    
     --------------------------------------------------
      ipyrad [v.0.1.70]
      Interactive assembly and analysis of RADseq data
     --------------------------------------------------
      loading Assembly: min8 [~/Downloads/pedicularis/min8.json]
      ipyparallel setup: Local connection to 4 Engines
    
      Step7: Filter and write output files for 13 Samples.
        Outfiles written to: ~/Downloads/pedicularis/min8_outfiles
        Saving Assembly.
    
     --------------------------------------------------
      ipyrad [v.0.1.70]
      Interactive assembly and analysis of RADseq data
     --------------------------------------------------
      loading Assembly: min12 [~/Downloads/pedicularis/min12.json]
      ipyparallel setup: Local connection to 4 Engines
    
      Step7: Filter and write output files for 13 Samples.
        Outfiles written to: ~/Downloads/pedicularis/min12_outfiles
        Saving Assembly.


Take a look at the stats summary 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Each assembly that finishes step 7 will create a stats.txt output summary
in the 'assembly_name'_outfiles/ directory. This includes information about 
which filters removed data from the assembly, how many loci were recovered
per sample, how many samples had data for each locus, and how many variable
sites are in the assembled data. 


.. code:: python

    cat ./pedicularis/min4_outfiles/min4_stats.txt


.. parsed-literal::


    ## The number of loci caught by each filter.
    ## ipyrad API location: [assembly].statsfiles.s7_filters
    
                               locus_filtering
    total_prefiltered_loci                1206
    filtered_by_rm_duplicates                2
    filtered_by_max_indels                 159
    filtered_by_max_snps                     0
    filtered_by_max_hetero                  15
    filtered_by_min_sample                 921
    filtered_by_edge_trim                    0
    total_filtered_loci                    221
    
    
    ## The number of loci recovered for each Sample.
    ## ipyrad API location: [assembly].stats_dfs.s7_samples
    
                            sample_coverage
    29154_superba                       151
    30556_thamno                        120
    30686_cyathophylla                  118
    32082_przewalskii                   132
    33413_thamno                        155
    33588_przewalskii                    89
    35236_rex                           154
    35855_rex                           145
    38362_rex                           154
    39618_rex                           156
    40578_rex                            90
    41478_cyathophylloides              156
    41954_cyathophylloides               97
    
    
    ## The number of loci for which N taxa have data.
    ## ipyrad API location: [assembly].stats_dfs.s7_loci
    
        locus_coverage  sum_coverage
    1              NaN             0
    2              NaN             0
    3              NaN             0
    4               65            65
    5               36           101
    6               16           117
    7               10           127
    8               10           137
    9                6           143
    10               2           145
    11              12           157
    12               7           164
    13              57           221
    
    
    ## The distribution of SNPs (var and pis) across loci.
    ## pis = parsimony informative site (minor allele in >1 sample)
    ## var = all variable sites (pis + autapomorphies)
    ## ipyrad API location: [assembly].stats_dfs.s7_snps
    
        var  sum_var   pis  sum_pis
    0   260      260  1140     1140
    1   130      390    45     1185
    2   130      520    12     1197
    3   133      653     3     1200
    4    90      743     4     1204
    5   110      853     0     1204
    6    77      930     0     1204
    7    70     1000     2     1206
    8    64     1064     0     1206
    9    32     1096     0     1206
    10   31     1127     0     1206
    11   30     1157     0     1206
    12   16     1173     0     1206
    13   14     1187     0     1206
    14    5     1192     0     1206
    15    6     1198     0     1206
    16    2     1200     0     1206
    17    4     1204     0     1206
    18    2     1206     0     1206


Take a peek at the .loci (easily human-readable) output
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
This is one fo the first places I usually look when an assembly finishes. It 
provides a clean view of the data with variable sites (-) and parsimony informative
SNPs (*) highlighted. Use the unix commands **less** or **head** to look at this
file briefly.

.. code:: bash

    ## head -n 50 prints just the first 50 lines of the file to stdout
    head -n 50 pedicularis/min4_outfiles/min4.loci


.. parsed-literal::

    29154_superba              CCTTGGTSACCTTMGCWCCWGAYGGRTCCTTCTTCTCCACACTCTTKATRACACCAACAGCAACAGTC
    32082_przewalskii          CCTTGGTSACCTTRGCWCCWGAYGG-TCCTTCTTCTCCACACTCTTGATRACACCAACAGCAACAGTC
    35236_rex                  CCTTGGTCACCTTAGCACCTGATGG-TCCTTCTTCTCCACACTCTTGATGACACCAACAGCAACAGTC
    38362_rex                  CCTTGGTCACCTTAGCACCTGATGGRTCCTTCTTCTCCACACTCTTGATGACACCAACAGCAAC-GTC
    //                                -     -  -  -  -  -                    -  -                  |
    33413_thamno               TAGACAACCAGTGCCTTCTTGTCTATCAGTCTCACACCTGTCTTCGGTACTTGCGGTACTTAGAAGCA
    33588_przewalskii          GAGACAACCAGTGCCTTCTTGTCTATCAGCCTCACACCTGTCTTCGGTACTTTCGGTACTTAGAAGCA
    35855_rex                  TAGACAACCAGTGCCGTCTTGTCTATCAGTCTCACACCTGTCTTCGGTACTTGCGGTACTTAGAAGCA
    38362_rex                  TAGACAACCAGTGCCTTCTTGTCTATCAGTCTCACACCTGTCTTCGGTACTTGCGGTACTTAGAAGCA
    39618_rex                  TAGACAACCAGTGCCTTCTTGTCTATCAGTCTCACACCTGTCTTCGGTACTTGCGGTACTTAGAAGCA
    40578_rex                  TAGACAACCAGTGCCTTCTTGTCTATCAGTCTCACACCTGTCTTCGGTACTTGCGGTACTTAGAAGCA
    41478_cyathophylloides     TAGACAACCAGTGCCGTCTTGTCTATCAGTCTCACACCTGTCTTCGGTACTTGCGGTACTTAGAAGCA
    41954_cyathophylloides     TAGACAACCAGTGCCGTCTTGTCTATCAGTCTCACACCTGTCTTCGGTACTTGCGGTACTTAGAAGCA
    //                         -              *             -                      -               |
    29154_superba              AGCAAGCGAAGAAAACGTAAGGGCGCGCGTTAGCACTCCTGCAAGAAAACGGC-CTAGCTAACGCGCCC
    30556_thamno               AGCAAGCGAAGAAAACGTAAGGGCGCGCGTTAGCACTCCTGCAAGAAAACGGC-CTAGCTAACGCGCCC
    30686_cyathophylla         AGCAAGCGAAGAAAACGTAAGGGCGCGCGTTAGCACTCCTGCAAGAAAACGGC-CTAGCTAACGCGCCC
    32082_przewalskii          AGCAAGCGAAGAAAACGTAAGGGCGCGCGTTAGCACTCCTGCAAGAAAACGGC-CTAGCTAACGCGCCC
    33413_thamno               AGCAAGCGAAGAAAACGTAAGGGCGCGCGTTAGCACTCCTGCAAGAAAACGGC-CTAGCTAACGCGCCC
    33588_przewalskii          AGCAAGCGAAGAAAACGTAAGGGCGCGCGTTAGCACTCCTGCAAGAAAACGGC-CTAGCTAACGCGCCC
    35236_rex                  AGCAAGCGAAGAAAACGTAAGGGCGCGCGTTAGCACTCCTGCAAGAAAACGGC-CTAGCTAACGCGCCC
    35855_rex                  AGCAAGCGAAGAAAACGTAAGGGCGCGCGTTAGCACTCCTGCAAGAAAACGGC-CTAGCTAACGCGCCC
    38362_rex                  AGCAAGCGAAGAAAACGTAAGGGCGCGCGTTAGCACTCCTGCAAGAAAACGGC-CTAGCTAACGCGCCC
    39618_rex                  AGCAAGCGAAGAAAACGTAAGGGCGCGCGTTAGCACTCCTGCAAGAAAACGGC-CTAGCTAACGCGCCC
    40578_rex                  AGCAAGCGAAGAAAACGTAAGGGCGCGCGTTAGCACTCCTGCAAGAAAACGGC-CTAGCTAACGCGCCC
    41478_cyathophylloides     AGCAAGCGAAGAAAACGTAAGGGCGCGCGTTAGCACTCCTGCAAGAAAACGGC-CTAGCTAACGCGCCC
    41954_cyathophylloides     AGCAAGCGAAGAAAACGTAAGGGCGCGCGTTAGCACTCCTGCAAGAAAACGGCNCTAGCTAACGCGCCC
    //                                                                                              |
    30686_cyathophylla         TAGCAATAAATGCAAGAATATTTACTTCCATAATTTCGTCGGTTTTTTAATTCGCAATAACTCGGGAT
    32082_przewalskii          TAGCAATAAATGCAAGAATATTGACTTCCATAATTTCGTCGGTTTTTTAATTCGCAATAACTCGGGAT
    33588_przewalskii          TAGCAATAAATGCAAGAATATTGACTTCCATAATTTCGTCGGTTTTTTAATTCGCAATAACTCGGGAT
    35236_rex                  TAGCAATAAATGCAAGAATATTKACTTCCATAATTTCGTCKGTTTTTTAATTCGCAATAACTCGGGAT
    38362_rex                  TAGCAATAAATGCAAGAATATTTACTTCCATAATTTCGTCTGTTTTTTAATTCGCAATAACTCGGGAT
    39618_rex                  TAGCAATAAATGCAAGAATATTTACTTCCATAATTTCGTCTGTTTTTTAATTCGCAATAACTCGGGAT
    40578_rex                  TAGCAATAAATGCAAGAATATTTACTTCCATAATTTCGTCTGTTTTTTAATTCGCAATAACTCGGGAT
    41478_cyathophylloides     TAGCAATAAATGCAAGAATATTT-CTTCCATAATTTCGTCGGTTTTTTAATTCGCAATAACTCGGGAT
    //                                               *                 *                           |
    35236_rex                  CTCTAGGTGGAGCTCCAGCTGGGTCTGAACCAGATCCTCCGTAAKCGGATCATCATGTGCGAGTTGAC
    35855_rex                  CTCTAGGTGGAGCTCCAGCTGGGTCTGAACCAGATCCTCCGTAAGCGGATCATCATGTGCGAGTGGAC
    38362_rex                  CTCTAGGTGGAGCTCCAGCTGGGTCTGAACCAGATCCTCCGTAAGCGGATCATCATGTGCGAGTTGAC
    39618_rex                  CTCTAGGTGGAGCTCCAGCTGGGTCTGAACCAGATCCTCCGTAAGCGGATCATCATGTGCGAGTTGAC
    //                                                                     -                   -   |
    30556_thamno               C-TTCTGATTAATCTG-AAATTGTAATCAAATGAAATYAAACAGCAAAAACAATGACTSGATAAACTA
    33413_thamno               CTTTCTGWTTAATCTGMAAATTGTAATCAAATGAAATCAAACARCAAAAACAATGACTYGAYAAWCYR
    35236_rex                  C-TTCTGATTAATCTG-AAATTGTAATCAAATGAAATCAAACA-CAAAAACAATGACT-GATAAACTA
    41478_cyathophylloides     CTTTCTGATTAATCTGCAAATTGTAATCAAATGAAATCAAACAGCAAAAACAATAACTTGATAAAATA
    //                                -        -                    -     -          -   -  -  ----|
    30556_thamno               GAAAGATWT-AYTGTAGACGTAWTKGATCRSAGGWKGAGGTGATGWATCATAWTCAT-ATCAGAGGAG
    38362_rex                  GAAAGATTTCACTGTAGACGTAATGGATCAGAGGTTGAGGTGATGRATCATAATCATGATCAGAGGWG
    39618_rex                  GAAAGATTTCACTGTAGACGTAWWGGATCMSAGGWKGAGGTGATGRATCATAATCATKATCAGAGGAG


peek at the .phy files
~~~~~~~~~~~~~~~~~~~~~~
This is the concatenated sequence file of all loci in the data set. It is typically
used in phylogenetic analyses, like in the program *raxml*. 


.. code:: bash

    ## cut -c 1-80 prints only the first 80 characters of the file
    cut -c 1-80 pedicularis/min4_outfiles/min4.phy


.. parsed-literal::

    13 15034
    29154_superba              CCTTGGTSACCTTMGCWCCWGAYGGRTCCTTCTTCTCCACACTCTTKATRACA
    30556_thamno               NNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN
    30686_cyathophylla         NNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN
    32082_przewalskii          CCTTGGTSACCTTRGCWCCWGAYGGNTCCTTCTTCTCCACACTCTTGATRACA
    33413_thamno               NNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN
    33588_przewalskii          NNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN
    35236_rex                  CCTTGGTCACCTTAGCACCTGATGGNTCCTTCTTCTCCACACTCTTGATGACA
    35855_rex                  NNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN
    38362_rex                  CCTTGGTCACCTTAGCACCTGATGGRTCCTTCTTCTCCACACTCTTGATGACA
    39618_rex                  NNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN
    40578_rex                  NNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN
    41478_cyathophylloides     NNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN
    41954_cyathophylloides     NNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN


peek at the .snp file
~~~~~~~~~~~~~~~~~~~~~
This is similar to the phylip file format, but only variable site columns are 
included. All SNPs are the file, in contrast to the .usnps file, which selects
only a single SNP per locus. 


.. code:: bash

    ## cut -c 1-80 prints only the first 80 characters of the file
    cut -c 1-80 pedicularis/min4_outfiles/min4.snp


.. parsed-literal::

    13 711
    29154_superba              SMWWYRKRNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNGYATNYGAMGT
    30556_thamno               NNNNNNNNNNNNNNNNA-YGGSTACTACWYWTKRSWKWW-TAGTAT-NNNNGT
    30686_cyathophylla         NNNNNNNNNNNNTGNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNCRACTT
    32082_przewalskii          SRWWY-GRNNNNGGNNNNNNNNNNNNNNNNNNNNNNNNNNNNGTATNNNNNGG
    33413_thamno               NNNNNNNNTTTGNNNNWMCRGYYWCYRYNNNNNNNNNNNNNNNNNNNNNNNGT
    33588_przewalskii          NNNNNNNNGTCTGGNNNNNNNNNNNNNNNNNNNNNNNNNNNNRTRKNNNNNGG
    35236_rex                  CAATT-GGNNNNKKKTA-C-G-TACTACNNNNNNNNNNNNNNG-ATMNNNNGT
    35855_rex                  NNNNNNNNTGTGNNGGNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNCGRCGT
    38362_rex                  CAATTRGGTTTGTTGTNNNNNNNNNNNNTCATGAGTTRAGTWNNNNMCGACGT
    39618_rex                  NNNNNNNNTTTGTTGTNNNNNNNNNNNNTCWWGMSWKRAKTAGTATMCGRCGT
    40578_rex                  NNNNNNNNTTTGTTNNNNNNNNNNNNNNNNNNNNNNNNNNNNGTATNNNNNGT
    41478_cyathophylloides     NNNNNNNNTGTGTGNNACCGATTAATACWKATKRSWKRAGYAGTATNCRACGT
    41954_cyathophylloides     NNNNNNNNTGTGNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNGT


peek at the .snp file for the min12 assembly
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Similar to above but you can see that there is much less missing data (Ns). 

.. code:: bash

    ## cut -c 1-80 prints only the first 80 characters of the file
    cut -c 1-80 pedicularis/min12_outfiles/min12.snp


.. parsed-literal::

    13 44
    29154_superba              GTTGACGCGGTTCTCCCCCGCGCAGTGCAGTCCGCTAANNAGTT
    30556_thamno               GTTGACGCGGTTCTCACCCGCGCAGCGCGGTACACTAAGAAATT
    30686_cyathophylla         TTTGACGCGGTTCTCACCCGCGCAGCGCGGTCCACTAAGAAATT
    32082_przewalskii          GGTGATGCAACCCCTACCCGCGTGGCGTARYCAGTTARGAGACC
    33413_thamno               GTTGACGCGGTTTTCACCCGCGCAGCGCGGTACACTAAGAAATT
    33588_przewalskii          GGTGATGCAACCCCTACCCGCGTG-TC-ARYCAGTTARGAGACC
    35236_rex                  GTTGACGCGGTTCTCACCCGCGCAGCGCGRYACACTWAKMAATT
    35855_rex                  GTKRMYKMGGTTCTCACCCGCGCAGCGCGGTACACTAAGANNNT
    38362_rex                  GTTGACGCGGTTCTCACCCGCGCAGCGCGGTCCACTAAGAAATT
    39618_rex                  GTTGACGCGGTTCTCAYYYRYRCAGCGCGGTCCACTAAGAAATT
    40578_rex                  GTTGACGCGGTTCTCACCCGCGCAGCGCGGTACACTAAGAAATT
    41478_cyathophylloides     GTTGACGCGGTTCTCACCCGCGCAATGCAGTCCGCCAAGAAATT
    41954_cyathophylloides     GTTGACGCGGTTCTCACCCGCGCAATGCAGTCCGCCAAGAAATT


downstream analyses
~~~~~~~~~~~~~~~~~~~
...
