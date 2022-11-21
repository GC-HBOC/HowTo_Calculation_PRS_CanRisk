# Variant check

# mappability

The following variant have low mappability when using 200mer (single end):

* chr1:121546196 A>G - ok in WGS
* chr5:44619400 A>G - ok in WGS
* chr7:101909160 G>A - ok in WGS/WES
* chr7:144351809 G>T - borderline as  ~10% of reads have MAPQ0 in WGS/WES

# not in gnomAD

The following variants are not in gnomAD:

* chr2:217091173 GA>G (AF=0.0364) - AC0 variant in gnomAD with 0% AF - should be removed?!
* chr5:58945885 C>T (AF=0.5762, Mavaddat only) - at end of 5 base deletion with 70% AF - change to deletion?
* chr6:87094101 T>C (AF=0.2809) - at end of 23 base deletion with 38% AF - change to deletion?
* chr17:30841059 G>T (AF=0.2573) - at start of 4 base MNP (GAGT>TCCA) - change to MNP if pipeline reports MNPs!


# comparison to WGS samples

The following variants were not found in 866 WGS samples (megSAP pipeline Tübingen using BWA-mem2, ABRA2, freebayes):

* chr2:217091173 GA>G	0	0
* chr5:58945885 C>T	0	0
* chr6:87094101 T>C	0	0

**Note: they are the same in 'not in gnomAD'**

These variants were found way too often for the given gnomAD frequency for europeans (NFE):

* chr10:38234698 C>A (AF=0.374) - found 353 times but AF 0.00025 - low complexity region with warning in gnomAD - should be removed?!
* chr2:9998855 T>C (AF=0.1168, Mavaddat only)	found 104 times but AF 0.00846 - completely chaotic region with warning in gnomAD - should be removed?!


# not covered in custom Twist exome

31 of the 313 variants are not covered with at lease 20x (avg coverage 100x) although in design of Twist custom exome V2 (Tübingen):

* chr1	7857010	7857021
* chr1	113903252	113903263
* chr1	121546190	121546201
* chr1	145830792	145830803
* chr2	88059300	88059311
* chr2	217091167	217091179
* chr3	141394011	141394024
* chr3	172567441	172567452
* chr4	91673702	91673719
* chr5	44619394	44619405
* chr5	72669174	72669185
* chr5	133071360	133071371
* chr6	21923573	21923584
* chr6	151734837	151734848
* chr7	100351026	100351037
* chr10	93532424	93532437
* chr11	65804954	65804965
* chr11	108396669	108396680
* chr12	184454	184465
* chr12	28194443	28194454
* chr14	104747635	104747646
* chr15	75458036	75458047
* chr15	100365608	100365619
* chr16	52504907	52504918
* chr17	30841053	30841064
* chr17	46206486	46206497
* chr17	55132407	55132418
* chr22	19778608	19778619
* chr22	28739549	28739560
* chr22	28807730	28807732
* chr22	29155878	29155889
