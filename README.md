# Scalable and Effective Arithmetic Tree Generation for Adder and Multiplier Designs

This repository contains the code for our NeurIPS 2024 **Spotlight** paper:
[*Scalable and Effective Arithmetic Tree Generation for Adder and Multiplier Designs*](https://proceedings.neurips.cc/paper_files/paper/2024/file/fb23cf87a9e04d7677b73c47acd060ef-Paper-Conference.pdf).

We propose an RL-based method for designing arithmetic hardware modules, including **adders** and **multipliers**.

An introduction video is available on Bilibili:
[*Crash Course: How to Design Computer Arithmetic Units: Adders and Multipliers*](https://www.bilibili.com/video/BV1fxxSebEgw/?vd_source=dca394f8d1037df4a246c2e079e79f99).



## Environment Setup (Updated in 2026)

This repo was developed with older versions of PyTorch and Python, so we recommend the following environment:

```bash
git clone https://github.com/laiyao1/ArithTreeRL.git
conda create -n arithtreerl python=3.9 -y
conda activate arithtreerl
conda install pytorch=1.10.1 torchvision torchaudio cudatoolkit=11.3 -c pytorch -y
conda install dgl-cuda11.3=0.7.2 -c dglteam -y
```



## Usage

### Adder design (theoretical metrics)

```bash
python adder.py --input_bit=64  --level_bound_delta=0
python adder.py --input_bit=128 --level_bound_delta=0
python adder.py --input_bit=128 --level_bound_delta=0 --demo
```

### Adder design (practical metrics)

> Note: Please copy `utils/fast_flow.tcl` and `utils/full_flow.tcl` to `OpenROAD/test` before running.

```bash
python adder_prac.py --input_bit=32 --adder_type=0 --openroad_path=/path/to/openroad
python adder_prac.py --input_bit=32 --adder_type=1 --openroad_path=/path/to/openroad
python adder_prac.py --input_bit=32 --adder_type=2 --openroad_path=/path/to/openroad
python select_adder.py --input_bit=32 --openroad_path=/path/to/openroad
```

### Multiplier design

```bash
python mult.py --input_bit=32  --area_w=0.01
python mult.py --input_bit=64  --area_w=0.01
python mult.py --input_bit=128 --area_w=0.01
```



## Expected Outputs

### Adder design (theoretical metrics)

Designed adders are represented by a binary matrix `A (N x N)` and dumped to:

- `cell_map/adder_{N}b/adder_{N}b_{L}l_{S}s_{ID}.log`

where:
- `N` = adder bit-width  
- `L` = logic level  
- `S` = size  
- `ID` = adder identifier

Timing reports are saved to:

- `time_log_{N}b_{L}_{SEED}.log`

### Adder design (practical metrics)

Designed adders are saved as:
- Verilog HDL files (`*.v`) and/or
- cell-map matrices (`*.log`)

By default, the script **only saves the cell-map logs** (to reduce output size).  
To also save Verilog HDL files, enable `--save_verilog`.

Logs are written to:

- `adder_prac_log/adder_{N}b/adder_{N}b_openroad_type{TID}_{TIME}.log`

where:
- `{TID}` = initial adder type (see table below)
- `{TIME}` = program start timestamp

| Adder Type ID | Adder |
|||
| 0 | Ripple-carry adder |
| 1 | Sklansky adder |
| 2 | Brent–Kung adder |

### Multiplier design

Best multipliers are logged in:

- `back_and_forth/`

Generated Verilog HDL files are saved in:

- `run_verilog_mult_mid/`
- `run_verilog_mult_add_mid/`

### Runtime notes

- For adder design with theoretical metrics, strong results can typically be found within **~3 hours** on our experimental hardware.
- For other tasks, the estimated runtime is reported in **Table A3** in the paper.



## Demo Outputs

### 1) Adder design (theoretical metrics)

```bash
python adder.py --input_bit=128 --level_bound_delta=0
```

Example adder output (cell map excerpt):

```txt
1 0 0 0 0 0 ...
1 1 0 0 0 0 ...
1 0 1 0 0 0 ...
1 0 1 1 0 0 ...
1 0 0 0 1 0 ...
1 0 0 0 1 1 ...
...
1 0 0 0 0 0 ...
```

Example timing log (format: level, size, clock time, search step):

| Level | Size | Clock Time | Search Step |
|||||

```txt
...
8.0 369.0   1.36    83
8.0 368.0   6.94    332
8.0 367.0   7.20    333
8.0 366.0   87.06   3810
8.0 365.0   148.33  6353
8.0 364.0   1669.44 71038
```

Results may vary depending on hardware and random seed. Another example:

```txt
...
8.0 369.0   31.90   565
8.0 368.0   40.90   726
8.0 367.0   41.41   727
8.0 366.0   997.32  16771
8.0 365.0   1774.52 29994
8.0 364.0   6128.24 105726
```

When `level_bound_delta=1`:

```txt
...
9.0 279.0   1.96    91
9.0 278.0   1.98    92
9.0 277.0   23.35   1008
9.0 276.0   23.51   1010
9.0 275.0   23.68   1011
9.0 274.0   1119.71 61278
9.0 273.0   1119.88 61279
```

(Logs can be large; `...` indicates omitted contents.)

### 2) Adder design (practical metrics)

```bash
python adder_prac.py --input_bit=32 --adder_type=0 --openroad_path=/path/to/openroad
```

Example Verilog HDL output:

```verilog
module main(a,b,s,cout);
input [31:0] a,b;
output [31:0] s;
output cout;
wire g11_8,g26_16,p8_8,c4,...
assign p0_0 = a[0] ^ b[0];
assign g0_0 = a[0] & b[0];
...
endmodule
```

Example log output format:

| File Name | Delay (ps) | Area (um^2) | * | Level | Size | Fanout | Step | Cache Hit | * | Timing (s) |
||:|:|:|:|:|:|:|:|:|:|

`*` are reserved fields for future use.

```txt
adder_32b_7_79_70199b11a9eeffb33f33691274f08116 450.97 401.13 0.0 7 79 16 1 0 0 1.23
adder_32b_8_82_8b6eaa7a64bf15b50f9b8abbb51d7145 408.76 416.82 0.0 8 82 15 2 0 0 1.86
adder_32b_8_84_583cb9fc4b850f0be7be0df992a414c2 411.60 441.83 0.0 8 84 15 3 0 0 2.50
adder_32b_8_83_b1077cbc7d258fcdb8cd10279b1f7630 412.09 423.21 0.0 8 83 15 4 0 0 3.12
adder_32b_8_84_5053d501e2c45c99f6854e08ed3a4265 405.76 430.12 0.0 8 84 14 5 0 0 3.75
```

When running `select_adder.py`, the selected top-K adders are written to:

- `adder_{N}b_full_openroad_{TIME}.log`

with four additional columns:

| Delay (ps, full flow) | Area (um^2, full flow) | * | Timing (s) |
|:|:|:|:|



## Parameters

Common:
- **input_bit** Bit-width of the adder/multiplier.
- **initial_adder_type** Initial adder type used by MCTS.
- **seed** Random seed.
- **openroad_path** Path to the OpenROAD installation.

For adder design:
- **level_bound_delta** Level upper bound offset (relative to `log2(N) + 1`).
- **max_save** Max number of adders stored per level threshold `L_th` (default: 2).
- **demo** Load / reproduce designs reported in the paper more easily.
- **save_verilog** Save generated adder Verilog files.
- **step** Max search steps.
- **adder_type** Initial adder type for practical-metric runs.
- **k** Top-K adders selected in the second-stage retrieval.

For multiplier design:
- **area_w** Weight for area.
- **max_iter** Max search iterations.
- **template** Compressor tree template file.
- **gamma** Decay factor.
- **lr** Learning rate.
- **batch_size** Batch size.



## Requirements

- [OpenROAD](https://github.com/The-OpenROAD-Project/OpenROAD) >= 2.0
- [Yosys](https://github.com/YosysHQ/yosys) >= 0.27
- [Python](https://www.python.org/) >= 3.9
- [PyTorch](https://pytorch.org/) >= 1.10
- [Gym](https://www.gymlibrary.dev/index.html) >= 0.21.0  
  - Note: Gym has been unmaintained since 2022. If you run into compatibility issues, consider switching to **Gymnasium** (drop-in replacement).
- [tqdm](https://tqdm.github.io/)
- Ubuntu >= 20.04

Other versions may also work but have not been tested.



## Installation

No installation is required for this codebase.

However, to evaluate multipliers you need to install **OpenROAD** and **Yosys** (installation time is typically ~1 hour).

Then copy the following synthesis scripts:

```bash
cp utils/fast_flow.tcl /path/to/OpenROAD/test
cp utils/full_flow.tcl /path/to/OpenROAD/test
```



## Testbench

We provide Verilog testbenches for generated adders and multipliers.

Each module is tested with **100** random addition/multiplication operations to validate correctness.

Testbench code is under `testbench/`, along with example generated modules. To run:

```bash
cd testbench
iverilog -o adder_sim adder.v adder_tb.v
vvp adder_sim
iverilog -o multiplier_sim multiplier.v multiplier_tb.v
vvp multiplier_sim
```



## Citation

If you find our paper or code useful, please cite:

```bibtex
@inproceedings{NEURIPS2024_fb23cf87,
 author = {Lai, Yao and Liu, Jinxin and Pan, David Z. and Luo, Ping},
 booktitle = {Advances in Neural Information Processing Systems},
 doi = {10.52202/079017-4416},
 editor = {A. Globerson and L. Mackey and D. Belgrave and A. Fan and U. Paquet and J. Tomczak and C. Zhang},
 pages = {139151--139178},
 publisher = {Curran Associates, Inc.},
 title = {Scalable and Effective Arithmetic Tree Generation for Adder and Multiplier Designs},
 volume = {37},
 year = {2024}
}
```
