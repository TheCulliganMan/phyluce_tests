# phyluce_tests
## Quality Control
```bash
for i in UCEs/*_R*.fastq.gz;
do
  echo $i; gunzip -c $i | wc -l;
done > line_counts.txt
```
Run illumiprocessor
```bash
for CONF in *config
  do
    illumiprocessor --input $(pwd)/UCEs --output uce-clean_$CONF --cores 56 --config $CONF --trimmomatic /home/genetics/ryan/zaglossus/Trimmomatic-0.36/trimmomatic-0.36.jar
  done
```
## Assembly
Build config file
```bash
export CONF=assembly.conf
rm -r uce-clean
mkdir uce-clean
for I in uce-clean_*
  do
    mv $I/* uce-clean
  done

echo "[samples]" > $CONF
for I in $(ls uce-clean);
  do
    echo $I:$(pwd)/uce-clean/$I;
  done >> $CONF

```
Run Velvet
```bash
mkdir -p log assemblies assemblies/velvet assemblies/abyss assemblies/trinity
phyluce_assembly_assemblo_velvet \
    --config $CONF \
    --output $(pwd)/assemblies/velvet \
    --kmer 35 \
    --subfolder split-adapter-quality-trimmed \
    --cores 12 \
    --clean \
    --log-path log
```
Run Abyss
```bash
phyluce_assembly_assemblo_abyss \
    --config $CONF \
    --output assemblies/abyss \
    --kmer 35 \
    --subfolder split-adapter-quality-trimmed \
    --cores 12 \
    --clean \
    --log-path log
```
Run Trinity: we will use this one.
```bash
phyluce_assembly_assemblo_trinity \
  --config $CONF \
  --output assemblies/trinity0 \
  --subfolder split-adapter-quality-trimmed \
  --clean \
  --cores 56 \
  --log-path log
```
## Identify UCE loci
USE MODIFIED PHYLUCE! RYAN'S GITHUB
need to do a second run on the assemblies/trinity0/contigs folder...
```bash
phyluce_assembly_match_contigs_to_probes \
    --contigs /home/genetics/ryan/zaglossus/assemblies/trinity-finals/contigs \
    --probes uce-tetrapods-roche-probes.fasta \
    --output /home/genetics/ryan/zaglossus/matches \
    --log-path log
```
