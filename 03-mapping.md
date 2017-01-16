---
layout: page
title: Análisis transcriptómicos
subtitle: Alineamiento de lecturas
minutes: 5
---
> ## Objetivos de aprendizaje {.objectives}
>
> *  Alinear datos de secuenciación a una referencia (genomas o transcriptomas)
> *  Entender como interpretar datos de alineamiento de secuenciación masiva
> *  Primer acercamiento a los formatos de coordenadas SAM y BAM

> ## Datos requeridos {.prereq}
>
> Para proceder con esta práctica, se requieren los resultados de tarea de 
> ensamble de transcriptoma de la clase pasada:
>
> *  Archivos fastq de lecturas filtradas por calidad usando Trimmomatic via Trinity
> *  Archivo fasta de ensamble de transcriptoma completo

Usaremos los archivos fastq que utilizamos la clase pasada, pero 
filtrados por calidad por Trimmomatic via Trinity 
resultantes de su tarea. Estos se encuentran el en directorio
`trinity_out_dir` con la extensión `*.qtrim.gz`. 

Con lecturas en pares, habrá casos en los que ambas lecturas sean filtradas (P), 
y casos en los que una lectura se conservó y la otra se desechó (U). Las lecturas 
filtradas se encuentran en los siguientes archivos (ejemplo):

~~~ {.output}
reads.left.fq.gz.P.qtrim.gz     
reads.left.fq.gz.U.qtrim.gz
reads.right.fq.gz.P.qtrim.gz
reads.right.fq.gz.U.qtrim.gz
~~~

También usaremos el genoma de referencia de nuestro organismo, 
*Saccharomyces pombe*:

[Sp_genome.fa](datasets/genome/Sp_genome.fa) 

Como ya sabemos, es buena idea primero ver que los datos están en el 
formato correcto. Para este ejercicio solo utilizaremos las lecturas
que conservaron sus pares después de ser filtradas:

~~~ {.bash}
$ for i in trinity_out_dir/*.P.qtrim.gz ; do zcat $i | head ; done 
$ ln -s ./trinity_out_dir/*.P.qtrim.gz .
~~~

Una vez que verificamos que las lecturas están el formato correcto,
vamos a alinear las lecturas y transcritos al genoma utilizando Bowtie2 via TopHat. 

Pueden encontrar el manual en el siguiente [link](https://ccb.jhu.edu/software/tophat/manual.shtml).

## Mapeando los transcritos al genoma

En los casos en los que contamos con una referencia, es muy útil analizar los transcritos
en su contexto genómico. En este ejercicio usaremos el programa de mapeo 
[GMAP](http://research-pub.gene.com/gmap/).

Primero instalemos el programa:

~~~ {.bash}
$ sudo apt-get update
$ sudo apt-get install gmap
$ cd /usr/lib/gmap/
$ sudo cp * /usr/bin
$ cd Directorio_con_transcriptoma
~~~

El primer paso para realizar el mapeo es generar el índice por medio del comando:

~~~ {.bash}

$ gmap_build -d genome -D . -k 13 Sp_genome.fa
~~~
~~~ {.output}
-k flag specified (not as 15), but not -b, so building with basesize == kmer size
Sorting chromosomes in chrom order.  To turn off or sort other ways, use the -s flag.
Running /usr/bin/fa_coords  -o ./genome.coords Sp_genome.fa
Opening file Sp_genome.fa
  Processed short contigs (<1000000 nt): .
============================================================
Contig mapping information has been written to file ./genome.coords.
You should look at this file, and edit it if necessary
If everything is okay, you should proceed by running
    make gmapdb
============================================================
Running /usr/bin/gmap_process  -c ./genome.coords Sp_genome.fa | /usr/bin/gmapindex -d genome -D . -A 
Reading coordinates from file ./genome.coords
Logging contig genome at genome:1..451226 in genome genome
Total genomic length = 451226 bp
Chromosome genome has universal coordinates 1..451226
Writing IIT file header information...done
Processing null division/chromosome...sorting...writing...done (1 intervals)
Writing IIT file footer information...done
Writing IIT file header information...done
Processing null division/chromosome...sorting...writing...done (1 intervals)
Writing IIT file footer information...done
Running /usr/bin/gmap_process  -c ./genome.coords Sp_genome.fa | /usr/bin/gmapindex -d genome -F . -D . -G
Genome length is 451226 nt
Trying to allocate 42303*4 bytes of memory...succeeded.  Building genome in memory.
Reading coordinates from file ./genome.coords
Writing contig genome to universal coordinates 1..451226 in genome genome
A total of 0 non-ACGTNX characters were seen in the genome.
Running cat ./genome.genomecomp | /usr/bin/gmapindex -b 13 -k 13 -q 3 -d genome -F . -D . -O
Allocating 67108865*4 bytes for offsets
Indexing offsets of oligomers in genome genome (13 bp every 3 bp), position 0
Writing 67108865 offsets compressed to file with total of 142909 positions...done
Running cat ./genome.genomecomp | /usr/bin/gmapindex -b 13 -k 13 -q 3 -d genome -F . -D . -P
Trying to allocate 142909*4 bytes of memory...succeeded.  Building positions in memory.
Indexing positions of oligomers in genome genome (13 bp every 3 bp), position 0
Writing 142909 genomic positions to file ./genome.ref133positions ...
Copying files to directory ./genome
~~~

Una vez generado el índice mapeamos los transcritos generados en la tarea al genoma:

~~~ {.bash}
$ gmap -n 0 -D . -d genome trinity_out_dir/Trinity.fasta -f samse > trinity_gmap.sam
~~~
~~~ {.output}
Trying to allocate 142909*4 bytes of memory...succeeded.  Building positions in memory.
Indexing positions of oligomers in genome genome (13 bp every 3 bp), position 0
Writing 142909 genomic positions to file ./genome.ref133positions ...
Copying files to directory ./genome
usuario@HP334:~/Documents/TEMPO$ gmap -n 0 -D . -d genome trinity_out_dir/Trinity.fasta -f samse > trinity_gmap.sam
GMAP version 2012-06-12 called with args: gmap -n 0 -D . -d genome trinity_out_dir/Trinity.fasta -f samse
Note: >1 sequence detected, so index files are being memory mapped.
  GMAP can run slowly at first while the computer starts to accumulate
  pages from the hard disk into its cache.  To copy index files into RAM
  instead of memory mapping, use -B 3, -B 4, or -B 5, if you have enough RAM.
  For more speed, also try multiple threads (-t <int>), if you have multiple processors or cores.
Pre-loading compressed genome....done (169,212 bytes, 42 pages, 0.00 sec)
Looking for index files in directory ./genome
  No gammaptrs file, because kmersize 13 == basesize 13
  Offsetscomp file is genome.ref13133offsetscomp
  Positions file is genome.ref133positions
Allocating memory for ref offsets, kmer 13, interval 3...done (268,435,460 bytes, 0.05 sec)
Pre-loading ref positions, kmer 13, interval 3....done (571,636 bytes, 140 pages, 0.00 sec)
No paths found for TR3|c0_g1_i1
No paths found for TR3|c0_g2_i1
No paths found for TR5|c0_g1_i1
No paths found for TR9|c0_g1_i1
No paths found for TR13|c0_g1_i1
No paths found for TR31|c0_g1_i1
No paths found for TR33|c0_g1_i1
No paths found for TR33|c1_g1_i1
No paths found for TR37|c0_g1_i1
No paths found for TR39|c0_g1_i1
No paths found for TR58|c0_g1_i1
No paths found for TR58|c0_g2_i1
No paths found for TR60|c0_g1_i1
No paths found for TR60|c0_g2_i1
No paths found for TR79|c0_g1_i1
No paths found for TR79|c0_g2_i1
No paths found for TR82|c0_g1_i1
No paths found for TR82|c0_g2_i1
No paths found for TR83|c0_g1_i1
No paths found for TR89|c0_g1_i1
No paths found for TR98|c0_g1_i1
No paths found for TR99|c2_g1_i1
No paths found for TR102|c0_g1_i1
No paths found for TR102|c1_g1_i1
No paths found for TR111|c0_g1_i1
No paths found for TR115|c0_g1_i1
No paths found for TR123|c0_g1_i1
No paths found for TR130|c0_g1_i1
No paths found for TR139|c0_g1_i1
No paths found for TR147|c0_g1_i1
No paths found for TR160|c0_g1_i1
No paths found for TR183|c0_g1_i1
No paths found for TR183|c0_g2_i1
No paths found for TR191|c0_g1_i1
No paths found for TR215|c0_g1_i1
Processed 354 queries in 4.14 seconds (85.51 queries/sec)
~~~

Si revisamos el resultado, es un archivo con coordenadas en un formato llamado
SAM (Sequence Alignment/Map format). Veamos en que consiste este formato:

~~~ {.bash}
$ more trinity_gmap.sam
~~~

### El formato SAM

Veamos un ejemplo más pequeño de este formato.
Supongamos que tenemos el siguiente alineamiento:

~~~ {.output}
Coor	12345678901234 5678901234567890123456789012345
ref	AGCATGTTAGATAA**GATAGCTGTGCTAGTAGGCAGTCAGCGCCAT
+r001/1	      TTAGATAAAGGATA*CTG
+r002	     aaaAGATAA*GGATA
+r003	   gcctaAGCTAA
+r004	                 ATAGCT..............TCAGC
-r003	                        ttagctTAGGC
-r001/2	                                      CAGCGGCAT
~~~

El formato SAM correspondiente será el siguiente:

~~~ {.output}
@HD	VN:1.5	SO:coordinate
@SQ	SN:ref	LN:45
r001	99	ref	7	30	8M2I4M1D3M	=	37	39	TTAGATAAAGGATACTG	*
r002	0	ref	9	30	3S6M1P1I4M	*	0	0	AAAAGATAAGGATA	*
r003	0	ref	9	30	5S6M	*	0	0	GCCTAAGCTAA	*	SA:Z:ref,29,-,6H5M,17,0;
r004	0	ref	16	30	6M14N5M	*	0	0	ATAGCTTCAGC	*
r003	2064	ref	29	17	6H5M	*	0	0	TAGGC	*	SA:Z:ref,9,+,5S6M,30,1;
r001	147	ref	37	30	9M	=	7	-39	CAGCGGCAT	*	NM:i:1
~~~

El formato SAM es un formato de texto plano que nos permite guardar datos de 
secuenciación en formato ASCII delimitado por tabulaciones. 

Está compuesto de dos secciones principales:

*  El encabezado
*  El alineamiento

La sección del **encabezado** comienza con el caracter `@` seguido por uno de lo códigos 
de dos letras que sirven para características de los alineamientos en este archivo. 
Cada línea esta delimitada por tabulaciones y, además de las líneas que comienzan con 
`@CO`, cada campo de datos tiene el formato `TAG:VALUE`, en donde `TAG` es una cadena
de dos caracteres que define el formato y contenido de `VALUE`. 

El encabezado no es indispensable pero contiene información acerca de la versión del 
archivo así como si está ordenado o no. Por ello es recomendable incluirlo.

La sección de **alineamiento** contiene la siguiente información:

1. **QNAME**	Nombre de la referencia, QNAME (SAM)/Nombre de la lectura(BAM). 
Se utiliza para agrupar alineamientos que están juntos, como es el caso de alineamientos
de lecturas por pares o una lectura que aparece en alineamientos múltiples. 
2.  **FLAG**	Set de información describiendo el alineamiento. Provee la siguiente información:
	* ¿Hay múltiples fragmentos?
	* ¿Todos los fragmentos están bien alineados?
	* ¿Está alineado este fragmento?
	* ¿No ha sido alineado el siguiente fragmento?
	* ¿Es esta referencia la cadena inversa?
	* ¿El el siguiente fragmento la cadena reversa?
	* ¿Es este el primer fragmento?
	* ¿Es este el último fragmento?
	* ¿Es este un alineamiento secundario?
	* ¿Esta lectura falló los filtros de calidad?
	* ¿Es esta lectura un duplicado por PCR o óptico?
3. **RNAME**	Nombre de la secuencia de referencia.
3. **POS**	Posición de alineamiento izquierda (base 1).
3. **MAPQ**	Calidad del alineamiento.
3. **CIGAR**	cadena CIGAR.
3. **RNEXT**	Nombre de referencia del par (mate) o la siguiente lectura.
3. **PNEXT**	Posición del par (mate) o la siguiente lectura.
3. **TLEN**	Longitud del alineamiento.
3. **SEQ**	La secuencia de prueba de este alineamiento (en este caso la secuencia de la lectura).
3. **QUAL**	La calidad de la lectura.
3. **TAGs**	Información adicional. 

> ## Cadenas CIGAR {.callout}
>
> La secuencia alineada a la referencia puede tener bases adicionales que no están en 
> la referencia o puede no tener bases en la lectura que si están en la referencial.
> La cadena CIGAR es una cadena que codifica cada base y la caracteristica de cada una en 
> el alineamiento.
> 
> Por ejemplo, la cadena CIGAR:
> 
> ~~~ {.output}
> CIGAR: 3M1I3M1D5M
> ~~~
> 
> indica que las primera 3 bases de la lectura alinea con la referencia (3M), la siguiente base
> no existe en la referencia (1I), las siguientes 3 bases alinean con la referencia (3M), la 
> siguiente base no existe en la lectura (1D), y 5 bases más alinean con la referencia (5M). 

Como pueden ver estos archivos contienen muchísima información que puede ser analizada
usando scripts que arrojen estadísticas del alineamiento. Programas como 
[Picard](http://broadinstitute.github.io/picard/) realizan
este tipo de análisis.

La versión comprimida de los archivos tipo SAM se conoce como BAM (binary sam). 
Convirtamos el archivo SAM a BAM usando samtools:

~~~ {.bash}
$ samtools view -Sb trinity_gmap.sam > trinity_gmap.bam
~~~

No abriremos este archivo ya que, dado que esta en formato binario, es ilegible. 
Sin embargo, tenemos que realizar dos últimos pasos visualizar 
los resultados:

~~~ {.bash}
$ samtools sort trinity_gmap.bam trinity_gmap
$ samtools index trinity_gmap.bam
~~~

El primer paso ordena los resultados por sus coordenadas y el segundo crea índices
para hacer más rápida la visualización usando un navegador. Visualizaremos esto datos
en la próxima clase.

## Mapeando las lecturas filtradas al genoma

Primero generaremos un índice de bowtie2 para el genoma:

~~~ {.bash}
$ bowtie2-build Sp_genome.fa Sp_genome 
$ ls *bt2
~~~ 
~~~ {.output}
Sp_genome.1.bt2        
Sp_genome.2.bt2              
Sp_genome.3.bt2       
Sp_genome.4.bt2           
Sp_genome.rev.1.bt2          
Sp_genome.rev.2.bt2
~~~

Usamos tophat2 para mapear las lecturas. Este programa nos permite dividir lecturas
que atraviesan sitios de splicing:

~~~ {.bash}
$ tophat2 -I 300 -i 20 Sp_genome \
 Sp_log.left.fq.gz.P.qtrim.gz,Sp_hs.left.fq.gz.P.qtrim.gz,Sp_ds.left.fq.gz.P.qtrim.gz,Sp_plat.left.fq.gz.P.qtrim.gz \
 Sp_log.right.fq.gz.P.qtrim.gz,Sp_hs.right.fq.gz.P.qtrim.gz,Sp_ds.right.fq.gz.P.qtrim.gz,Sp_plat.right.fq.gz.P.qtrim.gz
~~~ 
~~~ {.output}
[2016-04-04 14:18:50] Beginning TopHat run (v2.0.13)
-----------------------------------------------
[2016-04-04 14:18:50] Checking for Bowtie
		  Bowtie version:	 2.2.4.0
[2016-04-04 14:18:50] Checking for Bowtie index files (genome)..
[2016-04-04 14:18:50] Checking for reference FASTA file
[2016-04-04 14:18:50] Generating SAM header for Sp_genome
Error: could not open pipe gzip -cd < RNASEQ_data/Sp_log.left.fq.gz
usuario@HP334:~/Documents/TEMPO$ ls tophat2 -I 300 -i 20 Sp_genome \
>     Sp_log.left.fq.gz.P.qtrim.gz,Sp_hs.left.fq.gz.P.qtrim.gz,Sp_ds.left.fq.gz.P.qtrim.gz,Sp_plat.left.fq.gz.P.qtrim.gz \
>     Sp_log.right.fq.gz.P.qtrim.gz,Sp_hs.right.fq.gz.P.qtrim.gz,Sp_ds.right.fq.gz.P.qtrim.gz,Sp_plat.right.fq.gz.P.qtrim.gz
ls: cannot access tophat2: No such file or directory
ls: cannot access 20: No such file or directory
ls: cannot access Sp_genome: No such file or directory
ls: cannot access Sp_log.left.fq.gz.P.qtrim.gz,Sp_hs.left.fq.gz.P.qtrim.gz,Sp_ds.left.fq.gz.P.qtrim.gz,Sp_plat.left.fq.gz.P.qtrim.gz: No such file or directory
ls: cannot access Sp_log.right.fq.gz.P.qtrim.gz,Sp_hs.right.fq.gz.P.qtrim.gz,Sp_ds.right.fq.gz.P.qtrim.gz,Sp_plat.right.fq.gz.P.qtrim.gz: No such file or directory
usuario@HP334:~/Documents/TEMPO$  tophat2 -I 300 -i 20 Sp_genome     Sp_log.left.fq.gz.P.qtrim.gz,Sp_hs.left.fq.gz.P.qtrim.gz,Sp_ds.left.fq.gz.P.qtrim.gz,Sp_plat.left.fq.gz.P.qtrim.gz     Sp_log.right.fq.gz.P.qtrim.gz,Sp_hs.right.fq.gz.P.qtrim.gz,Sp_ds.right.fq.gz.P.qtrim.gz,Sp_plat.right.fq.gz.P.qtrim.gz

[2016-04-04 14:21:02] Beginning TopHat run (v2.0.13)
-----------------------------------------------
[2016-04-04 14:21:02] Checking for Bowtie
		  Bowtie version:	 2.2.4.0
[2016-04-04 14:21:02] Checking for Bowtie index files (genome)..
[2016-04-04 14:21:02] Checking for reference FASTA file
[2016-04-04 14:21:02] Generating SAM header for Sp_genome
[2016-04-04 14:21:02] Preparing reads
	 left reads: min. length=25, max. length=68, 329490 kept reads (432 discarded)
	right reads: min. length=25, max. length=68, 329760 kept reads (162 discarded)
[2016-04-04 14:21:10] Mapping left_kept_reads to genome Sp_genome with Bowtie2 
[2016-04-04 14:21:24] Mapping left_kept_reads_seg1 to genome Sp_genome with Bowtie2 (1/2)
[2016-04-04 14:21:24] Mapping left_kept_reads_seg2 to genome Sp_genome with Bowtie2 (2/2)
[2016-04-04 14:21:24] Mapping right_kept_reads to genome Sp_genome with Bowtie2 
[2016-04-04 14:21:39] Mapping right_kept_reads_seg1 to genome Sp_genome with Bowtie2 (1/2)
[2016-04-04 14:21:39] Mapping right_kept_reads_seg2 to genome Sp_genome with Bowtie2 (2/2)
[2016-04-04 14:21:40] Searching for junctions via segment mapping
	Coverage-search algorithm is turned on, making this step very slow
	Please try running TopHat again with the option (--no-coverage-search) if this step takes too much time or memory.
[2016-04-04 14:21:45] Retrieving sequences for splices
[2016-04-04 14:21:45] Indexing splices
Building a SMALL index
[2016-04-04 14:21:45] Mapping left_kept_reads_seg1 to genome segment_juncs with Bowtie2 (1/2)
[2016-04-04 14:21:45] Mapping left_kept_reads_seg2 to genome segment_juncs with Bowtie2 (2/2)
[2016-04-04 14:21:45] Joining segment hits
[2016-04-04 14:21:47] Mapping right_kept_reads_seg1 to genome segment_juncs with Bowtie2 (1/2)
[2016-04-04 14:21:47] Mapping right_kept_reads_seg2 to genome segment_juncs with Bowtie2 (2/2)
[2016-04-04 14:21:47] Joining segment hits
[2016-04-04 14:21:48] Reporting output tracks
-----------------------------------------------
[2016-04-04 14:22:15] A summary of the alignment counts can be found in ./tophat_out/align_summary.txt
[2016-04-04 14:22:15] Run complete: 00:01:12 elapsed
~~~

Exploramos el resultado, el cuál es un archivo tipo SAM. 

~~~ {.bash}
$ samtools view tophat_out/accepted_hits.bam | head
~~~ 

> ## Tarea - Alineando las lecturas filtradas al transcriptoma {.challenge}
>
> Hemos alineado las lecturas al genoma pero queremos alinearlas también directamente
> al transcriptoma. La tarea consiste en revisar el manual de TopHat2 y usar las opciones
> que nos permite mapear lecturas directamente a transcriptomas. 
> 
> Agreguen el archivo Trinity.fasta de referencia y el archivo de mapeo en formato BAM
> (ordenado e indizado) a su repositorio en un nuevo directorio llamado Transcriptome_Mapping.
> También agreguen un archivo de texto plano con el commando que utilizaron para realizar
> este análisis.
> Lo deberán agregar antes de la clase del viernes 8 de abril.
> 
> * **Pista 1:** No podrán utilizar el índice generado previamente.
> * **Pista 2:** No deberán iniciar un nuevo repositorio, solo tienen que agregar el nuevo 
> directorio y su contenido. 








