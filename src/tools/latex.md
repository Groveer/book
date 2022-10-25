# Latex 环境配置

## Archlinux 环境

### 安装基础软件
```bash
yay -S plantuml inkscape texlive-core texlive-bibtexextra texlive-latexextra texlive-langchinese texlive-langjapanese python-pip
```
```bash
pip install Pygments
```

可能需要一些字体：
```bash
yay -S noto-fonts-cjk
```

### 安装模板
```bash
git clone git@gitlabwh.uniontech.com:rd/latex/latex.git
cd Latex
mkdir -p ~/texmf/tex/latex/
cp -r uniontech ~/texmf/tex/latex/
```

### 打中文支持补丁

```patch
--- /usr/share/texmf-dist/tex/lualatex/plantuml/plantuml.lua    2022-04-17 16:12:47.000000000 +0800
+++ /tmp/plantuml.lua   2022-10-25 11:19:18.203071671 +0800
@@ -21,8 +21,9 @@ function convertPlantUmlToTikz(jobname,
     return
   end

+  local lang = os.getenv("LANG")
   texio.write("Executing PlantUML... ")
-  local cmd = "java -Djava.awt.headless=true -jar " .. plantUmlJar .. " -charset UTF-8 -t"
+  local cmd = "LC_CTYPE=" .. lang .. " java -Djava.awt.headless=true -jar " .. plantUmlJar .. " -charset UTF-8 -t"
   if (mode == "latex") then
     cmd = cmd .. "latex:nopreamble"
   else
```

```bash
sudo  patch --verbose /usr/share/texmf-dist/tex/lualatex/plantuml/plantuml.lua < patch
```

### 构建

```bash
PLANTUML_JAR=/usr/share/java/plantuml/plantuml.jar \
lualatex "-shell-escape" \
    -synctex=1 \
    -interaction=nonstopmode \
    -file-line-error \
    {FILE_NEME}
```

## Deepin 环境

### 安装基础软件

```bash
sudo apt install texlive-full plantuml inkscape
```
```bash
sudo apt install python3-pip
```
```bash
pip3 install Pygments
```

### 安装模块

```bash
git clone git@gitlabwh.uniontech.com:rd/latex/latex.git
cd latex
sudo make install
sudo make postinstall
```
### 打中文支持补丁

```
打中文支持补丁

```pacth
--- /usr/share/texlive/texmf-dist/tex/lualatex/plantuml/plantuml.lua    2018-03-09 06:56:58.000000000 +0800
+++ ./plantuml.lua   2022-08-10 15:19:58.000000000 +0800
@@ -21,8 +21,10 @@
     return
   end
 
+  local lang = os.getenv("LANG")
+
   texio.write("Executing PlantUML... ")
-  local cmd = "java -jar " .. plantUmlJar .. " -t"
+  local cmd = "LC_CTYPE=" .. lang .. " java -jar " .. plantUmlJar .. " -t"
   if (mode == "latex") then
     cmd = cmd .. "latex:nopreamble"
   else
```
```bash
sudo  patch --verbose /usr/share/texlive/texmf-dist/tex/lualatex/plantuml/plantuml.lua < patch
```

### 构建

```bash
PLANTUML_JAR=/usr/share/plantuml/plantuml.jar \
lualatex "-shell-escape" \
    -synctex=1 \
    -interaction=nonstopmode \
    -file-line-error \
    {FILE_NEME}
```

需要注意的是，Deepin 系统下的 plantuml 版本可能比较老旧（/usr/share/plantuml/plantuml.jar），可在其他系统上拷贝较新的版本或使用本文章附件：[plantuml 包](../resources/plantuml-1.2022.6.jar)
