# installing pomegranate (with python interface)

This note explains the steps for installing pomegranate using the package management system *conda*. This is not the only way to install pomegranate, but one of the fastest ways.  

If you don't have conda, first install [*Miniconda*](https://conda.io/en/latest/miniconda.html). 

We will create an environment to isolate these from other existing python installations and libraries in our system:

```
$ conda create --name pomegranate-env --yes
```

Next, activate the environment:

```
$ source activate pomegranate-env
```

First install *hdf5* version 1.8.14:

```
(pomegranate-env)$ conda install -c anaconda hdf5=1.8.14 --yes
```

and then install pomegranate using this command:

```
(pomegranate-env)$ conda install pomegranate
```

If this doesn't work out because some libraries are missing, you can install them just like we installed *hdf5*. 

You can deactivate the environment by running this command:

```
(pomegranate-env)$ source deactivate
```

TODO: Docker Image
