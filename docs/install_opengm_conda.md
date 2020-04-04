# installing OpenGM (with python interface)



This note explains the steps for installing the OpenGM and its python interface using the package management system *conda*. This is not the only way to install the python interface of OpenGM, but one of the fastest ways. For alternative installation methods, see the later sections.  

If you don't have conda, first install [*Miniconda*](https://conda.io/en/latest/miniconda.html). 

The python interface of OpenGM uses python 2. It also depends on certain versions of some python libraries. We will create an environment to isolate these from other existing python installations and libraries in our system:

```
$ conda create --name opengm-env --yes
```

Next, activate the environment:

```
$ source activate opengm-env
```

First install *hdf5* version 1.8.14:

```
(opengm-env)$ conda install -c anaconda hdf5=1.8.14 --yes
```

and then install openGM using this command:

```
(opengm-env)$ conda install -c ilastik/label/backup opengm --yes
```

If this doesn't work out because some libraries are missing, you can install them just like we installed *hdf5*. 

I also installed jupyter notebook:

```
(opengm-env)$ conda install jupyter  --yes
```



In order to test the installation, run the following piece of code:

```python
import opengm
import numpy

shape = [20, 20]
nl = 100
unaries = numpy.random.rand(*shape+[nl])
potts = opengm.PottsFunction([nl]*2,0.0,0.4)
gm = opengm.grid2d2Order(unaries=unaries,regularizer=potts)


p = opengm.InfParam(steps=10,
                    damping=0.5,
                    convergenceBound=0.001)
inf = opengm.inference.BeliefPropagation(gm, parameter=p)

inf.infer()

print inf.value()
```

This should print a number close to 200. 



You can deactivate the environment by running this command:

```
(opengm-env)$ source deactivate
```



## Alternative installation methods

TODO: using the docker image

TODO: using the packages of the distribution

TODO: build from source