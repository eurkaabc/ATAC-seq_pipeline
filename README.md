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


#####
SCRIPT=/mnt/sda/Public/Project/collabration/AoLab/20251208ATAC/analysis/0.script/02_callpeak.sh

ANALYSIS="/mnt/sda/Public/Project/collabration/AoLab/20251208ATAC/analysis"

[ -f "$SCRIPT" ] || { echo "ERROR: SCRIPT not found: $SCRIPT" >&2; exit 1; }
[ -d "$ANALYSIS" ] || { echo "ERROR: ANALYSIS dir not found: $ANALYSIS" >&2; exit 1; }


if [ -d "$ANALYSIS/3.alignment" ]; then
  BAMDIR="$ANALYSIS/3.alignment"
else
  # 兜底：全盘搜一个 final.bam 来推断目录（避免你在不同机器/路径变动）
  BAMDIR="$(find "$ANALYSIS" -maxdepth 5 -type f -name '*.final.bam' -print -quit | xargs -r dirname)"
fi

[ -n "${BAMDIR:-}" ] && [ -d "$BAMDIR" ] || { echo "ERROR: cannot locate BAMDIR under $ANALYSIS" >&2; exit 1; }
echo "[INFO] Using BAMDIR=$BAMDIR"

for rep in 3 4 5; do
  for grp in KO NTC; do
    sample="GZ25064836-STARD7-${rep}-${grp}"
    echo "==== Callpeak: $sample ===="
    bash "$SCRIPT" "$sample" "$BAMDIR"
  done
done

ANALYSIS="/mnt/sda/Public/Project/collabration/AoLab/20251208ATAC/analysis"
BAMDIR="$ANALYSIS/3.alignment"
PEAKBASE="$ANALYSIS/4.callpeak"
OUTDIR="$ANALYSIS/5.chipqc"
CSV="$OUTDIR/chipqc_input.csv"

mkdir -p "$OUTDIR"
[ -d "$BAMDIR" ] || { echo "ERROR: BAMDIR not found: $BAMDIR" >&2; exit 1; }
[ -d "$PEAKBASE" ] || { echo "ERROR: PEAKBASE not found: $PEAKBASE" >&2; exit 1; }

echo "SampleID,Tissue,Factor,Replicate,bamReads,Peaks,PeakCaller" > "$CSV"

for bam in "$BAMDIR"/*.final.bam; do
  [ -e "$bam" ] || continue
  sample="$(basename "$bam" .final.bam)"
  peaks="$PEAKBASE/$sample/${sample}_peaks.narrowPeak"

  # Replicate：尽量从样本名里提取 “-3- / -4- / -5-” 这种
  rep="$(echo "$sample" | sed -n 's/.*-\([0-9]\+\)-.*/\1/p')"
  [ -n "$rep" ] || rep="1"

  tissue="ATAC"
  factor="ATAC"

  if [ ! -s "$peaks" ]; then
    echo "[WARN] Peaks not found for $sample: $peaks (FRiP 之类指标可能算不了)" >&2
  fi

  echo "${sample},${tissue},${factor},${rep},${bam},${peaks},macs2" >> "$CSV"
done

echo "[OK] Wrote $CSV"



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

#!/usr/bin/env bash
set -euo pipefail

# ========= 配置区 =========
SCRIPT="/mnt/sda/Public/Project/collabration/AoLab/20251210ATAC/analysis/0.script/02_call_peak.sh"  # 修改为目标脚本路径
ANALYSIS="/mnt/sda/Public/Project/collabration/AoLab/20251210ATAC/analysis"  # 新样本路径

# 检查脚本路径是否存在
[ -f "$SCRIPT" ] || { echo "ERROR: SCRIPT not found: $SCRIPT" >&2; exit 1; }
[ -d "$ANALYSIS" ] || { echo "ERROR: ANALYSIS dir not found: $ANALYSIS" >&2; exit 1; }

# 判断 BAM 文件所在目录
BAMDIR="$ANALYSIS/3.alignment"
[ -d "$BAMDIR" ] || { echo "ERROR: Cannot locate BAMDIR under $ANALYSIS" >&2; exit 1; }
echo "[INFO] Using BAMDIR=$BAMDIR"

# 循环进行每个样本的 peak calling
for rep in 1 2 3; do
  for grp in KO WT; do
    sample="GZ25064838-Tango6-${grp}-${rep}"  # 样本名格式: GZ25064838-Tango6-KO-1, GZ25064838-Tango6-WT-1 等
    echo "==== Callpeak: $sample ===="
    bash "$SCRIPT" "$sample" "$BAMDIR"
  done
done
