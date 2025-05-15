### Guide to Setting Up an Ubuntu-based WSL and Eclipse Environment for Developing SRC

1. **Install Ubuntu:**
   - Open Windows PowerShell as an administrator.
   - Run the following command to install Ubuntu:
     ```bash
     wsl --install -d Ubuntu
     ```
   - Set up a user and password of your choice.

   **OR** if you plan to import a backup (ensure `linux_backup.tar` is copied to `C:\` directory):
   ```bash
   wsl --install
   wsl --import moas "C:\Linux" "C:\linux_backup.tar" --version 1
   ```

### Unit Testing Configuration
```text
-enableassertions
-Xmx4096M
-Dlog4j.configuration=file:///home/moas/source/src_1/java/unit_tests/cfg/log4j.properties
-Dlog4j2.enable.threadlocals=false
-Djacorb.config.dir=/
-Dorg.omg.CORBA.ORBClass=org.jacorb.orb.ORB
-Dorg.omg.CORBA.ORBSingletonClass=org.jacorb.orb.ORBSingleton
-Dosr-id=osr1
```

### IntelliJ Custom VM Options
```text
-Dawt.toolkit.name=WLToolkit
-Xmx8192m
```

### Git User Configuration
```bash
git config --global user.name "Firstname Lastname"
git config --global user.email "firstname.lastname@knapp.com"
```

### Directory and SSH Key Setup
```bash
mkdir -p $HOME/.local/bin
mkdir -p $HOME/source

ssh-keygen -t ed25519 -C $(hostname) -N "" -f $HOME/.ssh/id_ed25519
# Add the public key to GitLab
cat $HOME/.ssh/id_ed25519.pub
# Visit https://git.knapp.at/-/profile/keys to add the key

# Test SSH connection
ssh -T git@git.knapp.at
```
 


