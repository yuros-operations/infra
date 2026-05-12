# lvresized Documentation
## check your available disk space in volume group
```
sudo vgs
```
## lvresized
```
sudo lvresize -r -L +20G /dev/vgdata/lvroot
```
> Use the + sign if you want to increase the size. Otherwise, do not include it, because the size will be changed directly to the specified disk size.
