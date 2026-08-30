rust
=======

old version
---------------


rustup toolchain install 1.56.0

rustup default 1.56.0

cargo proxy
------------------

~/.cargo/config.toml

.. code-block::

        [http]

        proxy = "socks5h://127.0.0.1:8080" 

        timeout = 30        

        check-revoke = false 

        multiplexing = false 
