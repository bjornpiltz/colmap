Frequently Asked Questions
==========================

.. _faq-share-intrinsics:

Share intrinsics
----------------

COLMAP supports shared intrinsics for arbitrary groups of images and camera
models. Images share the same intrinsics, if they refer to the same camera, as
specified by the `camera_id` property in the database. You can add new cameras
and set shared intrinsics in the database management tool. Please, refer to
:ref:`Database Management <database-management>` for more information.


.. _faq-fix-intrinsics:

Fix intrinsics
--------------

By default, COLMAP tries to refine the intrinsic camera parameters (except
principal point) automatically during the reconstruction. Usually, if there are
enough images in the dataset and you share the intrinsics between multiple
images, the estimated intrinsic camera parameters in SfM should be better than
parameters manually obtained with a calibration pattern.

However, sometimes COLMAP's self-calibration routine might converge in
degenerate parameters, especially in case of the more complex camera models with
many distortion parameters. If you know the calibration parameters a priori, you
can fix different parameter groups during the reconstruction. Choose
``Reconstruction > Reconstruction options > Bundle Adj. > refine_*`` and check
which parameter group to refine or to keep constant. Even if you keep the
parameters constant during the reconstruction, you can refine the parameters in
a final global bundle adjustment by setting ``Reconstruction > Bundle adj.
options > refine_*`` and then running ``Reconstruction > Bundle adjustment``.


Principal point refinement
--------------------------

By default, COLMAP keeps the principal point constant during the reconstruction,
as principal point estimation is an ill-posed problem in general. Once all
images are reconstructed, the problem is most often constrained enough that you
can try to refine the principal point in global bundle adjustment, especially
when sharing intrinsic parameters between multiple images. Please, refer to
:ref:`Fix intrinsics <faq-fix-intrinsics>` for more information.


Increase number of 3D points
----------------------------

By default, COLMAP ignores two-view feature tracks in triangulation, resulting
in fewer 3D points than possible. Triangulation of two-view tracks can in rare
cases improve the stability of sparse image collections by providing additional
constraints in bundle adjustment. To also triangulate two-view tracks, unselect
the option ``Reconstruction > Reconstruction options > Triangulation >
ignore_two_view_tracks``.


.. _faq-merge-models:

Merge disconnected models
-------------------------

Sometimes COLMAP fails to reconstruct all images into the same model and hence
produces multiple sub-models. If those sub-models have common registered images,
they can be merged into a single model as post-processing step::

    ./src/exe/model_merger \
        --input_path1 /path/to/sub-model1 \
        --input_path2 /path/to/sub-model2 \
        --output_path /path/to/merged-model

To improve the quality of the alignment between the two sub-models, it is
recommended to run another global bundle adjustment after the merge::

    ./src/exe/bundle_adjuster \
        --input_path /path/to/merged-model \
        --output_path /path/to/refined-merged-model


.. _faq-dense-memory:

Reduce memory usage during dense reconstruction
-----------------------------------------------

You can either reduce the maximum image resolution by setting the option
``max_image_size`` or reduce the number of sources images in the
``stereo/patch-match.cfg`` file from e.g. ``__auto__ 30`` to ``__auto__ 10``
file. Note that enabling the ``geom_consistency`` option increases the required
GPU memory.

For large-scale reconstructions of several thousands of images, you should split
your sparse reconstruction into more manageable clusters of images using e.g.
CMVS [furukawa10]_. Otherwise, the fusion procedure might run out of memory, as
the implementation requires all depth and normal maps to be loaded at the same
time. Note that, for this use case, COLMAP's dense reconstruction pipeline also
supports the PMVS/CMVS folder structure when executed from the command-line.


.. _faq-dense-timeout:

Fix GPU freezes and timeouts during dense reconstruction
--------------------------------------------------------

The stereo reconstruction pipeline runs on the GPU using CUDA and puts the GPU
under heavy load. You might experience a display freeze or even a program crash
during the reconstruction. As a solution to this problem, you could use a
secondary GPU in your system, that is not connected to your display.
Alternatively, you can increase the GPU timeouts of your system, as detailed in
the following.

By default, the Windows operating system detects response problems from the GPU,
and recovers to a functional desktop by resetting the card and aborting the
stereo reconstruction process. The solution is to increase the so-called
"Timeout Detection & Recovery" (TDR) delay to a larger value. Please, refer to
the `NVIDIA Nsight documentation <https://goo.gl/d17IhT>`_ or to the `Microsoft
documentation <http://www.microsoft.com/whdc/device/display/wddm_timeout.mspx>`_
on how to increase the delay time under Windows.

The X window system under Linux/Unix has a similar feature and detects response
problems of the GPU. The easiest solution to avoid timeout problems under the X
window system is to shut it down and run the stereo reconstruction from the
command-line. Under Ubuntu, you could first stop X using::

    sudo service lightdm stop

And then run the dense reconstruction code from the command-line::

    ./src/exe/dense_mapper ...

Finally, you can restart your desktop environment with the following command::

    sudo service lightdm start
