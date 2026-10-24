Audio
=======


Intel 11 代 CPU 的 OED 黄色感叹号与无声问题
------------------------------------------------

适用场景
########

以 ThinkPad 为例，重新安装 Windows 后可能出现以下现象：

- 安装 ThinkPad 官方 Audio 驱动后，设备管理器中出现
  ``High Definition Audio`` 设备；
- Intel Smart Sound Technology OED 设备显示黄色感叹号；
- 系统找不到可用的声音输出设备，扬声器没有声音。

处理步骤
########

1. 前往 Lenovo 支持页面，根据 ThinkPad 的准确型号和 Windows 版本安装官方
   Audio 驱动。安装后如果仍然显示 ``High Definition Audio``、OED 黄色感叹号且
   系统无声，继续执行以下步骤。

2. 安装与机型匹配的 ``Realtek High Definition Audio (SST)`` 驱动。可参考以下
   驱动页面，但实际安装时应优先使用当前 ThinkPad 型号对应的官方版本：

   - `Realtek Audio 驱动 DRV202102190003（Audio-n32a123w.exe） <https://think.lenovo.com.cn/support/driver/driverdetail.aspx?DEditid=89528&driverID=undefined&treeid=3124500>`_
   - `Intel Smart Sound Technology (SST) Driver for Windows 10 64-bit <https://support.lenovo.com/us/en/downloads/ds503776>`_

3. 打开设备管理器，手动将 ``High Definition Audio`` 设备切换到刚安装的
   Realtek SST 驱动：

   #. 选中 ``High Definition Audio`` 设备，右键选择“更新驱动程序”；
   #. 选择“浏览我的电脑以查找驱动程序”；
   #. 选择“让我从计算机上的可用驱动程序列表中选取”；
   #. 在兼容硬件列表中选择 ``Realtek High Definition Audio (SST)``；
   #. 完成安装并按提示重新启动 Windows。

4. 重启后检查任务栏声音图标和“设置 → 系统 → 声音”。正常情况下扬声器输出设备
   会重新出现，声音恢复。

.. note::

   不同 ThinkPad 型号显示的设备名称可能略有不同。不要安装其他机型或其他 Windows
   版本的驱动；如果兼容列表中没有 Realtek SST，先确认第 2 步的驱动已经正确解压或
   安装，再重新打开设备管理器。

参考
####

`Intel 11 代 CPU 音频驱动问题处理参考 <https://www.bilibili.com/opus/497993911719182966?spm_id_from=333.1387.0.0>`_
