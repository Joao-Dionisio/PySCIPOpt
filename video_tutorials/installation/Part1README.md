# PySCIPOpt tutorial series Part 1: Installation and Basic Modeling

Note: the README's in these tutorials are written in Markdown. If you are using VSCode, you can press the default shortcut `Ctrl+Shift+V` to see the desired output. Other editors should have similar functionalities.

### Section 1: Installing PySCIPOpt

#### PyPI (pip)

The default way of installing PySCIPOpt is by using the command `pip install pyscipopt`. This will install the latest release of both SCIP and PySCIPOpt.

#### Anaconda

It is also possible to use Anaconda to install PySCIPOpt. You just need to make sure that you are not using the default environment. If you are not sure how to do this, follow [Anaconda's instructions](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#creating-an-environment-with-commands). After activating the non-base environment, the command `conda install --channel conda-forge pyscipopt` should do the trick.

#### Building from source

Instruction on building PySCIPOpt from source will be added on a later, when the part dedicated to contributing to PySCIPOpt is released. 

--

For more detailed information on the installation process, see the [Installation Guide](https://pyscipopt.readthedocs.io/en/latest/install.html) online.

### Section 2: Basic Modeling with PySCIPOpt

Let us create the model below

$$
\begin{align*}
\max 2x + 4y - \sum_{i \in {1, \dots, 10} iz_i} \\
\text{subject to} \quad & xy \leq 3 \\
&\sum_{i \in \lbrace 1, \dots, 10\rbrace} z_i \geq 20 \\

& x, z \geq 0\\
& y \in \lbrace 1, \dots, 5\rbrace

\end{align*}
$$

```python 
from pyscipopt import Model, quicksum

# create a new model
model = Model()

# create a simple variable
x = model.addVar()

# create a variable with some options
y = model.addVar(lb=1, ub=5, vtype="B", name="test")

# add a nonlinear constraint
model.addCons(model.addCons(x*y <= 3))

# add multiple indexed variables 
z = {}
for i in range(10):
    z[i] = model.addVar(lb=3, "z[%i]"%i)

# add a constraint using an iterator
model.addCons(quicksum(z[j] for j in range(10) >= 20))

# set a maximization objective
model.setObjective(2*x + 4*y - quicksum(i*z[i] for i in range(10)), sense="maximize")

# start the optimization process
model.optimize()
```

#### Quering the solution

To better understand SCIP's solving log, please refer to the [documentation](https://pyscipopt.readthedocs.io/en/latest/tutorials/logfile.html).

```python 
# get the optimal solution value 
print(model.getObjVal())

# get name of the variables and their value in the optimal solution 
for var in model.getVars():
    print(var.name, model.getVal(var))
``` 



#### Reading the output

Let us go over the output log to understand what is going on.

![title](Images/solver_output.png)


Presolving is a fundamental part of optimization, since it can greatly reduce the size of the problem

There is also the option to hide output by calling `model.hideOutput()` before optimizing.

#### FAQ



#### Feedback

These tutorials will benefit from your feedback. If you have any suggestions for this Part or for the tutorials in general, please let us know using the [discussions page](https://github.com/scipopt/PySCIPOpt/discussions) in the repo.




















#####################################################################
Introduction (Model Object, Solution Information, Parameter Settings)
#####################################################################


The ``Model`` object is the central Python object that you will interact with. To use the ``Model`` object
simply import it from the package directly.


`from pyscipopt import Model

model = Model()`

### Create a Model, Variables, and Constraints

While an empty Model is still something, we ultimately want a non-empty optimization problem. Let's
consider the basic optimization problem:

$$
    \begin{align*}
  &\min & \quad &2x + 3y -5z \\
  &\text{s.t.} & &x + y \leq 5\\
  & & &x+z \geq 3\\
  & & &y + z = 4\\
  & & &(x,y,z) \in \mathbb{R}_{\geq 0}
  \end{align*}
$$


We can construct the optimization problem as follows:

```python
  model = Model()
  x = model.addVar(vtype='C', lb=0, ub=None, name='x')
  y = model.addVar(vtype='C', lb=0, ub=None, name='y')
  z = model.addVar(vtype='C', lb=0, ub=None, name='z')
  cons_1 = model.addCons(x + y <= 5, name="cons_1")
  cons_1 = model.addCons(y + z >= 3, name="cons_2")
  cons_1 = model.addCons(x + y == 5, name="cons_3")
  model.setObjective(2 * x + 3 * y - 5 * z, sense="minimize")
  model.optimize()
``` 

That's it! We've built the optimization problem defined above and we've optimized it.
For how to read a Model from file see :doc:`this page </tutorials/readwrite>` and for best practices
on how to create more variables see :doc:`this page </tutorials/vartypes>`.

.. note:: ``vtype='C'`` here refers to a continuous variables.
  Providing the lb, ub was not necessary as they default to (0, None) for continuous variables.
  Providing the name attribute is not necessary either but is good practice.
  Providing the objective sense was also not necessary as it defaults to "minimize".

.. note:: An advantage of SCIP is that it can handle general non-linearities. See
  :doc:`this page </tutorials/expressions>` for more information on this.

### Query the Model for Solution Information

Now that we have successfully optimized our model, let's see some examples
of what information we can query. For example, the solving time, number of nodes,
optimal objective value, and the variable solution values in the optimal solution.

  solve_time = scip.getSolvingTime()
  num_nodes = scip.getNTotalNodes() # Note that getNNodes() is only the number of nodes for the current run (resets at restart)
  obj_val = scip.getObjVal()
  for scip_var in [x, y, z]:
      print(f"Variable {scip_var.name} has value {scip.getVal(scip_var)})

### Set / Get a Parameter

SCIP has an absolutely giant amount of parameters (see `here <https://www.scipopt.org/doc/html/PARAMETERS.php>`_).
There is one easily accessible function for setting individual parameters. For example,
if we want to set a time limit of 20s on the solving process then we would execute the following code:

  `scip.setParam("limits/time", 20)` 

To get the value of a parameter there is also one easily accessible function. For instance, we could
now check if the time limit has been set correctly with the following code.

  `time_limit = scip.getParam("limits/time")`

A user can set multiple parameters at once by creating a dictionary with keys corresponding to the
parameter names and values corresponding to the desired parameter values.

  `param_dict = {"limits/time": 20}
  scip.setParams(param_dict)`

To get the values of all parameters in a dictionary use the following command:

  `param_dict = scip.getParams()`

Finally, if you have a ``.set`` file (common for using SCIP via the command-line) that contains
all the parameter values that you wish to set, then one can use the command:

  `scip.readParams(path_to_file)`

### Copy a SCIP Model

A SCIP Model can also be copied. This can be done with the following logic:

  `alternate_model = Model(sourceModel=original_model) # Assuming original_model is a pyscipopt Model`

This model is completely independent from the source model. The data has been duplicated.
That is, calling ``model.optimize()`` at this point will have no effect on ``alternate_model``.

After optimizing users often struggle with reoptimization. To make changes to an
  already optimized model, one must first fo the following:

`model.freeTransform()`

  Without calling this function the Model can only be queried in its post optimized state.
  This is because the transformed problem and all the previous solving information
  is not automatically deleted, and thus stops a new optimization call.

To completely remove the SCIP model from memory use the following command:

`model.freeProb()`

  This command is potentially useful if there are memory concerns and one is creating a large amount
  of different SCIP models.



