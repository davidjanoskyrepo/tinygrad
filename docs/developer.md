The tinygrad framework has four pieces

* a PyTorch like <b>frontend</b>.
* a <b>scheduler</b> which breaks the compute into kernels.
* a <b>lowering</b> engine which converts ASTs into code that can run on the accelerator.
* an <b>execution</b> engine which can run that code.

## Frontend

Everything in [Tensor](tensor.md) is syntactic sugar around [function.py](function.md), where the forwards and backwards passes are implemented for the different mlops. There's about 25 of them, implemented using about 20 basic ops. Those basic ops go on to construct a graph of:

::: tinygrad.lazy.LazyBuffer
    options:
        show_source: false

The `LazyBuffer` graph specifies the compute in terms of low level tinygrad ops. Not all LazyBuffers will actually become realized. There's two types of LazyBuffers, base and view. base contains compute into a contiguous buffer, and view is a view (specified by a ShapeTracker). Inputs to a base can be either base or view, inputs to a view can only be a single base.

## Scheduling

The [scheduler](https://github.com/tinygrad/tinygrad/tree/master/tinygrad/engine/schedule.py) converts the graph of LazyBuffers into a list of `ScheduleItem`. One `ScheduleItem` is one kernel on the GPU, and the scheduler is responsible for breaking the large compute graph into subgraphs that can fit in a kernel. `ast` specifies what compute to run, and `bufs` specifies what buffers to run it on.

::: tinygrad.ops.ScheduleItem

## Lowering

The code in [realize](https://github.com/tinygrad/tinygrad/tree/master/tinygrad/engine/realize.py) lowers `ScheduleItem` to `ExecItem` with

::: tinygrad.engine.realize.lower_schedule

There's a ton of complexity hidden behind this, see the `codegen/` directory.

First we lower the AST to UOps, which is a linear list of the compute to be run. This is where the BEAM search happens. The UOps can be changed by `CompilerOptions`.

::: tinygrad.device.CompilerOptions

Then we render the UOps into code, then we compile the code to binary.

## Execution

Creating `ExecItem`, which has a run method

::: tinygrad.engine.realize.ExecItem
    options:
        members: true

Lists of `ExecItem` can be condensed into a single ExecItem with the Graph API (rename to Queue?)