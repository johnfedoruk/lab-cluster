# Wiser Labs

A local repository to quickly bring up and tear down infrastructure projects for testing purposes.

> ⚠️ **Warning**
>
> Make sure you are on the correct container clsuter before running the clustercmd command! Failure to do so could take down production systems!

## How to use

The project contains an executable bash script, `clustercmd`, which can be used to manage the infrastructure.

```plaintext
Usage: ./clustercmd [OPTION]

Options:
  --build, -b      Build the cluster
  --destroy, -d    Destroy the cluster
  --rebuild, -r    Rebuild the cluster
  --help, -h       Display this help message
```