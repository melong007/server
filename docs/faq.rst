..
  # Copyright (c) 2019-2020, NVIDIA CORPORATION. All rights reserved.
  #
  # Redistribution and use in source and binary forms, with or without
  # modification, are permitted provided that the following conditions
  # are met:
  #  * Redistributions of source code must retain the above copyright
  #    notice, this list of conditions and the following disclaimer.
  #  * Redistributions in binary form must reproduce the above copyright
  #    notice, this list of conditions and the following disclaimer in the
  #    documentation and/or other materials provided with the distribution.
  #  * Neither the name of NVIDIA CORPORATION nor the names of its
  #    contributors may be used to endorse or promote products derived
  #    from this software without specific prior written permission.
  #
  # THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS ``AS IS'' AND ANY
  # EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
  # IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
  # PURPOSE ARE DISCLAIMED.  IN NO EVENT SHALL THE COPYRIGHT OWNER OR
  # CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
  # EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
  # PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
  # PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY
  # OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
  # (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
  # OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

.. _section-faq:

FAQ
===

What are the advantages of running a model with Triton Inference Server compared to running directly using the model's framework API?
-------------------------------------------------------------------------------------------------------------------------------------

When using Triton Inference Server the inference result will be the
same as when using the model's framework directly. However, with
Triton we get benefits like :ref:`concurrent model execution
<section-concurrent-model-execution>` (the ability to run multiple
models at the same time on the same GPU) and :ref:`dynamic batching
<section-dynamic-batcher>` to get better throughput. We can also
:ref:`replace or upgrade models while Triton and client application
are running <section-model-management>`. Another benefit is that
Triton can be deployed as a Docker container, anywhere – on premises
and on public clouds. Triton Inference Server also :ref:`supports
several frameworks <section-model-repository>` such as TensorRT,
TensorFlow, PyTorch, and ONNX on both GPUs and CPUs leading to a
streamlined deployment.

Can Triton Inference Server run on systems that don't have GPUs?
----------------------------------------------------------------

Yes, see :ref:`section-running-triton-without-gpu`.

Can Triton Inference Server be used in non-Docker environments?
---------------------------------------------------------------

Yes. Triton Inference Server has a CMake build that allows the server
to be built from source making it more portable to non-Docker
environments. For more details, see
:ref:`section-building-the-server-with-cmake`. After building you can
then run Triton outside of Docker as described in
:ref:`section-running-triton-without-docker`.

Do you provide client libraries for languages other than C++ and Python?
------------------------------------------------------------------------

We provide C++ and Python client libraries to make it easy for users
to write client applications that communicate with Triton. We chose
those languages because they were likely to be popular and performant
in the ML inference space, but in the future we can possibly add
another language if there is a need.

We provide the GRPC API as a way to generate your own client library
for a large number of languages. By following the official GRPC
documentation and using `src/core/grpc\_service.proto
<https://github.com/triton-inference-server/server/blob/master/src/core/grpc_service.proto>`_
you can generate language bindings for all the languages supported by
GRPC. We provide two examples of this:

- Go:
  `https://github.com/triton-inference-server/server/tree/master/src/clients/go`_.

- Python:
  `https://github.com/triton-inference-server/server/blob/master/src/clients/python/api_v1/examples/grpc_image_client.py`_.

In general the client libraries (and client examples) are meant to be
just that, examples. We feel the client libraries are well written and
well tested, but they are not meant to serve every possible use
case. In some cases you may want to develop your own customized
library to suit your specific needs.

How would you use Triton Inference Server within the AWS environment?
---------------------------------------------------------------------

In an AWS environment, the Triton Inference Server docker container
can run on :ref:`CPU-only instances or GPU compute instances
<section-running-triton>`. Triton can run directly on the compute
instance or inside Elastic Kubernetes Service (EKS). In addition,
other AWS services such as Elastic Load Balancer (ELB) can be used for
load balancing traffic among multiple Triton instances. Elastic Block
Store (EBS) or S3 can be used for storing deep-learning models loaded
by the inference server.

How do I measure the performance of my model running in the Triton Inference Server?
------------------------------------------------------------------------------------

The Triton Inference Server exposes performance information in two
ways: by :ref:`Prometheus metrics <section-metrics>` and by the
:ref:`statistics endpoint <section-http-and-grpc-api>`.

A client application, :ref:`perf\_analyzer <section-perf-analyzer>`,
allows you to measure the performance of an individual model using a
synthetic load. The perf\_analyzer application is designed to show you
the tradeoff of latency vs. throughput.

How can I fully utilize the GPU with Triton Inference Server?
-------------------------------------------------------------

Triton Inference Server has several features designed to increase
GPU utilization:

* Triton can :ref:`simultaneous perform inference for multiple models
  <section-concurrent-model-execution>` (using either the same or
  different frameworks) using the same GPU.

* Triton can increase inference throughput by using :ref:`multiple
instances of the same model <section-concurrent-model-execution>` to
handle multiple simultaneous inferences requests to that model. Triton
chooses reasonable defaults but :ref:`you can also control the exact
level of concurrency <section-instance-groups>` on a model-by-model
basis.

* Triton can :ref:`batch together multiple inference requests into a
  single inference execution <section-dynamic-batcher>`. Typically,
  batching inference requests leads to much higher thoughput with only
  a relatively small increase in latency.

As a general rule, batching is the most beneficial way to increase GPU
utilization. So you should alway try enabling the :ref:`dynamic
batcher <section-dynamic-batcher>` with your models. Using multiple
instances of a model can also provide some benefit but is typically
most useful for models that have small compute requirements. Most
models will benefit from using two instances but more than that is
often not useful.

If I have a server with multiple GPUs should I use one Triton Inference Server to manage all GPUs or should I use multiple inference servers, one for each GPU?
---------------------------------------------------------------------------------------------------------------------------------------------------------------

Triton Inference Server will take advantage of all GPUs on the server
that it has access to. You can limit the GPUs available to Triton by
using the CUDA_VISIBLE_DEVICES environment variable (or with Docker
you can also use NVIDIA_VISIBLE_DEVICES or --gpus flag when launching
the container). When using multiple GPUs, Triton will distribute
inference request across the GPUs to keep them all equally
utilized. You can also :ref:`control more explicitly which models are
running on which GPUs <section-instance-groups>`.

In some deployment and orchestration environments (for example,
Kubernetes) it may be more desirable to partition a single multi-GPU
server into multiple *nodes*, each with one GPU. In this case the
orchestration environment will run a different Triton for each GPU and
an load balancer will be used to divide inference requests across the
available Triton instances.
