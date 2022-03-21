# SOP: Erstellung CanRsik-konformer VCF-Dateien für die PRS-Berechnung

## 1 Hintergrund & Zielsetzung

Das CanRisk Web-Tool (www.canrisk.org, [1]) ermöglicht die Berechnung des zukünftigen Risikos für eine Brust- oder Eierstockkrebserkrankung basierend auf verschiedenen genetischen und nicht-genetischen Faktoren. Zu den genetischen Faktoren kann dabei neben Mutationen in definierten Brust- und/oder Eierstockkrebs-Risikogenen auch der sogenannte Polygene Risikoscore (PRS) gehören. Der PRS stellt die Aggregation verschiedener (Niedrig-)Risikoallele, die isoliert betrachtet keinen relevanten Effekt auf das Erkrankungsrisiko haben, in einem einzigen Wert dar. Im Gegensatz zu den einzelnen Loci, können PRS-Werte in den Extremen der Verteilung einen klinisch relevanten Effekt auf das Erkrankungsrisiko haben, wie in mehreren Studien auf Grundlage großer Datensätze übereinstimmend gezeigt wurde [2–5]. 

Der PRS für ein Individuum *i* wird auf Grundlage eines gegebenen Sets von Risiko-Allelen der Größe *N* wie folgt berechnet:
![image](https://user-images.githubusercontent.com/17177177/150496138-fd8596eb-d009-49fd-a2a9-46061fb8b83b.png)

wobei &beta;<sub>j</sub>:= logarithimierter Odds-Ratio für Allel *j* und g<sub>ij</sub> := Anzahl der beobachteten Effekt-Allele in Person *i*.

Das CanRisk Web-Tool erlaubt die automatisierte Berechnung und Integration des PRS in die Risikoberechnung. 
Dazu können die Genotypen der betreffenden Loci als Datei im Variant Call Format (VCF) zur Verfügung gestellt werden. 
Das VCF-Format hat sich als Standardformat zur Darstellung genomischer Varianten etabliert; die aktuelle Version v4.3 ist unter https://samtools.github.io/hts-specs/VCFv4.3.pdf vollumfänglich spezifiziert. 
Um die Kompatibilität mit CanRisk zu gewährleisten, müssen spezifische Voraussetzungen in der VCF-Datei erfüllt sein:

• Alle relevanten PRS-Loci müssen angegeben sein, auch wenn kein alternatives Allel vorliegt oder aus technischen Gründen kein Genotyp festgestellt werden kann.

• Als Humangenomreferenz dient GRCh37 ("hg19").

• Referenz- und Effekt-Allele müssen genauso dargestellt werden wie vom CanRisk-Tool erwartet. 

Funktionale Beispiel-VCFs können unter
https://canrisk.atlassian.net/wiki/spaces/FAQS/pages/29884417/Variant+Call+Format+File+Examples heruntergeladen und getestet werden; eine Dokumentation mit erforderlichen Spezifikationen oder Vorgaben zu ihrer Erstellung ist jedoch nicht hinterlegt. Eine entsprechende Anleitung soll daher diese SOP bieten. Dabei wird davon ausgegangen, dass die betreffenden PRS-Loci via NGS genotypisiert wurden, und die Sequenzierungen als auf die Referenzsequenz kartierte ("gemappte") Reads im "binary alignment map" (BAM) Format vorliegen. 

## 2 Variant Calling

Die Betrachtung von in der Allgemeinbevölkerung weit verbreiteten Genotypen und ihrer Verteilung statt ausschließlich seltener, potentiell pathogener genetischer Variation stellt einen Paradigmenwechsel in der DNA-Sequenzanalyse dar. 
Aus diesem Grund bietet nicht jede Software zum Variant Calling passende Voraussetzungen zum Erstellen geeigneter VCF-Dateien. 
Im Unterschied zur “klassischen” Varianten-Detektion, bei der das gesamte Genom nach Abweichungen von der Referenzsequenz abgesucht wird, und ausschließlich diese in der resultierenden VCF reportiert werden, soll hier der Genotyp spezifizierter Loci eindeutig ermittelt werden. 
Für die PRS-Berechnung ist dabei wesentlich, dass das Nicht-Vorhandensein eines SNPs (durch Genotyp (GT) 0/0 in der VCF spezifiziert) von einem fehlenden Call aufgrund technischer Probleme eindeutig unterschieden werden kann.
Ein geeigneter Varianten-Caller muss somit also über die Funktionalität verfügen, eine VCF-Datei über die gesamte Menge der mit Aufruf spezifizierten Loci zu erstellen. 
Diese Option wird von den Varianten-Callern FreeBayes [6] (https://github.com/freebayes/freebayes), Strelka2 [7] (https://github.com/Illumina/strelka) und GATK HaplotypeCaller [8] geboten, die alle frei verfügbar sind. 

Die zu genotypisierenden PRS-Loci werden bei den genannten Tools mittels Angabe einer (komprimierten) VCF-Datei mit dem Programmaufruf definiert. 
Zur Einsparung von Ressourcen ist es außerdem empfehlenswert, die genomischen Zielregionen einzugrenzen, d.h., das Varianten-Calling mittels einer BED-Datei (http://genome.cse.ucsc.edu/FAQ/FAQformat.html#format1) auf jene Regionen zu beschränken, in denen SNPs genotypisiert werden sollen. 
Entsprechende kompatible VCF- und (komprimierte) BED-Dateien für diese Zwecke werden von der AG Bioinformatik zur Verfügung gestellt. **ToDo**


Die für die PRS-Berechnung mit CanRisk zu betrachtenden Allele sind in komma-separierten Textdateien (CSV-Dateiformat) spezifiziert und können hier
https://canrisk.atlassian.net/wiki/spaces/FAQS/pages/35979266/What+variants+are+used+in+the+PRS heruntergeladen werden. 
Diese Dateien mit Suffix &ast;.prs definieren Chromosom, genomische Position (Referenz hg19), Referenz- und Effekt-Allel, sowie die erwartete (europäische) Allelfrequenz des Effekt-Allels und den logarithmierten Odds-Ratio für jeden Locus: 

![image](https://user-images.githubusercontent.com/17177177/150498454-2b0202a1-93a1-4c42-8d6f-4535dd05bb18.png)


## 3 Post-Prozessierung der VCF-Datei

### 3.1. Ersetzung nicht-normalisierter Loci

Das CanRisk-Webtool erwartet die Angabe von Referenz- und Effekt-Allelen wie in den &ast;.prs-Dateien spezialisiert. 
Die Allele sind hier im wesentlichen links-normalisiert [9], dies trifft aber nicht auf alle Loci zu, wie etwa

```
#Name,Chromosome,Position,Reference_Allele,Effect_Allele,effect_allele_frequency,log_odds_ratio
4_84370124_TAA_TA,4,84370124,TAA,TA,0.5353,-0.0464
22_38583315_AAAAG_AAAAGAAAG,22,38583315,AAAAG,AAAAGAAAG,0.2819,-0.0471
```

Die links-normalisierten Allele TA/T und A/AAAG müssten daher in der VCF-Datei entsprechend ersetzt werden, dies kann manuell vorgenommen werden oder mithilfe eines Inhouse-Skripts. 

### 3.2 Ersetzung nicht-gecallter/low-quality Loci durch Allelfrequenzen

Das CanRisk Webtool erwartet die Angabe aller in der PRS-Berechnung zu berücksichtigenden Varianten in der betreffenden VCF-Datei. 
Standardmäßig geht dann der mit dem `GT`-Tag spezifierte Genotyp in den PRS ein (`GT`=`0/0` &rarr; g<sub>ij</sub>:=0 ; `GT`=`0/1` &rarr; g<sub>ij</sub>:=1 ; `GT`=`1/1` &rarr; g<sub>ij</sub>:=2). 
Der mögliche Ausfall eines Locus kann u.a. durch sein Fehlen, `GT`=`./.` oder Angabe eines dritten alternativen Allels in der VCF spezifiziert sein. 
Darüber hinaus besteht die Möglichkeit, gecallte Varianten im Post-Processing aus der VCF herauszufiltern, da die Mindestanforderungen für eine zuverlässige Genotypisierung, z.B. bezüglich der Read-Tiefe (VCF-Tag `DP`), nicht erfüllt sind. 
Die AG Bioinformatik GC-HBOC empfiehlt hierzu folgende Schwellenwerte:

* WGS: `DP` &geq; 15

* WES / targeted: `DP` &geq; 20


Kann ein für die PRS-Berechnung benötigter Locus nicht oder nicht mit ausreichender Evidenz genotypisiert werden, empfiehlt die AG Bioinformatik, hier den Genotyp durch die Dosage, d.h. das Doppelte der in Europa erwarteten Allelfrequenz, zu ersetzen. Die betreffende Allelfrequenz kann den &ast;.prs-Dateien entnommen werden. Die Angabe erfolgt mithilfe des Tags `DS` in der VCF-Datei. 

**Beispiel: Angabe von Dosage statt Genotyp**

&ast;.prs-Datei:

```
#alpha = 0.444
#Name,Chromosome,Position,Reference_Allele,Effect_Allele,effect_allele_frequency,
log_odds_ratio
1_100880328_A_T,1,100880328,A,T,0.4100,0.0373
1_10566215_A_G,1,10566215,A,G,0.3291,-0.0586
…
```

VCF-Datei (Minimal-Beispiel, sortiert):

```
##fileformat=VCFv4.3
#CHROM	POS	ID	REF	ALT	QUAL	FILTER	INFO	FORMAT	Sample
1	10566215	A	T	.	.	.	DS	0.6582
1	100880328	A	T	.	.	.	DS	0.8200
…
```


Aufgrund der Möglichkeit der Ersetzung von unbekannten Genotypen durch die doppelte erwartete Effekt-Allelfrequenz stellt sich die Frage, wie viele Loci imputiert werden können, ohne das Prädiktionsvermögen des PRS signifikant zu verringern. 
Mit jeder Substitution verringert sich die Varianz der zugrundeliegenden Verteilung, der individuelle standardisierte PRS liegt mit höherer Wahrscheinlichkeit nahe 0.


## 4 Abschließende Bemerkungen

### 4.1 Manueller PRS-Input

Neben Generierung und Upload entsprechender VCF-Dateien bietet das CanRisk-Webtool ebenfalls die Möglichkeit, den (nicht standardisierten) PRS direkt zu berechnen und im Web-Tool einzugeben. 
Je nach Setting kann diese Berechnung u.U. komfortabler sein als die Erstellung geeigneter VCF-Dateien. 
Eine entsprechende SOP zum Verfahren ist von der AG Bioinformatik geplant.

### 4.2 Abstammungs-Check

PRSs sind vorrangig für Europäer&ast;innen entwickelt und evaluiert, und daher für Nicht-Europäer&ast;innen nicht grundsätzlich anwendbar. 
Aktuelle Praxis im GC-HBOC ist das Abfragen der Herkunft der Vorfahren vor Anwendung des PRS. 
Grundsätzlich wäre aber auch ein Abstammungs-Check auf Grundlage der Genotypen der zur Verfügung stehenden, verbreiteten genomischen Varianten und ihren bekannter Weise erwarteten Allel-Frequenzen möglich.


# Appendix

## GATK HapltypeCaller

Beispielaufruf mit gatk v4.2.3

```
gatk HaplotypeCaller -I <sample>.bam -R <path/to/reference>.fa -O <sample>.vcf -L BCAC_313_PRS_regions.bed --alleles BCAC_313_PRS.vcf.gz --dbsnp BCAC_313_PRS.vcf.gz
```

## freebayes

Beispielaufruf mit freebayes 1.3.3

```
freebayes -f <path/to/reference>.fa --variant-input BCAC_313_PRS.vcf.gz --only-use-input-alleles -t BCAC_313_PRS_regions.bed <sample>.bam
```

freebayes returns not only the variants in the input VCF, but all variants. Thus, some post-processing might be necessary, like breaking multi-allelic variants and removing variants that were not part of the input variant list.

## Strelka

Beispielaufruf mit Strelka v2.9.2

```
python2 path/to/configureStrelkaGermlineWorkflow.py --bam <sample>.bam --forcedGT BCAC_313_PRS.vcf.gz --targeted --referenceFasta <path/to/refrence> --runDir <path/to/workdir>

python2 <path/to/workdir/>runWorkflow.py -m local -j <#nodes>
```

# Referenzen

[1]	Carver, T. et al. CanRisk Tool—A Web Interface for the Prediction of Breast and 	Ovarian Cancer Risk and the Likelihood of Carrying Genetic Pathogenic Variants. 	Cancer Epidemiol Biomarkers Prev (2020).

[2]	Mavaddat et al. Prediction of breast cancer risk based on profiling with common genetic variants. JNCI: Journal of the National Cancer Institute (2015).

[3]	Kuchenbaecker et al. Evaluation of polygenic risk scores for breast and ovarian cancer risk prediction in BRCA1 and BRCA2 mutation carriers. JNCI: Journal of the National Cancer Institute (2017).

[4]	Mavaddat, N. et al. Polygenic risk scores for prediction of breast cancer subtypes. The American Journal of Human Genetics (2019).

[5]	Borde et al. Performance of breast cancer polygenic risk scores in 760 female CHEK2 germline mutation carriers. JNCI: Journal of the National Cancer Institute (2021).

[6]	Garrison, E. & Marth, G. Haplotype-based variant detection from short-read sequencing. arXiv preprint (2012).

[7]	Sangtae, K et al. Strelka2: fast and accurate calling of germline and somatic variants. Nature methods (2018).

[8] McKenna A, Hanna M, Banks E, Sivachenko A, Cibulskis K, Kernytsky A, Garimella K, Altshuler D, Gabriel S, Daly M, DePristo MA. (2010). The Genome Analysis Toolkit: a MapReduce framework for analyzing next-generation DNA sequencing data. Genome Res, 20:1297-303. DOI: 10.1101/gr.107524.110.

[9]	Tan, A. et al. Unified representation of genetic variants. Bioinformatics (2015).
