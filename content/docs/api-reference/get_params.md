# dvc.api.get_params()

Returns all the <abbr>params</abbr> associated with the given
<abbr>stage</abbr>.

```py
def get_params(stage: str, flatten: bool = False) -> Dict:
```

#### Usage:

```py
import dvc.api

params = dvc.api.get_params("train")
```

## Description

Retrieves the `dvc params` associated with a given `dvc.yaml` stage.

Only the keys indicated in the
[`params` section of the stage](/doc/user-guide/project-structure/pipelines-files#stage-entries)
will be returned.

When multiple `dvc params` files are associated with a stage, this function
takes care of aggregating and selecting all the relevant keys, regardless of the
file format used.

## Parameters

- **`stage`** (required) - Name of the dvc.yaml stage to get params from.

- `flatten` - Whether to return flattened dict or not. _Default_: False.

  If True, `{"foo.bar": 1}` would be returned, instead of `{"foo": {"bar": 1}}`.

## Example: Accesing params inside a stage.

Given the following `dvc params` files:

```yaml
# params.yaml
foo:
  bar: 1
```

```json
# params.json
{"bar": 2}
```

And corresponding `dvc.yaml`:

```yaml
stages:
  merge-params:
    cmd: python merge_params.py
    params:
      - foo
      - params.json:
          - bar
    outs:
      - merged.json
```

We can use the `dvc.api.get_params()` function in order to retrieve `dvc params`
inside a python script:

```py
# merge_params.py
import json
import dvc.api

params = dvc.api.get_params("merge-params")

with open("merged.json", "w") as f:
    json.dump(params,f, indent=4)
```

If we run `dvc repro`, this will generate the following output in `merged.json`:

```json
{
  "foo": {
    "bar": 1
  },
  "bar": 2
}
```
