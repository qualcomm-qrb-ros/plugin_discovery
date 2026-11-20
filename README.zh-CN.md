# plugin_discovery platform 配置

[English](README.md) | **中文**

[plugin_discovery](https://github.com/qualcomm-qrb-ros/plugin_discovery) 的 platform 层配置:每台机器、每种 OS、每个域，Qualcomm 完全支持哪些 backend。它放在 `platform-config` 分支，和库分开，这样配置可以单独修改和发布，不用动库。

## 目录

```
<域>/<machine>-<os>-<域>.config.yaml     例如 camera/QAM8797P-QCLinux-camera.config.yaml
```

- `<machine>`:`/sys/devices/soc0/machine` 里的字符串。`<os>`:`Ubuntu` 或 `QCLinux`。名字比较时不区分大小写。
- 文件名里的域必须等于它所在的目录，文件里也必须有这个域的配置块，否则构建失败。
- 每个域的目录归该域的团队(见 `.github/CODEOWNERS`)：他们通过 pull request 增加机型、修改自己的 backend 列表。

## 包

一个 Debian 包 `plugin-discovery-platform-config`(架构 `all`)，把所有文件装到 `/usr/share/plugin_discovery/platforms.d/`。第一次开机时 `platform-detect` 把匹配机器和 OS 的文件链接到 `/etc/plugin_discovery/platform.d/`；包的 postinst 和 postrm 会运行 `platform-detect --force`，所以安装或卸载这个包都会刷新链接。

```bash
cmake -S . -B build && cd build && cpack -G DEB      # plugin-discovery-platform-config_<version>_all.deb
sudo dpkg -i plugin-discovery-platform-config_*_all.deb
pluginlib-detect -d camera                           # 合并后的结果
```

**Qualcomm Linux** 的根文件系统只读，且一个镜像只对应一个机型，所以不使用这个包和 `platform-detect`:镜像的 recipe 直接把所属机型的候选文件装成 `/etc/plugin_discovery/platform.d/<域>.config.yaml`。这个 recipe 在镜像 layer(`meta-qcom-robotics-sdk`)里，不在这里。

客户配置不在这个包里：它由客户自己的包装到 `/etc/plugin_discovery/config.d/`。

## 许可证

BSD-3-Clause-Clear，见 [`LICENSE`](LICENSE)。
