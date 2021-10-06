---
title: Data and Model Management Trail
---

As its name implies, DVC is used to control versions of data. It enables to keep
track of multiple versions of your datasets.

## Initialize a DVC project

Suppose we are working on a deep learning project to develop the next ground
breaking supervised learning model. We plan to test the classifier in MNIST
dataset, but we also plan to use a more difficult one, Fashion MNIST. We want to
keep track of these two datasets and replace with each other easily, without
changes in the code.

We need a way to track these two datasets as if they are versions of the same
text file. DVC is used for this kind of data and model file tracking as if they
are code files.

#### ✍🏻 We download MNIST data from a URL using wget/curl

We first download the example project from a point where only the source code
files are present.

```dvc
$ git clone https://github.com/iterative/example-data-management -b get-started
$ cd example-data-management
```

The data is not provided with the source code and we need to download it
separately.

```dvc
$ wget https://dvc.org/datasets/mnist.zip -o data/mnist.zip
```

> Later on, we'll see how to automate this procedure and how DVC can track the
> data along with the code. We're just starting journey.

## Adding data to DVC projects

We add data and model files (and directories) to DVC with `dvc add` command.

```dvc
$ dvc add data/mnist.zip
```

DVC stores information about the added file (or a directory) in a special `.dvc`
file named `data/mnist.zip.dvc` a small text file with a human-readable
[format](/doc/user-guide/project-structure/dvc-files). This metadata file is a
placeholder for the original data, and can be easily versioned like source code
with Git:

```dvc
$ git add data/mnist.zip.dvc data/.gitignore
$ git commit -m "Add zipped MNIST data"
```

The original data, meanwhile, is listed in `.gitignore`.

## Versioning data in DVC projects

Suppose you have run the [experiments] with MNIST and would like to see your
model's performance in another dataset. You can update the code to use a
different dataset but here, in order to demonstrate how DVC makes it easy to
update the data, we'll write Fashion-MNIST over MNIST.

```dvc
$ wget https://dvc.org/datasets/fashion-mnist.zip --force -o data/mnist.zip
```

Now, when we ask DVC about the changes in the workspace, it tells us that
`mnist.zip` has changed.

```dvc
$ dvc status
```

And we can add the newer version to DVC as well.

```dvc
$ dvc add data/mnist.zip
$ git add data/mnist.zip.dvc
$ git commit -m "Added Fashion MNIST dataset"
```

Now you have two different datasets in your cache, and you can switch between
them as if they are code files in a Git repository.

```dvc
$ git checkout HEAD~1
$ dvc checkout
```

Note that you can also keep these different version in separate Git branches or
tags. Their content is saved in `.dvc/cache` in the project root and only a
reference in the form of `.dvc` file is kept in Git.

Yes, DVC is technically not even a version control system! `.dvc` file contents
define data file versions. Git itself provides the version control. DVC in turn
creates these `.dvc` files, updates them, and synchronizes DVC-tracked data in
the <abbr>workspace</abbr> efficiently to match them.

<details>

### ℹ️ Large datasets versioning

In cases where you process very large datasets, you need an efficient mechanism
(in terms of space and performance) to share a lot of data, including different
versions. Do you use network attached storage (NAS)? Or a large external volume?
You can learn more about advanced workflows using these links:

- A [shared cache](/doc/user-guide/how-to/share-a-dvc-cache) can be set up to
  store, version and access a lot of data on a large shared volume efficiently.
- A quite advanced scenario is to track and version data directly on the remote
  storage (e.g. S3). See
  [Managing External Data](https://dvc.org/doc/user-guide/managing-external-data)
  to learn more.

</details>

## Sharing data and models

You can upload DVC-tracked data or model files with `dvc push`, so they're
safely stored [remotely](/doc/command-reference/remote). This also means they
can be retrieved on other environments later with `dvc pull`. First, we need to
set up a remote storage location:

```dvc
$ dvc remote add -d storage s3://mybucket/dvcstore
$ git add .dvc/config
$ git commit -m "Configure remote storage"
```

> DVC supports many remote storage types, including Amazon S3, SSH, Google
> Drive, Azure Blob Storage, and HDFS. See `dvc remote add` for more details and
> examples.

<details>

### ⚙️ Expand to set up remote storage.

DVC remotes let you store a copy of the data tracked by DVC outside of the local
cache (usually a cloud storage service). For simplicity, let's set up a _local
remote_:

```dvc
$ mkdir -p /tmp/dvcstore
$ dvc remote add -d myremote /tmp/dvcstore
$ git commit .dvc/config -m "Configure local remote"
```

> While the term "local remote" may seem contradictory, it doesn't have to be.
> The "local" part refers to the type of location: another directory in the file
> system. "Remote" is what we call storage for <abbr>DVC projects</abbr>. It's
> essentially a local data backup.

</details>

```dvc
$ dvc push
```

Usually, we also want to `git commit` and `git push` the corresponding `.dvc`
files.

## Pushing to/pulling from remotes

- Push the cache to the remote we created
- Clone the repository to somewhere (e.g. ssh or local folder)
- Pull the cache

Having DVC-tracked data and models stored remotely, it can be downloaded when
needed in other copies of this <abbr>project</abbr> with `dvc pull`. Usually, we
run it after `git clone` and `git pull`.

<details>

### ⚙️ Expand to delete locally cached data.

If you've run `dvc push`, you can delete the cache (`.dvc/cache`) and
`data/data.xml` to experiment with `dvc pull`:

```dvc
$ rm -rf .dvc/cache
$ rm -f data/data.xml
```

</details>

```dvc
$ dvc pull
```

> 📖 See also
> [Sharing Data and Model Files](/doc/use-cases/sharing-data-and-model-files)
> for more on basic collaboration workflows.

## Accessing public datasets and registries

- Get the Fashion MNIST data from dataset-registry

Having initialized a project in the previous section, we can get the data file
(which we'll be using later) like this:

```dvc
$ dvc get https://github.com/iterative/dataset-registry \
          get-started/data.xml -o data/data.xml
```

We use the fancy `dvc get` command to jump ahead a bit and show how a Git repo
becomes a source for datasets or models Ѡwhat we call a "data/model registry".
`dvc get` can download any file or directory tracked in a <abbr>DVC
repository</abbr>. It's like `wget`, but for DVC or Git repos. In this case we
download the latest version of the `data.xml` file from the
[dataset registry](https://github.com/iterative/dataset-registry) repo as the
data source.

> # Get Started: Data and Model Access
>
> We've learned how to _track_ data and models with DVC, and how to commit their
> versions to Git. The next questions are: How can we _use_ these artifacts
> outside of the project? How do we download a model to deploy it? How to
> download a specific version of a model? Or reuse datasets across different
> projects?
>
> > These questions tend to come up when you browse the files that DVC saves to
> > remote storage (e.g.
> > `s3://dvc-public/remote/get-started/fb/89904ef053f04d64eafcc3d70db673` 😱
> > instead of the original file name such as `model.pkl` or `data.xml`).
>
> Read on or watch our video to see how to find and access models and datasets
> with DVC.
>
> https://youtu.be/EE7Gk84OZY8
>
> Remember those `.dvc` files `dvc add` generates? Those files (and `dvc.lock`,
> which we'll cover later) have their history in Git. DVC's remote storage
> config is also saved in Git, and contains all the information needed to access
> and download any version of datasets, files, and models. It means that a Git
> repository with <abbr>DVC files</abbr> becomes an entry point, and can be used
> instead of accessing files directly.
>
> ## Find a file or directory
>
> You can use `dvc list` to explore a <abbr>DVC repository</abbr> hosted on any
> Git server. For example, let's see what's in the `get-started/` directory of
> our [dataset-registry](https://github.com/iterative/dataset-registry) repo:
>
> ```dvc
> $ dvc list https://github.com/iterative/dataset-registry get-started
> .gitignore
> data.xml
> data.xml.dvc
> ```
>
> The benefit of this command over browsing a Git hosting website is that the
> list includes files and directories tracked by both Git and DVC (`data.xml` is
> not visible if you
> [check GitHub](https://github.com/iterative/dataset-registry/tree/master/get-started)).
>
> ## Download
>
> One way is to simply download the data with `dvc get`. This is useful when
> working outside of a <abbr>DVC project</abbr> environment, for example in an
> automated ML model deployment task:
>
> ```dvc
> $ dvc get https://github.com/iterative/dataset-registry \
>           use-cases/cats-dogs
> ```
>
> When working inside another DVC project though, this is not the best strategy
> because the connection between the projects is lost — others won't know where
> the data came from or whether new versions are available.
>
> ## Import file or directory
>
> `dvc import` also downloads any file or directory, while also creating a
> `.dvc` file (which can be saved in the project):
>
> ```dvc
> $ dvc import https://github.com/iterative/dataset-registry \
>              get-started/data.xml -o data/data.xml
> ```
>
> This is similar to `dvc get` + `dvc add`, but the resulting `.dvc` files
> includes metadata to track changes in the source repository. This allows you
> to bring in changes from the data source later using `dvc update`.
>
> <details>
>
> #### 💡 Expand to see what happens under the hood.
>
> > Note that the
> > [dataset registry](https://github.com/iterative/dataset-registry) repository
> > doesn't actually contain a `get-started/data.xml` file. Like `dvc get`,
> > `dvc import` downloads from [remote storage](/doc/command-reference/remote).
>
> `.dvc` files created by `dvc import` have special fields, such as the data
> source `repo` and `path` (under `deps`):
>
> ```git
> +deps:
> +- path: get-started/data.xml
> +  repo:
> +    url: https://github.com/iterative/dataset-registry
> +    rev_lock: f31f5c4cdae787b4bdeb97a717687d44667d9e62
>  outs:
>  - md5: a304afb96060aad90176268345e10355
>    path: data.xml
> ```
>
> The `url` and `rev_lock` subfields under `repo` are used to save the origin
> and [version](https://git-scm.com/docs/revisions) of the dependency,
> respectively.
>
> </details>
>
> ## Python API
>
> It's also possible to integrate your data or models directly in source code
> with DVC's [Python API](/doc/api-reference). This lets you access the data
> contents directly from within an application at runtime. For example:
>
> ```py
> import dvc.api
>
> with dvc.api.open(
>     'get-started/data.xml',
>     repo='https://github.com/iterative/dataset-registry'
> ) as fd:
>     # fd is a file descriptor which can be processed normally
> ```

## Removing data from DVC projects

- Remove certain folders from workspace
- Delete the corresponding cache files