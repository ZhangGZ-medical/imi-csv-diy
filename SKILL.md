---
name: imi-csv-diy
description: >
  IMI 导针 CSV→xlsx 拆分技能。当用户提供了 {case}_trajs.csv 和 {case}_imi.csv 两个文件，
  并要求按指定 IMI deposit ID 区间拆分为多个 xlsx 文件时使用。
  触发词：CSV拆分、IMI拆分、trajs拆分、imi_csv、csv转xlsx、拆分到traj、deposit拆分。
agent_created: true
---

# IMI CSV→xlsx 拆分

将一对 CSV 文件（`{case}_trajs.csv` + `{case}_imi.csv`）按人工指定的 deposit ID 区间拆分为多个 xlsx 文件。

## 输入/输出

- **输入**：`{case}_trajs.csv`（轨迹 RAS 坐标）和 `{case}_imi.csv`（45 个 deposit 的 depth/angle/exit 参数）
- **人工输入**：3 个 ID 区间（如 1-15→traj1, 16-30→traj2, 31-45→traj3）
- **输出**：`{case}_traj1.xlsx`, `{case}_traj2.xlsx`, `{case}_traj3.xlsx`

## 工作流程

### 第一步：确认拆分区间

向用户确认 3 个 IMI deposit ID 区间如何分配。默认约定是 3 个 traj，但区间由人工根据图像判断。

如果用户已明确给出区间（如"1-15为traj1, 16-30为traj2, 31-45为traj3"），直接使用。

### 第二步：运行拆分脚本

使用 `scripts/split_csv_to_xlsx.py`：

```bash
python scripts/split_csv_to_xlsx.py <case_name> <r1_start>,<r1_end> <r2_start>,<r2_end> <r3_start>,<r3_end>
```

示例：
```bash
python scripts/split_csv_to_xlsx.py 225CKH 1,15 16,30 31,45
```

脚本自动完成：
- 读取 CSV，将 `trajs.csv` 的 3 条轨迹按顺序分配给 traj1/2/3
- 将 `imi.csv` 的 deposit 参数按指定区间切片
- 生成 3 个 xlsx，每个包含 `traj` 和 `deposits` 两个 sheet

### 第三步：验证生成结果

读取生成的 3 个 xlsx 的 sheet 名称和行列结构，确认与预期一致。

如果需要用 openpyxl 读取验证但环境未安装，先确保安装：
```bash
pip install openpyxl
```

## 输出格式说明

每个 xlsx 包含两个 sheet：

### traj sheet
- 6 行 × 3 列（无表头）
- Col A：条目标签（e# / t#）
- Col B：坐标标签（r# / a# / s#）
- Col C：数值
- 示例：e2, r1, 32.79 → entry e2 的 R 坐标

### deposits sheet
- Row 1：`deposit#` | 1 | 2 | ... | N（在每个文件内从 1 重新编号）
- Row 2：`depth` | values...
- Row 3：`angle` | values...
- Row 4：`exit` | values...
- Row 5-7：`R` / `A` / `S` 标签（空值占位，供后续 compute_deposits.py 回填）

## 依赖

- Python 3.x + openpyxl
