# imi-csv-diy

IMI 导针 CSV 到 xlsx 拆分工具。将一对 IMI 导出的 CSV 文件按人工判断的 deposit ID 区间，拆分为 3 个标准格式的 Excel 工作簿。

## 背景

在 IMI 手术规划工作流中，IMI 软件会导出两个 CSV：

- `{case}_trajs.csv` — 多条轨迹的 RAS 坐标（e1, e2, ... / t1, t2, ...）
- `{case}_imi.csv` — 45 个预定义 deposit 的参数（depth / angle / exit）

实际手术中不同导针负责不同区域，需要由医生在影像上判断每个导针覆盖哪些 deposit，然后将数据拆分到独立文件中供后续计算使用。

## 功能

将一对 CSV 自动拆分为 3 个 `.xlsx` 文件，每个文件包含：

| Sheet | 内容 |
|-------|------|
| `traj` | 6 行 RAS 坐标（entry + tip），3 列，无表头 |
| `deposits` | deposit 参数表，含 depth / angle / exit 数据行和 R / A / S 占位行 |

## 文件结构

```
imi-csv-diy/
|-- SKILL.md                     # 技能定义（供 WorkBuddy 加载）
|-- README.md                    # 本文件
|-- scripts/
|   |-- split_csv_to_xlsx.py     # 核心拆分脚本
```

## 安装

```bash
pip install openpyxl
```

## 用法

### 命令行

```bash
python scripts/split_csv_to_xlsx.py <case_name> <r1_start>,<r1_end> <r2_start>,<r2_end> <r3_start>,<r3_end>
```

参数说明：

| 参数 | 含义 | 示例 |
|------|------|------|
| `case_name` | 病例编号，对应 CSV 文件名前缀 | `225CKH` |
| `r1_start,r1_end` | traj1 的 deposit ID 区间（闭区间） | `1,15` |
| `r2_start,r2_end` | traj2 的 deposit ID 区间 | `16,30` |
| `r3_start,r3_end` | traj3 的 deposit ID 区间 | `31,45` |

### 示例

```bash
# 225CKH 病例：1-15 归 traj1，16-30 归 traj2，31-45 归 traj3
python scripts/split_csv_to_xlsx.py 225CKH 1,15 16,30 31,45
```

输出：
```
225CKH_traj1.xlsx: traj1, deposits 1-15 (15个)
225CKH_traj2.xlsx: traj2, deposits 16-30 (15个)
225CKH_traj3.xlsx: traj3, deposits 31-45 (15个)
完成
```

### 通过 WorkBuddy 对话

直接在对话中告诉 WorkBuddy 拆分区间：

> "用 imi-csv-diy 拆分 225CKH，1-15 为 traj1，16-30 为 traj2，31-45 为 traj3"

## 输入 CSV 格式

### {case}_trajs.csv

每 6 行一条轨迹，前 3 行为 entry 坐标，后 3 行为 tip 坐标：

```csv
e2,r1,32.79
e2,a1,-11.27
e2,s1,63.09
t2,r2,19.07
t2,a2,-11.64
t2,s2,10.87
e3,r3,32.79
...
```

### {case}_imi.csv

共 3 行数据 + 1 行表头（45 列，对应 45 个 deposit）：

```csv
,1,2,3,...,45
depth,-5,-5,-5,...,-25
angle,165,240,330,...,270
exit,8,8,8,...,8
```

## 输出 xlsx 格式

### traj sheet

```
e2    r1    32.79
e2    a1   -11.27
e2    s1    63.09
t2    r2    19.07
t2    a2   -11.64
t2    s2    10.87
```

### deposits sheet

```
deposit#   1     2     3    ...   N
depth     -5    -5    -5    ...  -30
angle     165   240   330   ...  180
exit       8     8     8    ...    8
R
A
S
```

deposit 编号在每个文件中从 1 重新开始，R / A / S 行为后续 `compute_deposits.py` 计算回填预留。

## 历史案例参考

| 病例 | traj1 ID 区间 | traj2 ID 区间 | traj3 ID 区间 |
|------|-------------|-------------|-------------|
| 231LYB | 1-15 | 16-29 | 30-45 |
| 225CKH | 1-15 | 16-30 | 31-45 |
