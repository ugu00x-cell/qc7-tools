# QC7つ道具 Python ツールセット

製造業の品質管理で使われる「QC7つ道具」をPythonで実装したツールセットです。
サンプルデータ付きですぐに動かせます。

## ツール一覧

| # | ファイル | ツール | 概要 |
|---|---------|--------|------|
| 1 | `pareto_chart.py` | パレート図 | 不良原因の件数を棒グラフ＋累積折れ線で表示。上位80%の原因を自動特定 |
| 2 | `histogram_chart.py` | ヒストグラム | 寸法データの分布を表示。規格線（USL/LSL）とCpkを自動計算 |
| 3 | `scatter_chart.py` | 散布図 | 2変数の関係を可視化。回帰直線と相関係数を自動計算 |
| 4 | `control_chart.py` | 管理図（Xbar-R） | 平均値と範囲の管理図を上下2段で表示。管理限界超えを赤丸でハイライト |
| 5 | `fishbone_chart.py` | 特性要因図 | 魚の骨図を描画。人・機械・材料・方法・環境・測定の6カテゴリ対応 |
| 6 | `stratification_chart.py` | 層別グラフ | 機械別・作業者別の分布を箱ひげ図で比較。異常グループを自動検出 |
| 7 | `checksheet_chart.py` | チェックシート | 不良種類×曜日のマトリクスをヒートマップで表示。最多不良・最多曜日を判定 |
| - | `qc7_dashboard.py` | 統合ダッシュボード | 7つ全てを1画面に統合表示。DXコンサル提案デモ用 |

## 必要な環境

- Python 3.10以上
- 以下のライブラリ

```bash
pip install matplotlib numpy pandas scipy seaborn
```

## 使い方

### サンプルデータで実行

各ツールはそのまま実行できます。

```bash
python pareto_chart.py
python histogram_chart.py
python scatter_chart.py
python control_chart.py
python fishbone_chart.py
python stratification_chart.py
python checksheet_chart.py
```

### 統合ダッシュボード

7つ全てを1画面で表示します。

```bash
python qc7_dashboard.py
```

### 自分のデータで使う

各ツールは関数としてインポートできます。

```python
# パレート図
from pareto_chart import create_pareto_chart

create_pareto_chart({"寸法不良": 45, "傷": 30, "バリ": 10})
```

```python
# ヒストグラム＋Cpk
from histogram_chart import create_histogram
import numpy as np

data = np.loadtxt("measurement.csv")
create_histogram(data, usl=51.0, lsl=49.0, title="ロットA")
```

```python
# 散布図
from scatter_chart import create_scatter_chart

create_scatter_chart(
    x=temperature, y=dimension_error,
    xlabel="加工温度（℃）", ylabel="寸法誤差（mm）",
)
```

```python
# 管理図（データ形状: サブグループ数 × サンプルサイズ）
from control_chart import create_control_chart
import numpy as np

data = np.loadtxt("subgroups.csv", delimiter=",")  # shape=(25, 4)
create_control_chart(data)
```

```python
# 特性要因図
from fishbone_chart import create_fishbone_chart

create_fishbone_chart(
    effect="バリ発生",
    causes={
        "人": ["送り速度ミス", "確認不足"],
        "機械": ["刃先摩耗", "クランプ不良"],
        "材料": ["硬度バラツキ"],
        "方法": ["条件表なし"],
        "環境": ["切削液劣化"],
        "測定": ["目視判定のバラツキ"],
    },
)
```

```python
# 層別グラフ
from stratification_chart import create_stratification_chart

create_stratification_chart({
    "機械A": data_a,
    "機械B": data_b,
    "機械C": data_c,
})
```

```python
# チェックシート
from checksheet_chart import create_checksheet
import numpy as np

data = np.array([[8, 6, 7], [4, 5, 3]])
create_checksheet(data, ["寸法不良", "傷"], ["月", "火", "水"])
```

## 各ツールの詳細

### 1. パレート図（`pareto_chart.py`）

不良原因を件数の多い順に並べ、累積比率を折れ線で重ねて表示します。
80%ラインを赤い点線で表示し、重点対策すべき原因を自動特定します。

**出力例：**
```
上位80%を占める原因：寸法不良、傷、バリ
```

### 2. ヒストグラム（`histogram_chart.py`）

測定データの分布をヒストグラムで表示します。
規格上限（USL）・規格下限（LSL）を赤い点線、平均値を青い点線で表示。
Cpk（工程能力指数）を自動計算し、判定結果をグラフ上に表示します。

**判定基準：**
| Cpk | 判定 |
|-----|------|
| 1.33以上 | 工程は安定 |
| 1.00〜1.33 | ギリギリ。要注意 |
| 1.00未満 | 能力不足。要改善 |

### 3. 散布図（`scatter_chart.py`）

2つの変数の関係を散布図で表示し、回帰直線を重ねます。
相関係数を自動計算し、相関の強さを判定します。

**判定基準：**
| |r| | 判定 |
|-----|------|
| 0.7以上 | 強い相関 |
| 0.4〜0.7 | 弱い相関 |
| 0.4未満 | 相関なし |

### 4. 管理図（`control_chart.py`）

Xbar-R管理図を上下2段で表示します。
管理限界線（UCL/LCL）を自動計算し、限界を超えた点を赤丸でハイライトします。
サンプルサイズ n=2〜10 に対応（A2, D3, D4 係数テーブル内蔵）。

### 5. 特性要因図（`fishbone_chart.py`）

魚の骨図（フィッシュボーンダイアグラム）を描画します。
大骨6本（人・機械・材料・方法・環境・測定）にカテゴリ別の色分けで小骨を配置します。

### 6. 層別グラフ（`stratification_chart.py`）

機械別・作業者別などグループごとの分布を箱ひげ図で比較します。
全体平均から2σ以上離れたグループを異常として自動検出します。

### 7. チェックシート（`checksheet_chart.py`）

不良種類×曜日のマトリクスをヒートマップで可視化します。
行・列の合計を自動計算し、最多不良種類・最多曜日を判定します。

### 統合ダッシュボード（`qc7_dashboard.py`）

7つ全てを1画面に配置した統合ダッシュボードです。
画面下部に分析サマリー（重点対策対象・Cpk・異常検出など）を表示します。

## ライセンス

MIT License
