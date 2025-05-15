### Guide to Setting Up an Ubuntu-based WSL and Eclipse Environment for Developing SRC

#### Enable WSL on Your Machine

1. **Get Administrative Privileges:**
   - Use `latool` to obtain administrative privileges.

2. **Install Ubuntu:**
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

#### Install the SRC Environment

- **Note:** Do not run the `setup_wsl` script if you have imported the `linux_backup`.

1. **Copy Attachments:**
   - Copy all attachments to your Linux user directory (`/home/<username>`).

2. **Open Linux Terminal:**
   - Start Ubuntu from the Start menu.

3. **Navigate to User Directory:**
   ```bash
   cd ~
   ```

4. **Make the `setup_wsl` Script Executable:**
   ```bash
   chmod 777 setup_wsl.sh
   ```

5. **Run the `setup_wsl` Script with Escalated Privileges:**
   ```bash
   sudo ./setup_wsl.sh <username>
   ```

#### Install Eclipse and JDK on Windows

1. **Run `download_eclipse_jdk.sh` from Windows PowerShell:**
   - (A `.bat` file is in progress for this step.)

2. **Navigate to Your Linux User Directory from Windows Explorer.**

3. **Copy Files to Windows Desktop:**
   - Copy `OpenJDK8U-jdk_x64_windows_hotspot_8u402b06.msi` and `eclipse-inst-jre-win64.exe` to your Windows Desktop.

4. **Install JDK:**
   - Run `OpenJDK8U-jdk_x64_windows_hotspot_8u402b06.msi` with administrative privileges.

5. **Install Eclipse:**
   - Run `eclipse-inst-jre-win64.exe`.

#### Clone SRC Repository

1. **Open Eclipse and Create a New Workspace.**

2. **Navigate to the Directory of Your Eclipse Workspace on WSL.**

3. **Clone Repositories:**
   ```bash
   git clone https://git.knapp.at/osr/project/walmart.git walmart
   # If it fails, try again with sudo
   sudo git clone https://git.knapp.at/osr/project/walmart.git walmart

   git clone https://git.knapp.at/osr/src.git src
   # If it fails, try again with sudo
   sudo git clone https://git.knapp.at/osr/src.git src
   ```

#### Configure GIT

1. **Execute in WSL:**
   ```bash
   git config core.fileMode false
   git config --global user.name "Firstname Lastname"
   git config --global user.email "firstname.lastname@knapp.com"
   ```

#### Build SRC on WSL

1. **Navigate to the Directories Where You Cloned Each SRC Repo.**

2. **Run Build Commands:**
   ```bash
   ant install
   ant test
   # Testing does not need to be completed, but the compile part must be done.
   ```

#### Setup Eclipse for SRC

1. **Open Eclipse and Create a New Java Project for Each of the Two Repos.**

2. **Disable "Build Automatically":**
   - Go to the **Project** menu and disable **Build Automatically**.

3. **Run `make_classpath.bat`:**
   - Copy `make_classpath.bat` to the root directory of each repo (`src`, `walmart`) and run it from Windows.

4. **Restart Eclipse and Wait for the Build/Refresh.**

#### Unit Tests

1. **Find the Appropriate Unit Test from the Eclipse Sidebar.**

2. **Run Unit Test:**
   - Click **Run As -> Configurations**.
   - On the **VM Arguments**, paste the contents of `run_args.txt`.


Configuration for unit testing:

-enableassertions
-Xmx4096M
-Dlog4j.configuration=file:///home/moas/source/src_1/java/unit_tests/cfg/log4j.properties
-Dlog4j2.enable.threadlocals=false
-Djacorb.config.dir=/
-Dorg.omg.CORBA.ORBClass=org.jacorb.orb.ORB
-Dorg.omg.CORBA.ORBSingletonClass=org.jacorb.orb.ORBSingleton
-Dosr-id=osr1

edit custom vm intellij

-Dawt.toolkit.name=WLToolkit
-Xmx8192m

ssh-keygen -t ed25519 -C $(hostname) -N "" -f $HOME/.ssh/id_ed25519
# Add ~/.ssh/id_ed25519.pub to https://git.knapp.at/-/profile/keys
ssh -T git@git.knapp.at
 



