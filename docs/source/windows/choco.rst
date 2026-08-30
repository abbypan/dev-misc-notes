choco
=======


.. code-block::

        Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

        ./install.ps1

        mv c:\programdata\chocolatey d:\software\chocolatey

        mklink /J c:\programdata\chocolatey d:\software\chocolatey

        choco config set --name="'proxy'" --value="'http://localhost:7070'"

        choco install firefox -y
        choco install vim -y
        choco install freecommander-xe.install -y
        choco install kitty -y
        choco install googlechrome -y
        choco install adb -y
        choco install 7zip -y
        choco install drawio -y

