# Annotation in reference genome graphs

## Motivation

Current methods of annotating genomes rely on genomic intervals as a core formalism.
There are some difficulties in generalizing this formalism to reference graphs.
A genomic interval corresponds to a path in the graph.
However, if we restrict the annotation to one path in the graph, the alternate alleles included in the graph are not included in the annotation.
We argue that connected subgraphs are a more appropriate formalism for genome graphs.

Using a new core formalism for annotation necessarily means that there is not currently infrastructure built out to support it.
We need exchangeable representations of the data, software support, and analysis tools to make the formalism useful for practitioners.

## Project

We developed a few proof-of-concepts for utilizing both interval-based features and continuous-valued annotations with genome graphs. Our goals were to...

* Agree on a basic data representation for annotating subgraphs with continuous valued data
* Specify a human-readable file format for expressing the data representation
* Produce a demonstration set of annotation data for a genome graph
* Augment VG to produce and work with these data
* Hook the file format into an existing graph visualization tool

## Genomic Interval to Graph Annotations

### Easy Case

![Annotation Import, The Easy Case](fig/annotation_easy_case.svg)

For short annotations within non-complex graph structures, integration of interval-based annotations using the reference path is relatively straight-forward.

### Harder Cases

![Annotation Import, The Harder Cases](fig/annotation_harder_cases.svg)

When annotations span over complex subgraphs it is hard to tell which base-pairs on which alternate allele nodes correspond to the reference annotation.
This is essentially a liftover problem that will need to be solved on a case-by-case basis with command line parameters.

## Gene-level RNAseq quantification pipeline

See the subproject-specific [README](gene_quant/README.md).

## Visualization using MoMIG

![MoMIG Genome Graph Visualization software screenshot](fig/momig_screenshot.png)

#### Visualization of `vg pack` format on MoMI-G

Before this hackathon, MoMI-G could visualize coverage only on a reference path. Using the `vg pack` format, we have enabled MoMI-G to visualize coverage on every node so that we can see read coverages mapped against a graph reference. We have also added an interface to retrieve a subset of the `vg pack` format in this [branch](https://github.com/vgteam/vg/pull/2185). It will be useful for visualization of gGFF format as a binary annotations over base-pairs in nodes.

![MoMIG Genome Graph Visualization software screenshot](fig/momig_final_screenshot.png)

![MoMIG Genome Graph Visualization software screenshot](fig/momig_final_screenshot2.png)

## The gGFF format

We have defined a generalization of the GFF3 format that replaces genomic intervals with a subgraph. It is a text-based, tab-separated file. Every line contains each of the following fields. If a field is to be ignored, it can be replaced with a "." (without quotes). The fields are

* **subgraph**: a comma separated list of intervals of sequences on nodes, along with orientation in the format `ID[start:end](+/-/?)`. The interval is 0-based and exclusive for the end index. If the strand is given as "-", the interval begins at the reverse strand of the final base and extends toward the first base of the node.
* **source**: the name of the program or database that generated the annotation
* **type**: the type of feature
* **score**: a floating point value
* **phase**: 0, 1, or 2 indicating the first base of the feature that is a codon, measuring from the source node in the subgraph
* **attributes**: a semi-colon separated list of tag-value pairs, with tags separated from the values by an "="

Example:

```
156619[22:32]+,156620[0:32]+,156621[0:32]+,156622[0:2]+ havana  exon    .       .       Parent=transcript:ENST00000624081;Name=ENSE00003760288;constitutive=1;ensembl_end_phase=1;ensembl_phase=0;exon_id=ENSE00003760288;rank=1;version=1
156619[22:32]+,156620[0:32]+,156621[0:32]+,156622[0:2]+ havana  CDS     .       0       ID=CDS:ENSP00000485664;Parent=transcript:ENST00000624081;protein_id=ENSP00000485664
156643[3:32]+,156644[0:32]+,156645[0:32]+,156646[0:32]+,156647[0:15]+   havana  exon    .       .       Parent=transcript:ENST00000624081;Name=ENSE00003758404;constitutive=1;ensembl_end_phase=0;ensembl_phase=1;exon_id=ENSE00003758404;rank=2;version=1
```

#### Converting to gGFF from GFF3

We have built this capacity into the `vg` toolkit. The following invocation will inject the annotations from a GFF3 file into a graph:

```
vg annotate -x graph.xg -s graph.snarls -f annotations.gff3 -g > annotations.ggff
```

The snarls can be computed from a `.vg` file with the `vg snarls` subcommand, and the XG index can be created from a `.vg` file using the `vg index` subcommand.

#### Operating on gGFF files

See the subproject-specific [README](ggfftools/README.md).

## Team Members

* Travis Wrightsman (tw493@cornell.edu)
* Jordan Eizenga (jeizenga@ucsc.edu)
* Rajeeva Musunuri (rmusunuri@nygenome.org)
* Toshiyuki Yokoyama (toshiyuki.t.yokoyama@gmail.com)

## Future Directions

* local alignment of nodes could help resolve annotations that span outside of snarls
  * both distance or best alignment methods could be used to traverse the graph and liftover annotation from the reference path to other paths
