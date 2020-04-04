# OpenGM quickstart

In this document, we apply the loopy belief propagation algorithm of OpenGM to a simple example. If you have not installed OpenGM (and its python interface), consult [this](install_opengm_conda.md) document first. 

## Defining the probabilistic model

Our first task is to define a probability model consisting of four binary variables in OpenGM. 

```python
import opengm
gm = opengm.gm([2, 2, 2, 2], operator='multiplier')
```

The first argument `[2, 2, 2, 2]` declares that the model consists of four binary variables. The second argument declares that the factors are *multiplied* together to construct the distribution. 

### Structure of the model

Our example model is represented by this Bayesian network:


<p align="center">
<img src="images/opengm_quickstart_bn.png" height="200px" >
</p>

As you know, this graph indicates that the distribution factorizes as follows:

<img src="images/opengm_quickstart_factorization.png" style="height:75%;width:75%;">

which can also be represented by the following factor graph:

<p align="center">
<img src="images/opengm_quickstart_fg.png" height="300px" >
</p>



### Parameters of the model

Next, we should define the values in our tabular factors (which in this example are conditional probabilities). We store these tables as `numpy` arrays. 


| X0   | f0   |
| ---- | ---- |
| 0    | 0.3  |
| 1    | 0.7  |

```python
import numpy 
f0_params = numpy.array([0.3, 0.7])
```

| X0   | X1   | f1   |
| ---- | ---- | ---- |
| 0    | 0    | 0.2  |
| 0    | 1    | 0.8  |
| 1    | 0    | 0.7  |
| 1    | 1    | 0.3  |

```python
f1_params = numpy.array([[0.2, 0.8],
                         [0.7, 0.3]])
```

| X0   | X2   | f2   |
| ---- | ---- | ---- |
| 0    | 0    | 0.4  |
| 0    | 1    | 0.6  |
| 1    | 0    | 0.3  |
| 1    | 1    | 0.7  |

```python
f2_params = numpy.array([[0.4, 0.6],
                         [0.3, 0.7]])
```
| X1   | X2   | X3   | f3   |
| ---- | ---- | ---- | ---- |
| 0    | 0    | 0    | 0.2  |
| 0    | 0    | 1    | 0.8  |
| 0    | 1    | 0    | 0.3  |
| 0    | 1    | 1    | 0.7  |
| 1    | 0    | 0    | 0.6  |
| 1    | 0    | 1    | 0.4  |
| 1    | 1    | 0    | 0.7  |
| 1    | 1    | 1    | 0.3  |

```python
f3_params = numpy.array([
    [[0.2, 0.8],
     [0.3, 0.7]],
    [[0.6, 0.4],
     [0.7, 0.3]]])
```

Finally, we define the factors of the model:

```python
gm.addFactor(gm.addFunction(f0_params),[0])
gm.addFactor(gm.addFunction(f1_params),[0, 1])
gm.addFactor(gm.addFunction(f2_params),[0, 2])
gm.addFactor(gm.addFunction(f3_params), [1, 2, 3])
```

The first argument of `addFactor` is the array containing the factor parameters, and the second argument is the indices of variables included in that factor. 