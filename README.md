# mkdocs-puml

[![PyPI version](https://badge.fury.io/py/mkdocs_puml.svg)](https://badge.fury.io/py/mkdocs_puml)

`mkdocs_puml` is a fast and simple package that brings PUML diagrams into `mkdocs`
documentation.

## Install

Run the following command to install this package

```shell
pip install mkdocs_puml
```

## How to use

To use puml with mkdocs, just add `mkdocs_puml.extensions` into
`markdown_extensions` block of your `mkdocs.yml` file.

```yaml
markdown_extensions:
    - mkdocs_puml.extensions:
        puml_url: https://www.plantuml.com/plantuml/
        num_workers: 5
```

Where
* `puml_url` is URL to the `PlantUML` service.
* `num_workers` is max amount of concurrent workers that requests plantuml service.
    This variable should take the value of average amount of diagrams on the page.
    Despite `num_workers` the extension processes each markdown page simultaneously.
    **NOTE** this setting is NOT required.

Now, you can put your puml diagrams into your `.md` documentation. For example,

<pre>
## PUML Diagram

```puml
@startuml
Bob -> Alice : hello
@enduml
```
</pre>

At the build phase `mkdocs` will send request to `puml_url` and substitute your
diagram with the `svg` image from response.

### Run PlantUML service with Docker

It is possible to run [plantuml/plantuml-server](https://hub.docker.com/r/plantuml/plantuml-server)
in Docker.

Add a new service to the `docker-compose.yml` file

```yaml
version: "3"
services:
  puml:
    image: plantuml/plantuml-server
    ports:
      - '8080:8080'
```

Then substitute `puml_url` setting with the local's one in the `mkdocs.yml` file

```yaml
markdown_extensions:
    - mkdocs_puml.extensions:
        puml_url: http://127.0.0.1:8080
```

Obviously, this approach works faster than
using [plantuml.com](https://www.plantuml.com/plantuml/).

### Standalone usage

You can use `PlantUML` converter on your own without `mkdocs`.
The example below shows how to do that.

```python
from mkdocs_puml.puml import PlantUML

puml_url = "https://www.plantuml.com/plantuml"

diagram1 = """
@startuml
Bob -> Alice : hello
@enduml
"""

diagram2 = """
@startuml
Jon -> Sansa : hello
@enduml
"""

puml = PlantUML(puml_url, num_worker=2)
svg_for_diag1, svg_for_diag2 = puml.translate([diagram1, diagram2])
```

## How it works

The package uses `PlantUML` as a service via HTTP. So, it encodes the diagram into
a string that can be passed via URL. Then it sends GET request to
the `PlantUML` service and receives `svg` image containing the diagram.

The markdown extension just parses `.md` documentation files and looks for

<pre>
```puml

```
</pre>

code blocks.

**NOTE** you must set `puml` keyword as an indicator that the plant uml is located in the block.

After the page is parsed, `mkdocs_puml` extension encodes diagrams and
sends requests to `PlantUML`. After responses are received, the package
substitutes `puml` diagrams with the `svg` images.

## License

The project is licensed under [MIT license](LICENSE).
