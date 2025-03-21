# New world

I write this note right after installing my Linux Mint OS on a dual boot disk with Windows 11.

This note is for summarizing what I have learned during this installation process.

## My experience

Firstly, as in Windows, I backup my data to my clouds: Mega (with 20GB), Dropbox (4GB), Google Drive (15GB), GitHub (Code), Youtube (Videos)

For a large amount of data, I have consumed many hours to zip then push. But finally, I found that some backups were not neccessary as I thought.
A little bit about how I divide my disk into partitions with Windows. I had C:/ (about 167GB) for the OS, D:/ (about 100GB) for the app installations and inside data, E:/ (about 100GB) for personal data like videos and P:/ (100GB) for developing.
In fact, I just need to cleanup D:/ drive by uninstalling unnecessary apps (most of them is for developing like Git, Docker, Android Studio Code,... and I want to move the work environment to Linux completely, Windows is for "keep intact with the past").
But, because I was afraid of being lost data, I have scan all the drives (except C:/, of course), deleted some data that I have had to consider too much... :(
However, I this cleaning is good, I have touched some files that I have never touched.

So, in conclusion, just need to backup data on a partition you want to install Linux. Then open Windows Partition management to unallocated it, shrink or merge if you want your Linux have more space.

Secondly, go to Linux Mint docs page to read about the [installation process](https://linuxmint-installation-guide.readthedocs.io/en/latest/).
- Install the ISO file
- Verify it: In windows, you need to install GnuPG for verify the checksum and gpg
- Create a bootable media with your USB (just need 4GB USB because Linux is thin). I use Etcher as the docs page recommended. It's pretty easy.
- Restart and Boot from USB
- Click on the Install Linux Mint on the Desktop to head to the installation process
- Choose language,...
- Choose installation type as "Something else" to manually config the partitions structure for your disk
- I have looked up the unallocated space (as I created with Windows)
  - Create a partition for EFI (512MB is ok, in fact is too much, Windows 11 just allocated over 200MB), this is where the computer looks up for the boot loader. In this case, your disk will have 2 EFI partitions, one for Windows, one for Linux Mint
  - Create a partition for Swapfile (about the size of the RAM, in my case, the RAM size is 8GB, but I allocate 4Gb for the Swap partition), which the OS use to save RAM data
  - Create a partition for / (root), which the system data is stored. In my case, I allocated 64GB for it, I think I will install pretty many applications
  - Create a partition for /home, which the user data is stored. I separate the root and home directory is because I want my data is independent from system data (in cases of reinstall OS)
  - The / and /home can be use Ext4, because it is the best for Linux
- Continue until success...

And I reach here. Thinking of noting what I have installed on my new world


## Tools and Configurations

Git and Docker were 2 applications I remembered to install first.

### Docker

For Docker, follow the [installation instructions with `apt`](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository) (I don't want to install Docker Desktop, I love CLI)
- Test with `sudo docker pull postgres`
- As the docs said
> The docker user group exists but contains no users, which is why you're required to use sudo to run Docker commands. Continue to [Linux postinstall](https://docs.docker.com/engine/install/linux-postinstall) to allow non-privileged users to run Docker commands and for other optional configuration steps.
- I need to add current user to `docker` group to be able to typing docker commands without `sudo`:
```bash
sudo usermod -aG docker $USER

# then restart the computer
```

### Git

For Git, it is pretty easy, just:
```bash
sudo apt install git
```

But, for Git, configuration is more interesting. In my case, I have 2 GitHub accounts: personal and work. I plan my mind to pull all work projects into a specific directory called `w` (just one letter, even if the Tab is better over time). I use personal account as default and switch to work account sometimes.
I have configured my Git as below (note the filename)

```ini
# ~/.gitconfig
[core]
	editor = "nano -w"
[alias]
	amend = "commit --amend"
	stash = "stash -u" # I always forget to stash untracked files...
	sum   = "log --oneline --graph --abbrev-commit"
[user]
	email = personal.email@example.com
	name  = my-name
[includeIf "gitdir:~/w/"]
	path = ~/.work.gitconfig

# ~/.work.gitconfig
[user]
	email = work.email@example.com
	name  = staff-name
```

Now is the time for SSH configurations for GitHub accounts. The [docs page](https://docs.github.com/en/authentication/connecting-to-github-with-ssh) is always the first choice.

Firstly, I generate the key pairs (public and private) for each accounts, place them into `~/.ssh` directory.
```bash
ssh-keygen -t ed25519 -C personal.email@example.com -f ~/.ssh/github-personal

ssh-keygen -t ed25519 -C work.email@example.com -f ~/.ssh/github-work
```

Then, add private keys into SSH Agent
```bash
# optional: start ssh-agent
eval "$(ssh-agent -s)"

ssh-add ~/.ssh/github-personal
#--- enter paraphrase (if have)

ssh-add ~/.ssh/github-work
#--- enter paraphrase (if have)
```

Open GitHub accounts and add public keys to it....

Finally, config to switch the keys for different hosts
```ini
# Personal GitHub account
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github-personal
    IdentitiesOnly yes

# Work GitHub account
Host work.github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github-work
    IdentitiesOnly yes
```
SSH Config is used to configure connections for different hosts and use cases with setting options like the hostname, username, identity file (private key), making it easier to manage multiple connections without needing to repeatedly enter configuration details.
In this case, we have 2 configurations for 2 type of connections into **same** host (_github.com_). In fact, `github.com` and `work.github.com` are 2 aliases for these 2 type of connections. When connect to the server (github.com), SSH find configuration in this file and apply it based on the aliases.
So, if you test with
```bash
ssh -T git@github.com
```
You will receive:
> Hi <personal-account-name>! You've successfully authenticated, but GitHub does not provide shell access.
and if you using
```bash
ssh -T git@work.github.com
```
you will see your work account name.

Do you see the point? When cloning from work repository, if you use `git clone git@github.com:staff-name/repository`, the used ssh key will be personal... :)
So, I will use Git Config to fix this case using [url.<base>.insteadOf option](https://git-scm.com/docs/git-config#Documentation/git-config.txt-urlltbasegtinsteadOf):
```ini
# ~/w/.gitconfig
[url "git@work.github.com:"]
	insteadOf = git@github.com:
```

10/10 for what I have learned today (2025-02-20)

### Visual Studio Code

I like VSCode for coding, the [docs page](https://code.visualstudio.com/docs/setup/linux#_install-vs-code-on-linux) has too many offers for the customization the editor.

I have just remembered following configs:
- Enable `Insert Final Newline` to add blank line automatically at the end of file (GitHub will complain)

With following extensions:
- **GitLens**: for highlight commit changes and compare branches
- **REST Client**: for testing api with curl (but more friendly) syntax
- **Tailwind Intellision**: for Tailwind syntax
- **Auto Rename Tag**: for auto editing HTML/XML tag pair
- **Markdown All in One**: for editing `*.md` files
- **Prettier**: for formatting JS, TS,...

### Terminal

Yeah, this is the time for Terminal.

Firstly is the prompt format. I config prompt for all users as below:

```bash
# /etc/bash.bashrc
export PS1="\[\033[1;31m\]\A \[\033[0;0m\]\[\033[1;33m\]\u@\[\033[1;32m\]\h \[\033[1;36m\][\W]\$\[\033[5;0m\] "
```

Next is the alias
```bash
# ~/.bash_aliases
alias cls=clear
```

Input language can not be missing. In Linux, I have installed `IBus` for Vietnamese input.
```bash
sudo apt install ibus-unikey

# optional
ibus-daemon -d

ibus restart
```
then add Vietnamese (Unikey) Input Source into Linux settings.

### NodeJS

NodeJS is a new choice for my learning career, even if not playing with React or NextJS, NodeJS is very crucial.

The installation instructions, is always from the [docs page](https://nodejs.org/en/download). Here, I chose to use NVM for node version management.

Just a runtime environment, so do not has many configurations.

### Context Menu

Today I've learned new world with **Nemo**, a default file manager in **Cinnamon** desktop environment (which I chose in Linux Mint installation). On thinking of creating new entry on context menu to open directory with VSCode, I found **Nemo Actions** for doing this.

Nemo Actions are defined in `.nemo_action` file in following directories:
- System-wide: `/usr/share/nemo/actions`
- User: `~/.local/share/nemo/actions`

Each file corresponds to one entry in context menu. The syntax is described in [this sample file](https://github.com/linuxmint/nemo/wiki/Documentation) or you can file local version of it at `/usr/share/nemo/action-info.md`.

I have created following context menu entries:

```ini
####### /usr/share/nemo/actions/77-open-with-code.nemo_action
[Nemo Action]

Name=Open with Visual Studio Code
Comment=Open all selected files (or current directory) on new window
Icon-Name=vscode
Extensions=any
Selection=any
Exec=code -n %F

####### /usr/share/nemo/actions/77-compare-with-code.nemo_action
[Nemo Action]

Name=Compare two files
Comment=Open two files in diff mode with Visual Studio Code                                 
Icon-Name=view-mirror-symbolic
Extensions=any
Selection=2  
Exec=code --diff %F
```

The list of `Icon-Name` can be found by: Right click a file's or directory's properties > Click on the icon in `Basic` Tab

### FortiClient

My company supports remote working policy, so I need a VPN Client. I have some troubles on install it on Linux Mint so throw a note here

```bash
cd ~/Downloads

# Link from https://www.fortinet.com/support/product-downloads#vpn
wget https://filestore.fortinet.com/forticlient/forticlient_vpn_7.4.0.1636_amd64.deb

#### If missing `libnss3-tools` ####
sudo apt install libnss3-tools

#### Linux Mint missing `libappindicator1` and it can not be installed with `apt` ####
# Download libappindicator1
wget http://mirrors.kernel.org/ubuntu/pool/universe/liba/libappindicator/libappindicator1_12.10.1+20.10.20200706.1-0ubuntu1_amd64.deb 

# Download dependency required by libappindicator1
wget http://mirrors.kernel.org/ubuntu/pool/universe/libd/libdbusmenu/libdbusmenu-gtk4_16.04.1+18.10.20180917-0ubuntu8_amd64.deb

# Install them
sudo apt install ./libappindicator1_12.10.1+20.10.20200706.1-0ubuntu1_amd64.deb ./libdbusmenu-gtk4_16.04.1+18.10.20180917-0ubuntu8_amd64.deb 

# Install forticlient downloaded from https://www.fortinet.com/support/product-downloads
sudo apt install ./forticlient_vpn_7.4.0.1636_amd64.deb
```
