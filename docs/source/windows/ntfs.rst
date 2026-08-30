ntfs
======

磁盘空间修复
---------------

.. code-block::bash

        diskpart
        list volume
        select volume C
        extend filesystem
        exit
