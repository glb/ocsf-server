# Open Cybersecurity Schema Framework Server

This is the Open Cybersecurity Schema Framework (OCSF) server source code repository.

## Obtaining the source code

Clone the GitHub OCFS WEB Server repository. Use `--recurse-submodules` to the `git clone` command, which will automatically initialize and update the schema submodule in the repository:

```bash
git clone --recurse-submodules https://github.com/ocsf/ocsf-server.git
```

## Build and run a schema server docker image

```bash
cd ocsf-server
docker build -t docker_ocsf:0.0.0 .
docker run -it --rm -p 8080:8080 docker_ocsf:0.0.0
```

Run the docker schema server with local schema extensions:
```bash
docker run -it --rm --volume /opt/sandbox/splunk:/app/schema/splunk -e SCHEMA_EXTENSION="splunk" -p 8080:8080 -p 8443:8443 rroupski370/ocsf:0.0.0
```

Run the docker schema server with added local schema extensions:
```bash
docker run -it --rm --volume /opt/sandbox/splunk:/app/schema/extensions/splunk -e SCHEMA_EXTENSION="extensions" -p 8080:8080 -p 8443:8443 rroupski370/ocsf:0.0.0
```

Run the docker schema server using a local schema:
```bash
docker run -it --rm --volume /opt/sandbox/ocsf-schema:/app/schema -p 8080:8080 -p 8443:8443 rroupski370/ocsf:0.0.0
```

## Development with docker-compose

The `docker-compose` environment enables development without needing to install any dependencies (apart from Docker/Podman and docker-compose) on the development machine.

When run, the standard `_build` and `deps` folders are created, along with a `.mix` folder. If the environment needs to be recreated for whatever reason, the `_build` folder can be removed and `docker-compose` brought down and up again and the environment will automatically rebuild.

### Run the ocsf-server and build the development container

```
docker-compose up
```

Then browse to the schema server at http://localhost:8080

### Testing the schema with docker-compose

**NOTE:** it is _not_ necessary to run the server with `docker-compose up` first in order to test the schema (or run any other commands in the development container).

```
# docker-compose run ocsf-elixir mix test 
Creating ocsf-server_ocsf-elixir_run ... done
Emulate Docker CLI using podman. Create /etc/containers/nodocker to quiet msg.


Finished in 0.00 seconds (0.00s async, 0.00s sync)
0 failures

Randomized with seed 933777
```

### Set aliases to avoid docker-compose inflicted RSI

```
source docker-source.sh
```

### Using aliases to run docker-compose commands

```
# testschema
Creating ocsf-server_ocsf-elixir_run ... done
Emulate Docker CLI using podman. Create /etc/containers/nodocker to quiet msg.


Finished in 0.00 seconds (0.00s async, 0.00s sync)
0 failures

Randomized with seed 636407
```

### Using environment variables to change docker-compose defaults

Optional environment variables can be placed in a `.env` file in the root of the repo to change the default behavior.

An `.env.sample` is provided, and the following options are available:

```
SCHEMA_PATH=./modules/schema    # Set the local schema path, eg. ../ocsf-schema, defaults to ./modules/schema
OCSF_SERVER_PORT=8080           # Set the port for Docker to listen on for forwarding traffic to the Schema Server, defaults to 8080
ELIXIR_VERSION=otp-25-alpine    # Set the Elixir container version for development, defaults to otp-25-alpine
```

## Local Usage

This section describes how to build and run the OCSF Schema Schema server.

### Required build tools

The Schema Server is written in [Elixir](https://elixir-lang.org) using the [Phoenix](https://phoenixframework.org/) web framework.

The Elixir site maintains a great installation page, see https://elixir-lang.org/install.html for help.

### Building the schema server

Elixir uses the [`mix`](https://hexdocs.pm/mix/Mix.html) build tool, which is included in the Elixir installation package..

#### Install the build tools

```bash
mix local.hex --force && mix local.rebar --force
```

#### Get the dependencies

Change to the schema directory, fetch and compile the dependencies:

```bash
cd ocsf-server
mix do deps.get, deps.compile
```

#### Compile the source code

```bash
mix compile
```

### Testing local schema changes

You can use Elixir `mix test` to test the changes made to the schema. For example to ensure the JSON files are correct or the attributes are defined.

Assuming the schema repo has been cloned in `../ocsf-schema` directory, then you can test the schema with this command:

```shell
SCHEMA_DIR=../ocsf-schema SCHEMA_EXTENSION=extensions mix test
```

Or, test the schema using the `module/schema` folder:

```shell
SCHEMA_DIR=modules/schema SCHEMA_EXTENSION=extensions mix test
```

If everything is correct, then you should not see any errors or warnings.

### Running the schema server

You can use the Elixir's interactive shell, [IEx](https://hexdocs.pm/iex/IEx.html), to start the schema server use:

```bash
iex -S mix phx.server
```

Now you can access the Schema server at [`localhost:8080`](http://localhost:8080) or [`localhost:8443`](https://localhost:8443).

### Runtime configuration

The schema server uses a number of environment variables.

| Variable Name    | Description |
| ---------------- | ----------- |
| HTTP_PORT        | The server HTTP  port number, default: `8080`|
| HTTPS_PORT       | The server HTTPS port number, default: `8443`|
| SCHEMA_DIR       | The directory containing the schema, default: `modules/schema` |
| SCHEMA_EXTENSION | The directory containing the schema extensions, relative to the SCHEMA_DIR |
| RELEASE_NODE     | The Erlang node name. Set ti if you want to run more than one server on the same computer |

```bash
SCHEMA_DIR=modules/schema SCHEMA_EXTENSION=extensions iex -S mix phx.server
```

## Releasing the schema server

This section describes how to make a release of the event schema server.

### Create a release

The schema server project uses the [`Elixir Releases`](https://hexdocs.pm/mix/Mix.Tasks.Release.html) to produce an Erlang/OTP release package. To make a production release package, run this command:

```bash
MIX_ENV=prod mix release --path dist
```

This command creates the release in the `./dist` directory.

You can use one of the following options to start the Schema server (see the above environment variables):

```bash
# starts the schema server in the foreground
> bin/schema_server start

# starts the schema server with IEx attached, like 'iex -S mix phx.server'
> bin/schema_server start_iex

# starts the schema server in the background, must be stopped with the 'bin/schema_server stop' command
> bin/schema_server daemon
```

For example to start the schema server, with all extensions, from the `dist` folder use:

```bash
cd dist
SCHEMA_DIR=schema SCHEMA_EXTENSION=extensions bin/schema_server start
```

For a complete listing of commands use:

```bash
bin/schema_server
```

### Deploy the release

A release is built on a **host**, a machine which contains Erlang, Elixir, and any other dependencies needed to compile the schema server. A release is then deployed to a **target**, potentially the same machine as the host, but usually a separate target host.

To deploy the schema server, copy the release archive file (`dist/schema_server-<version>.tar.gz`) from the release folder to the target. Extract the release files to disk from the archive. Note, the following must be the same between the **host** and the **target**:

- Target architecture (for example, x86_64 or ARM)
- Target vendor + operating system (for example, Windows, Linux, or Darwin/macOS)
- Target ABI (for example, musl or gnu)
