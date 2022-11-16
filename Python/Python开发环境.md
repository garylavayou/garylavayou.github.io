# Python开发环境配置

## Python环境

### 系统集成Python环境

以Ubuntu为例，系统自带Python 3，用于支持相关系统组件工作。如果某些软件的运行需要依赖Python软件包而系统并未预装，则可以使用软件源管理工具（`apt`）安装。

> 使用相关python命令时需要注意：系统自带的`python3/pip3`并没有创建符号链接`python/pip`，以便可以同时安装Python 2。


```shell
sudo apt install {python-package_name|python3-package_name}
sudo apt install python-is-python3  # 设置默认版本为Python3
```

如果软件源中找不到所需软件包，则可以通过`pip`工具从Python的软件仓库中下载软件包。系统的Python发行版默认未安装安装`pip`，需手动安装：

```shell
sudo apt install {python-pip|python3-pip} # Python package installer
```

`python3-dev`：Python软件开发依赖环境（*header files and a static library for Python (default).*），在Fedora/CentOS中命名为`python3-devel`。

##### 查找软件包

```shell
pip list	          # list installed packages
pip show <package>  # show information about installed packages
```

使用`pip`(`pip3`)安装和移除软件包：

```shell
pip install <pkgs> \
    --user                                 # 仅为当前用户安装，默认为系统范围安装
    -r requirements.txt \                  # 指定安装包声明文件
    -i https://pypi.doubanio.com/simple/ \ # 强制使用镜像站点
    --trusted-host 172.28.76.237           # 如果使用自建的代理站点，添加此选项
pip uninstall -r <requirements_file>
pip cache remove <pattern>|purge
```

> 使用`sudo`在系统范围安装。
>
> Ubuntu等Linux发行版已经内置许多python模块，应该优先使用软件源管理工具而非`pip`进行系统范围的Python软件包更新，否则可能导致系统功能出错。仅使用`pip`更新手动安装的软件包。如果不小心使用`pip`更新了`pip`，可以卸载更新后的版本并使用软件源管理工具重装。
>
> ```shell
> sudo python3 -m pip uninstall pip
> sudo apt install python3-pip --reinstall
> ```
>
> 如果需要创建开发环境，使用虚拟Python环境（[Virtualenv](#Virtualenv)）

##### 软件更新

```shell
pip list --outdated
pip install --upgrade package_name # 安装并更新依赖包（默认不更新）
pip-review  # pip install pip-review
# pip install pipupgrade
```

> 1. [achillesrasquinha/pipupgrade: 🗽 Like yarn outdated/upgrade, but for pip. Upgrade all your pip packages and automate your Python Dependency Management. (github.com)](https://github.com/achillesrasquinha/pipupgrade)
>
> 2. [jgonggrijp/pip-review: A tool to keep track of your Python package updates. (github.com)](https://github.com/jgonggrijp/pip-review)

使用较近的PyPi镜像站点可以加速软件包下载。

##### 离线安装

下载（不安装）安装包，默认同时下载依赖包。

```shell
pip download [--python-version 3.7.4] package_name[==version]
```

> `-d,--dest <dir>`: 下载目标文件夹（默认为当前工作目录）。
>
> 指定`--python-version`时，必须同时指定`--no-deps`或`--only-binary=:all:`。

安装离线包

```shell
pip install --no-index --find-links=/local_path package_name
```

> `--find-links`：如果指定本地路径或`file://url`，则优先在本地目录中寻找安装文件。
>
> `--no-index`：不使用在线仓库搜索依赖包；

   

### Anaconda/Miniconda

Anaconda发行版打包了Python常用的软件包，可以从[官网](https://repo.anaconda.com/archive/Anaconda3-2019.07-Linux-x86_64.sh)或[镜像](https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/Anaconda3-5.3.1-Linux-x86_64.sh)站点下载安装包。[Miniconda](https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/Miniconda3-latest-Linux-x86_64.sh)是精简发行版，仅包含python语言环境和conda工具，其他软件包均需要在线下载。

> Anaconda/Miniconda可以配置自定义的python环境屏蔽系统的Python，可以任意修改该环境而不会对系统产生影响。
>
> 在`Linux aarch64`平台上，[官方提供的安装包](https://docs.conda.io/en/master/miniconda.html)可能安装失败，可尝试使用[conda-forge/miniforge: A conda-forge distribution. (github.com)](https://github.com/conda-forge/miniforge)。
>
> - *`version 'GLIBC_2.25' not found`*

> Anaconda基础环境的Python版本参考发行注记：https://docs.anaconda.com/anaconda/reference/release-notes/。

#### 安装conda运行环境

```shell
bash conda_installer.sh -b -f -p $INSTALL_PATH  # 静默安装并指定安装路径
# -u更新已有安装；
```

静默安装模式不会执行初始化修改路径配置。如果安装期间未执行初始化，则Shell无法直接运行`conda`（不在路径中）。

```python
eval "$($CONDAROOT/bin/conda shell.bash hook)"   # 激活默认环境
conda init   # [yes]
```

也可以[使用包管理器安装](https://docs.conda.io/projects/conda/en/latest/user-guide/install/rpm-debian.html)，方便[卸载](#卸载Anaconda/Miniconda)。

##### 重新安装Anaconda/Miniconda

如果基础环境因为更新问题损坏（网络、杀毒软件拦截）需要重装，而不希望重装已配置的[虚拟环境](#Conda虚拟环境)，可以将虚拟环境目录`envs`备份，在重装基础环境后还原。

##### 卸载Anaconda/Miniconda

```shell
conda install anaconda-clean
anaconda-clean --yes  # backup all files/directories in ~/.anaconda_backup 
rm -rf $CONDA_HOME
```

最后清理`~/.bashrc`中的`conda`初始化代码。

#### conda配置

使用命令行修改配置项，

```shell
conda config --{system|env} SUB_COMMAND
```

`conda`配置文件包括三类：

- `--system`：修改系统配置位于`$CONDA_HOME/.condarc`；
- `--env`：修改当前激活环境的配置，位于`$CONDA_HOME/envs/$ENV/.condarc`；
- 如果未指定上述选项，则修改当前用户的配置，位于`$HOME/.condarc`。

修改配置：

```shell
conda config --append KEY VALUE      # 为配置项KEY追加一个值
             --prepend/add KEY VALUE # 为配置项KEY前端插入一个值
             --remove KEY VALUE      # 从配置项KEY的值列表中删除匹配的VALUE
             --set KEY VALUE         # 更新配置项的值
             --remove-key KEY        # 移除配置项
```

##### 列出当前的conda配置

```shell
conda config --show [CONFIG]
conda config --describe [CONFIG]  # 显示配置项的说明
```

##### 自动激活基础conda环境

是否在shell启动时启动Conda环境：

```shell
conda config --set auto_activate_base false  # => auto_activate_base: false
```

#### 使用`conda`进行包管理

```shell
conda info	# infomation of conda installation (.condarc)
conda list [package_name] [-n envname]
conda search name --info
conda {update|upgrade} {--all | package_names}
conda [<cmd>] --help
```

Anaconda/Miniconda使用`conda`作为软件包管理工具。更新`conda`工具：

```shell
conda update -n base -c defaults conda
```


> Anaconda发布的Python版本中，机器学习库`sklearn`打包在`scikit-learn`中。

固定包的版本：在`conda-meta`目录中创建一个名为`pinned`的文件并加入不希望更新的包名以及版本信息（可以使用`#`注释行）。

```shell
numpy 1.7.*     # stay on the 1.7 series
scipy==0.14.2  # fix to 0.14.2
```

```shell
conda update numpy --no-pin   # 忽略pinned文件中的声明
```

> 每次执行更新时，conda会检查`pinned`文件，因此使用`--no-pin`升级的包会恢复到`pinned`文件中声明的版本。

##### 清理缓存

当软件包升级后，旧版本的软件包可能不再有用，因此可以从本地缓存删除。

```shell
conda clean \ # Remove unused packages and caches.
      --all \ # index cache, lock files, unused packages and tarballs
      -i,--index-cache  \ # 清除索引缓存，保证用的是镜像站提供的索引。
      -p,--packages     \ # Remove unused packages
      -t,--tarballs     \ # remove cached package tarballs
      -l,--logfiles
      -d,--dry-run
```

##### 安装软件包

```shell
conda {remove|uninstall} package_names
conda install -c conda-forge vaex=4.0.0 # --dry-run
conda install package_names \
	--file requirements.txt \ # 安装文件给定的包
	--freeze-installed \      # 避免已安装包升级*
	-n| --name ENVNAME \      # 目标环境名（未指定则为当前环境，默认为base）
	-p,--prefix ENVPATH \     # 目标环境的目录
```

> `*`：`conda`尝试安装最新的包，为此将升级其依赖的安装包。使用`--freeze-installed`避免已安装包自动升级。

**指定仓库名**：`-c,--channel`指定安装软件包优先使用的仓库；可以重复使用该选项指定多个仓库（优先级从高到低）；如果未指定该参数或指定仓库不包含相应的软件包，`conda`会检查`.condarc`中设定的[`channels`中包含的其他仓库](#conda软件源)。

软件包声明文件格式：

```shell
channel::package_name=version=build_string # 使用指定使用*进行模糊匹配
```

> [使用指定的`channel`代替当前的仓库搜索配置](https://github.com/conda/conda/blob/4.4.0/CHANGELOG.md)，添加仓库前缀可能导致依赖解析失败（`pytorch`）。

常用声明格式：

```shell
pytorch             # 安装最新的兼容版本
cudatoolkit>=11.3   # 限制最低版本 （命令行使用时需要使用""对><转义，避免解释为重定向）
cudatoolkit=11.3.*  # 仅接受小版本更新（如果后续无build声明可省略.*）
cudatoolkit=11.3.1  # 安装固定版本
cudatoolkit==11.3   # 安装精确的指定版本
```

###### 安装离线包

```shell 
conda install --use-local   /path/XXX.tar.bz
```

在Anaconda/Miniconda的Python环境中也可以使用`pip`进行软件包管理。

*使用conda下载离线包*：`conda`只能将软件包加载到本地仓库（缓存）中，而无法下载到指定文件夹。

> `conda`本地仓库中的包无法用于`pip`安装。

###### 安装Conda软件源中没有的包

```shell
pip install --upgrade-strategy "only-if-needed" packname
```

> - 在使用conda安装尽可能多的包后，再使用`pip`安装额外包；
> - 在单独虚拟环境中使用`pip`(避免使用root环境)；使用`pip`后，Conda无法识别相应的更改；
> - 如果后续还需要使用Conda进行更新或安装而外包，则使用基本环境再重新创建一个环境后再使用`pip`；
> - 使用`-r, --requirements`指定需要使用`pip`安装的包。

https://www.anaconda.com/using-pip-in-a-conda-environment/

### 虚拟Python环境

#### virtualenv

在指定目录`ENV_DIR`创建一个虚拟Python环境。

```shell
python3 -m venv ENV_DIR
```

> 创建虚拟环境，默认包含Python解释器（与系统中Python解释版本相同）、标准库等。可以使用`pip`为虚拟环境安装额外的内容。
>
> 在Debian/Ubuntu系统中需要首先安装
>
> ```shell
> apt-get install python3-venv
> ```

Windows激活虚拟环境：

```shell
.\myenv\Scripts\activate.bat  # windows cmd
.\myenv\Scripts\Activate.ps1  # windows powershell
```

Linux激活虚拟环境：

```shell
source myenv/bin/activate   # on linux
which python  # 查看python路径
```

##### pipenv

[Pipenv: A Guide to the New Python Packaging Tool – Real Python](https://realpython.com/pipenv-guide/)

consolidate the `pip` & `virtualenv` into a single interface. 

#### Conda虚拟环境

> *It makes your project more self-contained as everything, including the required software, is contained in a single project directory.*

基于`conda`创建的虚拟环境并安装指定的软件包（和安装一样使用`--file requirements.txt`指定安装包）。Conda默认将虚拟环境置于`envs`目录下，并通过`--names/-n`选项指定要使用的环境名。使用`--prefix`可将虚拟环境置于任何路径下，但访问时也需要通过`--prefix`选项指定目录。

```shell
conda create -n pydev37 python=3.7 <pkgname=ver,...> \ # python=3*
             --prefix ./envs ...          # 指定虚拟环境的安装目录
conda create --name myclone --clone myenv # 复制已有环境
conda env remove --name myenv             # <=> conda remove -n env_name --all 
conda env list                            # <=> conda info --envs
```

> `*`：如果创建环境时未指定Python版本，可能导致依赖解析很慢。
>
> 支持创建Python 2.x环境。
>
> 虚拟环境中不会维护独立的软件包，而是由Conda维护一个统一的包目录（`pkgs`）。通过`conda-meta`目录中的信息，可以定位到其引用的包路径，包的档案文件路径以及软件源路径。
>
> 复制环境可直接将虚拟环境目录复制到`envs`目录下（如果虚拟环境路径发生变化，需要使用`conda-pack`进行[打包并在目标系统中解压并修复路径](#虚拟环境打包部署)）   

虚拟环境配置默认继承系统和用户配置，可以[为虚拟环境添加设置覆盖默认配置](#conda配置)。

查看conda环境更改历史：

```shell
conda list --revision[s]
conda install --revision=REVNUM   # 恢复历史版本
```

虚拟环境信息文件：

```shell
conda env export [--from-history] > environment.yml
```

更新环境

```shell
conda env update --prefix | --name --file environment.yml --prune
```

复制环境

```shell
conda env create -f environment.yml   #从配置文件创建虚拟环境
#=========================================
conda list --explicit > spec-file.txt
conda create --name myenv --file spec-file.txt
conda install --name myenv --file spec-file.txt
```



##### 使用conda虚拟环境

```shell
conda activate env_name
conda deactivate
conda config --set env_prompt '({name})'
```

> 在脚本中尝试激活命令时会出现conda环境未初始化的错误，可在脚本中执行`source ~/.bashrc`或将`~/.bashrc`中的conda初始化代码复制到用户脚本中。

[Python - Activate conda env through shell script - Stack Overflow](https://stackoverflow.com/questions/55507519/python-activate-conda-env-through-shell-script)

[Can't execute `conda activate` from bash script · Issue #7980 · conda/conda (github.com)](https://github.com/conda/conda/issues/7980)

#### 虚拟Python环境损坏

`AttributeError: module 'brotli' has no attribute 'error'`

解决方法：使用`conda remove`删除损坏的环境，并删除本地目录；

##### Pipeenv

#### 虚拟环境打包部署

`conda-pack`可以将虚拟Python环境打包并部署到其他位置（操作系统相同）。可创建一个包含`conda-pack`的虚拟环境专门用于打包。

```shell
conda create -n pack -c conda-forge "conda-pack>=0.7"  # pip install conda-pack=0.7
```

> *`CondaPackError: Files managed by conda were found to have been deleted/overwritten...`*：
>
> 1. Python升级到3.10后，其模块库目录下会创建一个`python3.1->python3.10`的符号链接；这会[导致旧版本的`conda-pack`无法正常工作](https://stackoverflow.com/questions/69992742/conda-pack-condapackerror-files-managed-by-conda-were)，需要使用`conda-pack>=0.7`。
> 2. 如果虚拟环境中使用`pip`安装了额外包，也可能导致上述错误。因此可将pip的安装包单独下载，再在目标环境中执行离线安装。

然后将目标环境打包环境为一个压缩档案文件（`tar.gz`）：

```shell
conda-pack --name my_env \
           --output out_name.tar.gz \ # 指定环境名
           -d target_env_path \   #目标主机上虚拟环境的路径
           --force
conda-pack -p /explicit/path/to/my_env  # 指定环境路径
```

在目标位置解压文件，并激活虚拟环境（要求目标环境安装`conda`）：

```shell
tar -xzf my_env.tar.gz -C conda_path/envs/my_env
source my_env/bin/activate # 激活环境 add `my_env/bin` to your path
conda-unpack               # 修复库的路径（可在未激活环境情况下运行，指定路径） 
```

`conda-unpack`是`conda-pack`打包到虚拟环境中的程序（`envname/bin`目录下），在解压后修复某些库的路径。==如果打包时指定`-d`选项，则不再打包`unpack`程序==。

[Conda-Pack — conda-pack 0.6.0 documentation](https://conda.github.io/conda-pack/).

### Arm64版本Python

支持aarch64 (arm64)的Conda环境：

- [conda-forge/miniforge: A conda-forge distribution. (github.com)](https://github.com/conda-forge/miniforge)
- https://repo.anaconda.com/archive/Anaconda3-2021.04-Linux-aarch64.shell

> 需要使用conda-forge作为默认源（miniforge的默认设置），且不能使用国内镜像源（同步不完整）。

## 软件源配置

### pip软件源

配置软件源和下载设置：

```shell
pip config set global.timeout 6000
pip config set global.index-url https://pypi.doubanio.com/simple/
pip config set install.use-mirrors true
pip config set install.mirrors https://pypi.doubanio.com/simple/
```

该配置文件位于`$HOME/.pip/pip.conf`（系统自带Python的配置文件位于`$HOME/.config/pip/pip.conf`）。也可以手动创建并编辑该文件（Windows下的配置文件位于`%HOMEPATH%\AppData\Roming\pip\pip.ini`），在文件中添加以下内容：

```shell
[global]
timeout = 6000
index-url = https://pypi.doubanio.com/simple
[install]
use-mirrors = true
mirrors = https://pypi.doubanio.com/simple
```

常用软件源包括：

- `https://pypi.doubanio.com`：豆瓣；
- `http://mirrors.aliyun.com/pypi`：阿里；
- `http://pypi.mirrors.ustc.edu.cn`：中国科学技术大学；
- `https://pypi.tuna.tsinghua.edu.cn`：[清华大学](https://mirrors.tuna.tsinghua.edu.cn/help/pypi/)；

本地镜像可用[Nexus配置](../应用软件/网络服务软件.md#pypi代理)。

### conda软件源

添加默认仓库`defaults`的地址：

```shell
conda config --add default_channels \
    https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/  # add default channel URL
conda config --set show_channel_urls yes  # 显示下载内容的URL
```

添加自定义仓库名称和地址：

```shell
conda config --{prepend|append} channels <new_channel>
conda config --set custom_channels.conda-forge \
    https://mirror.sjtu.edu.cn/anaconda/cloud/
```

直接编辑该文件：

```yaml
channels:
  - conda-forge
  - defaults
default_channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
custom_channels:
  conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch:     https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
```

配置文件中`channels`为使用`conda`时默认使用的一个或多个源，优先级从上到下递减。默认从高优先级的源选择同名软件包（`channel_priority=strict`）；反之优先选择高版本号的同名软件包。

```shell
conda config --set channel_priority strict
```

通过`default_channels`配置`defaults`仓库的实际地址；通过`custom_channels`配置其他仓库对应的地址；如果使用`--channel`选项或`channels`配置项指定的仓库未在`custom_channels`设定相应的地址信息，则默认从[Anaconda官方源](https://conda.anaconda.org/)查找该仓库。

列出当前的仓库配置

```shell
conda config --show-sources   # 显示配置文件的所有信息
conda config --show channels  # 显示配置文件中的channels内容
conda config --validate       # 验证仓库源
```

##### 仓库镜像

- `https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/`：[清华大学](https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/)，包含`pytorch`、`conda-forge`等多个仓库，其中`conda-forge`包含`linux-aarch64`架构的仓库，可用于ARM平台；
- `https://mirrors.bfsu.edu.cn/anaconda/cloud/`：[北京外国语大学](https://mirrors.bfsu.edu.cn/help/anaconda/)（与清华大学镜像源一致）；
- `https://mirror.sjtu.edu.cn/anaconda/cloud/`：[上海交通大学](https://mirrors.sjtug.sjtu.edu.cn/docs/anaconda)（不包含`conda-forge/linux-aarch64`）；
- `https://mirrors.aliyun.com/anaconda/cloud/`：[阿里云](https://developer.aliyun.com/mirror/anaconda)（不包含`conda-forge/linux-aarch64`）。

##### 使用[Nexus](../应用软件/网络服务软件.md#conda代理)可创建镜像代理

> Nexus Conda代理不会获取`current_repodata.json`，因此使用`conda`时总是会回退使用`repodata.json`，导致解析速度变慢（直接使用镜像源则不会有此问题）。`mamba`总是使用`repodata.json`不会受此影响。

多数镜像源仅包含最常用的仓库，如`conda-forge`、`pytorch`，而一些开源库在Anaconda官方仓库维护了独立的仓库（仅包含开源库相关包），因此要使用这些仓库需要使用Anaconda官方仓库作为代理源。

```ini
Name=anaconda-cloud
Remote=https://conda.anaconda.org/
URL=http://192.168.178.52:8081/repository/anaconda-cloud
```

#### 软件包下载优化

##### 安装mamba/libmamba加速依赖解析和下载

> *`mamba` is a reimplementation of the conda package manager in C++.*

`mamba`支持多线程并行下载仓库数据和包文件，使用`libsolv`以加速依赖解析（用于RPM包管理器）。

```shell
conda install -n base -c conda-forge mamba
```

安装完成后，使用`mamba`代替`conda`进行包管理，其语法与`conda`一致。

> 最新的`conda`版本（`22.9+`）提供`--experimental-solver libmamba`选项（需要安装`conda-libmamba-solver`包），使用`mamba`进行依赖解析，但仍采用`conda`下载包。

`mamba/libmamba`虽然加速解析和下载，但**不使用本地缓存导致重复下载**（*虽然检测到本地缓存包但似乎URL不匹配*）。如果是通过本地镜像仓库下载，则重复下载没有多大影响；反之，如果通过远程仓库下载，则重复下载比较浪费时间。

##### 配置HTTP代理

`conda`会自动检测系统代理配置，但不支持`$no_proxy`例外项。如果`conda`使用的代理配置与系统配置不同，可在`.condarc`中指定（同样不支持例外项）。

```yaml
proxy_servers:
  http: http://172.28.76.4:3128
  https: http://172.28.76.4:3128
```

> `conda`/`mamba`使用`urllib3`，会读取环境变量`$http[s]_proxy`；Windows中优先检测环境变量`$env:HTTP[s]_PROXY`，其次检测**网络设置**中的**代理设置**，可通过代理设置的例外项排除不需要通过代理访问的镜像源地址，例如本地仓库代理。）

##### 网络超时设置

当使用`nexus`镜像代理时，新内容会首先下载到代理服务器，然后再下载到本地。因此，代理服务器如果下载速度较慢会导致本地`conda`下载超时。这种情况，可将网络读取的超时限制调大。

```yaml
remote_read_timeout_secs: 180.0  # 60.0 by default
```

> `mamba`忽略该选项，==尝试设置`export MAMBA_NO_LOW_SPEED_LIMIT=1`==。

## 开发环境

### 标准库

- 系统管理：`os, sys, shutil`

- 日期时间：`datetime, time, calendar`

- 高级数据结构：`collections, array, enum `

- 输入输出控制：`io, fcntl, logging, getpass, warnings`

- 数据处理：`re`, `struct`

- 文件解析：`csv, configparser, json, xml, html`

  Python有三种方法解析XML，SAX，DOM，以及ElementTree。ElementTree就像一个轻量级的DOM，具有方便友好的API。代码可用性好，速度快，消耗内存少。

  > **注：**因DOM需要将XML数据映射到内存中的树，一是比较慢，二是比较耗内存，而SAX流式读取XML文件，比较快，占用内存少，但需要用户实现回调函数（handler）。

- 数据持久化：`pickle, marshal, sqlite3`

  `sqlite`使用文件保存数据，因此无需运行任何服务。Python内置`sqlite3`提供sqlite数据库的操作。

- 数据库：`mysql-connector-python`, `psycopg2`（PostgreSQL）

  DB-API 是一个规范. 它定义了一系列必须的对象和数据库存取方式, 以便为各种各样的底层数据库系统和多种多样的数据库接口程序提供一致的访问接口 。

  [Introduction to Python SQL Libraries](https://realpython.com/python-sql-libraries/).

- 网络通信：`socket,ipaddress`,  `xmlrpc`, `email,smtpd,smtplib,imaplib,poplib`,  `cgi`, `ftplib`, `uuid,base64,urllib.parse`

- HTTP：`http`, `urllib.request`

  > [urllib](https://docs.python.org/3.1/library/urllib.request.html#module-urllib.request)是一个内置在Python标准库中的模块，并使用`http.client`来实现HTTP和HTTPS协议的客户端。

- 密码学：`hashlib, hmac`

- 数据压缩归档：`zlib,gzip,bz2,lzma,zipfile,tarfile`；

- 任务调度：`asyncio, multiprocessing, threading, concurrent, signal, subprocess, queue, select, atexit`：

  > `concurrent.futures`

- 程序管理：`importlib, inspect, typing, argparse, getopt, traceback`；

- 调试：`timeit, trace`；

  [PyPI Download Stats (pypistats.org)](https://pypistats.org/top)

### 第三方库

- 语法格式：`autopep8`, `pylint`；

- 系统管理：`filelock, psutil`

- 数值计算：`numpy`、`scipy`；

- 数据可视化：`matplotlib, seaborn, bokeh, plotly, pyecharts`；

  > [Python可视化库_As的博客-CSDN博客_python可视化](https://blog.csdn.net/weixin_39777626/article/details/78598346)。

- 文档生成：[`pylatex`](https://jeltef.github.io/PyLaTeX/current/index.html), [`pdfkit`](https://pypi.org/project/pdfkit/)；

- Web开发：`django, flask, tornado, requests, beatifulsoup, scrapy`；`urllib3`, `requests`

  > [Requests](https://docs.python-requests.org/en/master/)由[urllib3](https://urllib3.readthedocs.io/en/latest/)提供支持。

- 图形界面开发：`wxPython, PyQT, TKinter`；

- 命令行工具：`click`、[`rich`](https://github.com/Textualize/rich)`。

- [机器学习](../机器学习/机器学习实践.md#搭建开发环境)。

##### 数据处理

`pandas`、`pandasql`、`pyarrow`、`dask`；

使用`conda`安装时`pyarrow`，默认的仓库安装版本较旧（`0.15`），导致兼容问题，因此使用`conda-forge`通道。

```python
conda install -c conda-forge pyarrow
```
```python
conda install dask
pip install "dask[complete]"    # Install everything
```

Pandas：

```python
pd.show_versions()   # 查看系统环境、pandas版本及其依赖包的版本
```

```python
pd.io.parquet.get_engine('auto')   # 获取parquet读写引擎
```

> 引入`pandas`包后立即调用该方法，解决与`matlab.engine`的环境冲突。



### Visual Studio

安装Python工作负载；

安装Python/Conda虚拟环境；

通过Python环境窗口，安装或卸载包或启动交互式Python命令环境；

<img src="Python开发环境.assets/image-20210522085526208.png" alt="image-20210522085526208" style="zoom:50%;" />



### Visual Studio Code

#### Python开发插件

- **Python**：Python语言支持，包括智能提示、语法检查、调试、代码导航、代码格式化、重构、测试等功能。
- **Pylance**：Python语言服务器。
- **autoDocstring**：自动生成Python文档。
- **Jupyter**（**Jupyter Notebook Renderers**）：支持在VS Code中使用Jupyter Notebook。
- **Better Comments**：有5种类型的注释高亮，分别用符号`* ? ! //todo`来区分。

#### Python环境配置

##### 插件环境变量

在项目目录中`${project}/.env`文件（默认位置，可通过`"python.envFile"`修改）中设置：

```shell
PYTHONPATH=path/to/project/lib   # 不添加export，可使用相对路径
```

此配置中的环境变量由Python插件读取，并在调试、语法检查、格式化、智能提示和测试工具中使用，对终端无效。

##### 终端环境变量

在工作区配置（`.vscode/settings.json`）中设置

```json
{
    "terminal.integrated.env.linux": {
        "PYTHONPATH": "path/to/project/lib",
        "OTHER_VARIABLE": "value"
    }
}
```

##### conda虚拟环境加载

Conda环境未正确加载会导致某些包（例如`numpy`）无法被导入。解决方案：

1. *为保证终端中Conda环境自动加载（终端启动时加载），执行*

   ```powershell
   conda init cmd.exe|powershell   # conda config --set auto_activate_base false
   ```

   初始化文件在`%USER%/Documents/WindowsPowerShell/profile.ps1`。

2. *在VS Code中配置Conda路径，从而使终端启动时自动激活Conda虚拟环境（`cmd`）*。

   ```json
   "python.condaPath": "C:\\tools\\miniconda3\\Scripts\\conda.exe"
   ```

> To ensure the environment is set up well from a shell perspective, one option is to use an Anaconda prompt with the activated environment to launch VS Code ==using the `code .` command==. At that point you just need to select the interpreter using the Command Palette or by clicking on the status bar.
>
> Conda environments can't be automatically activated in the VS Code Integrated Terminal if the default shell is *set to PowerShell*. 
>
> [PowerShell does not support automatic activation of conda virtual environment · Issue #11638 · microsoft/vscode-python (github.com)](https://github.com/microsoft/vscode-python/issues/11638)

#### 自动补全和语法分析配置

```json
{
   "python.autoComplete.extraPaths": [
      "/home/gary/dataproc/dataproc/lib"
   ],
   "python.analysis.extraPaths": [
      "/home/gary/dataproc/dataproc/lib"
   ]
}
```

##### 告警信息配置

忽略单行警告，在行末添加声明。如果要忽略同一文件，则将声明置于单独行。

```python
import config.logging_settings  # type: ignore   => pylance
import config.logging_settings  # pylint: disable=unused-import
```

> `pylint`可以放在`type`标记后。

全局忽略警告：

```json
"python.linting.pylintArgs": [
    "--disable=unused-import",
    "--disable=too-few-public-methods"
]  // => pylint
"python.analysis.diagnosticSeverityOverrides": {
    "reportUnusedImport": "information",
    "reportMissingImports": "none"
}  // ==> pylance
```

> Pylance和pylint可同时启用，因此需要设置两项配置。

[Type Check Diagnostics Settings · microsoft/pyright (github.com)](https://github.com/microsoft/pyright/blob/main/docs/configuration.md#type-check-diagnostics-settings)。

#### 单元测试

可将项目目录下的一个文件夹设置为单元测试目录（如`test`）；单元测试脚本需要位于该目录下，以`test_*.py`模式保存的文件将被自动发现为测试脚本[^vsctest]。单元测试环境会自动将测试目录和当前项目目录加入`PYTHONPATH`。

> **Tip**: Sometimes tests placed in subfolders aren't discovered because such test files cannot be imported. To make them importable, create an empty file named `__init__.py` in that folder. （为每一级测试目录创建`__init__.py`测试路径可被识别为包）
>
> 由于测试目录位于搜索路径的最前，因此应该避免测试目录中包含与系统模块或用户模块同名的文件或文件夹（导致其屏蔽系统/用户模块的导入）。

##### 单元测试项自动发现

如果单元测试脚本中包含错误（无法捕获异常），则在发现阶段会将其分类的错误测试项一栏（完整错误日志可在Python输出窗口中查看）；解决错误后会自动更新到对应的测试目录树下。

#### 在VS Code中使用Jupyter Notebook
需要安装`ipykernel`以启动内核。

### Jupyter

##### 安装

```shell
conda create -n jupyterlab -c conda-forge {jupyterlab|jupyterhub}
conda install -c conda-forge ipywidgets jupyterhub-idle-culler
```

#### JupyterLab

##### 路径信息

`jupyter --path`配置文件、数据等的默认路径（以下给出首选路径）。

```shell
config:
    $HOME/.jupyter 
data:
    $HOME/.local/share/jupyter
runtime:
    $HOME/.local/share/jupyter/runtime
```

生成配置文件：配置文件中包含默认值（可进行修改）

```shell
jupyter lab --generate-config  # -> $HOME/.jupyter/jupyter_lab_config.py
```

网络配置

```python
c.ServerApp.base_url='/analyze/jupyter'
c.ServerApp.ip = 'localhost' # --ip=0.0.0.0
c.ServerApp.port = 0         # --port=8888
```

> 如果使用代理出现403错误，尝试设置`c.ServerApp.disable_check_xsrf=True`禁用*cross-site-request-forgery*检查。

[NGINX](../应用软件/网络服务软件.md#Nginx)反向代理JupyterLab服务，参考[JupyterHub反向代理配置](#JupyterHub网络配置)。

运行环境配置

```python
c.ServerApp.root_dir='~/jupyter'
c.ServerApp.allow_root = False   # --allow-root: Allow the server to be run from root.
c.ServerApp.autoreload = True    # --autoreload
c.ServerApp.open_browser = False # --no-browser
```

安全配置

```python
c.ServerApp.allow_origin = '*'
c.ServerApp.certfile = ''
c.ServerApp.keyfile = ''
c.ServerApp.cookie_secret = b''
c.ServerApp.cookie_secret_file = ''
c.ServerApp.password = ''   # from jupyter_server.auth import passwd; passwd()
```

##### 启动JupyterLab服务

以系统服务启动：

```ini
[Service]
Type=simple
Environment="WORKDIR=/home/ml/jupyterlab"
Environment="LOGDIR=/var/log/jupyterlab"
ExecStartPre=/usr/bin/mkdir -p /tmp/jupyter $WORKDIR
ExecStart=/usr/local/miniconda3/envs/jupyter/bin/jupyter lab $OPTIONS \
          > ${LOGIDR}/server.log 2> ${LOGIDR}/server.error.log
TimeoutSec=10
ExecStop=/bin/kill -s TERM $MAINPID
User=ml
```



#### JupyterHub

##### 系统架构

<img src="Python开发环境.assets/jhub-fluxogram.jpeg" alt="JupyterHub subsystems" style="zoom: 50%;" />

Jupyter系统组件包括：

- **HTTP Proxy**：代理用户请求，对外的唯一的Web接口；
- **Hub**：
- **Spawners**：管理用户服务；
- **Authenticator**：用户鉴权组件。

各个服务通过HTTP进行通信，因此可以分离部署。

> *Connections to user servers go through the proxy, and not the hub itself. If the proxy stays running when the hub restarts (for maintenance, re-configuration, etc.), then user connections are not interrupted. For simplicity, by default the hub starts the proxy automatically, so if the hub restarts, the proxy restarts, and user connections are interrupted. It is easy to run the proxy separately, for information see the separate proxy page.*

```shell
jupyterhub token|upgrade-db -h
```

##### 配置JupyterHub

推荐以下安装配置：

- `/srv/jupyterhub` for all security and runtime files
- `/etc/jupyterhub` for all configuration files
- `/var/log` for log files

生成并修改配置文件（默认输出到当前目录，使用`-f`选项指定输出目录）。

```shell
jupyterhub --generate-config -f /etc/jupyterhub/jupyterhub_config.py  
jupyterhub -h,--help-all   # 列出所有命令行配置项
configurable-http-proxy -h # 代理配置项
```

除了上述命令列出的配置项外，所有配置项均可通过命令行选项进行配置以覆盖配置文件中的对应配置项。

```shell
--Class.trait='value'  # c.Class.trait in [jupyterhub_config.py]
```

验证配置：

```shell
jupyterhub --show-config -f config_file  # 列出当前生效的配置(仅非默认值项)
```

##### 数据存储配置

```python
c.JupyterHub.data_files_path = '/usr/local/share/jupyterhub'
c.JupyterHub.db_url = 'sqlite:////var/lib/jupyterhub/jupyterhub.sqlite'
```

> 在读取配置文件时：`sqlite:///`视为相对路径，绝对路径需要使用`sqlite:////`。

##### JupyterHub网络配置

`bind_url`配置JupyterHub代理服务（用户的Web访问地址，代替弃用的`ip`、`port`和`base_url`）。省略IP则监听所有地址接收的请求，默认URL根路径为`/`。

```python
c.JupyterHub.bind_url='http://:8000/'  # --url
```

`api_url`配置REST API接口的访问地址（Hub服务通过该接口与代理服务通信）。

```python
c.ConfigurableHTTPProxy.api_url = 'http://localhost:8001' 
```

`hub_bind_url`配置Hub服务的访问地址（代理与Spawners通过该地址与Hub通信，代替`hub_ip`和`hub_port`）。`hub_connect_url`用于分离部署时指定Hub服务的地址（代替`hub_connect_ip`/`hub_connect_port`）。

```python
c.JupyterHub.hub_bind_url = 'http://localhost:8081'
c.JupyterHub.hub_connect_url = 'http://hub_ip:8081'
```

[外部nginx反向代理](https://jupyterhub.readthedocs.io/en/stable/reference/config-proxy.html#nginx)：

```nginx
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}
server {
  location /jupyterhub {
    proxy_pass http://127.0.0.1:8000;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Host $host;  #  $http_host for http connection
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    # websocket headers
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;
    proxy_set_header X-Scheme $scheme;

    proxy_buffering off;
  }
}
```

Spawner网络配置：

```python
c.Spawner.ip = '127.0.0.1'
c.Spawner.port = 0
```

使用SSL：

```shell
--ssl-key my_ssl.key   # c.JupyterHub.ssl_key = '/path/to/my.key'
--ssl-cert my_ssl.cert # c.JupyterHub.ssl_cert = '/path/to/my.cert'
```

> 使用`jupyterhub --generate-certs`生成自签名证书和私钥。

##### 用户访问控制

配置JupyterHub用户

```python
c.Authenticator.allowed_users = {'mal', 'zoe', 'inara', 'kaylee'} # 初始化用户集合
c.Authenticator.admin_users = {'mal', 'zoe'}
```

> *if `allowed_users` is not set, then all authenticated users will be allowed into your hub.*

可选：使用系统用户进行鉴权：*creating that user via the system `adduser` command line tool.*

```python
c.LocalAuthenticator.create_system_users = True
# PAMAuthenticator -> LocalAuthenticator -> Authenticator
```

> *This approach is not recommended when running JupyterHub in situations where JupyterHub users map directly onto the system’s UNIX users.*

自动创建用户可能会失败（CentOS7），可提前创建系统用户并进行初始化。

```shell
allowed_users=(mal zoe inara)
for u in "${users[@]}"; do
    useradd -m -g ml $u;     # 添加用户并添加到预定义的组
    if [ $? -eq 0 ]; then    # 为新用户设置默认密码（用于JupyterHub初始登录验证，登陆后修改密码）
        passwd $u --stdin <<< '1234abcd'
    fi
    cat /etc/passwd | grep --color=always $u  # 显示用户基本信息
done
```

Hub和Proxy（`ConfigurableHTTPProxy `）之间通过一个`secret token`来验证请求。此令牌可通过`openssl`生成，保存在配置文件或环境变量中。如果未设置，则Hub会自动随机生成一个，那么重启Hub的同时需要重启Proxy。

```shell
c.ConfigurableHTTPProxy.api_token = 'abc123...def' 
export CONFIGPROXY_AUTH_TOKEN=$(openssl rand -hex 32)
```

`cookie secret`用于对浏览器中用于鉴权的Cookies信息进行加密，可通过`openssl`生成。

```shell
openssl rand -hex 32 > /srv/jupyterhub/jupyterhub_cookie_secret 
chmod 600 /srv/jupyterhub/jupyterhub_cookie_secret
export JPY_COOKIE_SECRET=$(openssl rand -hex 32)
```

有三种指定方式：配置为环境变量，在配置文件中指定该信息所在的文件或在配置文件中指定该信息。

```python
c.JupyterHub.cookie_secret_file = '/srv/jupyterhub/jupyterhub_cookie_secret'
c.JupyterHub.cookie_secret = bytes.fromhex('64 CHAR HEX STRING')
```

> 如果未配置上述任何方法，则`jupyterhub_cookie_secret`默认创建在运行JupyterHub的用户的主目录下。该文件应该仅有该用户的读写权限，否则服务不会启动。
>
> 使用环境变量配置的随机生成密码会在重启服务时重新生成，导致所有登录会话失效。

**基于角色的访问控制**：[**Scopes**](https://jupyterhub.readthedocs.io/en/stable/rbac/scopes.html) are specific permissions used to evaluate API requests. [**Roles**](https://jupyterhub.readthedocs.io/en/stable/rbac/roles.html) are collections of scopes that specify the level of what a client is allowed to do. 

```python
c.JupyterHub.load_roles = [
 {
   'name': 'server-rights',
   'description': 'Allows parties to start and stop user servers',
   'scopes': ['servers'],
   'users': ['alice', 'bob'],
   'services': ['idle-culler'],
   'groups': ['admin-group'],
 }
]
```

[Use Cases — Scopes in JupyterHub](https://jupyterhub.readthedocs.io/en/stable/rbac/use-cases.html)

##### 配置用户后台服务

```shell
c.Spawner.cmd = ['jupyterhub-singleuser']  # 后台启动命令
# c.Spawner.cmd=["jupyter-labhub"]
c.Spawner.notebook_dir = '~/notebooks'     # 工作目录路径(默认为~)
```

> *Since the single-user server extends the notebook server application, it still loads configuration from the jupyter_notebook_config.py config file. Each user may have one of these files in $HOME/.jupyter/. Jupyter also supports loading system-wide config files from /etc/jupyter/, which is the place to put configuration that you want to affect all of your users.*

修改用户的工作目录为Jupyter服务独占目录，防止与用户的其他服务的文件冲突。如果为用户配置了非默认工作目录，则需要在用户服务初始化前进行检查确认该目录存在。JupyterHub配置文件中提供了初始化过程的接口。

```shell
def my_hook(spawner):
    import os,pwd,shutil
    username = spawner.user.name
    path = f'/home/{username}/notebooks'
    if not os.path.exists(path):
        os.makedirs(path, mode=0o755, exist_ok=True)
        passwd = pwd.getpwnam(username)
        os.chown(path, passwd.pw_uid, passwd.pw_gid)
        src = '/usr/local/share/jupyterhub/.bashrc'
        if os.path.exists(src):
            dest = f'/home/{username}/.bashrc'
            shutil.copy(src, dest)
            os.chown(dest, passwd.pw_uid, passwd.pw_gid)
c.Spawner.pre_spawn_hook = my_hook
```

> 除了工作目录外，这里也附带配置了用户的Shell初始化文件（`.bashrc`中包含Python执行环境初始化，可将已有用户的Shell配置文件作为模板文件）。

资源分配控制

```python
c.JupyterHub.active_server_limit = 0
c.JupyterHub.active_user_window = 1800
c.JupyterHub.activity_resolution = 30
c.Spawner.cpu_guarantee = None
c.Spawner.cpu_limit = None
c.Spawner.mem_guarantee = None
c.Spawner.mem_limit = None
```

##### 启动JupyterHub

以系统服务运行JupyterHub：

```ini
[Service]
Type=simple
Environment="PATH=/usr/local/miniconda3/envs/jupyterhub/bin:..."
ExecStart=/usr/local/miniconda3/envs/jupyterhub/bin/jupyterhub -f \
          /etc/jupyterhub/jupyterhub_config.py
TimeoutSec=10
ExecStop=/bin/kill -s TERM $MAINPID
StandardOutput=journal
StandardError=journal
SyslogIdentifier=JupyterHub
User=root
```

使用管理员权限运行JupyterHub可允许多用户登录。

> [Using sudo to run JupyterHub without root privileges · jupyterhub/jupyterhub Wiki (github.com)](https://github.com/jupyterhub/jupyterhub/wiki/Using-sudo-to-run-JupyterHub-without-root-privileges)

#### Jupyter Docker Stack

https://jupyter-docker-stacks.readthedocs.io/en/latest/

Docker image for Core Stacks：https://jupyter-docker-stacks.readthedocs.io/en/latest/using/selecting.html#core-stacks

> https://hub.docker.com/u/jupyter

[Using Docker — JupyterHub 2.2.2 documentation](https://jupyterhub.readthedocs.io/en/stable/quickstart-docker.html)

[Zero to JupyterHub with Kubernetes — Zero to JupyterHub with Kubernetes documentation (zero-to-jupyterhub.readthedocs.io)](https://zero-to-jupyterhub.readthedocs.io/en/latest/)

JupyterLab Documentation：https://jupyterlab.readthedocs.io/en/latest/



#### 问题

1. 终端启动配置问题，无法像登录终端一样初始化：[jupyter - How to login to bash with current user, and the user's .bashrc file, in jupyterlab? - Stack Overflow](https://stackoverflow.com/questions/60073276/how-to-login-to-bash-with-current-user-and-the-users-bashrc-file-in-jupyterl)

   ```python
   c.ServerApp.terminado_settings = {'shell_command': ['/bin/bash']}
   ```

   

### [PyCharm](https://www.runoob.com/w3cnote/pycharm-windows-install.html)

支持Windows、Linux和mac。

## 辅助工具

**pyreadline**：命令行自动补全。在Windows平台中，如果命令记录中包含中文，那么`pyreadline`在读取历史记录时采用平台相关的编码（`gbk`）会出错。可以将其源码中默认的编码方式改为`utf-8`。

```python
# xxx\envs\data\lib\site-packages\pyreadline\lineeditor\history.py
for line in open(filename, 'r', encoding='utf-8'):
```

## 参考文献

[^jupyterhub]:

[^vsctest]: [Testing Python in Visual Studio Code](https://code.visualstudio.com/docs/python/testing).

