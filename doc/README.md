Bitcoin Classic

Setup
---------------------
[Bitcoin Classic](https://bitcoinclassic.com/) is a Bitcoin client and it builds the backbone of the network. However, it downloads and stores the entire history of Bitcoin transactions (which is currently several GBs); depending on the speed of your computer and network connection, the synchronization process can take anywhere from a few hours to a day or more.

### Need Help?

* See the documentation at the [Bitcoin Wiki](https://bitcoinfactswiki.github.io/)
for help and more information.
* Ask for help on [#bitcoin](http://webchat.freenode.net?channels=bitcoin) on Freenode. If you don't have an IRC client use [webchat here](http://webchat.freenode.net?channels=bitcoin).

Building
---------------------
The following are developer notes on how to build Bitcoin on your native platform. They are not complete guides, but include notes on the necessary libraries, compile flags, etc.

- [Gitian Building Guide](gitian-building.md)

Development
---------------------
The Bitcoin repo's [root README](/README.md) contains relevant information on the development process and automated testing.

- [Release Process](release-process.md)
- [Source Code Documentation (External Link)](https://dev.visucore.com/bitcoin/doxygen/)
- [Unit Tests](unit-tests.md)
- [Unauthenticated REST Interface](REST-interface.md)
- [Shared Libraries](shared-libraries.md)
- [Dnsseed Policy](dnsseed-policy.md)

### Miscellaneous
- [Files](files.md)
- [Tor Support](tor.md)
- [Init Scripts (systemd/upstart/openrc)](init.md)
