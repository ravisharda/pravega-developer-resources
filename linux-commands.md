# Linux Commands

## Finding and Searching

* Find a Pravega Java process (while avoiding seeing a wall of text shown on `ps -ef | grep java`): 

  ```
  pgrep -f logback.configurationFile
  ```
## VI and VIM

**VIM:**

* Copying file contents
  * Yank the whole file:
    ```bash
    $ ggVGy
    
    # Or alternatively: 
    $ gg"*yG
    # gg - gets the cursor to the first character
    # "*y - Start the yank command to the register * from the first line
    # G - end of the file
    ```


## Shell Scripts and Environment

**Further Reading:**
* https://askubuntu.com/questions/481715/why-doesnt-cd-work-in-a-shell-script

## Miscellaneous

* Show IP addresses: `ip addr show`
* Avoid having to type `sudo <command>`: `sudo -s`
* Disk space usage of a directory https://www.ostechnix.com/find-size-directory-linux/
* Remove all directories with name `build`: `find -path "*/build/*" -delete`
* Disk usage: 
  ```
  $ du -sh * 
  $ du -sh <dir>
  ```

## Links
* Installing Minikube: 
  * https://kubernetes.io/docs/tasks/tools/install-minikube/
  * I used steps mentioned [here](https://computingforgeeks.com/how-to-install-minikube-on-ubuntu-18-04/).
* Nano editor cheat sheet https://www.nano-editor.org/dist/latest/cheatsheet.html  
