WSL
-------------------------

开启WSL
###############

powershell 管理员


.. code-block::

    dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

    dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

或者通过开启windows功能: hyper-v，适用于linux的windows子系统

 
查看版本 
###############

.. code-block::

    wsl -l -o 

安装 
###############


.. code-block::

    wsl --install Ubuntu-20.04  

或者在windows store里安装


更新到 pre-release 版本
########################

在 PowerShell 中执行：


.. code-block:: powershell

    wsl --update --pre-release

    wsl --version


使用 wsl move 迁移到 D 盘
############################

较新版本的 WSL 可以直接移动发行版，无需先导出再导入。先通过 ``wsl -l -v``
确认发行版名称，然后在 PowerShell 中执行：


.. code-block:: powershell

    wsl --shutdown

    wsl --manage Ubuntu-20.04 --move D:\software\vm\ubuntu

如果当前 WSL 不支持 ``--move``，可先执行 ``wsl --update`` 更新 WSL，或使用下面的
导出、导入方式迁移。


使用导出、导入迁移到 D 盘
############################


.. code-block::

    wsl --shutdown

    wsl --export Ubuntu-20.04 d:\ub.tar

    wsl --unregister Ubuntu-20.04 

    wsl --import Ubuntu-20.04 d:\software\vm\ubuntu\  d:\ub.tar   

压缩wsl镜像
###############


.. code-block::

    wsl --shutdown

    diskpart

    DISKPART> select vdisk file = "xxxx\ext4.vhdx"

    DISKPART> compact vdisk
