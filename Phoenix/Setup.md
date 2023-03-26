Download the phoenix qcow from https://github.com/ExploitEducation/Phoenix/releases/download/v1.0.0-alpha-3/exploit-education-phoenix-amd64-v1.0.0-alpha-3.tar.xz

Extract it with 

```
tar xJf exploit-education-phoenix-amd64-v1.0.0-alpha-3.tar.xz`
```

Once extracted, make the bootscript executable with `chmod +x` and run it.

You can access the machine via ssh, creds are `user/user` and `root/root`
```
ssh user@localhost -p2222
```

The challenges are located in the `/opt/phoenix/amd64` directory.

## GDB GEF Fix

I've noticed when running GDB within the vm that it complains of python ascii errors:

```
Python Exception <class 'UnicodeEncodeError'> 'ascii' codec can't encode character '\u27a4' in position 12: ordinal not in range(128):
```

The fix for me was to run the following code at least once before gdb:

```
export LC_CTYPE=C.UTF-8
```

To make this persist, i added the line to .bashrc once I had connected via SSH.

```bash
echo "export LC_CTYPE=C.UTF-8" >> ~/.bashrc
```