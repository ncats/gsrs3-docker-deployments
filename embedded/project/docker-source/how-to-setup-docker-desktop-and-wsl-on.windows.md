# How to set up Docker Desktop and WSL with on windows

## Set up WSL and Ubuntu

go to CMD terminal
type `wsl --list`

If you don't see an Ubuntu installation, then install one.

```
## Do this once, for example. It will ask you for a username and password. 
wsl --install -d Ubuntu-24.04
```
## wt.exe

After intalled, it will be more convenient, to use `wt.exe` as it allows you to use tabbed terminal sessions. 

```
wt.exe --window last new-tab --profile "Ubuntu-24.04"
```

## Associate your Docker Desktop with WSL/Ubuntu

In Docker Desktop click on Settings/Gear Icon (top-right).
 
On left menu click on Resources > WSL Integration and enable Ubuntu  

Now you can create a folder in Ubuntu and clone the GSRS docker recipes repo. 

```
mkdir -p ~/gsrs3/docker && cd ~/gsrs3/docker`

clone gsrs3-docker-deployments
```


## Using web constainers that you create

You should be able to access containers from your Windows Firefox or other browser, for example like this.


```
http://localhost:9081/ginas/app/ui 

http://localhost:9081/api/v1/substances 

```


