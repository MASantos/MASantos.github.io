# TensorFlow: Installation

Wed. Dec 2nd 2015

## Overview 

I describe here briefly my attempts to install [TensorFlow][http://www.tensorflow.org/get_started/index.html] in my OSX 10.6.8 (Snow Leopard).

Direct installation within the system's python distribution risks disrupting it, and, without knowing yet if we'll get succeed running it, it 
does not seem reasonable. The third option suggested on that site is via Docker. However, Docker is not available for 10.7 and earlier. The only
option left is via VirtualEnv.

Alternatively, we can setup a VM with ubuntu and install test the Linux distribution. As it turned out, this was the only option that really worked.

## Installing via virtualenv

The installation of the python package worked w/o problems:
```
msantos@MBP-2[18:13]:~/DATASCIENCE$pip install --upgrade virtualenv
/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/site-packages/pip/_vendor/requests/packages/urllib3/util/ssl_.py:90: InsecurePlatformWarning: A true SSLContext object is not available. This prevents urllib3 from configuring SSL appropriately and may cause certain SSL connections to fail. For more information, see https://urllib3.readthedocs.org/en/latest/security.html#insecureplatformwarning.
  InsecurePlatformWarning
You are using pip version 7.1.0, however version 7.1.2 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
Collecting virtualenv
/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/site-packages/pip/_vendor/requests/packages/urllib3/util/ssl_.py:90: InsecurePlatformWarning: A true SSLContext object is not available. This prevents urllib3 from configuring SSL appropriately and may cause certain SSL connections to fail. For more information, see https://urllib3.readthedocs.org/en/latest/security.html#insecureplatformwarning.
  InsecurePlatformWarning
  Downloading virtualenv-13.1.2-py2.py3-none-any.whl (1.7MB)
    100% |████████████████████████████████| 1.7MB 124kB/s
Installing collected packages: virtualenv
Successfully installed virtualenv-13.1.2
```

The second step is intalling TensorFlow and numpy within a virtualenv. This worked as well w/o any problem:
```
msantos@MBP-2[18:13]:~/DATASCIENCE$virtualenv --system-site-packages ~/DATASCIENCE/tensorflow
New python executable in /Users/msantos/DATASCIENCE/tensorflow/bin/python
Installing setuptools, pip, wheel...done.
msantos@MBP-2[18:13]:~/DATASCIENCE$source tensorflow/bin/activate
(tensorflow)msantos@MBP-2[18:14]:~/DATASCIENCE$pip install --upgrade https://storage.googleapis.com/tensorflow/mac/tensorflow-0.5.0-py2-none-any.whl
Collecting tensorflow==0.5.0 from https://storage.googleapis.com/tensorflow/mac/tensorflow-0.5.0-py2-none-any.whl
/Users/msantos/DATASCIENCE/tensorflow/lib/python2.7/site-packages/pip/_vendor/requests/packages/urllib3/util/ssl_.py:90: InsecurePlatformWarning: A true SSLContext object is not available. This prevents urllib3 from configuring SSL appropriately and may cause certain SSL connections to fail. For more information, see https://urllib3.readthedocs.org/en/latest/security.html#insecureplatformwarning.
  InsecurePlatformWarning
  Downloading https://storage.googleapis.com/tensorflow/mac/tensorflow-0.5.0-py2-none-any.whl (9.8MB)
    100% |████████████████████████████████| 9.8MB 26kB/s
Collecting six>=1.10.0 (from tensorflow==0.5.0)
/Users/msantos/DATASCIENCE/tensorflow/lib/python2.7/site-packages/pip/_vendor/requests/packages/urllib3/util/ssl_.py:90: InsecurePlatformWarning: A true SSLContext object is not available. This prevents urllib3 from configuring SSL appropriately and may cause certain SSL connections to fail. For more information, see https://urllib3.readthedocs.org/en/latest/security.html#insecureplatformwarning.
  InsecurePlatformWarning
  Downloading six-1.10.0-py2.py3-none-any.whl
Collecting numpy>=1.9.2 (from tensorflow==0.5.0)
  Downloading numpy-1.10.1-cp27-none-macosx_10_6_intel.macosx_10_9_intel.macosx_10_9_x86_64.macosx_10_10_intel.macosx_10_10_x86_64.whl (3.7MB)
    100% |████████████████████████████████| 3.7MB 75kB/s
Installing collected packages: six, numpy, tensorflow
  Found existing installation: six 1.9.0
    Not uninstalling six at /Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/site-packages, outside environment /Users/msantos/DATASCIENCE/tensorflow/bin/..
  Found existing installation: numpy 1.8.0
    Not uninstalling numpy at /Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/site-packages, outside environment /Users/msantos/DATASCIENCE/tensorflow/bin/..
Successfully installed numpy-1.10.1 six-1.10.0 tensorflow-0.5.0
(tensorflow)msantos@MBP-2[18:14]:~/DATASCIENCE$
```
Notice the change in the prompt: we are already within the tensorflow env. Runny python from here will be the way to access TF API. 
The first step will always be import tensorflow, e.g., 'import tensorflow as tf'. Unfortunately, this is the step where we hit the wall:
```
(tensorflow)msantos@MBP-2[18:14]:~/DATASCIENCE$python
Python 2.7.6 (v2.7.6:3a1db0d2747e, Nov 10 2013, 00:42:54)
[GCC 4.2.1 (Apple Inc. build 5666) (dot 3)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import tensorflow as tf
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/Users/msantos/DATASCIENCE/tensorflow/lib/python2.7/site-packages/tensorflow/__init__.py", line 4, in <module>
    from tensorflow.python import *
  File "/Users/msantos/DATASCIENCE/tensorflow/lib/python2.7/site-packages/tensorflow/python/__init__.py", line 22, in <module>
    from tensorflow.python.client.client_lib import *
  File "/Users/msantos/DATASCIENCE/tensorflow/lib/python2.7/site-packages/tensorflow/python/client/client_lib.py", line 35, in <module>
    from tensorflow.python.client.session import InteractiveSession
  File "/Users/msantos/DATASCIENCE/tensorflow/lib/python2.7/site-packages/tensorflow/python/client/session.py", line 11, in <module>
    from tensorflow.python import pywrap_tensorflow as tf_session
  File "/Users/msantos/DATASCIENCE/tensorflow/lib/python2.7/site-packages/tensorflow/python/pywrap_tensorflow.py", line 28, in <module>
    _pywrap_tensorflow = swig_import_helper()
  File "/Users/msantos/DATASCIENCE/tensorflow/lib/python2.7/site-packages/tensorflow/python/pywrap_tensorflow.py", line 24, in swig_import_helper
    _mod = imp.load_module('_pywrap_tensorflow', fp, pathname, description)
ImportError: dlopen(/Users/msantos/DATASCIENCE/tensorflow/lib/python2.7/site-packages/tensorflow/python/_pywrap_tensorflow.so, 2): Library not loaded: /usr/lib/libc++.1.dylib
  Referenced from: /Users/msantos/DATASCIENCE/tensorflow/lib/python2.7/site-packages/tensorflow/python/_pywrap_tensorflow.so
  Reason: image not found
>>>
```
Chances to possible compile and locally install the new library libc++ for my Snow Leopard seem slim. Going for the Linux approach looks much more promising.

## Installing on a VirtualBox VM with Ubuntu 14.03 server

Only worth mentioning is that I selected the ssh server  as the only extra package when given the options of different servers to install. And that is more than enough
for spending well over an hour: It had many packages to upgrade and even the compilation of numpy and tensorflow took together way longer than the installation in OSX.

The latter also showed compilation warnings gallore...But in the end it looks that all went well.

## Conclusion

I'll be working within the ubuntu VM at the [deep learning meetup][http://www.meetup.com/Deep-Learning-Toronto-Meetup/events/227021984/] tomorrow morning.

So far the simple tests I tried (see TF's website) worked flawlessly. 

The idea of a graph of operations and a flow of data (as a multi-dimensional array, or tensor) is appealing and easy to grasp. Syntactically feels a bit odd
though as instead of writing (a+b) for two tensors a and b, we have to use the method .add(a,b). 

I think, however, that this may be a simple case of having an overview of the API and getting a sense of how basic operations are defined.

Overall the impression is good: Seems easy to get running and start thinking on implementing some code. There is even the option for an interactive which can
be used in conjuction with iPython (and possibly notebook I guess).

I go through a quick tutorial in the next post. I may edit this one later to add the basic tests as well as the interactive mode.
 




 
