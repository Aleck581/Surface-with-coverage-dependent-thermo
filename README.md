# 鉑表面的覆蓋率依賴熱力學

[Surface with coverage-dependent thermo.ipynb](Surface%20with%20coverage-dependent%20thermo.ipynb) 示範如何用 Cantera 計算鉑（Pt）表面上吸附態 CO（`CO*`）的標準態焓與熵。Notebook 在 **300 K、1 atm** 掃描表面覆蓋率，比較四種模型與密度泛函理論（DFT）資料，並以等高線圖展示 CO* 與吸附態氧（`O*`）的交互作用。[Cantera 官方範例](https://www.cantera.org/3.1/examples/python/thermo/coverage_dependent_surf.html)

## 執行方式

使用 Python 3.12、Cantera 3.1 以上、NumPy、Matplotlib 和 JupyterLab。可用 Conda 建立環境。若已有裝好這些套件的 `cantera-env`，可直接啟用。

```bash
conda create -n cantera-env -c conda-forge python=3.12 'cantera>=3.1' numpy matplotlib jupyterlab
conda activate cantera-env
jupyter lab "Surface with coverage-dependent thermo.ipynb"
```

請在專案目錄執行上述指令，開啟後選擇這個環境的 Python kernel，並執行 **Run → Run All Cells**。模型參數由 Cantera 的範例資料檔 [`example_data/covdepsurf.yaml`](https://testing.cantera.org/stable/examples/input/covdepsurf.html) 讀入。若出現找不到 YAML 的錯誤，可檢查 `ct.get_data_directories()` 列出的路徑，或將官方範例 YAML 存為專案內的 `example_data/covdepsurf.yaml`。

## 基本方程式

覆蓋率 $\theta_j$ 是物種 $j$ 佔據的表面位點比例。以 CO* 和 O* 共存的模型為例，三種位點狀態滿足 $\theta_{\mathrm{vac}} + \theta_{\mathrm{CO}^*} + \theta_{\mathrm{O}^*} = 1$，其中 $\theta_{\mathrm{vac}}$ 是空位比例。

Cantera 以低覆蓋率的熱力學性質為基準，再把各物種造成的側向交互作用加上去。用 $f$ 代表焓 $h$ 或熵 $s$，對表面物種 $i$ 可寫成

$$
f_i^\circ(T,\boldsymbol{\theta})
= f_{i,\mathrm{base}}^\circ(T) + \sum_j \Delta f_{ij}(T,\theta_j).
$$

$j=i$ 是自身交互作用（例如 CO*–CO*），$j\ne i$ 是交叉交互作用（例如 CO*–O*）。基準熱力學性質來自 YAML 中的 NASA7 多項式，熵的基準也包含參考覆蓋率的常數項。若設定了覆蓋率相關的熱容 $\Delta c_p$，Cantera 會分別將 $\Delta c_p$ 和 $\Delta c_p/T$ 從 298.15 K 積分到目標溫度，加到焓與熵的修正中。[Cantera 模型方程式](https://www.cantera.org/3.0/doxygen/html/db/d25/classCantera_1_1CoverageDependentSurfPhase.html) · [Cantera 實作](https://raw.githubusercontent.com/Cantera/cantera/v3.1.0/src/thermo/CoverageDependentSurfPhase.cpp)

| YAML 相名稱 | 覆蓋率修正模型 |
| --- | --- |
| `covdep_lin` | 線性。修正量與覆蓋率成正比，例如 $a\theta$。 |
| `covdep_pwlin` | 分段線性。在指定覆蓋率切換斜率，兩段在轉折點連續。 |
| `covdep_poly` | 多項式。Cantera 支援最高四次，本例使用到三次。 |
| `covdep_int` | 插值。依 YAML 表格，在相鄰覆蓋率資料點之間做線性插值。 |

等高線圖使用 `covdep_cross`。其中 CO* 標準態焓的線性修正項為

$$
\Delta h_{\mathrm{CO}^*,\mathrm{lin}}
= 0.845\,\theta_{\mathrm{CO}^*} + 0.412\,\theta_{\mathrm{O}^*}
\qquad [\mathrm{eV\ per\ molecule}].
$$

第一項的係數以 DFT 資料擬合，第二項的係數是為了示範交叉交互作用而設定。[範例 YAML 與參數來源](https://testing.cantera.org/stable/examples/input/covdepsurf.html)

## 如何讀圖

前兩張圖的橫軸是 CO* 覆蓋率，縱軸分別是 $h^\circ/(RT)$ 和 $s^\circ/R$，其中 $R$ 是氣體常數。黑點是用於建立模型的 DFT 資料，曲線顯示四種模型在各覆蓋率下的結果。

第三張圖的有效區域滿足 $\theta_{\mathrm{CO}^*}+\theta_{\mathrm{O}^*}\le 1$。目前 notebook 把 CO* 覆蓋率存成矩陣列索引，而 `contourf` 把列索引畫在縱軸，因此圖上的兩個軸標籤與資料對調。若重新繪圖並保留原標籤，可傳入 `CO_hrts.T`，並遮罩兩種覆蓋率總和超過 1 的格點。

## 參考資料

- J. M. Thomas 與 W. J. Thomas，*Principles and Practice of Heterogeneous Catalysis*，第 2 版，第 2 章，尤其是 §2.8〈Energetics of Adsorption〉。適合先讀吸附能與覆蓋率的物理背景。[Wiley-VCH](https://www.wiley-vch.de/en/areas-interest/natural-sciences/chemistry-11ch/organic-chemistry-11ch8/principles-and-practice-of-heterogeneous-catalysis-978-3-527-31458-4)
- R. J. Kee、M. E. Coltrin、P. Glarborg 與 H. Zhu，*Chemically Reacting Flow: Theory, Modeling, and Simulation*，第 2 版，第 16 章〈Heterogeneous Chemistry〉。適合了解表面熱力學與反應速率如何銜接。[Wiley](https://onlinelibrary.wiley.com/doi/book/10.1002/9781119186304)
- J. Bae、B. Kreitz、A. A. Peterson 與 C. F. Goldsmith（2025），〈Influence of Coverage Dependence on the Thermophysical Properties of Adsorbates and Its Impact on Microkinetic Models〉。說明覆蓋率修正的參數化和微觀動力學應用。[原始論文](https://doi.org/10.1021/acs.jcim.4c02167)
