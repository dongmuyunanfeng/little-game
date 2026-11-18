# 缤纷叠影 · 折叠还原

一个纯前端的空间推理小游戏，玩法参考《最强大脑》同款项目「缤纷叠影」：
**一整张三角彩纸，沿折痕把左边的「原图」折成右边的「结果图」。**

没有任何依赖、不需要构建，双击 `index.html` 就能玩，也可以直接丢到 GitHub Pages / Vercel / 任意静态托管上。

![玩法示意](docs/preview.png)

---

## 玩法

盘面是一整张彩纸：方格线把纸分成 n×n 格，每格再用两条对角线切成 **4 个三角色块**
（上 / 右 / 下 / 左）。方格外框线与对角线都是**折痕**。

| 规则 | 说明 |
| --- | --- |
| 折痕 | 鼠标移到任意一条内部折痕上，会亮起「要被折走的一侧」 |
| 方向 | 只能把**较少的一侧**折过去；两侧等宽时折左 / 上侧 |
| 镜像 | 翻面时三角色块左右或上下**镜像**，并覆盖对面原有的颜色 |
| 底层 | 被折走的一侧露出下一张同款彩纸，所以**看起来不变** |
| 通关 | 盘面颜色与结果图完全一致即通关，不限步数 |

计分：用时越短、步数越接近「参考步」越好，三星条件为 `步数 ≤ 参考步`。

### 操作

- 鼠标移上折痕 → 预览，点击 → 折叠
- 键盘：`Z` 撤销 · `R` 重来 · `H` 提示
- 提示按钮会用广度优先搜索找出 3 步内的最短解法并高亮那一条折痕

### 难度

| 难度 | 盘面 | 颜色 | 折数 |
| --- | --- | --- | --- |
| 入门 | 5×5 | 3 | 2 |
| 初级 | 6×6 | 4 | 3 |
| 中级 | 7×7 | 4 | 4 |
| 高级 | 8×8 | 5 | 6 |
| 挑战 | 9×9 | 6 | 8 |

题目由程序实时生成：先随机生成一张彩纸，再随机折叠 k 次，把折完的结果当作「结果图」。
所以每一道题**一定可解**，而且无限出题、不会重复。

---

## 本地运行

直接双击 `index.html` 即可（全部逻辑内联，`file://` 也能跑）。

想用本地服务器：

```bash
# 任选一种
python -m http.server 8080
npx serve .
```

然后打开 <http://127.0.0.1:8080>。

---

## 部署到 GitHub Pages

### 1. 建仓库并推送

```bash
cd binfen-dieying
git init
git add .
git commit -m "feat: 缤纷叠影 · 折叠还原"
git branch -M main
git remote add origin https://github.com/<你的用户名>/binfen-dieying.git
git push -u origin main
```

### 2. 打开 Pages

仓库 → **Settings** → 左侧 **Pages**：

- **Source** 选 `Deploy from a branch`
- **Branch** 选 `main`，目录选 `/ (root)`
- 点 **Save**

等 1 分钟左右，访问：

```
https://<你的用户名>.github.io/binfen-dieying/
```

> 仓库里已经放了 `.nojekyll`，可以跳过 Jekyll 处理，静态文件原样发布。

### 3. 绑定自己的域名

假设你的域名是 `fold.example.com`。

**a. 在仓库里加 CNAME**
把本目录的 `CNAME.sample` 复制成 `CNAME`（无后缀），内容只写域名一行：

```
fold.example.com
```

**b. 配 DNS**（在域名服务商处）

| 记录类型 | 主机记录 | 记录值 |
| --- | --- | --- |
| CNAME | `fold` | `<你的用户名>.github.io` |

如果用**根域名**（`example.com`）而不是子域名，CNAME 不适用，要配 A 记录：

| 类型 | 主机记录 | 记录值 |
| --- | --- | --- |
| A | `@` | `185.199.108.153` |
| A | `@` | `185.199.109.153` |
| A | `@` | `185.199.110.153` |
| A | `@` | `185.199.111.153` |

**c. 回填域名**
回到 **Settings → Pages → Custom domain** 填 `fold.example.com`，Save，DNS 检查通过后
勾选 **Enforce HTTPS**（证书由 GitHub 自动签发，通常几分钟到一小时生效）。

DNS 生效后，直接访问 `https://fold.example.com` 就是游戏本体，不需要任何路径。

> 其他托管方式同理：文件是纯静态的，Vercel / Netlify / Cloudflare Pages 直接拖进去就能上线，
> 绑定域名后在平台后台加一条 CNAME 即可。

---

## 项目结构

```
binfen-dieying/
├── index.html        # 游戏本体（HTML + CSS + JS 全部内联，无依赖）
├── README.md
├── LICENSE           # MIT
├── CNAME.sample      # 自定义域名模板
├── .nojekyll         # 关闭 GitHub Pages 的 Jekyll 处理
└── docs/
    └── preview.png   # 示意图
```

## 实现要点

- **折痕建模**：一条竖折痕位于第 `i` 列右侧（`i = 1..n-1`），左侧宽 `i`、右侧宽 `n-i`，
  永远折较窄的一侧，因此折叠后彩纸不会超出盘面。
- **三角镜像**：竖直折叠时 上→上、下→下、左↔右；水平折叠时 左→左、右→右、上↔下。
- **动画**：把源侧区域连同折痕线克隆成一层 `flap`，以折痕为 `transform-origin` 做
  `rotateY/rotateX(180deg)`，`backface-visibility` 默认可见，翻过去正好是镜像的那一面。
- **响应式**：色块全部用百分比定位 + `clip-path` 切三角，折痕用 `viewBox` 为 `0 0 n n`
  的 SVG 叠加（`vector-effect: non-scaling-stroke` 保证线宽不随尺寸变化），缩放不重排。

## License

[MIT](LICENSE)
