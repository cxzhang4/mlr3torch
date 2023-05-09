
<!-- README.md is generated from README.Rmd. Please edit that file -->

# mlr3torch

<!-- badges: start -->

[![Lifecycle:
experimental](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://lifecycle.r-lib.org/articles/stages.html#experimental)
[![r-cmd-check](https://github.com/mlr-org/mlr3torch/actions/workflows/r-cmd-check.yml/badge.svg)](https://github.com/mlr-org/mlr3torch/actions/workflows/r-cmd-check.yml)
[![CRAN
status](https://www.r-pkg.org/badges/version/mlr3torch)](https://CRAN.R-project.org/package=mlr3torch)
[![StackOverflow](https://img.shields.io/badge/stackoverflow-mlr3-orange.svg)](https://stackoverflow.com/questions/tagged/mlr3)
[![Mattermost](https://img.shields.io/badge/chat-mattermost-orange.svg)](https://lmmisld-lmu-stats-slds.srv.mwn.de/mlr_invite/)
<!-- badges: end -->

Deep Learning with torch and mlr3.

## Installation

``` r
remotes::install_github("mlr-org/mlr3torch")
```

## What is mlr3torch?

`mlr3torch` is a deep learning framework for the
[`mlr3`](https://mlr-org.com) ecosystem built on top of
[`torch`](https://torch.mlverse.org/). It allows to easily build, train
and evaluate deep learning models in a few lines of codes, without
needing to worry about low-level details. Off-the-shelf learners are
readily available, but custom architectures can be defined by connection
`PipeOpTorch` operators in an `mlr3pipelines::Graph`.

Using predefined learners such as a simple multi layer perceptron (MLP)
works just like any other mlr3 `Learner`.

``` r
learner_mlp = lrn("classif.mlp",
  # defining network parameters
  activation     = "relu",
  layers         = 1,
  d_hidden       = 20,
  # training parameters
  batch_size     = 16,
  epochs         = 50,
  device         = "cpu",
  # Defining the optimizer, loss, and callbacks
  optimizer      = t_opt("adam", lr = 0.1),
  loss           = t_loss("cross_entropy"),
  callbacks      = t_clbk("history"), # this saves the history in the learner
  # Measures to track
  measures_valid = msrs(c("classif.logloss", "classif.ce")),
  measures_train = msrs(c("classif.logloss", "classif.ce"))
)
```

This learner can for example be used for resampling or benchmarking. It
can also be tuned using [mlr3tuning](https://mlr3tuning.mlr-org.com/).

``` r
resample(
  learner    = learner_mlp,
  task       = tsk("iris"),
  resampling = rsmp("holdout")
)
#> <ResampleResult> with 1 resampling iterations
#>  task_id  learner_id resampling_id iteration warnings errors
#>     iris classif.mlp       holdout         1        0      0
```

Below, we construct the same architecture using `PipeOpTorch` objects.
The first pipeop – a `PipeOpTorchIngress` – defines the entrypoint of
the network. All subsequent pipeops define the neural network layers.

``` r
architecture = po("torch_ingress_num") %>>%
  po("nn_linear", out_features = 20) %>>%
  po("nn_relu") %>>%
  po("nn_head")
```

To turn this into a learner, we configure the loss, optimizer, callbacks
and the training arguments.

``` r
graph_mlp = architecture %>>%
  po("torch_loss", loss = t_loss("cross_entropy")) %>>%
  po("torch_optimizer", optimizer = t_opt("adam", lr = 0.1)) %>>%
  po("torch_callbacks", callbacks = t_clbk("history")) %>>%
  po("torch_model_classif", batch_size = 16, epochs = 50, device = "cpu")

learner_graph_mlp = as_learner(graph_mlp)
learner_graph_mlp$id = "graph_mlp"
```

## Feature Overview

  - Off-the-shelf architectures are readily available as mlr3
    `Learner`s.
  - Custom learners can be defined using the `Graph` language from
    `mlr3pipelines` or using `nn_module`s.
  - The package supports tabular and image data.
  - It is possible to customize the training process via (predefined or
    custom) callbacks.
  - The package is fully integrated into the `mlr3` ecosystem.

## Documentation

The easiest way to learn about `mlr3torch` is to read one of the
vignettes.

## Acknowledgements

  - Without the great R package `torch` none of this would have been
    possible.
  - The names for the callback stages are taken from
    [luz](https://mlverse.github.io/luz/), another high-level deep
    learning framework for R `torch`.
  - Building neural networks using `PipeOpTorch` operators is inspired
    by [keras](https://keras.io/).

## Bugs, Questions, Feedback

*mlr3torch* is a free and open source software project that encourages
participation and feedback. If you have any issues, questions,
suggestions or feedback, please do not hesitate to open an “issue” about
it on the GitHub page\!

In case of problems / bugs, it is often helpful if you provide a
“minimum working example” that showcases the behaviour (but don’t
worry about this if the bug is obvious).

Please understand that the resources of the project are limited:
response may sometimes be delayed by a few days, and some feature
suggestions may be rejected if they are deemed too tangential to the
vision behind the project.
