# Install prerequisite Files

```bash
sudo apt install freeradius-python3 -y
ldd /usr/lib/freeradius/rlm_python3.so # Identify python version used to build the module mine is python3.10

git clone https://github.com/Manawyrm/freeradius-osmohlr-gsup.git
cd freeradius-osmohlr-gsup/
python3.10 -m pip install build
sudo -H python3.10 -m pip install build
sudo -H python3.10 -m  build
sudo -H python3.10 -m pip install .
sudo mkdir /etc/freeradius/3.0/scripts
sudo cp -r freeradius_osmohlr_gsup /etc/freeradius/3.0/scripts

# edit the file /etc/freeradius/3.0/sites-enabled/default
edit the file /etc/freeradius/3.0/mods-available/python3
add the python path as: 
python_path="/usr/bin/python3.10:/usr/lib/python310.zip:/usr/lib/python3.10:/usr/lib/python3.10/lib-dynload:/home/uwelekezo/.local/lib/python3.10/site-packages:/usr/local/lib/python3.10/dist-packages:/usr/lib/python3/dist-packages:/usr/lib/python3.10/dist-packages:/etc/freeradius/3.0/scripts"

module = freeradius-osmohlr-gsup


finally link the python3 to mods-enabled
sudo ln -s /etc/freeradius/3.0/mods-available/python3 /etc/freeradius/3.0/mods-enabled/python3

```


# Get the python path from system You'll need the paths to be referenced in /etc/freeradius/3.0/mods-available/python3 .

### enumerate python version path
```bash
root@build:/etc/freeradius/3.0/mods-enabled# /usr/bin/python3.9 -c "import sys; print(sys.executable); print(sys.path)"



/usr/bin/python3.9  ['', '/usr/lib/python39.zip', '/usr/lib/python3.9', '/usr/lib/python3.9/lib-dynload', '/usr/local/lib/python3.9/dist-packages', '/usr/lib/python3/dist-packages']
```
root@build:/etc/freeradius/3.0/mods-enabled# ln -s ../mods-available/python3 ./


immunity@build:~$ which python3.9
/usr/bin/python3.9

#### Set /etc/freeradius/3.0/mods-enabled/python3 to the following
```
 python_path="/usr/lib/python39.zip:/usr/bin/python3.9:/etc/freeradius/3.0/scripts:/usr/lib/python3.9/lib-dynload:/usr/local/lib/python3.9/dist-packages:/usr/lib/python3/dist-packages"
```


############### install python3 support module from the apt store
 sudo apt install freeradius-python3 -y


# test
freeradius -X

## When you come across the following error :
```
 # Instantiating module "gsup" from file /etc/freeradius/3.0/mods-enabled/gsup
Fatal Python error: drop_gil: GIL is not locked
Python runtime state: initialized

Current thread 0x00007fed5c444340 (most recent call first):
<no Python frame>
Aborted (core dumped)
```
This means version mismatch e.g. freeradius was compiled for example with version 3.8 (rlm_python3.so) and the GSUP wheel you have installed was compiled with a different version e.g. python3.9

You can confirm this as follows: 
```
/etc/freeradius/3.0# ldd /usr/lib/freeradius/rlm_python3.so
        linux-vdso.so.1 (0x00007fff3498b000)
        libpython3.8.so.1.0 => /lib/x86_64-linux-gnu/libpython3.8.so.1.0 (0x00007f9c9404f000)
---- Version 3.8


        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f9c93e5d000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f9c945bc000)
        libexpat.so.1 => /lib/x86_64-linux-gnu/libexpat.so.1 (0x00007f9c93e2f000)
        libz.so.1 => /lib/x86_64-linux-gnu/libz.so.1 (0x00007f9c93e13000)
        libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f9c93df0000)
        libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f9c93dea000)
        libutil.so.1 => /lib/x86_64-linux-gnu/libutil.so.1 (0x00007f9c93de3000)
        libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f9c93c94000)
```
---------------- issue - When compiling module using python3.8 there is a missing dependency python module "
```
ERROR: Could not find a version that satisfies the requirement pyosmocom>=0.0.1 (from freeradius-osmohlr-gsup==0.0.1) (from versions: none)
ERROR: No matching distribution found for pyosmocom>=0.0.1 (from freeradius-osmohlr-gsup==0.0.1)
```
The pyosmocom>=0.0.1 module is available on version python3.9 and above not available on version python3.8

The module must be compiled with same system version of python else radius server will "Core Dump from GIL locking from version mismatch" 



