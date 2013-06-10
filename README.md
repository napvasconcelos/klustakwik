KlustaKwik
==========

Documentation for KlustaKwik and Masked KlustaKwik
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------

0) Introduction
------------------------
------------------------

KlustaKwik is an implementation of a hard Expectation-Maximization algorithm for the purposes of clustering, a major application of which is the spike-sorting of neurophysiological data. Older versions of the code can be found here:
[Sourceforge](http://sourceforge.net/projects/klustakwik/).

Masked KlustaKwik is a new algorithm designed to be used in conjunction with [SpikeDetekt](http://klusta-team.github.io/spikedetekt) for clustering spike waveforms recorded on large dense probes with high-channel counts, and [KlustaViewa](http://klusta-team.github.io/klustaviewa) for manual verification and adjustment of clustering results.

The new algorithm takes advantage of the fact that spikes tend to occur only on a subset of the features, with the remainder of the channels containing only multi-unit noise. The information of the relevant channels for each spike is encoded in an additional .fmask file which is output from SpikeDetekt, along with the usual .fet features file. The vectors in the .fmask file are the same size as those in the .fet file, but are restricted to lie between 0 and 1. *Unmasked* channels are channels on which spiking activity has been found to occur by the program SpikeDetekt,
whereas *masked* channels contain only noise. The .fmask file is a text file, every line of which is a vector
giving the positions of the unmasked channels. In the .fmask file, **1** denotes *unmasked* and **0** denotes
*masked*, values between 0 and 1 are also permitted at the boundaries of detected spikes.

1) Essential input files
---------------------

KlustaKwik 3.0 is backward compatible with previous versions. To use it in "classic" mode, just run the same command you would have for version 2.x (as documented on the sourceforge page linked above). It will produce the same results, but should run about 10 times faster. In this mode, KlustaKwik takes a single input file (mydata.fet.n) containing feature vectors, and produces an output file (mydata.clu.n) containing cluster numbers. It also produces a log file (mydata.klg.n).

To use in "masked" mode, you need to provide also a file with mask information (mydata.fmask.n). This text file is produced by SpikeDetekt, and contains a set of floating-point vectors between 0 and 1 indicating the weight with which different features should be used in clustering each point.  

The suffix .n allows you to keep track of data recorded on a probe with mutliple shanks. Thus, for shank n, the relevant files would be the following  output from SpikeDetekt:

    mydata.fet.n
    mydata.fmask.n

2) Command line input
----------------------
----------------------

A typical command to run the masked version of KlustaKwik therefore looks as follows in a linux terminal:

    [yourterminal]$./KlustaKwik yourfetfilename shanknumber -UseDistributional 1 -UseMaskedInitialConditions 1 -AssignToFirstClosestMask 1 -MaxPossibleClusters 500 -MinClusters 130 -MaxClusters 130 -PenaltyK 1 -PenaltyKLogN 0 -UseFeatures 111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111110
    
e.g. if your .fet file is called **recording.fet.4** (and your other files are **recording.mask.4**, **recording.fmask.4**)  for the fourth shank, then the command looks something like:

    [yourterminal]$./KlustaKwik recording 4 -UseDistributional 1 -UseMaskedInitialConditions 1 -AssignToFirstClosestMask 1 -MaxPossibleClusters 500 -MinClusters 130 -MaxClusters 130 -PenaltyK 1 -PenaltyKLogN 0 -UseFeatures 111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111110

You may consider writing a script to generate such a complicated command. To understand the various options employed above, see the next section.

**We apologize for the current somewhat complicated set-up. Everything will be simplified once beta testing has been completed - slightly simplified on 23/04/13.**


3) Parameters
-------------------
-------------------

The current release of masked KlustaKwik has an enormous range of parameters which can be adjusted
according to the user's needs. Many unnecessary parameters will soon be phased out, but in the interim
to help beta-testers, we will describe a few of the relevant ones.

    Usage: MaskedKlustaKwik FileBase ElecNo [Arguments]

    Arguments (with default values): 

    FileBase    electrode
    ElecNo	1
    MinClusters	20
    MaxClusters	30
    MaxPossibleClusters	100
    nStarts	1
    RandomSeed	1
    Debug	0
    Verbose	1
    UseFeatures	11111111111100001
    DistDump	0
    DistThresh	6.907755
    FullStepEvery	20
    ChangedThresh	0.050000
    Log	1
    Screen	1
    MaxIter	500
    StartCluFile	
    SplitEvery	40
    PenaltyK	0.000000
    PenaltyKLogN	1.000000
    Subset	1
    PriorPoint	1
    SaveSorted	0
    SaveCovarianceMeans	0
    UseMaskedInitialConditions	0
    AssignToFirstClosestMask	0
    UseDistributional	0
    help	0

The above defaults cause KlustaKwik (classical EM algorithm) to run exactly as previous versions, but 10 times faster.
The only difference is that the parameter PenaltyMix has been replaced with two parameters, PenaltyK and PenaltyKLogN.

+ **Penalties**

 

PenaltyMix 1 corresponds to (PenaltyK 0, PenaltyKLogN 1) (Bayesian Information Criterion) *default*

PenaltyMix 0 corresponds to (PenaltyK 1, PenaltyKLogN ) (Akaike Information Criterion)

The parameters PenaltyK and PenaltyKLogN can only be given positive values.
The higher the values, the fewer clusters you obtain. Higher penalties discourage cluster splitting.

AIC is recommended for larger probes. Evidence suggests anything between AIC and BIC gives reasonable results. 


+ **UseDistributional** (*default* 0)

To use KlustaKwik in "masked" mode, set this to 1.
This enables the use of the new `masked Expectation-Maximization' algorithm. To use the new algorithm, it is recommended that you change most of the defaults.

It has been observed that using soft masks of the form. e.g.:

0 0 0 0.3 0.3 0.3 1 1 1 1 1 1 0.4 0.4 0.4 0 0 0 0 0 0 

leads to improved clusterings than using binary masks:

0 0 0 0 0 0 1 1 1 1 1 1 0 0 0 0 0 0 0 0 0 

So your .fmask file will work better if it contains floats.

+ **UseFeatures** (*default* 


Given a .fet file containing vectors of a set length, you may chose which features are used for the automatic clustering process and omit others. 

Your choice of which features to use is expressed in a string of 0's and 1's. Include a **1** for every feature you would like to include and a **0** for every feature you want to leave out (e.g. features that corresponding to bad channels that you don't want).

e.g. Your .fet file looks as follows (97-dimensional data obtained from taking the first 3 principal components from each channel of a 32-channel probe, and the final feature, time):

    97
    -43206 -6407 652 5181 3675 11860 -30066 -145 -3720 -33491 -15892 5840 -57344 11025 -16524 -8815 -4046 -157 -30493 -7224 507 -676 9700 -6809 -39397 -211 -5468 -22511 -352 -186 -1127 -10679 9435 -25851 -22021 4238 -26672 -17418 11085 10035 -2864 9975 -20875 -29624 13367 -14849 -8633 4408 -19751 -24882 8184 -28165 4853 2136 -21114 -10823 3004 -8716 -4686 4793 11529 -9505 8930 13056 5133 3442 -1085 -4045 4965 -63934 -3827 -6996 -46672 4384 -7775 -29050 -549 -9260 -24985 20865 -14532 -17785 -6168 862 -22615 1593 -5319 -972 -1660 10063 -106 1600 5378 1963 -4880 7357 2
    -43496 -2123 1697 7471 2085 15302 -30109 2679 -2230 -34578 -13000 6082 -56265 17120 -14871 -9465 -2789 -1098 -31967 -3309 -255 -74 10289 -6540 -39518 3964 -4349 -22094 1344 1581 -1050 -11070 9237 -28378 -19400 2499 -27143 -15752 11980 10508 -3931 10804 -22956 -28054 11357 -15002 -8109 5675 -22333 -22982 6313 -26356 7494 4534 -21650 -8983 2692 -8909 -4412 5892 10953 -10840 8443 14319 3268 5287 -236 -4808 6393 -64397 2215 -5170 -45979 8712 -5203 -29763 2625 -9344 -22857 23174 -11572 -17982 -5042 1361 -22808 4433 -5423 1133 -3507 14082 1741 292 7829 2157 -5736 8542 2
    -33069 18324 -563 -108726 -20512 -11860 -30497 10828 20051 -31983 -2309 24404 -11377 22760 14372 -27932 -2754 7252 -9373 43163 -5022 -48048 22884 -11330 -15395 36739 28855 -14419 13872 -976 -45777 305 5922 -4145 9977 5397 -9010 -3391 -13322 7835 278 30982 -47770 -15958 -364 -22770 1765 3022 -1997 8759 13377 -42757 -19253 27619 6931 -4810 1240 -14390 -39503 -9258 2612 -25492 -7973 -16064 -13039 -3647 -17640 -11906 11890 2259 -8161 -12922 -23660 -28462 7323 -18757 -25355 -6518 -15544 -26716 12220 3591 -9433 -32399 -5565 -36616 -7977 18442 -10641 -28989 -14625 -404 -1207 6840 -17422 -3009 41
    2903 -6480 -8171 -6906 -14576 10433 29882 -4122 -7722 6632 29771 9732 -5294 -3917 50886 -7037 2175 4469 11817 -4083 -7448 -7708 3939 3240 -14890 2721 4889 -11259 20546 11612 -36244 -5598 27674 9815 -4252 -16080 -13403 6912 11855 8481 11759 -24013 -41589 -36261 11447 -32228 2745 -16386 -9704 4388 -20931 3564 32431 -9779 -5692 26247 -11884 -1589 37336 12236 3088 32036 -18553 14636 18875 1886 22541 10968 -14971 8588 23946 -23505 -26785 70034 -29746 -28573 50845 -24927 -166980 114185 -57570 -83298 67056 -39670 -103696 90495 -36092 -52810 31849 -5378 -6437 3499 -34861 -11266 -8738 24618 84

If you want to include all the PCA's but not time you need the following string:

    111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111110



+ **UseMaskedInitialConditions**, **AssignToFirstClosestMask** and **SplitEvery**

The quality of clustering obtained from EM-type algorithms is sensitive to the initialization. A good way to initialize that works better than random initializations is to start with clusters which depend on unique masks or clusters containing similar masks. Set both of these options to 1 to enable masked initializations. At the same time set **MinClusters**= **MaxClusters** to the number of distinct masks (also set MaxPossibleClusters to a value greater than this). 

(If you do not know the number of distinct masks, run KlustaKwik, with arbitrary values of MinClusters and MaxClusters, read the first line giving the number of distinct masks, and then prematurely terminate the program).

Starting with the total number of distinct masks is not usually necessary and prohibitive in terms of memory usage for large datasets. Therefore the options 
**-UseMaskedInitialConditions 1 -AssignToFirstClosestMask 1** used together enable a fixed number of starting clusters (determined by MinClusters and MaxClusters) by 
randomly selecting a fixed number of distinct derived binary masks and assigning points according to their their nearest mask according to Hamming distance.
e.g. the derived binary mask of 

0 0 0 0.3 0.3 0.3 1 1 1 1 1 1 0.4 0.4 0.4 0 0 0 0 0 0

is

0 0 0 0 0 0 1 1 1 1 1 1 0 0 0 0 0 0 0 0 0. 

**SplitEvery** is an integer which is the number of iterations after which KlustaKwik attempts to split existing clusters.
When using masked initializations, to save time due to excessive splitting, set **SplitEvery** to a large number, close
to the number of distinct masks or the number of chosen starting masks. 



+ **PriorPoint**
Please set this to 1 at all times when using Masked KlustaKwik.



4) Glossary of Parameters
-------------------------
-------------------------

**Filebase** - Name of your .fet and .mask file, e.g. if your feature file is called mydata.fet.1, then Filebase is *mydata*.

**ElecNo** - Shank number of your probe.

**MinClusters** - The minimum number of starting clusters.

**MaxClusters** - The maximum number of starting clusters.

**MaxPossibleClusters** - The largest permitted number of clusters.





