---
title: "Installation"
weight: 2
---

# Installation


Detailed instructions for installing Go on various operating systems can be found on the official Go website. 

Here's a brief overview.

## maxOS Setup

### Use official installer
1. [Download installer](https://go.dev/doc/install)
2. Open the package file you downloaded and follow the prompts to install Go.
3. Check the installed version of Go, open Terminal and typing the following command: 
```
go version
```
### Use Homebrew

Prerequisite:

- Installed the package manager [Homebrew](https://brew.sh/) 

Then:

```
brew install go
```

## Windows Setup

1. [Download installer](https://go.dev/doc/install)
2. Open the MSI file you downloaded and follow the prompts to install Go.
3. Open windows CMD, type the following command:

```
go version
``` 

## Linux Setup

1. [Download official tarball](https://go.dev/dl/) for your linux distribution. For example:
```
wget https://go.dev/dl/go1.24.3.linux-amd64.tar.gz
```

2. Remove any previous Go installation by deleting the /usr/local/go folder (if it exists), then reinstall.
```
 rm -rf /usr/local/go && tar -C /usr/local -xzf go1.24.3.linux-amd64.tar.gz
```

3. Add PATH.
```
export PATH=$PATH:/usr/local/go/bin
```

4. Verify the install version of GO.
```
go version
```


