#ai #utilities

fabric is an open-source framework for augmenting humans using AI. It provides a modular framework for solving specific problems using a crowdsourced set of AI prompts that can be used anywhere.  
[danielmiessler/fabric](https://github.com/danielmiessler/fabric)

## Installation

> [!info]  
> Fabric is switching to a go installer in the future.

 Navigate to where you want the Fabric project to live on your system in a semi-permanent place on your computer.

```shell
# Find a home for Fabric
cd /where/you/keep/code
```

Clone the project to your computer.

```shell
# Clone Fabric to your computer
git clone https://github.com/danielmiessler/fabric.git
```

Enter Fabric's main directory.

```shell
# Enter the project folder (where you cloned it)
cd fabric
```

Install pipx.

```shell
sudo apt install pipx
```

Install fabric.

```shell
pipx install .
```

Run setup.

```shell
fabric --setup
```

Restart your shell to reload everything.

```bash
source ~/.zshrc
```

Now you are up and running! You can test by running the help.

```shell
# Making sure the paths are set up correctly
fabric --help
```

FFmpeg is a command-line tool that records, transcodes, mixes, formats, and streams multimedia content. If we don't install it, our prompts get a warning.
```bash
apt install ffmpeg
```

## Usage

Make sure that you are running [[Ollama]] locallly.

List all available patterns.

```bash
fabric --list
```

Use the summarize pattern. See [[Utilities#Pbcopy and Pbpaste|pbpaste]] in case you don't have it configured yet.

```bash
pbpaste | fabric -sp summarize;
```

Use a specific model.

```bash
pbpaste | fabric --model llama3:latest -p summarize 
```

You can chain patterns.

```bash
# The 's' parameter is used to enable streaming.
# However that means you can't pipe it to another command.
pbpaste | fabric -p summarize | fabric -sp write_essay
```

You can even go batshit insane and summarize youtube videos.

```bash
yt --transcript https://www.youtube.com/watch\?v\=UbDyjIIGaxQ | fabric -sp extract_wisdom
```