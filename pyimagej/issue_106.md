`ij.op().create()` ambiguity with `PTService.create()`
===

When attempting to use/interact with `ij.op().create()` an overloading error is thrown. There is an ambiguity issue (bug?) where `JPype` gets confused between `PTService.create()` and what we want `ij.op().create()`.

```python
>>> import imagej
>>> ij = imagej.init()
>>> ij.op().threshold() # something that works
<java object 'net.imagej.ops.threshold.ThresholdNamespace'>
>>> ij.op().create()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: No matching overloads found for org.scijava.plugin.PTService.create(), options are:
	public default org.scijava.plugin.SciJavaPlugin org.scijava.plugin.PTService.create(java.lang.Class)

```

For now the workaround is to be explicit and import the create namespace manually.

```python
>>> import imagej
>>> import scyjava as sj
>>> ij = imagej.init()
>>> CreateNamespace = sj.jimport('net.imagej.ops.create.CreateNamespace')
>>> CreateNamespace
<java class 'net.imagej.ops.create.CreateNamespace'>
>>> dir(CreateNamespace)
['__class__', '__del__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'context', 'equals', 'getClass', 'getContext', 'getName', 'hashCode', 'img', 'imgFactory', 'imgLabeling', 'imgPlus', 'integerType', 'kernel', 'kernel2ndDerivBiGauss', 'kernelBiGauss', 'kernelDiffraction', 'kernelGabor', 'kernelGaborComplexDouble', 'kernelGaborComplexFloat', 'kernelGaborDouble', 'kernelGaborFloat', 'kernelGauss', 'kernelLog', 'kernelSobel', 'labelingMapping', 'nativeType', 'notify', 'notifyAll', 'object', 'ops', 'setContext', 'setEnvironment', 'setName', 'toString', 'wait']
```

Here's an example of how code should run (but doesn't because of this ambiguity issue) working with this workaround.

**Should work:**
```python
psf = ij.op().create().kernelDiffraction(psfSize, numericalAperture, wavelength, riSample, riImmersion, xySpacing, zSpacing, depth, FloatType())
```

**Workaround:**
```python
import scyjava as sj

# import the CreateNamespace
CreateNamespace = sj.jimport('net.imagej.ops.create.CreateNamespace')

psf = ij.op().namespace(CreateNamespace).kernelDiffraction(psfSize, numericalAperture, wavelength, riSample, riImmersion, xySpacing, zSpacing, depth, FloatType())
```

Note instead of using `ij.op().create()`, use `ij.op().namespace()` and pass it the CreateNamespace.