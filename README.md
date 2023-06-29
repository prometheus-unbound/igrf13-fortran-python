# igrf13-fortran-python

## About

This repository contains Fortran code for generating geomagnetic field values from the Interational Geomagnetic Reference Field IGRF13, released in 2019. The code is modified from its orignal source: 

[noaa.gov/IAGA/vmod/igrf13.f](https://www.ngdc.noaa.gov/IAGA/vmod/igrf13.f)https://www.ngdc.noaa.gov/IAGA/vmod/igrf13.f

so that it can be compiled by `numpy.f2py` and used in Python.

## Modifications

The modifications to the original code involve implicit variable declarations, i.e.:

``` fortran
      implicit double precision (a-h,o-z)
```

and instead make key variables explicit:

``` fortran
      integer,intent(in)::isv
      double precision,intent(in)::date
      integer,intent(in)::itype
      double precision,intent(in)::alt
      double precision,intent(in)::colat
      double precision,intent(in)::elong
      double precision,intent(out)::x
      double precision,intent(out)::y
      double precision,intent(out)::z
      double precision,intent(out)::f
```

The reason for this is to enable `F2PY`, the Fortran to Python interface generator, to compile Fortran code for use in Python. By making variables explicit, we give the compiler the ability to determine which variables are inputs versus outputs. It is not able to do this otherwise:

``` python
print(igrf13.igrf13syn.__doc__) # verify compiled file gives correct inputs and outputs
```

should produce the following if compiled correctly (see following section on how to compile):


``` python
x,y,z,f = igrf13syn(isv,date,itype,alt,colat,elong)

Wrapper for ``igrf13syn``.

Parameters
----------
isv : input int
date : input float
itype : input int
alt : input float
colat : input float
elong : input float

Returns
-------
x : float
y : float
z : float
f : float
```

## How to compile and use
Clone this repository and put the `igrf13.f` in a place that is accessible. Provided that you have `numpy` (conda/pip install if you do not), run the following command to compile the Fortran code for your system architecture (use `f2py3` instead of `f2py` if you are running this for Python3 and not Python2):

``` python
from numpy import f2py
with open("path/to/file/igrf13.f") as sourcefile:
    sourcecode = sourcefile.read()
f2py.compile(sourcecode, modulename='igrf13')

```

`F2PY` will create an `ìgrf13.so` file in the working directory where you launched Python. If in doubt, run `!pwd` (note that !pwd works only in jupyterlab/notebook) to locate where that is. Go to that directory and create a folder called `igrf13` and move `ìgrf13.so` into that folder. In addition, create an empty file called `__init__.py` in that created folder. This can be summarized as such:

```
cd /path/given/by/!pwd
mkdir igrf13
mv igrf13.so igrf13
cd igrf13
touch __init.py__
```

Once this is done, you should now be able to import the `igrf13` module:

``` python
from igrf13 import igrf13
```

And now you should be able to perform such computations:

```python
colat = 90 - mag_data.lat_wgs84
elong = (360 + mag_data.long_wgs84) % 360

isv = 0
date = 1981 + 2.0/12
itype = 1
alt = 400 * 0.3048 / 1000

x = np.empty_like(elong)
y = np.empty_like(elong)
z = np.empty_like(elong)
f = np.empty_like(elong)
intensity = np.empty_like(elong)
I = np.empty_like(elong)
D = np.empty_like(elong)

for i in np.arange(colat.shape[0]):
    x[i], y[i], z[i], f[i] = igrf13.igrf13syn(isv, date, itype, alt, colat[i], elong[i])
    intensity[i], I[i], D[i] = v2a([x[i], y[i], z[i]])
```

As mentioned earlier you can verify if things have compiled correctly by confirming if the compiler has corrected seperated the subroutines and the input and output parameters via:

```python
print(igrf13.igrf13syn.__doc__)
```

## Miscellaneous
This repository also contains an already compiled .so file (`igrf13.so`); which would work for those who are using Python 2.7 and Macbook Pro 2019 (Intel) architecture. It is recommended however that you compile your own version. I suppose most people are using some version of Python3 nowawadays.

For MacOS users, make sure your system is up to date, and that your Clang and GFortran are compatible with each other by running `gfortran -v` and `clang -v` to check if nothing is out of date. If not (and you are running into issues where error codes are mentioning gcc, c.lang., llvm, etc.), it is recommended to run:

```
sudo rm -rf /Library/Developer/CommandLineTools
sudo xcode-select --install
````

as noted by ```brew upgrade```.
