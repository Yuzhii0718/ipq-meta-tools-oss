# CDT and GPT generate

CDT is Qualcomm Configuration Data Table.

GPT is GUID Partition Table.

```bash
python2.7 prepareSingleImage.py --arch ipq6018 --fltype emmc --gencdt --genpart
```

The 1G SDRAM cdt binary will be:

meta-tools/ipq6018/in/cdt-AP-CP03-C2_Arthur_512M16(1G)_DDR3.bin

The original FSEIASLD-64G-J02 eMMC gpt binary will be:

meta-tools/ipq6018/in/gpt_main0.bin
