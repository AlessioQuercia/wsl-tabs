# Open a new tab (or split pane) in the same directory in WSL
An essential feature for me in terminals is the possibility to quickly open a new tab by pressing ctrl+t (much like in a browser) to make a new search. Differently from the browsers, where you often want to start from the home page, in terminals it is useful to start from the previously used directory. Windows Subsystem for Linux allows both to open new tabs (ctrl+t) and to split pane vertically (alt+plus) and horizontally (alt+-). However, at the moment it does not offer (as far as I know) a ready-to-use feature to open a new tab (or split pane) in the same directory you were working on. By default, WSL will open the new tabs/panes in the %USERPROFILE% directory, as specified in the settings, but it is also possible to specify the _startingDirectory_ in the _settings.json_ file. The only problem is that it will be always the same, unless you change it every time.

Inspired by [this](https://github.com/microsoft/terminal/issues/3158#issuecomment-654539219) and [this](https://stackoverflow.com/questions/57120932/how-to-spawn-a-new-tab-in-same-directory-as-previous-directory), I wrote the following function:

```
sd() {
  settings='/c/Users/<USERNAME>/AppData/Local/Packages/Microsoft.WindowsTerminal_8wekyb3d8bbwe/LocalState/settings.json'
  startDir=\"startingDirectory\"
  sed -i -E "s|$startDir: .*\"|$startDir: \"$(wslpath -m "$(pwd)")\"|g" $settings
}
```
where \<USERNAME\> has to be replaced by your account username. 

Once called, it changes all occurrences of _startingDirectory_ in your _settings.json_ file to your current directory. Since I don't want to call it manually every time I plan to open a new tab, I overrode **cd** and **z** commands to make them call **sd** as follows:

```
cd() {
    builtin cd "$@"; sd
}
```
```
z_store() {
    z "$@"; sd
}
alias z=z_store
```
You can put these two functions in your _.bashrc_ or _.zshrc_ to use **sd** automatically every time you access a directory. 

This "trick" works both for new tabs and split panes. An example is shown below:

<img src="gifs/test.gif" alt="test" width="800"/>

However, there is a known problem: if you open a tab in "Home", cd to "Beautiful_directory", then open a new tab (it will open in "Beautiful_directory" as expected), then from the new tab cd back to "Home", then if you switch the first tab (which is still in "Beautiful_directory") and you open a new tab (or split pane) from there, it will open a new tab (or split pane) in "Home", because it is the last accessed directory (using cd). You can avoid this by manually calling **sd** before opening a new tab (or just using **z** to get back to your desired directory), as shown below:

<img src="gifs/bug.gif" alt="bug" width="800"/>

Lastly, I wrote a batch script to achieve the same result with Command Prompt. Unfortunately, naming the batch file _cd_ and updating the **PATH** accordingly does not override Windows **cd** function, and I did not find a way to override it, therefore I wrote the following script:
```
@echo off
cd %1
set settings=/c/Users/<USERNAME>/AppData/Local/Packages/Microsoft.WindowsTerminal_8wekyb3d8bbwe/LocalState/settings.json
set startDir="startingDirectory"
set lastDir="%CD:\=/%"
sed -i -E 's;%startDir%: .*^";%startDir%: %lastDir%;g' %settings%
```
where \<USERNAME\> has to be replaced by your account username. 
  
I named the batch script _cdd.bat_ and put its directory to the PATH environment variable. After doing so, it is possible to just call _cdd_ to achieve the desired result. An example is shown below:


<img src="gifs/cdd.gif" alt="cdd" width="800"/>
