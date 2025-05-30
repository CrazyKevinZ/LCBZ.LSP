# LCBZ.LSP AutoCAD里程桩号标注及对应坐标提取保存全自动插件

## 一、功能实现说明

**LCBZ.LSP** 是用于在 AutoCAD 中自动批量标注线性对象（如多段线、样条线、直线等）上的“里程桩号”及对应短线的 AutoLISP 脚本。其主要功能包括：

### 1. 支持对象类型
- 适用于 AutoCAD 中的多段线（LWPOLYLINE/POLYLINE）、样条曲线（SPLINE）、直线（LINE）等支持 `vlax-curve` 或已内部支持的对象。
- 直线（LINE）对象已集成专用处理逻辑，可与多段线、样条线同样自动标注。

### 2. 交互式参数设置
- **选择对象**：提示用户通过对象选择方式，选取需要标注的线性对象（支持直线）。
- **自定义起点**：要求用户以对象捕捉方式点取起始位置，支持端点、中点等。
- **参数输入**：依次输入或选择
  - 里程前缀（如“K”），可选
  - 起始里程数值（默认0）
  - 标注间距（默认100）
  - 文字高度（默认5）
  - 短线长度（默认25）
  - 小数位数（默认0）
  - 短线对齐方式（支持 Center/Left/Right，默认 Right）
  - 是否标注整数倍里程点（可选）
  - 是否合并等距与整数倍里程点（可选）

### 3. 里程桩号点自动计算
- 按设定的起点、间距、是否合并等逻辑自动计算所有需标注的里程桩号。
- 终点桩号**始终强制标注**，即使不在等距序列内。

### 4. 桩号与短线标注
- 自动沿曲线或直线精确计算每个桩号点的坐标和切线/法线方向。
- 对于直线，采用直线插值算法定位桩号点和方向，与曲线处理一致。
- 根据对齐方式（Center/Right/Left）智能绘制短线和文字，自动处理插入点的偏移，避免文字与短线重叠。
- **强制文字左对齐**，无论当前文字样式为何。
- 默认文字颜色 BYLAYER，可按需调整。

### 5. 里程号格式控制
- 普通桩号按用户设定小数位数显示；
- **终点桩号如为小数，自动强制三位小数显示**，否则按全局设定；
- 自动补零，使“公里+米”格式对齐美观（如 1+005.000）。

### 6. 结果导出
- 标注结果以 TXT 或 CSV 格式导出，内容为：桩号文本、X 坐标、Y 坐标。

---

## 二、使用说明

1. **加载脚本**  
   在 AutoCAD 命令行输入：
   ```
   APPLOAD
   ```
   选择 `LCBZ.LSP` 文件加载。

2. **运行命令**  
   命令行输入：
   ```
   LCBZ
   ```
   按提示完成对象选择、参数输入。

3. **参数建议**
   - 对于曲线折点太多、线段极短时，建议将标注间距设为大于单段长度，避免标注点密集。
   - “小数位数”一般选 0~3，终点桩号为小数时脚本自动三位小数，无需人工干预。

4. **文字颜色与样式**
   - 默认颜色 BYLAYER，如需全部红色，将 `cons 62 256` 改为 `cons 62 1`。
   - 文字始终左对齐，不受当前样式影响。

5. **对齐方式说明**
   - Center：短线以桩点为中点，文字在短线上方偏左。
   - Right：短线从桩点向右/上，文字在短线上方偏左（默认）。
   - Left：短线从桩点向左/上，文字在短线上方偏左。

6. **导出结果**
   - 可选 TXT 或 CSV，文件内容为“桩号文本、X坐标、Y坐标”。

---

## 三、修改与深化说明

### 1. 支持更多对象/场景
- 已支持直线（LINE）对象，自动切换参数与点计算方式。
- 可通过扩展 `vlax-curve` 能力，支持更多曲线类型。
- 如需支持圆、圆弧等对象，需对对象类型作进一步适配。

### 2. 文字内容格式自定义
- 可在 `format-mile` 函数中修改桩号输出风格，如去掉“+”分割、调整补零规则等。

### 3. 颜色、图层、文字样式
- 颜色：将 `(cons 62 256)` 改为指定 AutoCAD 颜色号。
- 图层：可在 `entmakex` 中增加 `(cons 8 "LayerName")` 指定图层。
- 文字样式：可加 `(cons 7 "TextStyle")` 控制文字样式。

### 4. 精度与容错
- 桩号点若超出曲线或直线总长，脚本自动忽略。
- 曲线自交、极短线段等极端情况，AutoCAD 本身 API 可能存在精度限制，建议提前简化曲线。

### 5. 交互优化
- 可将部分参数设为命令行变量，便于批量处理。
- 可增加预览、批量撤销等功能。

---

## 四、常见问题解答

1. **桩号不显示或密集重叠？**
   - 检查间距设置是否太小，或曲线是否过度碎片化。

2. **终点桩号小数不显示三位？**
   - 只要终点不是整数，脚本自动三位小数；若为整数则不显示小数位。

3. **如何让所有文字都红色？**
   - 把 `entmakex` 中的 `(cons 62 256)` 改为 `(cons 62 1)` 即可。

4. **支持多段线闭合吗？**
   - 支持，首尾闭合时终点桩号处理与普通曲线一致。

5. **支持直线（LINE）对象吗？**
   - 支持，脚本已集成直线对象逻辑，选取直线后可正常沿线标注桩号。

---

如需进一步定制或遇到特殊标注场景，请联系我进一步补充需求！
如需使用请与我取得联系。
