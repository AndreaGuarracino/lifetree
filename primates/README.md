# wfmash 13 primate assemblies

## Mapping

The `wfmash-v0.12.0_map-p70.sh` script is used to generate mappings between all genomes and each target genome.
By running this for all genomes, we can evoke an all-to-all homology mapping and base-level alignment in parallel.

Mapping is accomplished by the `mashmap3` algorithm as parameterized in `wfmash -m`.
It's achieved by splitting each query genome into overlapping 5kb segments and mapping each segment to the chosen target.
Mappings are filtered to keep only those with >70% identity and a cumulative length >20kb. 

The script runs `wfmash` with these key parameters:

- `-m` - generate mappings (instead of alignments)  
- `-p 70` - minimum 70% identity
- `-s 5k` - segment size 5kb
- `-c 20k` - chains with cumulative 20kb mapping

This results in PAF format mapping files for each target sequence in the genome.

## Alignment

The `wfmash-v0.12.0_aln-p70.sh` script can then be used to convert the mappings to alignments.

It takes the PAF mapping files generated in the previous step as input. 

It runs `wfmash` again for each target, this time in alignment mode, using the mappings to guide the alignment process.

This results in aligned PAF files for each target sequence which include `--eqx` compatible cigars usable in rustybam, seqwish, and other tools.

## Visualization

The `plot.R` script is then used to visualize the mappings. In this example we use the homology mappings only, for simplicity.

It loads the mapping PAF file and bins the mappings into windows along each target chromosome using `bedtools intersect`.

Then for each target chromosome, it generates a heatmap showing which query sequences align to that target, filtering for queries above a minimum length.

The heatmaps are colored by query sequence identity and scaled based on the number of unique query sequences. 

This results in PDF and PNG heatmap images for each target chromosome showing the mapping coverage and origins.

## Usage

1. Generate mappings with `wfmash-v0.12.0_map-p70.sh` or alignments with `wfmash-v0.12.0_aln-p70.sh`.

2. Bin and process mappings into windows with `overlaps.sh`

3. Visualize each chromosome with `plot.R` 

For example:

```
# Generate mappings with an array job
sbatch -a 1-13 wfmash-v0.12.0_map-p70.sh 

# Bin mappings 
./overlaps.sh mappings.paf genome.fa.gz prefix 100000

# Visualize
./plot.R binned_mappings.bed.tsv 1000000
```

This will generate heatmap images for each chromosome showing the mapping results.

## Data

* Mappings only (data shown here) https://garrisonlab.s3.amazonaws.com/t2t-primates/primates13-v0.1-wfmash-01f812e5-map.tar.gz 
* Alignments plus mappings (full cigars for all homologies shown here) https://garrisonlab.s3.amazonaws.com/t2t-primates/primates13-v0.1-wfmash-01f812e5.tar.gz
* Assemblies with names formatted as shown here: https://garrisonlab.s3.amazonaws.com/t2t-primates/primates13.fa.gz 
