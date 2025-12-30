# ATAC-seq_pipeline

#一个样品
RAW=/mnt/sda/Public/Project/collabration/AoLab/20251208ATAC/rawdata
SCRIPT=/mnt/sda/Public/Project/collabration/AoLab/20251208ATAC/analysis/01_qc_map_mm10.sh

sh ${SCRIPT} \
  ${RAW}/Sample_GZ25064836-STARD7-3-KO \
  STARD7_3_KO \
  GZ25064836-STARD7-3-KO_combined



#多个样品
RAW=/mnt/sda/Public/Project/collabration/AoLab/20251208ATAC/rawdata
SCRIPT=/mnt/sda/Public/Project/collabration/AoLab/20251208ATAC/analysis/01_qc_map_mm10.sh

for d in ${RAW}/Sample_GZ25064836-*; do
    fastq_dir=${d}

    # 找到 *_combined_R1.fastq* 文件，去掉 _R1.fastq.gz 得到前缀
    prefix=$(basename $(ls ${fastq_dir}/*_combined_R1.fastq* | head -n1) | sed 's/_R1\.fastq.*//')

    # 输出前缀用目录名去掉 Sample_
    sample_name=$(basename ${d} | sed 's/^Sample_//')

    echo "Running sample: ${sample_name}"
    echo "  fastq_dir:   ${fastq_dir}"
    echo "  prefix:      ${prefix}"

    sh ${SCRIPT} ${fastq_dir} ${sample_name} ${prefix}
done

#tango6
RAW=/mnt/sda/Public/Project/collabration/AoLab/20251210ATAC/rawdata
SCRIPT=/mnt/sda/Public/Project/collabration/AoLab/20251210ATAC/analysis/01_qc_map_mm10.sh

for d in ${RAW}/Sample_GZ25064838-*; do
    fastq_dir=${d}

    # 找到 *_combined_R1.fastq* 文件，去掉 _R1.fastq.gz 得到前缀
    prefix=$(basename $(ls ${fastq_dir}/*_combined_R1.fastq* | head -n1) | sed 's/_R1\.fastq.*//')

    # 输出前缀用目录名去掉 Sample_
    sample_name=$(basename ${d} | sed 's/^Sample_//')

    echo "Running sample: ${sample_name}"
    echo "  fastq_dir:   ${fastq_dir}"
    echo "  prefix:      ${prefix}"

    sh ${SCRIPT} ${fastq_dir} ${sample_name} ${prefix}
done

