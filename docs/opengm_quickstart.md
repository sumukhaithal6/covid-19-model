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



## Setting evidence

What will happen to our model if we add the evidence *X2 = 1* ? This will change into one which does not contain this variable, and its factor are reduced to the rows which are consistent with the evidence:

```python
gm2, vs = gm.fixVariables([2], [1])
```

The first argument is the list of evidence variables, and the second argument is their values. `vs` is a list that maps the variable indices in the new (reduced) model to the old one:

```python
print(vs)
```
```
[0 1 3]
```

We can inspect the factors of the new model, too:
```python
from __future__ import print_function
for f in gm2.factors():
    print(f.variableIndices, f.copyValues())
```

Take some time to inspect how these factors are produced by reducing the original factors according to the evidence. Remember that variable with index 2 in these factors is the one with index 3 in the original model (see the `vs` list):
```
[0, ] [ 0.3  0.7]
[0, 1, ] [ 0.2  0.7  0.8  0.3]
[0, ] [ 0.6  0.7]
[1, 2, ] [ 0.3  0.7  0.7  0.3]
[] [ 1.]
```
The last factor in this list is a dummy factor. 



## Calculating the marginals

Let us now set the evidence *X_3 = 1* and calculate the marginals of other variables subject to this evidence. We will use the belief propagation algorithm for this purpose. First set the evidence:

```python
gm3, vs3 = gm.fixVariables([3], [1])
```

Then we set up the inference algorithm:

```python
inference_parameters = opengm.InfParam(
    steps=10, 
    damping=0.5,
    convergenceBound=0.001)

inference_algorithm = opengm.inference.BeliefPropagation(
    gm3,
    accumulator='integrator',
    parameter=inference_parameters)
```

By setting the value `integrator` value for the `accumulator` argument, we specify that we want the algorithm to compute the *marginals*. 

<font color="red">
TODO: Check and explain the other parameters of the belief propagation algorithm
</font>

Finally, we run the inference algorithm:

```python
inference_algorithm.infer()
```
Once the inference step is performed, we can get the marginals:

```python
for i in vs3:
    print(inference_algorithm.marginals(i))
```
```
[[ 0.22971516  0.77028484]]
[[ 0.72716989  0.27283011]]
[[ 0.36386238  0.63613762]]
```



## Monitoring the progress of inference algorithm

Sometimes we want to track certain values as the inference algorithm progresses. For example, let us print the marginals of one of the variables in each iteration of the belief propagation algorithm. We will do this using a call-back mechanism:

```python
class PyCallback(object):
    def __init__(self):
        pass
    def begin(self, inference):
        print("begin")
    def end(self, inference):
        print("end")
    def visit(self, inference):
        print("marginals of variable 2: ", inference.marginals(2))

callback=PyCallback()        
visitor=inference_algorith.pythonVisitor(callback,visitNth=1)
inference_algorithm.infer(visitor)
```

The output will be something similar to this:

```
begin
marginals of variable 2:  [[ 0.43992727  0.56007273]]
marginals of variable 2:  [[ 0.40775119  0.59224881]]
marginals of variable 2:  [[ 0.39028824  0.60971176]]
marginals of variable 2:  [[ 0.3803642  0.6196358]]
marginals of variable 2:  [[ 0.37435229  0.62564771]]
marginals of variable 2:  [[ 0.37047283  0.62952717]]
marginals of variable 2:  [[ 0.36784895  0.63215105]]
marginals of variable 2:  [[ 0.36602756  0.63397244]]
marginals of variable 2:  [[ 0.36475245  0.63524755]]
marginals of variable 2:  [[ 0.36386238  0.63613762]]
end
```



The code blocks of this guide are available in [this](opengm_quickstart_notebook.ipynb) jupyter notebook. 

