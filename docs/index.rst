vtkInterface Overview
=====================
``vtkInterface`` is a Python module that simplifies the interface with VTK by using numpy and direct array access and more general classes to work with meshes and plotting.  It's designed to make working with VTK more pythonic and more straightforward.

This module is suited creating engineering plots for presentations and research papers as well as being a supporting module for other mesh dependent Python modules that would like to simplify hundreds of lines of code into just a few lines.


Installation
------------
If you have a working copy of VTK, installation is simply::

    $ pip install vtkInterface
    
You can also visit `PyPi <http://pypi.python.org/pypi/vtkInterface>`_ or `GitHub <https://github.com/akaszynski/vtkInterface>`_ to download the source.

See :ref:`install_ref` for more details.


Why?
----
VTK is an excellent visualization toolkit, and with Python bindings it should be able to combine the speed of C++ with the rapid prototyping of Python.  However, despite this VTK code programmed in Python generally looks the same as its C++ counterpart.  This module seeks to simpify mesh creation and plotting without losing functionality.

Compare two approachs for loading and plotting a surface mesh from a file:


Plotting a Mesh using Python's VTK
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Using this `example <http://www.vtk.org/Wiki/VTK/Examples/Python/STLReader>`_, loading and plotting an STL file requires a lot of code when using only the ``vtk`` module.

.. code:: python

    import vtk

    # create reader
    reader = vtk.vtkSTLReader()
    reader.SetFileName("myfile.stl")
     
    mapper = vtk.vtkPolyDataMapper()
    if vtk.VTK_MAJOR_VERSION <= 5:
        mapper.SetInput(reader.GetOutput())
    else:
        mapper.SetInputConnection(reader.GetOutputPort())

    # create actor
    actor = vtk.vtkActor()
    actor.SetMapper(mapper)
     
    # Create a rendering window and renderer
    ren = vtk.vtkRenderer()
    renWin = vtk.vtkRenderWindow()
    renWin.AddRenderer(ren)
     
    # Create a renderwindowinteractor
    iren = vtk.vtkRenderWindowInteractor()
    iren.SetRenderWindow(renWin)
     
    # Assign actor to the renderer
    ren.AddActor(actor)
     
    # Enable user interface interactor
    iren.Initialize()
    renWin.Render()
    iren.Start()

    # clean up objects
    del iren
    del renWin


Plot a Mesh using vtkInterface
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
The same stl can be loaded and plotted using vtkInterface with:

.. code:: python

    import vtkInterface
    
    filename = "myfile.stl"
    mesh = vtkInterface.LoadMesh(filename)
    mesh.Plot()

The mesh object is more pythonic and the code is much more straightforward.  Garbage collection is taken care automatically and the renderer is cleaned up after the user closes the vtk plotting window.


Advanced Plotting with Numpy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~
When combined with numpy, you can make some truly spectacular plots:

.. code:: python

    import vtkInterface
    import numpy as np
    
    # Make a grid
    x, y, z = np.meshgrid(np.linspace(-5, 5, 20),
                          np.linspace(-5, 5, 20),
                          np.linspace(-5, 5, 5))
    
    points = np.empty((x.size, 3))
    points[:, 0] = x.ravel('F')
    points[:, 1] = y.ravel('F')
    points[:, 2] = z.ravel('F')
    
    # Compute a direction for the vector field
    direction = np.sin(points)**3
    
    # plot using the plotting class
    plobj = vtkInterface.PlotClass()
    plobj.AddArrows(points, direction, 0.5)
    plobj.SetBackground([0, 0, 0]) # RGB set to black
    plobj.Plot()
    del plobj

.. image:: ./images/vectorfield.png

While not everything can be simplified without losing functionality, many of the objects can.  For example, triangular surface meshes in VTK can be subdivided but every other object in VTK cannot.  It then makes sense that a subdivided method be added to the existing triangular surface mesh.  That way, subdivision can be performed with:

.. code:: python

    submesh = mesh.Subdivide('linear', nsub=3)

and ``help(mesh.Subdivide)`` yields a useful helpdoc::

    Help on function Subdivide in module vtkInterface.utilities:
    
    Subdivide(self, nsub, subfilter='linear'):
        Increase the number of triangles in a single, connected triangular
        mesh.

        Uses one of the following vtk subdivision filters to subdivide a mesh.
        vtkButterflySubdivisionFilter
        vtkLoopSubdivisionFilter
        vtkLinearSubdivisionFilter

        Linear subdivision results in the fastest mesh subdivision, but it
        does not smooth mesh edges, but rather splits each triangle into 4
        smaller triangles.

        Butterfly and loop subdivision perform smoothing when dividing, and may
        introduce artifacts into the mesh when dividing.

        Subdivision filter appears to fail for multiple part meshes.  Should
        be one single mesh.


        Parameters
        ----------
        nsub : int
            Number of subdivisions.  Each subdivision creates 4 new triangles,
            so the number of resulting triangles is nface*4**nsub where nface
            is the current number of faces.

        subfilter : string, optional
            Can be one of the following: 'butterfly', 'loop', 'linear'

        Returns
        -------
        mesh : Polydata object
            vtkInterface polydata object.

        Examples
        --------
        >>> from vtkInterface import examples
        >>> import vtkInterface

        >>> mesh = vtkInterface.PolyData(examples.planefile)
        >>> submesh = mesh.Subdivide(1, 'loop')


Contents
========
.. toctree::
   :maxdepth: 2

   installation   
   examples
   common
   polydata
   grids

..
   Indices and tables
   ==================
   * :ref:`genindex`
   * :ref:`modindex`
   * :ref:`search`
