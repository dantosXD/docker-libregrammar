# Dockerfile for LibreGrammar
This repository contains a Dockerfile to create a Docker image for [LibreGrammar](https://github.com/TiagoSantos81/languagetool), a LanguageTool fork maintained
by [TiagoSantos81](https://github.com/TiagoSantos81).

I wrote this image since I'm looking for a Job, so I can't afford to pay LanguageTool premium and LibreGrammar activates most of the rules.

# Setup

## Prebuilt images

At the moment I don't provide a built image in none of the registries. You should build the image on your own.

## Setup using the Dockerfile
This approach could be used when you plan to make changes to the `Dockerfile`.
```
git clone https://github.com/py-crash/docker-libregrammar.git -b libregrammar --config core.autocrlf=input
cd libregrammar
docker build -t libregrammar .
docker run --rm -it -p 8081:8081 libregrammar
```

# Configuration

## Java heap size
LibreGrammar will be started with a minimal heap size (`-Xms`) of `256m` and a maximum (`-Xmx`) of `512m`. You can overwrite these defaults by setting the [environment variables](https://docs.docker.com/engine/reference/commandline/run/#set-environment-variables--e---env---env-file) `Java_Xms` and `Java_Xmx`.

An example startup configuration:
```
docker run --rm -it -p 8081:8081 -e Java_Xms=512m -e Java_Xmx=2g libregrammar
```

## LibreGrammar HTTPServerConfig
You are able to use the [HTTPServerConfig](https://languagetool.org/development/api/org/languagetool/server/HTTPServerConfig.html) configuration options by prefixing the fields with `langtool_` and setting them as [environment variables](https://docs.docker.com/engine/reference/commandline/run/#set-environment-variables--e---env---env-file).

An example startup configuration:
```
docker run --rm -it -p 8081:8081 -e langtool_pipelinePrewarming=true -e Java_Xms=1g -e Java_Xmx=2g libregrammar
```

## Using n-gram datasets
> LibreGrammar can make use of large n-gram data sets to detect errors with words that are often confused, like __their__ and __there__.

*Source: [https://dev.languagetool.org/finding-errors-using-n-gram-data](https://dev.languagetool.org/finding-errors-using-n-gram-data)*

[Download](http://languagetool.org/download/ngram-data/) the n-gram dataset(s) to your local machine and mount the local n-gram data directory to the `/ngrams` directory in the Docker container [using the `-v` configuration](https://docs.docker.com/engine/reference/commandline/run/#mount-volume--v---read-only) and set the `languageModel` configuration to the `/ngrams` folder.

An example startup configuration:
```
docker run --rm -it -p 8081:8081 -e langtool_languageModel=/ngrams -v local/path/to/ngrams:/ngrams libregrammar
```

## Improving the spell checker

> You can improve the spell checker without touching the dictionary. For single words (no spaces), you can add your words to one of these files:
> * `spelling.txt`: words that the spell checker will ignore and use to generate corrections if someone types a similar word
> * `ignore.txt`: words that the spell checker will ignore but not use to generate corrections
> * `prohibited.txt`: words that should be considered incorrect even though the spell checker would accept them

*Source: [https://dev.languagetool.org/hunspell-support](https://dev.languagetool.org/hunspell-support)*

The following `Dockerfile` contains an example on how to add words to `spelling.txt`. It assumes you have your own list of words in `en_spelling_additions.txt` next to the `Dockerfile`. It assumes you already built the LibreGrammar image.
```
FROM libregrammar

# Improving the spell checker
# http://wiki.languagetool.org/hunspell-support
USER root
COPY en_spelling_additions.txt en_spelling_additions.txt
RUN  (echo; cat en_spelling_additions.txt) >> org/languagetool/resource/en/hunspell/spelling.txt
USER libregrammar
```

You can build & run the custom Dockerfile with the following two commands:
```
docker build -t libregrammar-custom .
docker run --rm -it -p 8081:8081 libregrammar-custom
```

You can add words to other languages by changing the `en` language tag in the target path. Note that for some languages, e.g. for `nl` the `spelling.txt` file is not in the `hunspell` folder: `org/languagetool/resource/nl/spelling/spelling.txt`.

# Docker Compose

This image can also be used with [Docker Compose](https://docs.docker.com/compose/). An example `docker-compose.yml` would be:

```yaml
version: "3"

services:
  libregrammar:
    build: ./docker-libregrammar
    container_name: libregrammar
    ports:
        - 8081:8081  # Using default port from the image
    environment:
        - langtool_languageModel=/ngrams  # OPTIONAL: Using ngrams data
        - Java_Xms=512m  # OPTIONAL: Setting a minimal Java heap size of 512 mib
        - Java_Xmx=1g  # OPTIONAL: Setting a maximum Java heap size of 1 Gib
    volumes:
        - /path/to/ngrams/data:/ngrams
```

This assumes you have cloned the repo into a folder called `docker-libregrammar` in the same path as your docker-compose.yml

# Usage
By default this image is configured to listen on port 8081.

An example cURL request:
```
curl --data "language=en-US&text=a simple test" http://localhost:8081/v2/check
```

Please refer to the [official LanguageTool documentation](https://dev.languagetool.org/) and to the
[Libregrammmar Repo](https://github.com/TiagoSantos81/languagetool) for further usage instructions.
