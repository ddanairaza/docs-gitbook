Since the current versions of the Desktop Wallet are not backward compatible, it is required to remove the files manually before upgrading to the next major version.

Points to mention:
* On Mac and Windows the wallet files reside in directories which are not shown by default, yet they can be accessed via a command line interface (see the examples below).
* On Mac and Windows the wallet files reside (by default) in a home directory of the user. In the examples below the user name is `alice` on all the platforms.
* The suggested commands will unconditionally remove the files, an action which cannot be undone.

### On Windows
* Push windows button+R, the Run window will appear. Type cmd and click OK.
* In the window that appears, paste the following command and click enter:

``` sh
RD /s /q "C:\Users\alice\AppData\Local\Beam Wallet"
```

### On Mac
* Open the Terminal app, paste the following command and click enter:

``` sh
rm -R "/Users/alice/Library/Application Support/Beam Wallet"
```

### On Linux
* Run the following command in terminal:

``` sh
rm -R ".local/share/Beam Wallet"
```
